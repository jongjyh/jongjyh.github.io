---
title: 论文研读 ｜ UniLMv2
tags: [论文]
date: 2022-04-10 17:05:00
categories: 论文&实现
---

# UniLMv2

微软提出的另一个尝试统一autoencoder和autoregressive的一个预训练大模型

[Code](https://github.com/microsoft/unilm) & [Paper](https://arxiv.org/abs/2002.12804)

# Methodology

论文提出了以下三个方法尝试在autoencoder的框架下把autoregressive融合进来

1. Pseudo Mask
2. blockwise factorization 
3. PAR

UniLM提出增加一种新的Mask（i.e Pseudo Mask）使LM能够以autoeocode的方式训练，这样在预训练时就能同时进行autoencoder和autoregressive式的训练，由于这仍是cloze式的预训练方式，所以能直接使用诸如Bert这类纯encoder模型。

## Pseudo Mask  

**存在的问题**

- 传统的Mask在输入中随机遮盖几个词，用未遮盖的词去预测Mask中的词语，但这忽略了Mask之间的关系（被遮盖住了不知道是什么）。
- 在autoregresive的预训练目标中，我们往往能够利用上文中的信息，但无法使用下文的信息（单方向预测），所以autoregresive和autoencoder其实是一种互补的预训练方式
- AR前边预测出的mask能够被attempt但AE不行，AE中拥有的下文信息却但AR attmpt不到

为了解决以上问题，Pseudo Mask被提出

![截屏2022-04-02 下午4.40.47](https://tva1.sinaimg.cn/large/e6c9d24ely1h0vgw6yp2kj21eu0sowlg.jpg)

1. Pseudo Mask可以看做是和传统的Mask同样的东西，只不过这是在AR下的Mask
2. Pseudo Mask与XL-net中的处理相类似，做了因子排序的处理，每次sample一种排序方式比如（4，5 --> 2）用前边的因子去预测后边的因子，这样就能补齐AR中无法attmpt下文的缺陷了
3. 值得注意的是P Mask拥有相同的Position Embedding，因此即使被打乱了顺序模型仍然知道被mask的词之间的顺序
4. 实际Input变成了两段，前边是传统的Mask式，后边是Pseudo Mask式，但后边的输入序列只有Masked Token+Pseudo Mask

## PAR & Blockwise factorization

与传统AR不同，AR往往是一个token一个token的预测，PAR采用了一种**span**的方式，具体来说：

- 随机sample一种因子序列
- 前边的因子必须一次性预测下个因子中内部所有的token
- PAR采用的Mask（40%span，60%single token）

![截屏2022-04-02 下午4.52.49](https://tva1.sinaimg.cn/large/e6c9d24ely1h0vh7tx6hvj21ec0d6tcg.jpg)

## Finetuning

- 对于NLU任务，和Bert类似把[CLS]当作Input embedding，把[CLS]输出的向量直接输入到task-specific layer
- 对于NLG任务，输入是src+target （i.e [SOS] SRC [EOS] TGT [EOS]），但是target是被mask的，所以finetune的目标是恢复target，复现的代码中是为用pseudo mask替换了所有的target token，然后用bert预测所有的pseudo mask原本的token

## Experiment

![截屏2022-04-02 下午5.02.45](https://tva1.sinaimg.cn/large/e6c9d24ely1h0vhi6b54aj21ek0nmagj.jpg)

![image-20220402171023121](https://tva1.sinaimg.cn/large/e6c9d24ely1h0vhq3gw4ij21g40ja455.jpg)

![image-20220402171532109](https://tva1.sinaimg.cn/large/e6c9d24ely1h0vhvfppv5j21da0k8wjs.jpg)

- UniLMv2效果无论是在NLU还是NLG上效果都是很好的，在NLG上甚至比肩了一些large模型