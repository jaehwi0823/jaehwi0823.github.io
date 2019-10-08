---
layout: article
title: Annotating Object Instatnces with a Polygon-RNN review
tags: [paper, review, ml, object, detection, polygon, rnn]
---

# Annotating Object Instatnces with a Polygon-RNN

<br> 2018.06.23
<br> Jaehwi Park

## 1. Introduction

> - The Goal of this paper is to make Annotating process faster and as precise as current datasets

![F1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/polygonRNN/Figure1.png)


## 2. Related Work

> - Semi-automatic annotation
> - Annotation tools
> - Object instance segmentation

## 3. Polygon-RNN

> 1. Image representation via CNN
> 2. Predict a vertex at every time step

![F2](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/polygonRNN/Figure2.png)


### 3.1.1 Image Representation via a CNN with Skip Connections

> - VGG-16 architecture
> - No fully connected layers as well as the last max-pooling layer
> - skip-connection !?
> - all 3x3 kernel followed by a ReLU

![vgg16](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/polygonRNN/vgg16.png)

### 3.1.2 RNN for Vertex Prediction

> - two-layer __ConvLSTM__ with kernel size of 3x3 and 16 channels 
> - vertex prediction as a classification task
> - Output: one-hot encoding of (DxD+1) grid
> - (D x D + 1): possible 2D position of the vertex
> - last one: The end-of-sequence token

![ConvLSTM](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/polygonRNN/ConvLSTM.png)

__ treat the first vertex as special __
> - two layers after CNN encoder
> - one branch predicts object boundaries
> - the other takes boundaries + features => predicts the vertices of the polygon
> - binary classification problem in each cell

### 3.2 Training

> - cross-entropy at each time step of the RNN
> - smooth not to over penalize 근접한 결과 => ??

__first vertex__

> - multi-task loss
> - logistic loss for every location in the grid

### 3.3 Inference and Annotators in the Loop

> - taking the vertex with the highest log-prob at each time step
> - the annotator can correct the prediction at any time step => feed into the next time-step

### 3.4 Implementation Details

> - D = 28
> - perform polygon simplification with zero error: 
<br>1. remove vertices on a line <br> 2. leave only one on a grid position
