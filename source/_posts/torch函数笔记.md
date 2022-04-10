---
title: torch常用函数（不断更新）
tags: [Pytorch]
date: 2022-04-07 17:05:00
categories: Pytorch学习
---

# torch常用函数

- torch.squeeze <->torch.unsqueeze para:dim=  制定维度降维/升维，[3,1,2]<->[3,2]

- Torch.scatter 使用索引数组index对原tensor定位修改

- torch.gather 根据索引数组index取出原tensor中的值

- torch.split <-> torch.stack 把tensor分开[3,1,2]->[tensor[1,2],tensor[1,2],tensor[1,2]] stack是逆操作

  - stack 用法1:list([tensor1,tensor1]) --> tensor([tensor1,tensor2]),tensor1.size()==tensor2.size()

- Torch.cat 在指定dim上合并两个tensor [3,1],[3,1] ->[3,2] or [6,1]

- torch.reshape<-> torch.views 重新调整tensor的维度，但是view要求tensor的必须是连续的

- permute 

- torch.einsum 爱因斯坦计数法

- TORCH.NN.UTILS.RNN.PACK_PADDED_SEQUENCE

- expand 可以指定定维度进行重复  (1,3)->(10,3)  (3,1)->(3,10) (3,2)->(2,3,2)

  - repeat创建新tensor，expand仍是原tensor

- F.pad(tensor,p1d,...)  tensor(3,1,2) p1d(1,1,2,0,0,2)  ->tensor(0+3+2,2+1+0,1+2+1) ->(5,3,4)

- Tensor.detach() 返回一个从一个图中分离出的新tensor，对这个新tensor任何求导都不会传播到原tensor

  - Tensor.copy()返回的新tensor求导会传播到原来的图

- Tril 返回一个2D（或者带Batch的2D） tensor的下三角矩阵，*diagonal* 是斜方向上的偏移量，如果为n则保留下三角线+上三角线n的区域

- Tensor.new() 返回一个与原tensor相同type、device的**无内容无大小**的新tensor

- torch.arg*

  - tensor.nonzero(as_tuple=False)  返回tensor中非零元素的**索引**数组

    - as_tuple的用法，**转置**

      ```python
      import torch
      a = torch.arange(4).view((2,2))
      print(a)
      print(a.nonzero(as_tuple=True))
      print(a.nonzero())
      
      tensor([[0, 1],
              [2, 3]])
      (tensor([0, 1, 1]), tensor([1, 0, 1]))
      tensor([[0, 1],
              [1, 0],
              [1, 1]])
      ```

    - nonzero()配合bool mask可以返回某一条件的索引

      ```python
      import torch
      a = torch.arange(12).view((3,4))
      print(a)
      (a>6).nonzero()
      ```

      ```python
      tensor([[ 0,  1,  2,  3],
              [ 4,  5,  6,  7],
              [ 8,  9, 10, 11]])
      tensor([[1, 3],
              [2, 0],
              [2, 1],
              [2, 2],
              [2, 3]]) 
      ```

    - 配合as_tuple也可以直接返回某一条件的值

      ```python
      import torch
      a = torch.arange(16).view((4,4))
      print(a)
      index = (a>8).nonzero(as_tuple=True)
      print(index)
      a[index]
      
      tensor([[ 0,  1,  2,  3],
              [ 4,  5,  6,  7],
              [ 8,  9, 10, 11],
              [12, 13, 14, 15]])
      (tensor([2, 2, 2, 3, 3, 3, 3]), tensor([1, 2, 3, 0, 1, 2, 3]))
      tensor([ 9, 10, 11, 12, 13, 14, 15])
      ```

      

  - argsort 返回tensor排序后的索引顺序，a.argsort()[i] : 第i小的元素的索引

- out = tensor.where(condition,x,y) 如果 满足condition out 设为x，不满足为y **三元运算符**

  