---
layout: article
title: Manifold Mixup review
tags: [paper, review, ml, maifold, mixup]
---


# Manifold Mixup: Better Representations by Interpolating Hidden States

2019.04.21. <br>
Jaehwi Park

## 1. Introduction

### [Figure1]
![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/Figure1.png)

<br>

### [Figure2]
![Figure2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/Figure2.png)

> - slightly different dist.를 만났을 때, high-confidence로 틀리는 일이 잦음
> - 그래서 dist. shift 또는 adversarial attack에 잘 대응할 수 없음

---

> - 현재 SOTA NN에서도 아래의 문제들이 아직 존재함
>   - Decision Boundary가 보통 sharp 하고 data에 닫혀있음
>   - Hidden representation space 보통은 high confidence prediction 위주임 __on and off of the data manifold__

---

> - Interporlation이 특성을 결합하는 좋은 방법이고,
> - High-level representations가 보통 더 낮은 차원에서 선형 분리가 쉬우므로,
> - __Linear Combinations of Hidden Representations__가 의미있는 feature space를 탐색해줄 것이라고 생각함

---

> - mixup manifold로 얻어지는 두 가지 바람직한 특성은,
>   - the class-representations are flattened into a minimal amount of directions of variation
>   - all points in-between these flat representations, most unobserved during training and off the data manifold, are assigned __low-confidence predictions.__

---

## 2. Manifold Mixup

1. 하나의 랜덤 layer k를 선택 (from a set of eligible layers S)
2. 2개의 랜덤 minibatches (x,y), (x', y')를 feedforward해서 layer k까지 진행
3. Mix$_\lambda(a,b)$ 연산 수행

![mix](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/mix.png)


> - Mix$_\lambda(a,b) = \lambda \times a + (1- \lambda) \times b$
> if one-hot labels exists
> - $\lambda$ ~ $Beta(\alpha, \alpha)$

--- 


4. layer k 이후에는 mixed minibatch를 활용해서 output까지 산출 <br>
5. Loss를 계산하고, 이전의 모든 graph에 back-prop을 수행

![loss](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/loss.png)

[Implementation 참고] <br>
1) (k, $\lambda$)는 minibatch마다 sampling <br>
2) Input mixup은 한 개의 minibatch를 활용 ~ __한 개의 minibatch를 복사__한 뒤 서로 다르게 __shuffle__해서 사용

## 3. Manifold Mixup Flattens Representations

### 3.1 Theory

> __Theorem 1.__
> - Hidden 차원이 데이터셋의 클래스 차원보다 클 경우, loss를 0으로 만드는 선형함수 f가 존재함<br>
(If dim(H) >= d - 1 then, J(P)=0 and corresponding minimizer f is a linear function from H)

> __Corollary 1.__
> - Theorem 1.이 성립할 때의 g는 dim(H) - d + 1 의 subspace로 representation을 한정시킴

> __If Manifold Mixup loss is minimized,__
> - the representation of each class lies on a subspace of dim(H)-d+1
> - 극단적인 케이스는 dim(H) = d - 1
> - 극단적인 케이스에서 hidden representation은 collapse to a single point되고, 어느 방향으로도 움직이지 않게됨
> - 좀 더 general한 케이스인 larger dim(H)에서, 대부분의 H-space 방향이 비어있게 됨 (class-conditional manifold에서)

![Figure3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/Figure3.png)

### 3.2 Empirical Investigation of Flattening

> - MNIST에 각종 regularization 기법을 쓰고, largest SV와 sum of SVs를 계산함
> - Largest의 경우: BASE(51.73), Weight decay(33.76), Dropout(28.83), IM(33.46), __MM(31.65)__
> - Sum의 경우: BASE(78.67), Weight decay(73.36), Dropout(77.47), IM(66.89), __MM(40.98)__

### 3.3 Why is Flattening Representations Desirable?

> - smaller volume hidden representations --> a randomly sampled hidden representation within the convex hull spanned by the data in this space is more likely to have a classification score with lower confidence
> - compression has been linked to generalization

![Table1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/Table1.png)
![Table2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/Table2.png)
![Table4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/mixup-manifold/Table4.png)
