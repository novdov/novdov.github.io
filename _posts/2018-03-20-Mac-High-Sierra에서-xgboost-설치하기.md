---
title: Mac High Sierra에서 XGBoost 설치하기
date: 2018-03-20 18:50:43
categories:
- Machine Learning
tags:
- machine learning
- python
---

###### 2018. 03. 20

## Mac High Sierra에서 XGBoost 설치하기

**XGBoost**

- 분산형 그래디언트 부스팅
- 여러 개의 결정 트리를 묶어 강력한 모델을 만드는 또다른 앙상블 방법
- 랜덤포레스트와 달리 이전 트리의 오차를 보완하는 방식으로 순차적으로 트리를 만듦
- 무작위성이 없음
- 메모리를 적게 사용하며 예측도 빠름 (트리가 많이 추가될수록 성능이 좋아짐)
- XGBoost는 빠르고, 튜닝하기 쉬움
- 강력하고 널리 사용되는 모델 중 하나



### Installation

**2018.03.20 기준 XGBoost 최신 버전: 0.7.0**

**[사용 중인 프로그램과 환경]**

```
OS - macOS High Sierra 10.13.3
Xcode 9.2
brew 1.5.11
gcc 7.3.0.1 (2018.03.20 기준)
python 3.6.3 anaconda3 custom (conda 4.4.3)
```

- git에서 해당 프로젝트를 클론

```shell
$ git clone --recursive https://github.com/dmlc/xgboost.git
```

- gcc 컴파일러 설치 (2018.03.20 기준으로 7.3.0.1 버전이 설치된다.)

```shell
$ brew install gcc --without-multilib
```

- 최신버전의 gcc를 사용하고 있다면 make 파일을 변경해야 한다. 이렇게 해야 멀티스레드를 지원한다.

```shell
$ cd xgboost
$ cp make/config.mk ./config.mk
$ make -j4
```

 ``make -j4`` 명령에서  `clang: error: unsupported option '-fopenmp'` 오류를 마주하게 되는데, 이를 해결하는 방법은 다음과 같다. 

참고 문서: <https://github.com/ppwwyyxx/OpenPano/issues/16> (OpenMP 오류 해결)

- `clang: error: unsupported option '-fopenmp'` 오류 해결 방법

아래의 명령은 디렉토리를 따라가면서 현재 설치된 버전에 맞게 실행하면 된다. 아래 명령은 XGBoost 디렉토리가 아닌, 루트 디렉토리에서 실행했다.

```shell
$ export CXX=/usr/local/Cellar/gcc/7.3.0_1/bin/g++-7
$ brew install eigen
```

- 아래 명령으로 XGBoost를 빌드한다.

```shell
./build.sh
```

다음 메시지가 나오면 성공이다.

```shell
make: Nothing to be done for `all'.
Successfully build multi-thread xgboost
```

- 마지막으로 다음 명령으로 XGBoost 설치 완료한다.

```shell
$ pip install -e python-package
```

다음 메시지가 나오면 성공이다.

```shell
Installing collected packages: xgboost
Running setup.py develop for xgboost
Successfully installed xgboost
```

Jupyter Notebook에서도 `import xgboost`가 잘 동작한다.

```python
import pip
pip.main(['install', 'xgboost'])
```

```
Requirement already satisfied: xgboost in /Users/sunwoongkim/xgboost/python-package
Requirement already satisfied: numpy in /Users/sunwoongkim/anaconda3/lib/python3.6/site-packages (from xgboost)
Requirement already satisfied: scipy in /Users/sunwoongkim/anaconda3/lib/python3.6/site-packages (from xgboost)
```

설치 기본 사항 참고

- 오늘 코드: <http://corazzon.github.io/xgboost-install-mac-osx>
- XGBoost 공식 문서: <http://xgboost.readthedocs.io/en/latest/build.html>

