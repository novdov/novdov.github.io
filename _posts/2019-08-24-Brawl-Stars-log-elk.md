---
title: '브롤스타즈 전투 기록 Elasticsearch와 Kibana에 올리기'
date: 2019-08-24 19:00:00
description: 브롤스타즈 전투 기록을 Elasticsearch와 Kibana에 올려 확인합니다.
categories:
- Data Anaylsis
- ELK
tags:
- data-anylsis
- game
- elk
---

파이썬을 이용한 시각화는 원하는 대로 데이터를 가공할 수 있다는 장점이 있지만 아무래도 매번 그래프를 그리기에는 노력이 많이 드는 것이 사실입니다. 특히 Plotly 같은 도구를 사용한다면 더욱 그렇죠. 그래서 계속해서 쌓일 브롤스타즈 전투 기록을 효율적으로 관리하고 분석하기 위해 Elasticsearch와 Kibana를 활용하기로 했습니다. 진행 환경은 맥 OS Mojave, Elasticsearch와 Kibana는 7.3입니다. (모두 localhost에서 진행했습니다.) 이 글에서 Elasticsearch와 Kibana 사용법을 다루진 않습니다. (다만, 주의할 점은 6.x, 7.x으로 업데이트 되면서 type 이 없어졌기 때문에 Mapping 시 주의해야 합니다. 공식 홈페이지에서는 하나의 index 내의 여러 type 대신 개별 index로 데이터를 관리할 것을 추천하고 있습니다.)



## 1. Elasticsearch & Kibana 설치

### Elasticsearch & Kibana 설치

[Elasticsearch](https://www.elastic.co/kr/downloads/elasticsearch)와 [Kibana](https://www.elastic.co/guide/en/kibana/7.3/brew.html)는 모두 Homebrew를 이용해 설치했습니다. 아래의 명령어로 설치해 줍니다. 맥에서는 설치 후 딱히 설정을 건드려 줄 건 없습니다. 저는 간편한 커맨드 입력을 위해 환경 변수를 등록해주었습니다. 다운로드 페이지에서 어떤 경로에 파일이 설치되는지 안내 되어 있어 이를 참고하면 됩니다.

```bash
# Elasticsearch 최신 버전 설치
brew install elastic/tap/elasticsearch-full

# Kibana 최신 버전 설치
brew install elastic/tap/kibana-full
```



### Python API 설치

Elasticsearch는 Rest API로 작동하며, 기본적으로 커맨드 라인에서 `curl` 명령어로 사용할 수 있습니다. 하지만 이는 매우 번거로운 게 사실이므로, 파이썬 API를 사용합니다. `pip3 install elasticsearch` 로 API를 설치해 주면 됩니다.



## 2. 전투 기록 읽기

다음으로 하루마다 기록한 전투 기록을 읽어 오겠습니다. 아래의 함수를 이용합니다.

```python
def read_all_logs(data_dir):
    def _read_log(filename):
        _df = pd.read_csv(filename).fillna({"name": "NULL"})
        return _df, set(_df["hash"].unique())
    
    log_files = sorted(glob.glob(str(data_dir.joinpath("*.csv"))))
    
    total_hashes = set()
    df_list = []
    for log_file in log_files:
        df, hashes = _read_log(log_file)
        
        unique_hashes = list(hashes - total_hashes)
        df_list.append(df[df["hash"].isin(unique_hashes)])
        total_hashes = total_hashes.union(set(unique_hashes))
    
    return pd.concat(df_list, axis=0)
```



데이터프레임 병합 시 중복되는 전투 기록은 제외합니다. 지금은 임시로 위와 같이 진행하지만 더 효율적인 방법으로 바꿀 예정입니다. 나흘 간 수집한 전투 기록은 6,754건의 전투 기록에 대한 총 40,524건입니다.



## 3. Elasticsearch에 데이터 입력하기

다음으로는 데이터프레임을 Elasticsearch에 입력해야 합니다. 먼저 데이터를 입력하기 위해 인덱스를 만들어 줍니다. 매핑 또한 함께 입력해 줍니다. 문자열은 `keyword` 로 등록해 Kibana에서 분석할 수 있게 합니다.



### 인덱스 생성

```python
from elasticsearch import Elasticsearch

def create_index(es: Elasticsearch, index_name: str, body: dict):
    if es.indices.exists(index_name):
        es.indices.delete(index_name)
    es.indices.create(
        index=index_name,
        body=body,
    )
```



먼저 위의 함수를 이용해 인덱스를 생성합니다. 실제 인덱스는 아래왁 같이 생성했습니다. `Elasticsearch()` 클라이언트는 기본 설정인 `localhost:9200` 으로 연결됩니다.

```python
index_name = "brawlstars" 
body = {
    "settings": {
        "number_of_shards": 1
    },
    "mappings": {
        "properties": {
            "mode": {"type": "keyword"},
            "result": {"type": "keyword"},
            "duration": {"type": "integer"},
            "map": {"type": "keyword"},
            "battle_time": {"type": "keyword"},
            "brawler": {"type": "keyword"},
            "brawler_power": {"type": "integer"},
            "brawler_trophies": {"type": "integer"},
            "name": {"type": "keyword"},
            "tag": {"type": "keyword"},
            "team": {"type": "integer"},
            "star_player": {"type": "integer"},
            "hash": {"type": "keyword"},
        }
    }
}

es = Elasticsearch()
create_index(es, index_name, body)
```



### 데이터 입력

데이터 입력은 간단합니다. 아래와 같이 입력하면 간단하게 끝납니다.

```python
from tqdm import tqdm

for i, sample in tqdm(enumerate(df.to_dict("record")), total=len(df)):
    try:
        res = es.index(index=index_name, body=sample, doc_type="_doc", id=i)
    except:
        print(i)
```



모든 과정을 완료하면 Kibana에서 다음과 같이 데이터를 확인할 수 있습니다.

![](https://drive.google.com/uc?id=1Z0X18vZho1mV6WYIKEFKlAHAWAc-q8ex)



## 4. Kibana를 이용한 시각화

Kibana의 Visualize를 활용해 데이터를 시각화할 수 있습니다. 몇 가지 정보를 Kibana로 확인해 보겠습니다.



### 브롤러 게임 모드별 선택 비율

<iframe src="http://localhost:5601/app/kibana#/visualize/edit/5ee4ffa0-c5b1-11e9-b88d-c5acefe042a0?embed=true&_g=(filters%3A!())" height="600" width="800"></iframe>

지난번과 양상이 비슷하네요. 다만 쉘리의 바운티 플레이 비율이 더 늘어났습니다. 틱, 지니, 보, 포코 등은 여전히 젬그랩 플레이 비율이 높습니다.



### 브롤러의 게임 모드별 승률

<iframe src="http://localhost:5601/app/kibana#/visualize/edit/fabffa40-c5c7-11e9-b88d-c5acefe042a0?embed=true&_g=(filters%3A!())" height="600" width="800"></iframe>



브롤러의 게임 모드별 승률은 위와 같습니다. 젬그랩에서는 프랭크와 쉘리의 승률이 가장 높습니다. 참고로 수집된 젬그랩 맵은 11개로, 전체 젬그랩 맵 중 절반 이상입니다. 그럼에도 쉘리와 프랭크가 여전히 상위 승률을 차지하고 있네요. 높은 체력, 강력한 데미지와 특수 공격, 어렵지 않은 조작이 그 이유인 것 같습니다. 바운티에서도 쉘리와 프랭크가 활약했네요. 부쉬가 많고, 장애물이 있는 맵의 데이터가 추가되어서 그런 것 같습니다. 그 외에 특이할 만한 점은 하이스트에서는 타라가, 시즈 모드에서는 브록의 승률이 매우 높다는 점입니다. 하이스트에서의 타라의 높은 승률은 예상치 못했던 부분입니다. (확인해보니 하이스트 모드에서의 타라 플레이는 14판 밖에 되지 않으며, 일종의 아웃라이어로 봐도 될 것 같습니다.)



## 5. 마무리

브롤스타즈 전투 기록을 수집하고 Elasticsearch와 Kibana까지 활용하면서 새로운 것들을 공부하고 있습니다. 막연히 어렵게만 느껴졌었는데 실제로 간단한 수준의 분석과 시각화를 위해서라면 한 두시간 정도의 공부로도 충분히 분석 환경을 만들 수 있을 것 같습니다.