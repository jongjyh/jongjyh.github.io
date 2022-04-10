---
title: Python 中的装饰器
tags: [Python]
date: 2022-04-09 17:05:00
categories: Python学习
---

# Python 中的装饰器

最近正在看transformers中的一些源码，transfomers中使用了Python语言中许多特性，比如装饰器等，结合[Python3-Cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/chapters/p09_meta_programming.html)一边看源码一边学习一些特性

## 装饰器

为了增加代码的复用，经常使用装饰器来修改或者增加**函数或者类**的行为，下面从最基础的使用开始介绍

### 无参装饰器

一个最简单的装饰器:

```python
def log(func):
  def someprocess():
    print("before call")
    func()
  return someprocess

@log
def test():
  print("it's a test func")
  
before call
it's a test func

```

使用@等价于

``` python
test = log(test)
```

### 带参装饰器

有的情况下我们需要**装饰器带参**，但是**又想保留不带参数**的情况：

```python
from functools import partial

def log(func=None,*,post_message=None):
  def someprocess():
    print("before call")
    func()
    if post_message:
        print(post_message)
  # 如果以log(post_messag=..)调用，此处返回一个固定了post_message的参数的log装饰器   
  if func is None:
      return partial(log,post_message=post_message)
  # 如果以log方式调用则直接返回被装饰的函数
  return someprocess

@log
def test():
  print("it's a test func")

test() 

@log(post_message="after call")
def test2():
  print("it's a test func")

test2()
```

- *代表了后边的参数必须用关键字参数传参，同理还有**\\**这个符号表示之前的参数必须使用位置参数传参数
- partial函数的用法，*partial(func,arg1,arg2...)固定func函数的参数，返回一个缩减了参数的**可调用函数**

