# CELoss 和 BCELoss

CE 和BCE都是深度学习里常用来衡量prediction和label之间的差距的损失函数，但是这两者在用法和表达式上又有些差别

## 表达式

- BCE虽然形式上是CE中的特例（分类数n=2），但是实际上BCELoss也可以处理多分类的问题
- BCE的激活函数是sigmoid
- **当N是batch_size时，这只是一个用BCE解决二分类的问题，当N是n_labels时，则是用BCE解决多分类问题**
  - 用BCE解决多分类的问题是对每个labels回答YES or No，即执行n_labels次二分类就变成了多分类
- BCE的输入一般是 [ n_labels,1 ]，**需自己把一个batch中的BCELoss相加**

![image-20220406125736981](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zwwbct65j2124030wem.jpg)

- CE的激活函数是softmax

- CE可以直接应用在多分类或二分类问题上

-  CE的输入是 [ batch_size, n_labels ]

  ![image-20220406130832519](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zx7oil1ej213003y0t1.jpg)

## 区别

1. 为什么激活函数不同？
   - 因为**对负样本的处理不同**，CE中的Loss看似只提高了正例的概率，没有对负例的概率做降低，但是softmax是一个容易使大者越大，小者越小的激活函数，当正例概率提高，负例的概率自然就会降低。
   - BCE中由于没有softmax，所以需要在做二分类时关注负例，表现在(1-yi)*log(1-xi)上，这部分的损失会被记入，通过这种方式降低负例的概率
2. 为什么要用BCE处理多分类？
   - softmax计算成本问题