

## Introduction

최근 활발히 연구되고 있는 NMT 모델은 다음의 속성을 따른다.

- Decoder는 source sequence representation에 대해 attention을 사용한다.
- Encoder와 Decoder는 두 개의 다른 모듈이며, Encoder는 Decoder의 작동 전에 source sentence의 encoding을 마쳐야 한다.

논문에서는 NMT 모델이 위의 속성 없이 얼마나 성능을 낼 수 있는지 확인한다.

이를 위해 논문에서는 Bah- danau et al. (2014)의 모델로 시작해 attention을 제거하고 Encoder와 Decoder를 하나의 간단한 모델로 합친다. 이 모델은 Zaremba et al. (2014)의 langaugae model를 닮았다.