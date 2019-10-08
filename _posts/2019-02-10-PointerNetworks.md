---
layout: article
title: Pointer Networks review
tags: [paper, review, ml, pointer, network]
---

# Pointer Networks

2019.02.10  <br>
Jaehwi Park <br>

## 1. Introduction

> - 고전 RNN은 input과 output이 fixed frame rate에 한정됐음
> - Seq2Seq 패러다임이 그러한 한계를 극복함
>   - input을 embedding하고, embedded featrue가 output을 생성함
> - Attention 매커니즘이 등장하며 Seq2Seq 성능이 대폭 개선됨
> - 그런데 output length를 자유롭게 조절할 수 없는 문제가 있음

### Main Contribution
> 1. Pointer Network라는 새로운 구조를 제안
>   - 간단한데 효율적
>   - softmax 확률분포를 pointer로 사용해서 variable length 문제를 해결
> 2. Pointer Network가 세 개의 non-trivial algorithmic problems를 해결함을 보임
>   - 학습시보다 points가 더 많을 때도 작동(Generalization)
> 3. Pointer Network는 작은 규모 (n<50)의 TSP류 문제의 풀이방법임
>   - 순수하게 Data Driven 접근방법
>   - computationally intractable(다루기 어려운) 문제를 해결해냄

## 2. Models

### 2.1 Sequence-to-Sequence Model (Seq2Seq)

![exp1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/PtrNet/exp1.png)

> - $(\mathcal{C}^{\mathcal{P}}, \mathcal{P})$ 는 학습 데이터 pair
> - Seq2Seq 모델은 조건부 확률인 $p(\mathcal{C}^{\mathcal{P}} | \mathcal{P};\theta)$을 계산
> - $\mathcal{P}=\{ P_1, ..., P_n \}$는 n개의 sequence vector
> - $\mathcal{C}^{\mathcal{P}}=\{ C_1,...,C_{m(\mathcal{P})} \}$는 값이 1부터 n사이인 $m(\mathcal{P})$개의 순서
> - 본 논문에서는 LSTM을 사용
>   - Encoder + Decoder을 각각 생성


```python
"""
pytorch로 구현된 코드의 일부를 발췌
(https://github.com/shiretzet/PointerNet/blob/master/PointerNet.py)
"""
class PointerNet(nn.Module):
    def __init__(self, embedding_dim,
                 hidden_dim,
                 lstm_layers,
                 dropout,
                 bidir=False):

        super(PointerNet, self).__init__()
        self.encoder = Encoder(embedding_dim,
                               hidden_dim,
                               lstm_layers,
                               dropout,
                               bidir)
        self.decoder = Decoder(embedding_dim, hidden_dim)
        
class Encoder(nn.Module):
    def __init__(self, embedding_dim,
                 hidden_dim,
                 n_layers,
                 dropout,
                 bidir):
        super(Encoder, self).__init__()
        self.lstm = nn.LSTM(embedding_dim,
                            self.hidden_dim,
                            n_layers,
                            dropout=dropout,
                            bidirectional=bidir)

class Decoder(nn.Module):
    def __init__(self, embedding_dim,
                 hidden_dim):
        super(Decoder, self).__init__()
        self.input_to_hidden = nn.Linear(embedding_dim, 4 * hidden_dim)
        self.hidden_to_hidden = nn.Linear(hidden_dim, 4 * hidden_dim)
        self.hidden_out = nn.Linear(hidden_dim * 2, hidden_dim)
```

> __Inference__
> - 일단, Seq2Seq 모형에서 $\mathcal{C}^{\mathcal{P}}$를 생성
> - 본 논문이 해결하려는 문제들은 복잡한 조합(Combinatorial) 경우의 수 문제
> - 그래서 Beam Search로 Optimal Solution 생성: [참조](https://ratsgo.github.io/deep%20learning/2017/06/26/beamsearch/)

> __Seq2Seq의 문제점 (중요!)__
> - Seq2Seq 모델은 output의 길이가 Input을 재배열하는 것이므로 n으로 고정됨
> - 즉, 학습한 모형이 n개의 sequence로 학습되면, m개의 data를 input으로 받는 문제는 해결할 수 없음.. 새로운 모형이 필요

### 2.2 Content Based Input Attention

![exp3](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/PtrNet/exp3.png)

> - $(e_1, ..., e_n)$ : Encoder hidden states
> - $(d_1, ..., d_{m(\mathcal{P})})$ : Decoder hidden states
![hidden_states](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/PtrNet/lstm.png)

__Attention__
> 1. Encoder와 Decoder의 output을 적절히 섞고
> 2. 그것을 n개의 확률로 변경해서
> 3. Encoder의 output에 가중치를 부여한 $d'_i$를 만든 후
> 4. FinalOutput = concat($d'_i, d_i$)
> 5. FinalOutput은 Prediction과 다음 time step의 input으로 활용

### 2.3 Ptr-Net

> - Attention을 적용해도 variable size 문제는 해결되지 않음
> - 그래서 새롭게 Attention을 '조금' 바꿨더니 해결!

![exp4](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/PtrNet/exp4.png)

> - $u^i$를 바로 "Pointer"로 활용
>   - $u^i$는 사실 길이 n의 vector
>   - 그러므로 위 수식은 i번째 정답인 $C_i$가 될 확률 분포를 $softmax(u^i)$로 활용하겠다는 뜻


```python
class Decoder(nn.Module):
    def __init__(self, embedding_dim,
                 hidden_dim):
        self.att = Attention(hidden_dim, hidden_dim)
        self.hidden_out = nn.Linear(hidden_dim * 2, hidden_dim)
    def forward(self, embedded_inputs,
            decoder_input,
            hidden,
            context):
        """
        :param Tensor embedded_inputs: Embedded inputs of Pointer-Net
        :param Tensor decoder_input: First decoder's input
        :param Tensor hidden: First decoder's hidden states
        :param Tensor context: Encoder's outputs
        :return: (Output probabilities, Pointers indices), last hidden state
        """
        def step(x, hidden):
            # Attention section
            hidden_t, output = self.att(h_t, context, torch.eq(mask, 0))
            hidden_t = F.tanh(self.hidden_out(torch.cat((hidden_t, h_t), 1)))

class Attention(nn.Module):

    def forward(self, input,
            context,
            mask):
        """
        :param Tensor input: Hidden state h
        :param Tensor context: Attention context
        :param ByteTensor mask: Selection mask
        :return: tuple of - (Attentioned hidden state, Alphas)
        """
        # (batch, seq_len)
        att = torch.bmm(V, self.tanh(inp + ctx)).squeeze(1)
        if len(att[mask]) > 0:
            att[mask] = self.inf[mask]
        alpha = self.softmax(att)
        hidden_state = torch.bmm(ctx, alpha.unsqueeze(2)).squeeze(2)

        return hidden_state, alpha
```

![Figure1](https://raw.githubusercontent.com/jaehwi0823/jaehwi0823.github.io/master/_image/PtrNet/Figure1.png)
