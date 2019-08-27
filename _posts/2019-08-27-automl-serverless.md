---
title: 'BigQuery, AutoML, Google AI Platform을 이용한 머신러닝 파이프라인 개발'
date: 2019-08-27 19:00:00
description: BigQuery, AutoML, Google AI Platform을 이용해 데이터 적재, 모델 개발/평가, 서비리스 배포 파이프라인을 간단하게 경험했습니다/
categories:
- DeepLearning
- GCP
tags:
- dl
- gcp
- cloud
- mlops
- automl
---

시작은 텐서플로우 코리아에 올라운 [이 글](https://www.facebook.com/groups/TensorFlowKR/permalink/971390023202056/) 이었습니다. MLOps가 중요하고, 공부해야 한다고 생각만 하고 있다가 이 글을 읽고 빠르게 한번 도전해 봐야겠다고 생각했습니다. 참가를 목적에 둔 건 아니고, 현재 개발한 부분도 손 볼 곳이 한두군데는 아니지만, 간접적으로 클라우드 상에서의 머신러닝 모델 개발, 배포를 경험하는데 의의를 뒀습니다. 관련 레포는 [여기](https://github.com/novdov/ml-pipeline)입니다.



요구 사항은 다음과 같습니다.

- BigQuery에 데이러를 적재해 사용할 것
- AutoML을 사용할 것
- 모델별 리스트, 버전 관리 가능할 것
- Serverless로 배포할 것



여기서는 모두 GCP를 사용했습니다. 무료 크레딧 덕분에 부담 없이 도전할 수 있기 때문입니다. 사용한 데이터는 MNIST입니다.



## 1. BigQuery 데이터 적재

먼저 MNIST를 다운 받고, BigQuery에 업로드 하기 위한 준비를 합니다. 데이터를 다운 받고, 파싱 한뒤 `txt.gz` 형태로 저장합니다. 최종 저장 형태를 `txt.gz` 로 설정한 건 [이 예제](https://cloud.google.com/tpu/docs/tutorials/mnist?hl=ko) 를 보고 결정했습니다. 관련 API 설명은 검색해보시면 꽤 자세하게 서술되어 있습니다. GCP에 프로젝트를 만들고, BigQuery 데이터셋을 만들어줍니다. 그런 뒤, `train/valid/text` 를 로컬에서 업로드 해줍니다.

![](https://drive.google.com/uc?id=1jIt2Tvije-vFPmRC28U3HYJURUwn8b8h)

데이터는 정규화를 하지 않은 채 올려주었습니다. 처음에는 정규화를 한 `float` 으로 테이블을 구성했었는데, 문자열로 변환하는 과정에서 부동소수점이 많이 날아갔고, 학습이 제대로 되지 않았기 때문입니다. (77% 정도의 정확도) 학습 시 데이터를 불러올 때는 파이썬 API를 사용합니다. 아래는 `Dataset` 클래스의 `get_data()` 메서드입니다. 테이블을 조회한 뒤 `ndarray` 이미지와 레이블을 리턴합니다.

```python
    def get_data(self):
        def _string_to_float(_raw_image: str):
            arr = np.asarray(_raw_image.split(","), "float")
            if self.unflatten:
                return arr.reshape([28, 28])
            return arr

        mode = self.mode
        dataset_ref = self.client.dataset(self.dataset_id)

        print(f"Mode: {mode} -> Load data from table")
        rows = self.client.list_rows(dataset_ref.table(self.TABLES[self.mode]))
        arrows = rows.to_arrow()
        arrows_dict = arrows.to_pydict()

        labels, images = arrows_dict["key"], arrows_dict["image"]
        labels = np.array(labels)
        images = np.array(
            [_string_to_float(image) for image in tqdm.tqdm(images)], "float"
        )

        # Result from BigQuery are sorted by key.
        data = list(zip(images, labels))
        random.shuffle(data)
        images, labels = zip(*data)
        return np.array(images, "float"), np.array(labels, "int")
```



## 2. AutoML

AutoML은 처음 경험해 봤습니다. 사용한 라이브러리는 구글의 `adanet` 입니다. MNIST 관련 예제가 [노트북](https://colab.research.google.com/github/tensorflow/adanet/blob/master/adanet/examples/tutorials/customizing_adanet.ipynb)으로 자세하게 제공되고, 많은 부분을 참고했습니다. 기본적인 API는 `tf.estimator.Estimator` 로 구현되어 있습니다. `adanet.Estimator` 에 feature columns과 서브네트워크를 만들어내는 메서드를 넘겨주면 됩니다.

관련 코드는 아래와 같습니다. 일단은 간단한 DNN만 이용했습니다. 예제에는 베이스라인, DNN, CNN을 소개하고 있습니다.

```python
class DNNGenerator:
    """DNN subnetwork Generator using `adanet.examples.simple_dnn.Generator`"""

    FEATURE_KEY = "image"

    def __init__(self, learning_rate: int):
        self.learning_rate = learning_rate

    def build_subnetwork_generator(self, unflatten: bool = False):
        if unflatten:
            shape = [28, 28, 1]
        else:
            shape = [int(28 * 28)]

        feature_columns = [
            tf.feature_column.numeric_column(self.FEATURE_KEY, shape=shape)
        ]

        return simple_dnn.Generator(
            feature_columns=feature_columns,
            optimizer=tf.train.AdamOptimizer(self.learning_rate),
            seed=SEED,
        )
```

먼저 DNN 네트워크를 생성하는 `DNNGenerator` 를 선언해주었습니다. 별다른 작업이 필요 없이 아주 간단합니다. 실제 학습과 평가를 위해 Estimator를 선언해 주는 부분도 기존의 Estimator API 사용과 동일합니다. 아래는 `AdanetTrainer` 의 `create_estimator()` 라는 메서드입니다. 일반 Estimator와 거의 동일하며, `train_and_evaluate()` 메서드를 호출할 때 TrainSpec과 EvalSpec을 넘겨주면 됩니다. 모델 서빙을 위한 `exporters()` 메서드도 선언해줍니다.

```python
    def create_estimator(
        self,
        subnetwork_generator,
        adanet_iterations: int,
        evaluator: Evaluator,
        config: tf.estimator.RunConfig,
    ) -> tf.estimator.Estimator:
        return adanet.Estimator(
            head=self.head,
            subnetwork_generator=subnetwork_generator,
            max_iteration_steps=self.train_steps // adanet_iterations,
            evaluator=evaluator,
            config=config,
        )
    
    def train_and_evaluate(
        self, estimator: tf.estimator.Estimator, train_input_fn, eval_input_fn
    ):
        results, _ = tf.estimator.train_and_evaluate(
            estimator,
            train_spec=self.get_train_spec(train_input_fn),
            eval_spec=self.get_eval_spec(eval_input_fn),
        )
        return results
     
    def exporters(self):
        def _serving_input_receiver_fn():
            receiver_tensor = {
                "image": tf.compat.v1.placeholder(
                    dtype=tf.float32, shape=[None, 28, 28, 1], name="image"
                )
            }
            features = {"image": receiver_tensor["image"]}
            return tf.estimator.export.ServingInputReceiver(features, receiver_tensor)

        return tf.estimator.LatestExporter(
            name="serving", serving_input_receiver_fn=_serving_input_receiver_fn
        )
```



위와 같이 코드를 작성하고 실제 학습/평가를 진행하면 CPU에서도 꽤 빠른 시간 안에 높은 정확도를 얻을 수 있습니다. (약 96% 정도의 정확도) 일반적인 DNN보다 좋은 성능을 얻을 수 있습니다. 정확한 구현 원리는 논문을 읽어 봐야 알 수 있겠지만, 간단한 문제에서는 적절하게 활용하면 유용할 것 같습니다.



## 3. 서빙

서버리스 모델을 위해 GCP AI Platform을 이용합니다. 관련 예제는 node.js로 많이 작성이 되어 있는데 [파이썬으로 된 자료](https://cloud.google.com/blog/products/ai-machine-learning/empower-your-ai-platform-trained-serverless-endpoints-with-machine-learning-on-google-cloud-functions)를 참고했습니다. 자세한 설정 등은 해당 글에 상세히 소개 되어 있으므로 참고하시면 좋을 것 같습니다. 서버와의 통신을 위해 `InferAPI` 라는 클래스를 작성했습니다. GCP에서 모델을 호출하고 결과를 받아옵니다. 역시 파이썬 API를 사용합니다. 작성에 어렵지 않습니다. 첫 번째 모델을 배포하고 난 뒤부터는 모델을 학습하면 해당 모델의 결과와 서버에 배포된 모델의 성능을 비교하는 데 사용되기도 합니다. (실제로는 매번 호출을 피하기 위해 각 모델별 결과를 로깅합니다.)

```python
    def predict(self, images: List[List[float]], batch_size: int = 100) -> Mapping:
        def _parse_results(list_of_dict: List[Mapping], key: str):
            return [res[key].pop() for res in list_of_dict]

        response = {"class_ids": []}

        for start in tqdm.tqdm(range(0, len(images), batch_size)):
            end = min(start + batch_size, len(images))
            _response = self.request(images[start:end]).execute()

            predictions = _response["predictions"]
            for key in response.keys():
                response[key].extend(_parse_results(predictions, key))
        return response

    def request(self, images: List[List[float]]):
        def _generate_payloads(_images):
            return {"instances": [{"image": image} for image in _images]}

        return self.service.projects().predict(
            name=f"projects/{self.project_id}/models/{self.model_name}",
            body=_generate_payloads(images),
        )

    def get_model_meta(self):
        name = f"projects/{self.project_id}/models/{self.model_name}"
        if self.version is not None:
            name += f"/versions/{self.version}"
            response = (
                self.service.projects().models().versions().get(name=name).execute()
            )
            return response
        response = self.service.projects().models().get(name=name).execute()
        return response["defaultVersion"]
```



## 4. 마무리

기본적인 기능들은 완성했지만 제대로 된 파이프라인을 구성하려면 고려해야 할 것이 많습니다. HTTP를 이용한 Cloud Function 이용, 유저 데이터 BigQuery 적재 등 고려해야 할 것이 많고, 학습도 클라우드에서 진행되도록 자동화할 것이 많이 남아 있습니다. (현재 학습은 로컬에서 이루어지고 있습니다.)

![](https://drive.google.com/uc?id=1TdrVYrTUA3_gkxMAc9RxjwbtbGf8Lh29)

깔끔하지 않은 부분도 많지만, 클라우드 상에서의 데이터, 모델 개발 및 서빙을 익숙하게 다룰줄 안다면 큰 자산이 될 거라고 생각합니다. 특히 데이터와 모델의 버전 관리, 자동화된 실험 등을 모두 효과적으로 해내는 것이 실무에서 머신러닝 모델 개발 과정에 큰 도움이 됟 것은 당연하기 때문이죠.