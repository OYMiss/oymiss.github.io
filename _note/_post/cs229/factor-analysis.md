---
mathjax: true
title: 因子分析
date: 2020-01-18 19:29:08
tags: [cs229, 机器学习]
categories: 机器学习
---

因子分析（Factor analysis）是一种用来描述可观测的有关联的变量（observed, correlated variables）与叫做因子（factors）的数量比较少的不可观测变量（unobserved variables）之间的关系统计方法。

> 比如，六个可观测变量可能只由两个不可观测变量决定。

在因子分析模型中，可观测变量为因子的线性组合加上误差项。因子分析的目的就是找到这些独立的隐变量。

### 概率模型

假设有 $n$ 个可观测随机变量（observable random variables）$x_1, \dots, x_n$，均值为 $\mu_1, \dots, \mu_n$。

同时有一些未知常量 $l_{ij}$ 和 $k$ 个不可观测随机变量（也被叫做 common factors）。这里有 $i \in 1, \dots, n$，$j \in 1,\dots, k$ 和 $k < n$。并有：

$$
x_i - \mu_i = l_{i1}F_1 + \cdots + l_{ik}F_k + \epsilon_i
$$

<!-- more -->

这里 $\epsilon_i$ 是零均值和有限方差（zero mean and finite variance）的不可观测随机误差项（unobserved stochastic error terms）。

写成矩阵的形式：

$$
x - \mu = LF + \epsilon
$$

如果有 $m$ 个观测值，可以得到这些维度 $x_{n \times m}$，$L_{n\times k}$ 和 $F_{k\times m}$。$x$ 和 $F$ 的每一个列向量表示一个特定的观测值，并且 $L$ 不会根据观测值发生变化。

下面再做一些关于 $F$ 的假设：

1. $F$ 和 $\epsilon$ 是独立的。
2. $\text{E}[F] = 0$。
3. $\text{Cov}(F) = I$，所有因子是不相关的。

满足上面的解的 $F$ 被定义为因子（factors）并且 $L$ 为负荷矩阵（loading matrix）。

假设 $\text{Cov}(x - \mu) = \Sigma$，于是有：

$$
\begin{aligned}
    \Sigma &= \text{Cov}(x - \mu) = \text{Cov}(LF + \epsilon) \\
    &= L\text{Cov}(F)L^T + \text{Cov}(\epsilon) \\
    &= LL^T + \Psi
\end{aligned}
$$

其中 $\Psi = \text{Cov}(\epsilon)$。

### 利用因子分析的混合高斯模型

#### 引入问题
如果有来自混合高斯的数据 $x^{(i)} \in \mathbb{R}^n$，可以使用 EM 算法去拟合这个混合模型。这里有个条件是训练集的大小 $m$ 远远大于数据的维度 $n$。

如果 $n \gg m$，在这种情况下，就算数据只由一个高斯产生也不容易去拟合。利用最大似然法可以得到：

$$
\begin{aligned}
&\mu = \frac 1m\sum_{i=1}^m x^{(i)} \\
&\Sigma = \frac 1m\sum_{i=1}^m (x^{(i)}-\mu)(x^{(i)}-\mu)^T
\end{aligned}
$$

但是，这里的 $\Sigma$ 是奇异矩阵。（$(x^{(i)}-\mu)(x^{(i)}-\mu)^T$ 的秩为 $1$，$m$ 个相加得到 $\Sigma$，$\Sigma$ 的秩也小于等于 $m$，于是小于 $n$）

通过对 $\Sigma$ 做一定的限制，在没有足够数据的情况下不去拟合整个协方差矩阵。例如，可以限制 $\Sigma$ 为一个对角矩阵（甚至可以限制为 $\sigma I$，所有元素相同），这样可以得到：

$$
\Sigma_{jj} = \frac 1m \sum_{i=1}^m (x_j^{(i)}-\mu_j)^2
$$

高斯分布的等高线是一些椭圆，这种情况下，这些椭圆的长轴和短轴是和坐标系平行的。

#### 建立模型

现在假设一个联合分布 $(x,z)$ 如下所示，其中 $z \in \mathbb{R}^k$ 是隐变量（latent random variabl）：

$$
\begin{aligned}
z &\sim \mathcal{N}(0,I) \\
x|z &\sim \mathcal{N}(\mu+\Lambda z,\Psi)
\end{aligned}
$$

这个模型的参数为 $\mu \in \mathbb{R}^n$、$\Lambda \in \mathbb{R}^{n \times k}$ 和对角矩阵 $\Psi \in \mathbb{R}^{n \times n}$，这里 $k$ 一般小于 $n$。

可以假设数据点 $x^{(i)}$ 是先从 $k$ 维多元正态分布 $z^{(i)}$ 采样，然后通过计算 $\mu + \Lambda z^{(i)}$ 将其映射到 $n$ 维空间，最后在加上 $\Psi$ 的噪声。

同样的可以这样定义：

$$
\begin{aligned}
z &\sim \mathcal{N}(0,I) \\
\epsilon &\sim \mathcal{N}(0,\Psi) \\
x &= \mu + \Lambda z + \epsilon
\end{aligned}
$$

这里 $z$ 和 $\epsilon$ 是独立的。

经过计算可以得到：

$$
\begin{bmatrix}
z\\
x
\end{bmatrix}
\sim
\mathcal{N}
\left(
\begin{bmatrix}
\vec{0}\\ \mu
\end{bmatrix},
\begin{bmatrix}
I&\Lambda^T\\ \Lambda&\Lambda\Lambda^T+\Psi
\end{bmatrix}
\right)
$$

可以得到 $x$ 的边缘分布为 $x \sim \mathcal{N}(\mu,\Lambda\Lambda^T +\Psi)$，如果给定训练集 $\{x^{(i)}; i = 1, \dots, m\}$，可以得到对数似然函数：

$$
l(\mu,\Lambda,\Psi)=\log\prod_{i=1}^m\frac{1}
{(2\pi)^{n/2}|\Lambda\Lambda^T+\Psi|^{1/2}}
\exp(-\frac 12(x^{(i)}-\mu)^T(\Lambda\Lambda^T+\Psi)^{-1}(x^{(i)}-\mu))
$$

然后利用 EM 算法对模型进行求解。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)

[3] [Factor analysis](https://en.wikipedia.org/wiki/Factor_analysis)
