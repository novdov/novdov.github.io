---
title: '논문 초록을 요약해서 슬랙으로 전송하기'
date: 2019-08-27 19:00:00
description: arxiv.org 최신 논문의 초록을 요약해서 슬랙으로 전송하는 구조를 만들어 봅니다. 
categories:
- DeepLearning
- GCP
tags:
- dl
- gcp
---

개인적으로 BERT를 이용해 논문 초록을 요약한 다음, 매일 슬랙으로 전송하는 간단한 프로젝트를 진행하고 있습니다. 간단한 모델이라도 fine-tuning하는 데 시간이 걸리니, 일단 바로 사용할 수 있는 간단한 구조를 짰습니다.

- 매일 발표되는 관심 분야의 논문 초록을 요약하고
- 매일 아침 슬랙으로 전송

이라는 간단한 구조입니다. 프로젝트 구조는 아래와 같습니다. 전체 코드는 [GitHub](https://github.com/novdov/papersum)에서 확인할 수 있습니다.

```bash
.
├── papersum
│   ├── __init__.py
│   ├── __main__.py
│   ├── api.py
│   ├── controller.py
│   └── crawler
│       ├── __init__.py
│       └── crawler.py
├── run.sh
└── tox.ini
```

- `crawler.py` 에서 논문 정보를 수집합니다.
- `controller.py` 에서 초록을 요약하고 텍스트를 가공합니다.
- `api.py` 에서 슬랙으로 결과를 메세지로 전송합니다.



## 정보 수집 & 슬랙 메세지 전송

### 논문 정보 수집

먼저 매일 발표되는 논문의 정보 (제목, 초록, URL)를 수집해야 합니다. `requests` 와 `bs4` 를 이용합니다.

```python
class Crawler:
    BASE_URL = "https://arxiv.org"
    URL = BASE_URL + "/list/cs.CL/recent"

    def __init__(self):
        self.query_url = self._get_query()
        self.soup = BeautifulSoup(requests.get(self.query_url).content, "html.parser")

        self.contents = self.get_page_contents()

    def get_recent_article_urls(self):
        def _get_article_url(attr):
            return self.BASE_URL + attr.span.a["href"]

        all_recent_article_list = self.contents.find_all("dl")
        raw_article_list = all_recent_article_list[0].find_all("dt")
        article_urls = [
            _get_article_url(raw_article) for raw_article in raw_article_list
        ]
        return article_urls

    def get_recent_days_list(self):
        days_list = self.contents.find("ul").find_all("li")
        return days_list

    def get_page_contents(self):
        return self.soup.find("div", {"id": "dlpage"})

    def _get_query(self):
        total_entries = self._get_total_entries()
        _query_fmt = f"/list/cs.CL/pastweek?show={total_entries}"
        return self.BASE_URL + _query_fmt

    def _get_total_entries(self):
        soup = BeautifulSoup(requests.get(self.URL).content, "html.parser")
        base_contents = soup.find("div", {"id": "dlpage"})
        return int(base_contents.find("small").text.split(" ")[3])
```

![](https://drive.google.com/uc?id=1-endf_T6TAT2YtGhHIbgbmp3S91duGj-)

- `_get_query()` 메서드로 위의 페이지에 접속합니다.

- `get_recent_article_urls()` 메서드로 최근 날짜의 논문 URL을 가져옵니다.

이렇게 수집한 논문별 URL을 가지고 `controller.py` 에서 논문별 정보를 수집합니다.

```python
class PapersumController:
    def __init__(self):
        self.crawler = Crawler()
        self.model = SingleModel()

    def get_article_info_list(self):
        article_urls = self.crawler.get_recent_article_urls()
        return [self.get_article_info(url) for url in article_urls]

    def get_article_info(self, url):
        def _clean_text(text):
            return text.replace("\n", " ").strip()

        soup = BeautifulSoup(requests.get(url).content, "html.parser")
        contents = soup.find("div", {"id": "content"})

        title = " ".join(contents.find("h1", "title mathjax").text.split(":")[1:])
        abstract = " ".join(contents.find("blockquote").text.split(":")[1:])
        return {
            "title": _clean_text(title),
            "abstract": _clean_text(self.summarize(_clean_text(abstract))),
            "url": url,
        }

    def summarize(self, text, ratio=0.2, min_length=40, max_length=600):
        return "".join(
            self.model(text, ratio=ratio, min_length=min_length, max_length=max_length)
        )
```

- `get_article_info()` 메서드에서는 제목, 초록, URL을 가져온 뒤 초록은 요약합니다.
  - `_clean_text()` 함수는 텍스트의 개행 문자를 처리하는 함수입니다.
- 논문 요약에는 [`bert-extractive-summarizer`](https://github.com/dmmiller612/bert-extractive-summarizer)  라는 라이브러리를 이용합니다.



### 슬랙 메세지 전송

마지막으로 최종 결과를 슬랙에 전송합니다. 슬랙의 incoming webhook을 사용합니다. 메세지는 payload에서 `text` 키에 해당합니다. 이를 위해 제목, 초록 요약 결과, URL을 하나의 문자열로 만들어 줍니다.

```python
class SlackAPI:
    def __init__(self, controller):
        self.controller = controller
        self.user_name = "SUMMARIZER"
        self.webhook_url = open("/etc/slack_url.txt").read().strip()
        self.headers = {"Content-Type": "application/json"}

    def send_slack(self):
        input_payloads = self.get_input_payloads()
        for payload in input_payloads:
            requests.post(
                self.webhook_url, data=json.dumps(payload), headers=self.headers
            )

    def get_input_payloads(self):
        info_list = self.controller.get_article_info_list()
        text_fmt = "{title}\n\n{abstract}\n\n{url}"
        return [
            {"username": self.user_name, "text": text_fmt.format(**info)}
            for info in info_list
        ]
```

완성된 코드는 GCP의 VM 인스턴스에 올려 매일 아침 9시마다 작업을 수행하도록 크론 작업으로 설정했습니다.



이렇게 하면 슬랙에서 다음과 같은 메세지를 받을 수 있습니다.

![](https://github.com/novdov/papersum/blob/master/example01.png?raw=true)

요약 결과는 나쁘지 않습니다. 프로젝트 완성 동안 아무것도 안 하는 것보다는 빠르게 결과를 받아보는 편이 훨씬 좋다고 생각합니다.



## 마무리

### 한계점

빠르게 결과물을 받아볼 수 있게 한 부분은 좋지만, 현재 구조에는 큰 한계점이 있습니다. 현재 30여개의 논문 정보를 수집하고 메세지를 전송하는데 3분 정도 걸립니다. wamup에 약 1분, 그리고 요약 inference에 약 1분 50초 정도 소요됩니다.

- 사용한 `bert-extractive-summarizer` 는 배치 inference를 지원하지 않습니다. 게다가 하나의 샘플을 inference 하는 데에도 2~4초 정도 걸립니다. 메모리도 많이 소요됩니다.
- 해당 작업을 위해 기존의 g1-small에서 v2 cpu 2코어 (2.8Ghz), 메모리 8GB로 인스턴스를 업그레이드 했습니다. 목표는 더 낮은 등급의 인스턴스에서 속도와 메모리를 최적화하는 것입니다.
  - 물론 이를 위해서는 BERT를 직접 fine-tuning 하고 모델 경량화를 진행해야 합니다.



하지만 매일 아침 딱 한번 수행하는 데다가, 당장 inference 속도가 그렇게 중요하지는 않습니다. 가장 중요했던 부분은 최대한 빠르게 작동하는 구조를 만들고 논문을 접할 수 있게 하는 것이었기 때문이었죠. 논문 초록 요약 봇의 MVP 버전이라고 해도 될 것 같습니다. 앞으로의 계획은 BERT를 이용해 직접 fine-tuning 하고 여러 최적화를 거치는 것으로 프로젝트를 완성하는 것입니다.