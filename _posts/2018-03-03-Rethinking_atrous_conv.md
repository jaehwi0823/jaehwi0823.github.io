---
layout: article
title: Rethinkng Atrous Convolution for Semantic Image Segmentation review
tags: [paper, review, ml, conv, sematic, segmentation, deeplab]
---

# Rethinkng Atrous Convolution for Semantic Image Segmentation - DeepLab v3

## 1. Introduction

1. Reduced feature resolution caused by consecutive pooling or convolution strideing --> Atrous Convolution
2. The existance of objects at multiple scales --> Image Pyramid + Encoder-decoder Structure + Atrous Conv. + Spatial Pyramid Pooling(SPP)

## 2. Related Work

### 2.1 Image Pyramid

![Image_Pyramid.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Image_Pyramid.png)

Feed each scale input to a DCNN and merge the feature maps from all the scales. Apply multi-scale inputs sequentially from coarse-to-fine.
However, it doesn't scale well for larger/deeper DCNNs due to limited GPU.

1. From small scale inputs, the long-range context
2. From large scale inputs, the small objects details

### 2.2 Encoder - decoder

![Encoder_Decoder.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Encoder_Decoder.png)

### 2.3 Context module

Encoding long-range context. <br>
One effective method is to incorporate **DenseCRF**.

![DenseCRF.jpeg](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/DenseCRF.jpeg)
<center>[Image reference](http://slideplayer.com/slide/784090/)</center>

### 2.4 Spatial Pyramid Pooling (SPP)


![SPP.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/SPP.png)



### 2.5 Atrous Convolution(Dilated Conv)

Atrous Convolution: powerful tool to 
    1. explicitly adjust filter's field of view 
    2. control the resolution of feature responses computed by DCNN

![Atrous_detail.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Atrous_detail.png)

![Dilated_Conv.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Dilated_Conv.png)


## 3. Method
### 3.1 Atrous Convolution for Dense Feature Extraction

Atrous Conv.

![Dilated_Conv2-1.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Dilated_Conv2-1.png)

![Dilated_Conv2-2.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Dilated_Conv2-2.png)

![Dilated_Conv2-3.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Dilated_Conv2-3.png)

<center> [Image Reference](http://iamyoonkim.tistory.com/18) </center>


### 3.2 Going Deeper with Atrous Conv.

** Atrous Convolution **
Control the [output_stride] with **atrous rate ** ** *r* **
1. Duplicate several copies of the last ResNet block
2. 3 x 3 conv except for the last one with stride 2

![Figure3.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Figure3.png)

** Multi-grid Method **
<br> one block with multi-rate Atrous Conv Filters
<br> EX) Multi_Grid = (1,2,4)


### 3.3 Atrous Spatial Pyramid Pooling(ASPP)

ASPP is effectively captures multi-scale information

![ASPP.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/ASPP.png)
![ASPP_DeepLab.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/ASPP_DeepLab.png)

<h4> * Problem </h4>
As the sampling rate becomes larger, the number of valid filter weights becomes smaller. 
<br>valid weights: ???

![Figure4.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Figure4.png)

<h4> * Solution </h4>
1. Apply global average pooling on the last feature map of the model
2. Feed the resulting image-level features (&BN) to a 1x1 conv with 256 filters
3. Bilinearly upsample the feature to the desired spatial dimension

![Figure5.png](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/Figure5.png)

__Related Code__

![ASPP](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/rethinkingAtrousConv/ASPP_Structure.png)
