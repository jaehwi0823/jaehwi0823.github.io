---
layout: article
title: Once for All review
tags: [paper, review, ml, ofa]
---

# Once for All: Train One Network and Specialize it for Efficient Deployment

2019.10.05 <br>
jaehwi Park

## 0. Abstract

 - 매번 모형을 처음부터 학습하는 것: high cost
 - "Once for All(OFA)"은 여러 시나리오들을 커버할 수 있도록, 모형 학습과 모형 탐색을 분리합니다.
 - 특화된 모형을 학습하는 것 대신, 다양한 architectural settings가 가능한 OFA 모형을 추천합니다.
 - 주어진 deployment scenario에 맞게, OFA network에서 특화된 sub-network를 학습없이 추후에 찾을 수 있습니다. -> training time: O(1)
 - 그러나, 많은 sub-networks 간의 간섭을 피할 수 가 없습니다.
 - 그래서, "the progressive shrinking" 알고리즘을 제안합니다.
 - 해당 알고리즘 덕분에 동등한 성능을 가진 $10^{19}$ 이상의 sub-networks 학습이 가능합니다.
 - OFA는 NAS SOTA 보다 훨씬 빠르며, 유사하거나 더 나은 성능을 보여줍니다.

### <center><strong>  [Figure1: Concept Overview] </strong></center>

![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Figure1.png)

## 1. Introduction

 - 어떻게 작업 환경(latency, energy consumption 등)이 다른 많은 플랫폼들에 딥러닝 모형을 deploy 할 것인지가 중요한 이슈가 됐다고 합니다.
 > 예시> iPhone-XS-Max와 5-year-old iPhone-6 모두에서 잘 작동해야 한다는데..
 - 다양한 하드웨어 사양들에 최적화하기 위해, 지금까지는 __1) compact models__ & __2) model compression__ 들이 연구됐습니다.
 - 그럼에도 모든 deployment scenario에 맞는 수 많은 모형들을 만들어내는 것은 현실적으로 어렵습니다.
 - 매번 특정 사양에 맞게 Architecture design이 필요하고, 그에 맞게 모형을 처음부터 재학습해야하기 때문입니다.

---

 - 본 논문에서 즉시 다양한 환경에 배포될 수 있는 Once-for-All network를 소개합니다.
 - OFA network는 Inference시, 환경에 맞게 전체 네트워크 중 일부분만을 선택해 진행됩니다.
 - OFA network는 서로 다른 [depths], [widths], [kernel sizes], and [resolutions] 를 갖는 sub-networks로 이루어져 있습니다.
 - OFA network는 모형 학습(training) 단계와, 모형 선택(specialization) 단계가 분리되어 있습니다.
 - 학습은 단 하나의 OFA network를 학습하지만, 모든 sub-networks가 서로 방해하지 않으며 정확도를 높이도록 진행합니다.
 - Interfering with each other 문제를 해결하기 위해, "the progressive shrinking algorithm"을 제안합니다.

## 2. Related Work

### 2.1 Efficient Deep Learning

 - 모형을 효율적으로 배포하기 위해 두 가지 방법이 제안되어 왔습니다.
 - 먼저, 효율적 구조의 모형들이 제안됐습니다.
 > - SqueezeNet
 > - MobileNets
 > - ShuffleNets
 
 - 또한, 모형 압축 방법들이 제안됐습니다.
 > - Network pruning by removing redundant units or channels
 > - Quantization approaches

### 2.2 Neural Architecture Search

 - NAS는 모형을 스스로 디자인하고, 학습합니다.
 - 초기 NAS모형은 모형 배포 환경을 고려치 않았습니다.
 > - NASNet
 > - AmoebaNet
 
 - 최신 NAS 방법론은 직접적으로 하드웨어 feedback을 받아 reward signal에 반영합니다.
 - 그리하여 여러 환경에 최적화된 모형을 찾아낼 수 있지만, 항상 재학습이 필요하므로 scalable 하지 못합니다.

### 2.3 Dynamic Neural Networks

 - Dynamic NN은 하나의 모형으로 여러가지 환경에서 사용할 수 있게 하는 방법입니다.
 - ResNet50 같은 모형에서 input images에 따라 특정 구간을 건너뛰어 사용하는 것입니다.
 - adaptively하게 구간을 건너뛰기 위해 추가적인 controller 또는 gating modules를 제안하기도 했습니다.
 - 어떤 모형은 현재의 예측 confidence에 따라 모형 중간에서 예측을 종료하는 제시하기도 했습니다.
 - "Slimmable Nets"는 수 개의 다른 [width] 옵션을 주기도 했습니다.
 > - MobileNetV2 0.35, 0.5, 0.75, 1.0
 - 이런 모형들은 computational cost가 적었지만, 학습되지 않은 환경에서 성능이 많이 저하됐으며 유연성이 굉장히 제한적이었습니다.

## 3. Method

### 3.1 Problem Formalization

 - Training the Once-for-All network는 다음과 같이 표현할 수 있습니다. 

 > <center> <strong> $min_{W_{O}} \displaystyle\sum_{arch_i} \mathcal{L}_{val} (C(W_O, arch_i)) $ </strong> </center>
 > - $W_O$: Once-for-All network weights <br>
 > - $arch_i$: architectural configurations <br>
 > - $C(W_O, arch_i)$: a selection scheme <br>

 - 본 논문의 모형은 다양한 [depths], [widths], [kernel sizes], and [resolutions] 를 갖습니다.
 - dilation 또는 #groups는 future work로 연구가 필요합니다.

### 3.2 Training the Once-for-all Network

__Preliminary.__

 - CNN은 일반적으로 몇 개의 단계가 반복되므로, building blocks들의 반복으로 생각할 수 있습니다.
 - 이러한 구조에서 모형에 다양성을 추가할 수 있습니다.
 > - network-level: elastic input sizes
 > - stage-level: elastic depths (skipping different numbers of blocks)
 > - block-level: elastic widths/kernel sizes (different # of channels & kernels)

__A Progressive Shrinking Approach.__

 - Once-for-All network를 한 번에 학습하는 것은 매우 어려운 일입니다.
 - 그러므로 a sequence of sub-tasks 최적화 문제로 치환합니다.

### <center><strong>  [Figure2: Once-for-All network training] </strong></center>

![Figure2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Figure2.png)

 - 먼저 [depths], [widths], [kernel sizes] 옵션의 최대값을 가지며 다양한 resolutions $\mathcal{R} \in \{128, 132, ..., 224 \}$를 갖는 모형을 학습합니다.
 - 그리고 점진적으로 더 작은 [depths], [widths], [kernel sizes] 값의 조합을 갖는 sub-networks를 학습합니다.
 - 이와 같은 방법은 세 가지 장점을 갖는다고 합니다.
 > - First, 한 번에 학습하는 것보다 학습이 훨씬 쉬워집니다.
 > - Second, 작은 모형들이 앞서 학습된 커다란 모형의 도움을 받아 학습이 쉬워집니다. 먼저 학습된 모형이 중요한 가중치들을 확보하여 좋은 출발점(good initialiation)을 제공하기 때문입니다. 이를 knowledge-distillation 기법이라고 합니다.
 > - Third, ordering to the shared weights를 제공하여, sub-networks가 larger sub-networks의 성능을 저하시키는 것을 막아줍니다.
 
---

__1) Elastic Resuolution__
> - 이론적으로 어떤 resolutions도 CNN에 입력될 수 있지만, 실무에서 학습에 활용되지 않은 size의 input에 대해서는 모형의 performance가 급격히 떨어지는 것을 관찰할 수 있습니다.
> - 그러므로 __BATCH별 서로 다른 image size를 채택하여__ elastic resolution이 적용될 수 있게 합니다.
> - 이는 data loader를 수정하여 쉽게 구현 가능합니다.

__2) Elastic Kernel Size__

> - 7x7 커널의 중앙은 5x5 커널 역할을 할 수 있고, 이는 또한 3x3 커널에 적용될 수 있습니다.
> - 여기에서 challenging point는 모든 centered sub-kernel들은 공유되지만 상황에 따라 다양한 역할을 해야한다는 것입니다.
> - 이 커널들의 weights는 각 상황에 따라 다른 분포를 갖거나, 다른 magnitude가 필요할지도 모릅니다.
> - 그러므로, 모두 같은 값을 갖도록 강제하는 것은 어떤 sub-networks의 성능을 떨어트릴 수 있습니다.
> - 그래서, weights의 공유가 필요할 때는 __"kernel transformation matrices"__ 를 도입했습니다.
> - block마다 서로 다른 transformation matrix를 사용하지만, 동일 블럭 내 동일 채널에서는 그 값을 공유합니다.
> - 그러므로 추가되는 가중치는 $(25^2 + 9^2)$ 뿐입니다.

### <center><strong>  [Figure3: Kernel Transformation Matrices] </strong></center>

![Figure3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Figure3.png)

__3) Elastic Depth__

> - Elastic Depth는 Stage-level의 작업입니다.
> - 하나의 stage는 output resolution이 동일한 a sequence of building blocks로 구성됩니다.
> - 또한, 각각의 building block은 한 개의 depth-wise convolution과 두 개의 point-wise convolutions로 구성됩니다.
> - 원래는 N blocks의 stage를 D blocks로 만들기 위해, 앞선 D blocks를 선택하고 나머지 N-D blocks를 건너뜁니다. <br>
(\*비교: NAS에서는 임의의 D blocks를 선택함)
> - 


### <center><strong>  [Figure4: Elastic Depth] </strong></center>

![Figure4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Figure4.png)

__4) Elastic Width__

> - width는 채널 수를 의미합니다.
> - 각각의 layer마다 서로 다른 "expansion ratio"를 선택할 수 있도록 했습니다.
> - "progressive shrinking scheme"을 위해 채널별 중요도(L1 norm)를 바탕으로 layer 내의 채널들을 정렬했습니다.
> - 정렬된 채널들 중 sub-network에서 필요한 만큼을 따로 선택했습니다.

### <center><strong>  [Figure5: Elastic Width] </strong></center>

![Figure5](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Figure5.png)

__5) Knowledge Distillation__

> - 학습 데이터의 hard labels와 학습된 full-network의 soft labels를 모두 사용했습니다.
> - $Loss = Loss_{hard} + \lambda Loss_{soft}$

### 3.3 Specialized Model Deployment with Once-for-All Network

 - OFA network를 학습했다면, deployment scenario에 따라 specialized sub-network를 선정합니다.
 - 타겟 환경의 효율 제약조건을 만족시키는 동시에 정확도를 최적화하는 sub-network를 선정해야 합니다.
 - 이 단계에서 search algorithm은 어떤 것을 써도 괜찮지만, RL / EV / GD 등은 추가적인 탐색 시간을 소요합니다.
 - 그래서 임의로 sub-network subsets을 뽑아, "Accuracy - Latency" 테이블을 생성합니다. 
 - 해당 테이블에서 목표 환경의 Latency에 해당하는 Accuracy값이 가장 큰 sub-network를 최종적으로 선정합니다.
 - 논문을 위한 실험에서는 16K sub-networks에 학습 데이터에서 뽑은 10K 검증 데이터로 accuracy를 측정했습니다.
 - 이 과정은 학습 환경과 동일하게, multiple input image sizes를 활용했습니다.
 - resolution이 증가하면 accuracy가 점진적으로 증가하는 경향성을 확인하기도 했습니다.
 - 속도 향상을 위해 resolution stride를 16으로 했습니다. (160, 176, ...)
 - 측정되지 않은 resulution은 비례식으로 근사했습니다.
 > 근사예시) $Acc_i(164) = (Acc_i(176) - Acc_i(160)) \times \frac{FLOP_i(164) - FLOP_i(160)}{FLOP_i(176) - FLOP_i(160)} + Acc_i(160)$

## 4. Experiments

### 4.1 Training the Once-for-All Network on ImageNet

__Training Details.__

 - Input sizes: 128 to 224 with a stride 4
 - Elastic Depth: 2, 3 or 4
 - Elastic width: 4, 5 or 6
 - Elastic kernel size: 3, 5 or 7
 - 그러므로 with 5 stages, there are $((3 \times 3)^2 + (3 \times 3)^3 + (3 \times 3)^4)^5 \approx 2 \times 10^{19}$ sub-networks
 > - Stage가 다섯개: (Each Stage)$^5$
 > - 만약 Depth(Block) = 2: (Each Block)$^2$
 > - width & kernel size: 1개의 블럭은 (width $\times$ kernel size) weights

![Table2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Table2.png)

### 4.2 Specialized Sub-networks for Different Hardware Platforms and Constraints

![Figure6](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/OFA/Figure6.png)
