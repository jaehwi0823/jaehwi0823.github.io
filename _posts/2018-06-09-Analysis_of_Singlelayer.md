---
layout: article
title: An Analysis of Single-Layer Networks in Unsupervised Feature Learning review
tags: [paper, review, ml, unsupervised, feature]
---

# An Analysis of Single-Layer Networks in Unsupervised Feature Learning

2018.6.9 <br>
Jaehwi Park

## Abstract

<br>
본 논문에서는 몇몇 매우매우 간단한 요소들이 매우 중요하다는 것을 보였다고 합니다. <br>
아래는 Set-up에 대한 요약입니다.

1. Benchmark Datasets
> - NORB: intended for 3d object detection, last update:2005  <br>
> - CIFAR

2. Off-the-shelf feature learning algorithms
> - sparse auto-encoders
> - sparse RBMs
> - K-means clustering
> - Gaussian mixtures

3. Network: only single-layer networks

4. Details to vary
> - the receptive field size
> - number of hidden nodes
> - the step-size (stride)
> - whitening


본 논문에서 알고리즘의 선택만큼이나 매우 중요하다고 확인된 두 가지는 아래와 같습니다. 
> - the large numbers of hidden nodes
> - Dense feature extraction

이 두 가지를 pushed to limits 했을 때, CIFAR & NORB 모두에서 SOTA 결과를 얻었다고 합니다. <br>
더 놀라운 사실은 __K-means clustering 에서 가장 좋은 결과__를 얻었다는 것 입니다. 매우 빠르고, 모형 구조 설계 외의 Hyper-parameters도 없는 알고리즘인데도 SOTA 결과를 보여줬다고 합니다.

## 1. Introduction

일단 옛날 논문이라 좋은 feature을 만들어내는 Current solution을 이렇게 표현합니다.
> greedily "pre-training" several layers of features, one layer at a time



## 2. Related work

Deep한 구조를 위한 many new schemes가 제안됐으나 비지도 학습 모듈이 가장 심도있게 연구됐다고 합니다.

이전 연구로는 ..
- pooling, normalization, rectification between layers
- pooling activation function or coding scheme



## 3. Unsupervised feature learning framework

### 3.1 Feature Learning

Extracting random sub-patches from unlabeled input images
>  - Each patch has dimension [w, w, d]
>  - then, construct a dataset of m randomly sampled patches
>  - apply the pre-processing and unsupervised learning steps

<br>
**3.1.1. Pre-processing**
> - Normalization
> - Whitening는 실험대상

**3.1.2. Unsupervised learning**
> 1. Sparse auto-encoders
> 2. Sparse RBMs
> 3. K-means clustering
> 4. Gaussian mixtures

---
** Sparse auto-encoder **

[Autoencoder](http://solarisailab.com/archives/113)
> - 항등함수를 학습하여 함축적인 Featuer 탐색
> - Sparsity 부여


![Autoencoder](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/analysisOfSingleLayer/autoencoder.png)

___
__Sparse restricted Boltzmann machine (RBM)__

모르겠어요...<br>
help me!

[보충설명](http://sanghyukchun.github.io/75/)

---
__K-means clustering__

[K-menas](http://sanghyukchun.github.io/69/)
> - 간단함
> - 그 대신 Local Minima에 수렴

![K-means](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/analysisOfSingleLayer/K-means.png)

<br>
논문에서는 두 가지 K-means를 사용함. $f_k(x)$가 input x가 k번째 cluster에 속하는 정도일때,
> - the standard 1-of-K: (hard)
>> $f_k(x)=1$ if $k= argmin_j||c^{(j)} - x||_2^2$ <br> otherwise 0
> - softer version: (triangle)
>>  $f_k(x) = max(0, \mu (z) - z_k)$ <br> $z_k = ||x-c^{(k)}||_2$

---
__Gaussian mixtueres (GMM)__

[GMM](http://sanghyukchun.github.io/69/)
> - EM 알고리즘으로 솔루션 탐색
> - 초기값은 K-means 알고리즘 활용

![GMM](http://sanghyukchun.github.io/images/post/69-4.gif)



---

### 3.2 Featuer Extraction and Classification

위 알고리즘으로 추출된 "new representation"을 labeled training images에 적용해서 분류를 학습합니다.

__3.2.1 Convolutional Extraction__
> - 한 개의 patch의 "new representation"을 얻기 위해 특징추출기 f를 many sub-patches에 적용합니다.
> - [n, n, d] -> [(n-w)/s+1, (n-w)/s+1, k]
> - pooling over 4 quadrants -> summation
> - create a feature vector for classification

![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/analysisOfSingleLayer/Figure1.png)

__3.2.2 Classification__

SVM with L2를 사용했고, hyper parameters는 cross-validation으로 결정했다고 합니다.

## 4. Experiments and Analysis

실험 옵션들은 아래와 같습니다.
> 1. Whitenming
> 2. the number of features K
> 3. the stride s
> 4. receptive field size w

![Figure3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/analysisOfSingleLayer/Figure3.png)
![Figure4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/analysisOfSingleLayer/Figure4.png)

### 4.1 Visualization

> - Autoencoders, sparse RBMs가 Gabor filters와 비슷한 결과를 본 논문 전에도 이미 잘 알려져 있었답니다.
> - Clustering Algorithms도 비슷한 결과를 처음이라고 합니다.

### 4.2 Effect of whitening
> - autoencoders and RBM에서는 역할이 애매하다. <br>
(feature 수를 어짜피 많이 가져갈 거니까)
> - Clustering Algorithm에서는 whitening 차이가 크다. <br>
(clustering에서는 데이터의 correlations를 고려하지 않기 때문이라는데)

### 4.3 Number of features

> - Feature 갯수를 바꿔가며 실험: 100 / 200 / 400 / 800 / 1200 / 1600
> - Feature 수가 많으면 많을 수록 좋은 성능을 보임
> - Hyper parameter tuning이 필요없는 __K-means clustering의 성능이 제일 좋았다는 것__이 주목할 만한 성과라 합니다.



### 4.4 Effect of stride

> - stride 크기에 따른 연산비용이 부담되지만.. Stride는 1일때가 성능이 제일 좋음

### 4.5 Effect of receptive field size

> - small size works better
> - especially, when the input size is small
