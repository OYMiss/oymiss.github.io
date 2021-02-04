---
mathjax: true
title: k-means 聚类算法
date: 2020-01-16 14:06:36
tags: [cs229, 机器学习]
categories: 机器学习
---

在聚类问题中，给定一个训练集 $\{ x^{(1)}, \dots, x^{(m)}\}$，把这些数据分成一些有联系的“群”，标签 $y^{(i)}$ 没有给出，这是一个无监督学习问题。

k-means 聚类算法的步骤如下：

1. 随机初始化聚类中心（cluster centroids）$\mu_1, \mu_2, \dots, \mu_k \in \mathbb{R}^n$。
2. 重复直到收敛
   1. 对于每个 $i \in \{1, \dots, m\}$ 令
      $$
      c^{(i)}=\arg\min_j||x^{(i)}-\mu_j||^2
      $$
   2. 对于每个 $j \in \{1, \dots, k\}$ 令
      $$
      \mu_j=\frac{\sum_{i=1}^m1\{c^{(i)}=j\}x^{(i)}}{\sum_{i=1}^m1\{c^{(i)}=j\}}
      $$

<!-- more -->

下图显示了 k-means 聚类算法的过程：

<img src="/asset/k-means-clustering/process.png" width = "80%" height = "80%" alt="process"/>

这个算法一定能保证收敛吗？

定义一个失真函数（distortion function）：

$$
J(c,\mu)=\sum_{i=1}^m ||x^{(i)}-\mu_{c^{(i)}}||^2
$$

可以看出来，k-means 聚类算法实际上是对 $J$ 做坐标下降算法。也就先固定 $\mu$ 然后改变 $c$ 使得失真函数变小，然后再固定 $c$ 通过改变 $\mu$ 来对失真函数进行最小化。

失真函数不是一个非凸函数（non-convex function），这个算法可能会收敛到局部最优解，可以通过多次随机初始化 $\mu$ 来避免这个问题。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
