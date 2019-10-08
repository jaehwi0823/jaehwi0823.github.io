---
layout: article
title: Partial Convolution based Padding review
tags: [paper, review, ml, conv, padding, image]
---


# Partial Convolution based Padding

2019.01.13. <br>
Jaehwi Park

## 1. Introduction

> - 널리 알려진 Padding은 세 가지가 있음
>> - 1) zero padding
>> - 2) reflection padding
>> - 3) replication padding
> - 위 방법들은 그럴듯한 데이터를 edge에서 활용하기 때문에 비현실적인 값이 입혀질 수 있음
> - 이러한 비현실적인 값들도 원본 이미지와 동일하게 취급받으므로 네트워크가 혼란스러워 함

> - 본 논문에서는 Partial Convolution을 소개하고,
> - 우리의 방법론이 padding type에 robust함을 보이고,
> - 본 방법론이 semantic segmentation 중 boundaries에서 성능이 높아짐을 보임
> ![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Figure1.png)

## 3. Formulation & Analysis
### 3.1 Partial Convolution

![PC](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/partialConv.png)

> - X가 이미지이고, M이 hole을 구분하는 binary Mask일 때, 위와 같이 계산됨
> - hole이 아닌 부분만 연산을 하고, 가중치만 곱해주면 끝!

### 3.2 Partial Convolution based Padding

![Figure2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Figure2.png)

> - notation이 조금 복잡해 보이지만, 결국 내용은 같음
> - 0으로 padding되지 않은 부분만 연산을하고, 나중에 그 비중만큼 가중치를 곱해줌


## 4. Implementation & Experiments

### 4.0 Inference Time

![Table1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Table1.png)

> - Pytorch로 구현하며, 구현 팁이 나와있음
> - 논문의 방법을 따르면, Inference Time이 첫번째 Conv 이후 별 차이가 없어지게 됨

### 4.1 Image Classification

> - 모든 network case에서 성능이 기존보다 좋게 나오며
> - 특히, 가장 널리 쓰이는 zero-padding 방법에 비해 stv가 낮게 나와 안정적임
> - 위 테이블은 best model, 아래 테이블은 가장 마지막 5 epoch의 check points들의 평균값

![Table2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Table2.png)
![Table3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Table3.png)

__Convergence Speed__

> - 학습 수렴속도가 빠르다는데.. 이건 잘 모르겠음
![Figure3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Figure3.png)

__Activation Map Visualization__

> - zero padding에서 분류를 실패한 사진을 partial에서 분류를 성공했을 때 activation map을 살펴봄
> - zero는 padding된 부분에서 큰 가중치가 부여된 것을 확인할 수 있었음.. 실패의 원인

![Figure4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Figure4.png)

__Cross Testing__
> - zero padding과 partial conv의 학습과 inference를 서로 섞어봄
> - zero 학습, partial conv 추론은 엉망으로 나오는데
> - partial 학습, zero 추론은 비교적 잘 나오는 것으로 보아
> - partial 모델이 더 강건하고 중요한 것만 캐치를 잘 하는듯함

![Table4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/partialConv/Table4.png)


