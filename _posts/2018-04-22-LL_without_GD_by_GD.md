---
layout: article
title: Learning to Learn without Gradient Descent by Gradient Descent review
tags: [paper, review, ml, automl]
---

# Learning to Learn without Gradient Descent by Gradient Descent

2018.04.22 <br>
Jaehwi Park

## Abstract

RNN Optimizer
Gaussian Process Bandits

## 1. Introduction

**The Learner** can implement and be trained by many different algorithms

> - Gradient Descent <br>
> - Evolutionary Strategies <br>
> - Simulated Annealing <br>
> - Reinforcement Learning <br>


In this work, **the goal** of meta-learning is to produce an algorithm **for global black-box optimization.**

$X^* = argmin_{x\in\chi}f(x)$, <br>
where, <br>
&nbsp;&nbsp; $\chi$ is some search space of interest. <br>
&nbsp;&nbsp; $f$ is a black-box function


### Bayesian Optimization
One of the most popular Black-box optimization methods. <br>
It is a sequential model-based decision making approach with two components.

> ** 1. Probabilistic model **
 - consisting of a prior distribution that captures our beliefs about the behavior of the unkown objective function 
>  > Beta-Bernoulli bandit <br>
   > Random forest <br>
   > Bayesian neural network <br>
   > Gaussian process (GP)

> ** 2. Acquisition function **
 - which is optimized at each step so as to trade-off exploration and exploitation
>  > Thompson sampling
   > Information gain
   > Probability of improvement
   > Expected improvement
   > Upper confidence bounds

> - optimizing at each step can be a significant cost and has some theoretical concerns

<br>
** This paper shows "A Learning to learn Approach" VS "Bayesian Optimization" **
 1. a large number of differentiable functions generated with a GP to train RNN
 2. Two types of RNN
   - LSTMs (Long Short Term Memory Networks)
   - DNCs  (Differentiable Neural Computers)



## 2. Learning Black-box Optimization

A black-box optimization algorithm can be summarized by the following loop:

> 1. Given the current state of knowledge $h_t$ propose a query point $x_t$
> 2. Observe the response $y_t$
> 3. Update any internal statistics to produce $h_{t+1}$

In this work,
> $h_t, x_t = RNN_{\theta}(h_{t-1}, x_{t-1}, y_{t-1})$, <br>
> $y_t$ ~ $p(y \mid x_t)$
![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/LLwGG/Figure1.png)

### 2.1 Loss Function

** summed loss **

> $L_{sum}(\theta) = \mathbb{E}_{f,y_{1:T-1}}[\displaystyle\sum_{t=1}^T f(x_t)]$ <br>
> A key reason to prefer $L_{sum}$ is that the amount of information conveyed by $L_{final}$ is temporally very sparse.
> Nothing explicitly encourages the optimizer itself to explore


** Expected posterior improvemet Loss **

> $L_{EI}(\theta) = -\mathbb{E}_{f,y_{1:T-1}}[\displaystyle\sum_{t=1}^T EI(x_t \mid y_{1:t-1})]$ <br>


** Observed improvemet Loss **

> $L_{OI}(\theta) = \mathbb{E}_{f,y_{1:T-1}}[\displaystyle\sum_{t=1}^T min (f(x_t) - min_{i<t}(f(x_i)) ,0 )]$ <br>



### 2.2 Training Function Distribution

Training Distribution: ** Gaussian Process **
> The joint distribution of fuction values at any finite set of query points follows a multivariate Gaussian distribution


