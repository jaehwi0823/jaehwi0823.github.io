---
layout: article
title: Stand-Alone Self-Attention in Vision Models review
tags: [paper, review, ml, attention, vision]
---

# Stand-Alone Self-Attention in Vision Models

## 1. Introduction

 - Capturing long-range interactions for convolutions is challenging because of their poor scaling properties with respect to large receptive fieds
 - Recently attention modules have been employed in discriminative computer vision models to boost the performance of traditional CNNS
 - Most notably, a channel-based attention mechanism termed __Squeeze-Excite__
 - Global attention layers as an add-on to existing CNN, limiting its usage to small inputs which typically require significant downsampling of the original image

## 2. Background
### 2.1 Convolutions


```python
from __future__ import print_function
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import datasets, transforms
from torch.autograd import Variable
from torchsummary import summary


train_dataset = datasets.MNIST(root='./mnist_data/', train=True, transform=transforms.ToTensor(), download=True)
test_dataset = datasets.MNIST(root='./mnist_data/', train=False, transform=transforms.ToTensor())

batch_size = 64

train_loader = torch.utils.data.DataLoader(dataset=train_dataset, batch_size=batch_size, shuffle=True) 
test_loader = torch.utils.data.DataLoader(dataset=test_dataset, batch_size=batch_size, shuffle=False)
```


```python
class BasicCNN(nn.Module):

    def __init__(self):
        super(BasicCNN, self).__init__()
        self.conv1 = nn.Conv2d(1, 10, kernel_size=5)
        self.conv2 = nn.Conv2d(10, 20, kernel_size=3)
        self.mp = nn.MaxPool2d(2)
        self.fc = nn.Linear(500, 10)

    def forward(self, x):
        in_size = x.size(0)
        x = F.relu(self.mp(self.conv1(x)))
        x = F.relu(self.mp(self.conv2(x)))
        x = x.view(in_size, -1)  # flatten the tensor
        x = self.fc(x)
        return F.log_softmax(x)

model = BasicCNN()

optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.5)
```


```python
summary(model, (1, 28, 28))
```

    ----------------------------------------------------------------
            Layer (type)               Output Shape         Param #
    ================================================================
                Conv2d-1           [-1, 10, 24, 24]             260
             MaxPool2d-2           [-1, 10, 12, 12]               0
                Conv2d-3           [-1, 20, 10, 10]           1,820
             MaxPool2d-4             [-1, 20, 5, 5]               0
                Linear-5                   [-1, 10]           5,010
    ================================================================
    Total params: 7,090
    Trainable params: 7,090
    Non-trainable params: 0
    ----------------------------------------------------------------
    Input size (MB): 0.00
    Forward/backward pass size (MB): 0.07
    Params size (MB): 0.03
    Estimated Total Size (MB): 0.10
    ----------------------------------------------------------------


    /home/jaehwi/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:16: UserWarning: Implicit dimension choice for log_softmax has been deprecated. Change the call to include dim=X as an argument.
      app.launch_new_instance()



```python
def train(epoch):
    model.train()
    for batch_idx, (data, target) in enumerate(train_loader):
        data, target = Variable(data), Variable(target)
        optimizer.zero_grad()
        output = model(data)
        loss = F.nll_loss(output, target)
        loss.backward()
        optimizer.step()
        if batch_idx % 10 == 0:
            print('Train Epoch: {} [{}/{} ({:.0f}%)]\tLoss: {:.6f}'.format(
                epoch, batch_idx * len(data), len(train_loader.dataset),
                100. * batch_idx / len(train_loader), loss.data[0]))

def test():
    model.eval()
    test_loss = 0
    correct = 0
    for data, target in test_loader:
        data, target = Variable(data, volatile=True), Variable(target)
        output = model(data)
        # sum up batch loss
        test_loss += F.nll_loss(output, target, size_average=False).data[0]
        # get the index of the max log-probability
        pred = output.data.max(1, keepdim=True)[1]
        correct += pred.eq(target.data.view_as(pred)).cpu().sum()

    test_loss /= len(test_loader.dataset)
    print('\nTest set: Average loss: {:.4f}, Accuracy: {}/{} ({:.0f}%)\n'.format(
        test_loss, correct, len(test_loader.dataset),
        100. * correct / len(test_loader.dataset)))

for epoch in range(1, 2):
    train(epoch)
    test()
```    
    Test set: Average loss: 0.0517, Accuracy: 9826/10000 (98%)
    


### 2.2 Self-Attention
  - Self-Attention is defined as attention applied to a single context
  - Query, Keys, Values: http://jalammar.github.io/illustrated-transformer/
  - This paper removes entire convolutions and uses only local self-attention

![Figure3-4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/Stand_alone_self_attention/Figure3-4.png)

# <center> $y_{ij} = \sum$softmax$_{ab}(q_{ij}^\intercal k_{ab} + q_{ij}^\intercal r_{a-i,b-j}) v_{ab}$ </center>

  - __Relative attention__ is used
  - The rlative distance is factorized across dimensions
  - so, each element $ab$ recieves two distances
  - The row and column offsets are associated with dimension $d_{out}/2$ -> concat


```python
class AttentionConv(nn.Module):
    def __init__(self, in_channels, out_channels, kernel_size, stride=1, padding=0, bias=False):
        super(AttentionConv, self).__init__()
        self.out_channels = out_channels
        self.kernel_size = kernel_size
        self.stride = stride
        self.padding = padding

        self.rel_h = nn.Parameter(torch.randn(out_channels // 2, 1, 1, kernel_size, 1), requires_grad=True)
        self.rel_w = nn.Parameter(torch.randn(out_channels // 2, 1, 1, 1, kernel_size), requires_grad=True)

        self.key_conv = nn.Conv2d(in_channels, out_channels, kernel_size=1, bias=bias)
        self.query_conv = nn.Conv2d(in_channels, out_channels, kernel_size=1, bias=bias)
        self.value_conv = nn.Conv2d(in_channels, out_channels, kernel_size=1, bias=bias)
        
    def forward(self, x):
        batch, channels, height, width = x.size

        padded_x = F.pad(x, [self.padding, self.padding, self.padding, self.padding])
        q_out = self.query_conv(x)
        k_out = self.key_conv(padded_x)
        v_out = self.value_conv(padded_x)
        
        k_out = k_out.unfold(2, self.kernel_size, self.stride).unfold(3, self.kernel_size, self.stride)
        v_out = v_out.unfold(2, self.kernel_size, self.stride).unfold(3, self.kernel_size, self.stride)

        v_out_h, v_out_w = v_out.split(self.out_channels // 2, dim=1)
        v_out = torch.cat((v_out_h + self.rel_h, v_out_w + self.rel_w), dim=1)

        k_out = k_out.contiguous().view(batch, self.groups, self.out_channels // self.groups, height, width, -1)
        v_out = v_out.contiguous().view(batch, self.groups, self.out_channels // self.groups, height, width, -1)

        q_out = q_out.view(batch, self.groups, self.out_channels // self.groups, height, width, 1)

        out = q_out * k_out
        out = F.softmax(out, dim=-1)
        out = torch.einsum('bnchwk,bnchwk -> bnchw', out, v_out).view(batch, -1, height, width)

        return out
```

## 3. Fully Attentional Vision Models
### 3.1 Replacing Spatial Convolutions

 - replace every instance of a spatial convolution with an attention layer
 - A 2x2 average pooling with stride 2 operation follows
 - for ResNet: swaps the 3x3 spatial convolution with a self-attention layer
 
![ResNet](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/Stand_alone_self_attention/ResNet.png)

### 3.2 Replacing the Convolutional Stem
 - Stemp: the initial layers of a CNN
 - Due to input images being larget, the stem typically differs from the core block

