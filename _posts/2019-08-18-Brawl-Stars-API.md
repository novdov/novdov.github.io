---
title: '브롤스타즈 Python API로 전투 기록 확인하기'
date: 2019-08-18 19:00:00
description: 브롤스타즈 API를 이용해 전투 기록을 저장하고 불러옵니다.
categories:
- Data Anaylsis
tags:
- data-anylsis
---



저는 브롤스타즈를 즐겨합니다. 항상 전투 기록으로 어떤 분석을 해보고 싶었는데, 브롤스타즈 API를 마침 찾아 이를 이용해 전투 기록을 저장하고 확인해봤습니다.



## 1. 브롤스타즈 API

슈퍼셀 공식 API가 있으면 좋겠지만, 그렇진 않은 것 같습니다. 대신 [Brawl Stats](https://brawlstats.com/) 와 같이 비공식적으로 브롤스타즈 플레이 데이터를 보여주는 곳은 있습니다. 이런 서비스가 있다면 누군가 API를 만들었을 거라는 생각에 관련 API를 찾던 도중, [brawlstats](https://github.com/SharpBit/brawlstats) 라는 파이썬 라이브러리를 발견했습니다. 최근까지 개발이 이루어지고 있어 보여 괜찮아보였습니다. 

사용법은 간단해서 자체 토큰을 발급받고 `Client` 를 호출하면 됩니다. 토큰을 발급받으려면 디스코드 계정이 필요합니다. API 문서도 자세하게 기술되어 있어서 간편하게 사용할 수 있습니다.



## 2. 전투 기록 확인하기

관심 있는 부분은 전투 기록이므로, 이를 파싱해서 저장하고 불러오기 위한 작업이 필요합니다. 먼저 pip로 `brawlstats` 를 설치하고 `Client` 를 선언해줍니다. `token` 은 발급 받은 토큰 (문자열) 입니다.

```python
import brawlstats
client = brawlstats.Client(token)
```



전투 기록은 `get_battle_logs(tag)` 메서드로 가져올 수 있습니다. `tag`는 브롤스타즈의 유저 아이디 (#으로 시작하는 태그이며 메서드의 인자로 넣을 때는 #은 제외)입니다. 리턴하는 값은 두 개의 `BattleLog` 클래스를 담고 있는 리스트인데, 두 개 모두 같은 기록을 가지고 있어서 첫 번째를 사용합니다.

`BattleLog` 클래스에서 전투 기록을 `raw_data: List[Mapping]` 라는 인스턴스로 접근할 수 있습니다. 로그는 최근 25건의 기록이며, 아래와 같은 모습을 하고 있습니다. 게임 모드, 전투 결과, 스타 플레이어, 플레이어 정보 등을 확인할 수 있습니다. 단 게임 모드가 빅게임 같은 이벤트 모드일때는 `result` 등이 표시되지 않습니다. 저장할 때는 이벤트 경기를 제외한 3 vs 3 매치 (젬그랩, 하이스트, 바운티, 브롤볼) 만 저장합니다. (3 vs 3 매치만 플레이하기도 하거니와, 분석 목표 자체가 3 vs 3 매치에 좀 더 초점을 맞추고 있기 때문이기도 합니다.)

```
'battle': {'duration': 123,
            'mode': 'gemGrab',
            'result': 'victory',
            "battle_time": battle_log["battleTime"],
            'starPlayer': {'brawler': {'id': 16000014,
                                       'name': 'BO',
                                       'power': 5,
                                       'trophies': 361},
                           'name': 'DannyPhantom',
                           'tag': '#GRVVUCRQ'},
            'teams': [[{'brawler': {'id': 16000025,
                                    'name': 'CARL',
                                    'power': 5,
                                    'trophies': 391},
                        'name': 'Lemonsilverhaze',
                        'tag': '#LPUV88VQ'},
                       {'brawler': {'id': 16000001,
                                    'name': 'COLT',
                                    'power': 4,
                                    'trophies': 152},
                        'name': 'SLAT',
                        'tag': '#9LULYCYV8'},
                       {'brawler': {'id': 16000005,
                                    'name': 'SPIKE',
                                    'power': 10,
                                    'trophies': 424},
                        'name': 'Element',
                        'tag': '#QRCGUVC'}],
                      [{'brawler': {'id': 16000005,
                                    'name': 'SPIKE',
                                    'power': 4,
                                    'trophies': 365},
                        'name': '꿀꿀',
                        'tag': '#2YYV9GLLR'},
                       {'brawler': {'id': 16000014,
                                    'name': 'BO',
                                    'power': 5,
                                    'trophies': 361},
                        'name': 'DannyPhantom',
                        'tag': '#GRVVUCRQ'},
                       {'brawler': {'id': 16000001,
                                    'name': 'COLT',
                                    'power': 10,
                                    'trophies': 404},
                        'name': 'titoboss71',
                        'tag': '#G9RY9UVJ'}]],
            'trophyChange': 8,
            'type': 'ranked'},
 'battleTime': '20190818T021356.000Z',
 'event': {'id': 15000040, 'map': 'Chill Cave', 'mode': 'gemGrab'}}
```



저장할 형식은 각 지표를 컬럼으로, 각 유저를 로우로 가지는 csv 입니다. 이를 위해 먼저 각 로그를 파싱하는 헬퍼 함수를 정의합니다. (주의: 변수 타입 힌트는 파이썬 3.6 이상부터 사용할 수 있습니다.)

```python
def parse_single_battle_log(battle_log: Mapping, tag: str) -> Optional[List[Mapping]]:
     if battle_log["battle"]["mode"] in ["bigGame"]:
         return None

     info_list = []
     battle_info = {
         "mode": battle_log["battle"]["mode"],
         "result": battle_log["battle"]["result"],
         "duration": battle_log["battle"]["duration"],
         "map": battle_log["event"]["map"],
         "battle_time": battle_log["battleTime"],
     }

     star_player = battle_log["battle"]["starPlayer"]["tag"]

     teams: List[List[Mapping]] = battle_log["battle"]["teams"]
     for team in teams:

         players = [log["tag"][1:] for log in team]
         side = 1 if tag in players else 0

         for player in team:
             team_info = {
                 "brawler": player["brawler"]["name"],
                 "brawler_power": player["brawler"]["power"],
                 "brawler_trophies": player["brawler"]["trophies"],
                 "name": player["name"],
                 "tag": player["tag"],
                 "team": side,
                 "star_player": 1 if player["tag"] == star_player else 0
             }
             info_list.append({**battle_info, **team_info})
     return info_list
```



위의 함수를 이용해서 25건의 전투 기록을 파싱하고 csv로 저장합니다. 브롤스타즈의 전투 기록은 최근 25건까지만 저장되므로 주기적인 로그 저장을 위해 GCP의 VM 인스턴스를 띄운 뒤, 8시간마다 기록을 저장하도록 크론 작업을 설정해 주었습니다. VM 인스턴스는 저렴한 f1-micro(vCPU 1개, 0.6GB 메모리)로 설정했습니다.

전투 기록을 저장하고 판다스로 불러온 모습은 가장 최근 경기 기록은 아래와 같습니다. 각 전투 식별은 `battle_time` 으로 합니다. 또한 기록에서 `result` 와 `team` 은 모두 본인 기준입니다. `team` 은 같은 팀이면 1, 상대 팀은 0으로 표시했고, 스타 플레이어 또한 0과 1로 표시했습니다.

![](https://i.imgur.com/BiXPgs7.png)



## 3. 마무리

하루에 수십판 씩 플레이 하지 않기 때문에 분석을 시작하려면 꽤 시간이 지나야 할 것 같습니다. 또한 훨씬 일찍 시작했더라면 어땠을까 하는 아쉬움이 남습니다. 항상 생각만 가지고 있다가 이제서야 시작한 게 참 아쉽습니다. 데이터가 어느 정도 모이면 간단하게라도 맵과 조합, 매칭 등에 대해서 살펴보려고 합니다.

