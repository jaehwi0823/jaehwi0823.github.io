---
layout: article
title: AUTO-KERAS review
tags: [paper, review, ml, automl, keras]
---


# AUTO-KERAS: EFFICIENT NEURAL ARCHITECTURE SEARCH WITH NETWORK MORPHISM

<br>2018.11.18
<br>Jaehwi.Park

## 1. Introduction

> - 지금까지 대부분의 NAS는 강화학습 or 유전알고리즘 기반이라, 연산비용이 높음. O(Nt) 인데 N이 매우 커야함
> - __Network Morphism__은 구조를 변경하면서도 그 functionality를 유지하는 장점이 있음
> - Bayesian Optimization 방식은 다른 NAS에 비해 적은 epoch으로도 better performance를 얻을 수 있음!
> - __Network Morphism__에서는 어떻게 구조를 수정(operation selection)할지를 결정하는 것이 가장 중요함
>> - 현재 SOTA(Cai et al., 2018)는 강화학습 기반으로 시간이 너무 오래걸림
>> - 간단한 접근 (Elsken et al., 2017)은 Local optimum에 빠지기 너무 쉬움

---
> - __Bayesian Optimization__를 __NAS__에 적용하는데 몇 가지 어려운 점이 있음
>> - 1) 전통적으로 __underlying Gaussian Process__는 유클리디언 공간에서 사용되는데, 신경망구조는 그렇지 않음
>> - 2) Acquisition 함수가 최적화 돼야 하는데 이게 tree-structured 이므로, gradient-based update를 쓰기가 어려움
>> - 3) skip-connection이 허용되는 탐색공간에서의 신경망 변형 operation은 매우 복잡함
<br> 　어떤 operation은 중간 결과물을 수정시켜서 consistency를 깨트릴 수 있음

---

> - 본 논문에서는 Bayesian Optimization을 활용하여 매번 가장 promising한 operation을 선택함
> - consistency를 유지하며 구조를 변경하기 위해 얼마만큼의 operation이 필요할지 측정함
> - 또한, tree-structured search space에서 사용 가능한 새로운 __acquisition function optimizer__를 제안함
> - 그리고, 신경망 변화를 담을 수 있는 graph-level morphism을 정의함

## 2. Problem Statement

> Notation
> - $\mathcal{F}$: a neural architecture search space
> - $X$: the input data
> - $Cost( \cdot )$: the cost metric
> - $\mathcal{f}^*$: optimal neural network
> - $\theta_{\mathcal{f}^*}$: trained parameters of optimal neural network
> - $w(f)$: # of parameters in $\mathcal{f}$
> - $G_f = (V_f, E_f)$: the computational graph of a neural network
> - $u \in V_f$: intermediate input tensor
> - $v \in V_f$: intermediate output tensor
> - $e_{u→v} \in E_f$: directed edge,
> - $u \prec v$: v 에서 u로 연결

___

> Problem Definition
> - $\mathcal{f}^* = argmin_{\mathcal{f} \in \mathcal{F}} min_{\theta_f} Cost(\mathcal{f}(X;\theta_f))$,
> - where $\theta_f \in \mathbb{R}^{w(f)}$

---

> The serch space $\mathcal{F}$ satisfies
> - (1) $G_f$ is a directed acyclic graph (DAG)
> - (2) $\forall u v \in V_f, (u \prec v) \lor (u \prec v)$
>> - skip connection을 포함하는 개념

## 3. Network Morphism with Bayesian Optimization

> 전통적인 베이지안 최적화는 세 단계로 이루어짐
>> - (1) __UPDATE__: 현재 구조에서 GP를 학습
>> - (2) __GENERATION__: Acquisition Func.를 최적화시켜 다음 구조를 생성
>> - (3) __OBSERVATION__: 새 구조에서 신경망을 학습시켜 성과 측정

### 3.1 Neural Network Kernel for Gaussian Process

> NAS 공간은 유클리디언 공간이 아닌데 GP를 학습해야함
> - 레이어 또는 패러미터 갯수가 계속 달라지므로 NAS에서 신경망 구조를 직접적으로 벡터화하는 것은 실용적이지 못함
> - Gaussian Process는 kernel method의 일종이므로, 본 논문에서는 __a neural network kernel function__을 새롭게 제안함

___

> __KERNEL Definition__
> - "DEEP GRAPH KERNEL" 논문의 아이디어를 차용해서 edit-distance를 정의함
> - edit-distance는 신경망이 $f_a → f_b$ 변경되기 위해 필요한 operation 숫자임
> - Kernel Function
>> $\kappa (f_a, f_b) = e^{-\rho^2(d(f_a,f_b))}$, <br>
>> where $d(\cdot , \cdot)$ is the edit-distnace, whose range is $[0, + \infty]$ <br>
>> $\rho$ is a mapping function from origin metric space to a new one, using Bourgain Theorem (Bourgain, 1985)
> - the edit-distance를 구하는 문제는 NP-hard problem이라서 approximated solution을 제안함
>> $d(f_a, f_b) = D_l(L_a, L_b) + \lambda D_s(S_a, S_b)$
>> - $D_l$ is the edit-distance
>> - $L_a , L_b$ are Sets of layers
>> - $D_s$ is the approximated edit-distance for morphing skip-connection
>> - $S_a, S_b$ are Sets of skip-connections
>> - $\lambda$ is the balancing factor

---

> __Calculating $D_l$__
> - $D_l(L_a, L_b) = min\displaystyle\sum_{i=1}^{|L_a|} d_l(l_a^{(i)},\varphi_l(l_a^{(i)})) + \mid |L_b| - |L_a| \mid$, <br>
> where $|L_b| > |L_a|$ <br>
> $\varphi_l : L_a → L_b$ is an injective function of layers <br>
> satisfying $\forall i <j, \varphi_l(l_a^{(i)}) \prec \varphi_l(l_a^{(j)})$ <br>
> if $L_a$ and $L_b$ are all sorted in topological order <br><br>
> - $d_l(l_a, l_b) = \displaystyle\frac{|w(l_a) - w(l_b)|}{max[w(l_a),w(l_b)]}$
> <br> where w(l) is the width of layer l
> ![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/autokeras/Figure1.png) <br><br>
> - $D_l(L_a,L_b)$ 수식에서,
>> - 앞의 시그마 부분은 노드별 차이를 의미하고,
>> - 뒤 절대값 부분은 전체 노드의 숫자를 의미함
>> - 또한, 매칭되는 쌍($l_a^{(i)}$, $\varphi_l(l_a^{(i)})$)을 찾는 방법은 Dynamic Programming을 활용함
>>> - $A_{i,j} = max[A_{i-1,j}+1, A_{i,j-1}+1, A_{i-1,j-1}+d_l(l_a^{(i)},l_b^{(j)})]$
>>> - $A_{i,j}$는 $D_l(L_a^{(i)}, L_b^{(j)})$의 최솟값이고
>>> - $L_a^{(i)}$는 a 신경망

---
> __Calculating $D_s$__
> - $D_s(S_a, S_b) = min\displaystyle\sum_{i=1}^{|S_a|} d_s(s_a^{(i)},\varphi_s(s_a^{(i)})) + \mid |S_b| - |S_a| \mid$
>> - $|S_b| - |S_a|$ 부분은 매칭되지 않은, 잔여 skip-connection을 의미함
> - $d_s(s_a, s_b) = \displaystyle\frac{|u(s_a) - u(s_b)| + |\delta(s_a) - \delta(s_b)|}{max[u(s_a),u(s_b)]+max[\delta(s_a), \delta(s_b)]}$
>> - $u(s)$는 skip-connection이 시작하는 topological rank이며,
>> - $\delta(s)$는 skip-connection이 이동하는 거리 (마지막 rank - 첫 rank)
> - $D_s$ minimization은 "the bipartite graph problem(이분 그래프 매칭 문제)"으로 생각할 수 있으며, Hungarian algorithm(Kuhn, 1955)으로 계산함


---
> __Proof of Kernel Validity__
> - Theorem 1, Theorem 2에서 증명하고 있음..

---
### 3.2 Acquisition Function Optimization for Tree Structured Space

> Acquisition Function의 최적화는 두 번째로 해결해야 하는 사항이었음. <br> 왜냐하면 마찬가지로 Search Space가 Euclidean Space가 아니라서 Traditional Acquisition Function을 쓸 수 없기 때문임
> - 본 연구에서는 Upper-confidence bound (UCB) (Auer et al., 2002)를 선택함
>> - $\alpha(f) = \mu(y_f) - \beta\sigma(y_f)$,
>> - where $y_f = min_{\theta_f} Cost(f(X;\theta_f))$: Cost(f)에서 가능한 가장 작은 추정치
>> - $\beta$는 balancing factor: Exploration & Exploitation 사이의 균형
>> - $\mu(y_f), \sigma(y_f)$는 변수 $y_f$의 posterior mean과 s.d.
> - UCB 함수값은 탐색 history에서 손실함수값 $c^{(i)}$와의 직접적인 비교가 가능함
>> - Search History: $\mathcal{H} = (f^{(i)},\theta^{(i)},c^{(i)})$ 
> - 새롭게 생성된 신경망: $\hat{f} = argmin_f \alpha(f)$

---
> __the tree-structured space__
> - $\alpha(f)$ 최적화 과정에서, $\hat{f}$를 $f^{(i)}$와 $O$로부터 얻을 수 있음
>> - $f^{(i)}$는 Search History 내의 관측된 신경망이고,
>> - $O$는 변경되기 위한 operations sequence
> - Morph $f$ to $\hat{f}$:
>> - $\hat{f} \gets \mathcal{M}(f, O)$

---
> 가장 흔한 "Network Morphism"의 약점은 네트워크가 계속 커진다는 것임
>> - 계속 커지기만 하기 때문에 작은 구조에서의 탐색(exploration)이 충분히 이뤄지지 않음
>> - "Network Morphism" NAS의 다른 사례(Elskenet al., 2018)에서도 항상 네트워크가 엄청 크게 만들어져서 끝남
>> - 그 이유는 기준 신경망이 항상 직전 스텝의 the best-architecture

> - 그런데 본 논문의 tree-structured search는 History내의 작은 구조도 여러번 다시 선택될 수 있기 때문에 다르다고 함

---
> 그런데 현재의 주류인 gradient-based or Newton-like method는 tree-strucrued에서는 사용할 수 없음
> - 그래서 tree-strucrued에서는 TreeBO (Jenatton et al., 2017)가 많이 사용됐지만, 본 논문과는 약간 달라서 사용할 수 없었음
> - NASBOT (Kandasamy et al., 2018)에서는 evolutionary 알고리즘이 사용됐지만 이건 계산비용이 너무 높았음
> - 그래서 __A* search and Simulated Annealing__을 기반으로 한 새로운 방법을 제안함
>> - A*은 tree-structure search를 위한 것임
>> - Simulated Annealing은 탐색과 이용의 균형을 위한 것임
>> - Algorithm 1의 Input: ① minimum temperature $T_{low}$, ② S.A.를 위한 비율 r, ③ search history $\mathcal{H}$
>> - Algorithm 1의 Output: ① $f \in \mathcal{H}$, ② Operation Sequences $O$




> __Algorithm 1__ <br>
> ![Algorithm1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/autokeras/Algorithm1.png)
> - line 02 ~ 06: priority queue (PQ)에 f를 cost 순서대로 넣음
> - line 07 ~ 07: setting in A* search
> - line 08 ~ 10: 가장 작은 acquisition function 값을 갖는 신경망부터 차례대로 queue에서 꺼냄
>> - $\Omega$는 $f$에 적용 가능한 모든 operation
>> - $f' \gets \mathcal{M}(f, O)$는 신경망 $f$에 operation sequences $o$를 적용해서 변형
> - line 11 ~ 13: Exploration 정도를 조절, 일부만 PUSH
>> - $\displaystyle{e^{\frac{c_{min} - \alpha(f')}{T}}}$는 S.A.에서 일반적으로 사용하는 수치
> - line 14 ~ 16: $f_{min}$, $c_{min}$ 업데이트

### 3.3 Graph-Level Network Morphism Operations

> 본 연구의 세 번째 과제는 중간 중간의 output tensor의 shape을 구조 변경과 무관하게 유지하는 것
> - 이전 연구에서는 이것을 단순하게 처리하기 위해 layer-level 변형만을 고려했음
> - 본 논문에서는 단일 graph(노드) 단위의 변형을 고려함
> - 신경망에 반영할 수 있는 operation은 네 가지가 있음 (Elsken et al., 2017)
>> 1. __deep(G,u)__: inserting a layer to $f$
>>> - G는 그래프
>>> - u는 추가할 위치 노드
>> 2. __wide(G,u)__: widening a node in $f$
>>> - 여기에서의 u는 변경된 output tensor shape 값을 지닌 노드
>> 3. __add(G,u,v)__: adding an additive connection from node $u$ to node $v$
>> 4. __concat(G,u,v)__: adding a concatenative connection from node $u$ to node $v$

---
> - Deep(G, u)는 새로 추가된 layer의 가중치만 초기화해주면 되지만, 나머지 세 개는 더 복잡함
> - __① wide(G,u)__
>> - 먼저, 네트워크에서 변경할 부분을 더 잘 설정하기 위해 _effective area $\gamma$_를 정의해야함
>> - $\gamma$는 그래프 노드들의 집합이며, 아래의 규칙에 의해 재귀적으로 정의됨
>>> $L_s$가 set of fully-connected layers and convolutional layes일때,
>>> 1. $u_0 \in \gamma$
>>> 2. $v \in \gamma, if \exists e_{u \to v} \notin L_s, u \in \gamma$
>>> 3. $v \in \gamma, if \exists e_{v \to u} \notin L_s, v \in \gamma$
>> - wide(G,u)는 두 개의 layers sets를 변경해야 함
>>> 1. $L_p = \{ e_{u \to v} | v \in \gamma \}$: 이전(previous) layer set으로, output tensor가 늘어나야 함
>>> 2. $L_n = \{ e_{u \to v} | u \in \gamma \}$: 다음(next) layer set으로, input tensor가 늘어나야 함
> - __② add(G,$u_0$,$v_0$)__
>> - skip-connection을 위한 추가적인 pooling layer가 필요할 수 있음
>> - $u_0$와 $v_0$는 채널수가 같지만, pooling layer 때문에 shape이 다를 수 있음
>> - 그래서 $u_0$에서 $v_0$까지의 모든 pooling layer들을 한 데 모은 효과와 같은 pooling layers set이 필요함
>>> - $L_o = \{ e \in L_{pool} | e \in p_{u_0 \to v_0} \}$
>>> - $p_{u_0 \to v_0}$는 any path btw them
>>> - $L_{Pool}$은 pooling layer set
>> - 또다른 $L_c$가 $L_0$ 다음에 사용돼 $u_0$와 $v_0$의 width를 맞춰줌
> - __③ concat(G,$u_0$,$v_0$)__
>> - 연결되는 tensor는 원래의 $v_0$ shape 보다 큼(wider)
>> - 연결되는 tensor은 새로운 레이어 $L_c$의 input이 되어 width를 줄여 $v_0$와 같도록 만듦
>> - add와 마찬가지로 추가적인 pooling layer set이 필요할 수도 있음

### 3.4 Architecture Performance Estimation

> - 성과를 측정하는 여러 방법이 있지만, 가장 좋은 방법은 직접 학습을 시키는 것임
> - 본 연구의 training process에 두 가지 필수사항이 있는데,
>> 1. training process는 여러가지 다른 신경망에 적용 가능해야 함. 신경망마다 converge하는 epoch 숫자 등이 달라지는 것을 고려해야 한다는 의미
>>> - Threshold를 두고 몇 epoch동안 변화가 없으면 early stop
>> 2. performance curve에서 noise의 영향을 받으면 안됨
>>> - mean of metric values를 the final metric value 대신 사용함. 물론 기준은 validation set

### 3.5 Time Complexity Analysis

> - $O$(Update + Generation) << $O$(Observation)

## 5. Experiment

__제한된 시간(12시간)__을 제약으로 실험했을때,

![Table1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/autokeras/Table1.png)

> - 기본적으로 Bayesian 류의 성능이 좋음
> - 제안한 방법이 기존 SOTA보다 좋음

---

![Figure4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/autokeras/Figure4.png)

> - 두 개의 hyperparameter: Distance, UCB의 영향력이 비슷함
> - A\* 대신 breadth-first search를 사용한 BFS는 속도는 제일 빠른데 성능은 영..
> - BO는 성능 탐색은 잘 하는 것 같은데 느림
> - 그래서 제안한 방법이 제일 좋음
