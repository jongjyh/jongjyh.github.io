---

title: JMeter 测试接口指南
tags: [开发,测试]
date: 2022-04-10 14:42:00
categories: 测试

---

# JMeter 测试接口指南

[JMeter下载地址](https://jmeter.apache.org/download_jmeter.cgi)

## 编写测试脚本

1. 添加线程组，配置对应的并发要求

   ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zteuvlo1j20ct0ahwfb.jpg)

   ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztf1nf1rj20uo0d5q3s.jpg)

##### 		线程组主要参数详解：

- 线程数：虚拟用户数。一个虚拟用户占用一个进程或线程。模拟多少用户访问也就填写多少个线程数量。

- Ramp-Up时间(秒)：设置的虚拟用户数需要多长时间全部启动。如果线程数为`100`，准备时长为`5`，那么需要`5`秒钟启动`100`个线程，也就是每秒钟启动`20`个线程。 相当于每秒模拟`20`个用户进行访问，设置为零我理解为并发访问。
- 循环次数：如果线程数为`100`，循环次数为`100`。那么总请求数为`100*100=10000` 。如果勾选了“永远”，那么所有线程会一直发送请求，直到选择停止运行脚本。

2. 添加HTTP请求

   - 右键点击 “你的线程组” → “添加” → “取样器” → “HTTP请求”

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztji68tnj20eq0h1abo.jpg)

   - 配置你需要测试的IP，端口，headers，data等

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztkl12slj20xc0cldhu.jpg)

3. 添加监视器

   - 监视器是用来返回每次请求返回的结果并进行统计的工具

   - 一般聚合统计、查看结果数较为实用，其他的也可以尝试

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztmvd67fj20eo0jd0ug.jpg)

   ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zto2ytqxj20e40j30um.jpg)

4. 为请求添加变量

   - 有些情况下我们的请求中需要一些变量（比如每次请求时都需要更改data中的某一个值），这时就可以增加变量

   - 右键点击 “你的线程组” → “添加” → “配置元件” → “用户定义的变量”：![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztqxzf5vj20g30ldtaz.jpg)

   - 新增一个用户名参数

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztrfv7uij20vn0inwfg.jpg)

   - 在Http请求中使用该参数，格式为：${key} 

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztrxitooj20v60madho.jpg)

   - 改变参数后，每次请求时username将是一个变量，每次请求不同的值（如果你不是设置为常量的话）

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztsk42j7j20vc0lf0vi.jpg)

5. 开始测试

   - 点击绿色按钮则JMeter会按照测试计划由上至下（如果你有多个任务需要执行）执行，右边的按钮是清除结果

   ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zttuw3pxj20xc0id0u9.jpg)

6. 查看测试结果

   - 一般聚合报告中有着我们需要的测试参数，如P95,P99等

     ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ztv7kfs4j20vj09t75h.jpg)