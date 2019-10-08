---
layout: article
title: DATA MINING OF INPUTS - ANALYSING MAGNITUDE AND FUNCTIONAL MEASURES review
tags: [paper, review, ml, datamining, inputs]
---

# DATA MINING OF INPUTS: ANALYSING MAGNITUDE AND FUNCTIONAL MEASURES


<br> 2019.03.10.
<br> Jaehwi Park

## 1. Input Significance
### 1.1 Network topology - GIS data

> - 네트워크 구조는 [16-10-5]의 간단한 신경망
> - 문제는 간단한 예측 문제로, 고도 기울기 강수량 온도 등이 있을 때 숲의 타입을 맞추는 분류 문제

### 1.2 Analysis Techniques

> - Garson Method(G):
>> 1. $n_h$: Hidden node 수, 본 문제는 16개
>> 2. $n_i$: Input node 수, 본 문제는 10개
>> 3. k: 영향도를 측정할 특정 k
>> 4. $G_{ik}$: 특정 i의 k에대한 "proportional contriution"
>> ![Function1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Function1.png)
>> 5 문제점: positive & negative cancelation

> - Wong, Gedeon and Taggart Method:
>> 1. Garson과 똑같은데, inner term에 절댓값만 적용
>> ![Function2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Function2.png)
>> 2. 문제점: 부호가 완전 무시됨
>> 3. 본 논문의 base method

> - Milne Method(M):
>> 1. Garson과 똑같은데, 절댓값 적용 방식을 살짝 바꿈
>> ![Function3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Function3.png)
>> 2. 그냥 비교군 중 하나

> - Our Method(Q):
>> 1. Wond, Gedeon and Taggart Method를 활용함
>> ![Function4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Function4.png)
>> ![Function5](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Function5.png)

### 1.3 Brute force analysis (B)

> - 제시한 방법들의 효과 증명을 위한 비교지표 생성을 위해 만듦
> - 변수 한 개만 제거할 때는 inconsistent results가 생성돼어 변수 두 개를 제거함
> - 16개 inputs 중 2개를 제외할 수 있는 경우의 수: 120
> - 각 방법을 4번씩 학습했을 때의 TSS를 sorting한 결과가 아래와 같음
> - 갑자기 TSS가 크게 증가하는 경우들이 중요한 변수들이 빠진 경우임
> ![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Figure1.png)

### 1.4 Comparison of results

![Table1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Table1.png)

> - Brute force (B)방법과 다른 방법들의 중요 변수 측정 결과를 비교함
> - 가장 중요한 5개 변수와 가장 중요하지 않은 5개의 변수를 나열했을 때, 본 논문의 방법(Q)이 60%의 정확도를 가짐

![Figure6](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Figure6.png)
> - 특히 본 논문의 방법(Q)은 여러번 시행하더라도 결과의 일관성을 보유함

## 2. Use of Measures
### 2.1 Network topology - Medical data

> - 네트워크 구조는 [12-7-1]인 신경망
> - eye gazing data로 신경분열증 환자인지를 맞추는 분류 문제

### 2.2 Magnitude measures of contributions
> - 위에서 소개한 방법들을 magnitude measures로 활용

### 2.3 Functional measures

> - Distinctiveness analysis:
>> - hidden neuron activations(다차원 vector)의 angle을 계산해서 유사도를 측정함

![Function6](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/ContribDataMinv2/Function6.png)

