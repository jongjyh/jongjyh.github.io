---
title: Pytorch 中的 ModuleList & Sequential
tags: [Pytorch]
date: 2022-04-08 14:52:00
categories: Pytorch学习
---

# Pytorch 中的 ModuleList & Sequential

ModuleList & Sequential 都是 torch.nn中重要的容器类，是为了方便定义结构化的可复用的网络结构而产生，但是两者的功能又有略微不同。

## 不同点

1. 场景不同，ModuleList 可拓展性更强，sequential更方便。

2. ModuleList需要可以自定义运算顺序，Sequential必须按照定义的顺序依次计算

``` python 
class SeqNet(nn.Module):
    def __init__(self):
        super().__init__()
        # 注意这里Sequential的参数是多个对象，而ModList的是一个数组
        self.encoder = nn.Sequential(
            nn.Linear(10,128),
            nn.ReLU(),
            nn.Linear(128,50)
        )
    def forward(self,x):
      # Sequential直接输入X则会按照定义的顺序依次运算
        return self.encoder(x)
class ModNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.encoder = nn.ModuleList([
            nn.Linear(10,128),
            nn.ReLU(),
            nn.Linear(128,50)
        ])
    def forward(self,x):
      # 如果按照以下方式输入则会报Not Implement error 说明如何运算需要自己定义
      # return self.encoder(x)
      # 需要我们自己定义整个运算过程
        for block in self.encoder:
            x = block(x)
        return x

```

- 可以看到ModuleList拓展性更强，如何运算完全取决于我们如何定义。
- 在一些结构化很强的模型中使用Sequential更方便，但是对于一些灵活的模型ModuleList更好
- 两者初始化的参数也有略微不同，Sequential是多个module对象，ModList是一个数组