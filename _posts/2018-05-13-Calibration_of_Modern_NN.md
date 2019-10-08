---
layout: article
title: On Calibration of Modern Neural Network review
tags: [paper, review, ml, object, calibration]
---

# On Calibration of Modern Neural Network

2018.05.13. <br>
Jaehwi Park

## 1. Introduction

We discover that modern neural networks, unlike those from a decade ago, are poorly calibrated.
<br><br>
In real-world decision making systems, classification networks must not only be accurate, <br>
but also **should indicate when they are likely to be incorrect.**
<br><br>
Our goal is not only to understand why neural networks
have become miscalibrated, but also to identify what methods
can alleviate this problem.
<br><br>
![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/CalibrationOfModernNN/Figure1.png)




## 2. Definition

**Perfect Calibration **
> $\mathbb{P}(\hat{Y}=Y \mid \hat{P}=p) = p, \forall p \in [0,1]$ <br>
> Confidence 와 실제 확률이 같은 경우

**Reliability Diagram -> Figure1
> 그냥 Histogram 입니다. <br> accuracy는 실제 맞춘 비율, Confidence는 평균 확률


**Expected Calibration Error (ECE)
> $  \mathbb{E}_{\hat{P}} [\mid \mathbb{P}(\hat{Y}=Y \mid \hat{P}=p)-p \mid]$ <br>
> 평균 오차

**Maximum Calibration Error (MCE)
> $  max [\mid \mathbb{P}(\hat{Y}=Y \mid \hat{P}=p)-p \mid]$ <br>
> MAX 오차

** Negative Log Likelihood (NLL)
> $ \mathcal{L} = - \displaystyle\sum_{i=1}^{n} log(\hat{\pi} (y_i | x_i)) $ <br>
> NLL은 모형의 Loss 입니다.
> 확률분포의 Distance로 해석할 수 있다고 합니다. [상세설명](https://ratsgo.github.io/deep%20learning/2017/09/24/loss/)


## 3. Observing Miscalibration

![Figure2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/CalibrationOfModernNN/Figure2.png)

1. with 64 conv filters / layer
2. fixed the depth at 14 layers
3. 6 layer Conv Net
4. 110-layer ResNet

### Model Capacity

Although increasing depth and width may reduce classification error, we observe that these increases negatively affect model calibration.
> Depth and Width 가 Calibration 에 Negative Effect를 주는듯 함

During training, after the model is able to correctly classify (almost) all training samples, NLL can be further minimized by increasing the confidence of predictions. Increased model capacity will lower training NLL, and thus the model will be more (over)confident on average.
> 학습중에 accuracy가 100% 


### Batch Normalization

While it is difficult to pinpoint exactly how Batch Normalization affects the final predictions of a model, we do observe that models trained with Batch Normalization tend to be more miscalibrated. 
> 잘 모르겠지만 쓰면 좋긴 한데 Calibration은 나빠짐 <br>
> (참고) BN은 학습속도를 높이고, 다른 RG기법 필요없게 하고, 성능을 좋아지게 할 때도 있다고 합니다.

### Weight Decay

> (참고) BN 때문에 "less L2"가 정규화를 더 잘한다는 주장이 있다네요. 그런가요?

While the model exhibits both over-regularization and under-regularization with respect to classification error, it does not appear that calibration is negatively impacted by having too much weight decay. Model calibration continues to improve when more regularization is added, well after the point of achieving optimal accuracy.
> WD를 많이 해주면 Optimal Error는 얻을 수 없으나 Calibration이 떨어지는 것을 확인할 수 있습니다.






### NLL

The disconnect occurs because neural networks can **overfit to NLL without overfitting to the 0/1 loss.** NLL overfits during the remainder of training. Surprisingly, **overfitting to NLL is beneficial to classification accuracy**.
> 이 논문의 Killing Part가 아닐까 합니다. <br>

![Figure3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/CalibrationOfModernNN/Figure3.png)
 <center> **learning rate is dropped at epoch 250!** </center>
> TEST NLL이 증가하는데, TEST ERROR가 낮아지고 있습니다. <br>

the network learns better classification accuracy at the expense of well-modeled probabilities.
> 확률값(Confidence)을 희생해서 정확도를 얻는 것이 Miscalibration의 이유라고 합니다.
> 그런데 왜 그러한 현상이 발생하는지에 대한 설명은 없습니다...

The observed disconnect between NLL and 0/1 loss suggests that these high capacity models are not necessarily immune from overfitting, but rather, overfitting manifests in probabilistic error rather than classification error.
> Capacity가 큰 모형에서 Overfitting이 어느정도 필요하다고 이해할 수 있다고 합니다.

## Calibration Methods
*All methods are post-processing steps that produce (calibrated) probabilities. Each method requires a hold-out validation set, which in practice can be the same set used for hyperparameter tuning. We assume that the training, validation, and test sets are drawn from the same distribution.*
> 1. 모든 방법은 일단 학습이 끝난 후(post-processing)에 확률값을 보정합니다. <br>
> 2. 모든 학습은 Train / Validation / Test Set으로 구성돼야 합니다. <br>
> 3. 모든 Train ~ Test Set이 같은 분포에서 추출됐다고 가정합니다.

### 4.1 Calibrating Binary Models
skip

**[Platt Scaling]**

$\hat{q_i} = \sigma(az_i+b) $, <br>
$\sigma$ is softmax, <br>
$z_i$ is logit(NN output), <br>
Parameters a and b can be optimized using the NLL loss over the validation set.


### 4.2 Extension to Multiclass Models
skip

**[Temperature Scaling]** <br>

Temperature scaling, the simplest extension of Platt scaling, uses a single scalar parameter T > 0 for all classes.

$\hat{q_i} = max_k \sigma(z_i / T)^{(k)} $, <br>

> 단순히 logit vector을 T>0 로 나눠서 Softmax 함수를 통과시킵니다. <br>
> 적절한 T의 크기는 Validation Set의 값을 활용합니다. <br>
> 임의의 상수로 나눠준 것이므로 기존의 rank는 유지됩니다. <br>


