---
mathjax: true
title: 广义线性模型
date: 2019-12-31 8:35:22
tags: [cs229, 机器学习]
categories: 机器学习
---

在回归模型中有 $y|x;\theta \sim N (\mu,\sigma^2)$ ，在分类模型中有 $y|x;\theta \sim \text{Bernoulli}(\phi)$，这两种模型都属于广义线性模型。

### 指数家族

$$
p(y;\eta) =b(y)\exp(\eta^TT(y)-a(\eta)) = b(y)e^{\eta^TT(y)} e^{-a(\eta)}
$$


$\eta$ 被叫做自然参数（natural parameter）也叫做典范参数 （canonical parameter）。

$T(y)$ 叫做充分统计量（sufficient statistic），通常 $T(y) = y$。

<!--more-->


$a(\eta)$ 是一个对数分割函数（log partition function）。

> $e^{−a(\eta)}$ 这个量本质上扮演了归一化常数（normalization constant）的角色，也就是确保 $p(y; \eta)$ 的总和或者积分等于 $1$

对于伯努利分布：

$$
\begin{aligned} p(y;\phi)
&= \phi ^y(1-\phi)^{1-y}\\
&=\exp\left(y \log \phi + (1-y)\log(1-\phi)\right)\\
&=\exp\left( \left(log (\frac {\phi}{1-\phi})\right)y+\log (1-\phi) \right) \\
\end{aligned}
$$
令 $\eta = \log (\frac \phi {1 − \phi})$，此时 $\phi = 1/ (1 + e^{−\eta} )$ ，得到
$$
\begin{aligned}
p(y;\phi) &=\exp(\eta y + \log(1 - \phi)) \\
&=\exp(\eta y + \log(1 + e^{\eta}))
\end{aligned}
$$

对于正态分布（为了简化，令 $\sigma = 1$）：

$$
\begin{aligned} p(y;\mu)
&= \frac 1{\sqrt{2\pi}}\exp (- \frac 12 (y-\mu)^2) \\
& = \frac 1{\sqrt{2\pi}}\exp (- \frac 12 y^2) \cdot\exp (\mu y -\frac 12 \mu^2) \\
\end{aligned}
$$
这里得到 $\eta = \mu$ 和 $a(\eta) = \mu^2/2 = \eta^2/2$ 。

### 构建 GLM

通常，令 $T(y) = y$，先对 $y$ 关于 $x$ 的条件分布进行三个假设：

1. $y | x; \theta \sim \text{ExponentialFamily}(\eta)$
2. $g(\eta) = E[y | \eta]$
3. $\eta = \theta^T x$，于是可以写成 $h(x) = E[y|x]$。

目的是求 $h(x)$，指数家族函数里面可以得到 $\eta = g^{-1}(\mu)$，其中 $\mu$ 也就是 $E[y|\eta]$。

$g^{-1}(\mu)$ 表示典型链接函数（canonical link functions）。
$g(\eta)$ 表示典型响应函数（canonical response function）。

#### 普通最小二乘法（Ordinary Least Squares ）

目标变量（target variable ）$y$ 在广义线性模型中也叫做响应变量（response variable ）。在给定 $x$ 的情况下，$y$ 服从正态分布 $\mathcal{N}(\mu, \sigma^2)$。

对于正态分布，有 $\eta = \mu$，可以得到：
$$
\begin{aligned}
h_{\theta}(x)
&= E[y | x; \theta] \\
&= \mu \\
&= \eta \\
&= \theta^Tx
\end{aligned}
$$

#### 逻辑回归（Logistic Regression ）

已知 $y|x; \theta \sim \text{Bernoulli}(\phi)$，有 $E[y|x; \theta] = \phi$，其中 $\phi = 1/(1 + e^{-\eta})$。

根据上面的三个假设可以得到：
$$
\begin{aligned}
h_{\theta}(x)
&= E[y | x; \theta] \\
&= \phi \\
&= 1 / (1 + e^{-\eta}) \\
&= 1 / (1 + e^{-\theta^Tx})
\end{aligned}
$$

#### Softmax 回归（Softmax Regression ）

当二元分类问题变成了多元问题，这时候 $y \in \{1, 2, \dots, k\}$ 而不是 $y\in\{1, 2\}$，当给定 $x$ 的情况下，$y$ 服从二项分布（multinomial distribution）。

二项分布具有 $k$ 个（实际上有一个是多余的，因为 $\sum_i \phi = 1$）参数 $\phi_1,\phi_2, \dots,\phi _k$ 来表示每个类别出现的概率。为了方便，依然令 $\phi_k = 1 - \sum_{i=1}^{k-1}\phi_i$。不同于上面两种分布中令 $T(y) = y$，这里令 $T(y) \in \mathbb{R}^{k-1}$ 并有，
$$
T(1)= \begin{bmatrix}
1\\ 0\\ 0\\ \vdots \\ 0\\
\end{bmatrix},
T(2)= \begin{bmatrix}
0\\ 1\\ 0\\ \vdots \\ 0\\
\end{bmatrix},
T(3)= \begin{bmatrix}
0\\ 0\\ 1\\ \vdots \\ 0\\
\end{bmatrix},
T(k-1)= \begin{bmatrix}
0\\ 0\\ 0\\ \vdots \\ 1\\
\end{bmatrix},
T(k)= \begin{bmatrix}
0\\ 0\\ 0\\ \vdots \\ 0\\
\end{bmatrix}
$$

引入一个符号 $1\{\cdot\}$，表示指示函数（indicator function），有 $1\{True\} = 1$ 和 $1\{False\} = 0$。于是就可以表示 $(T(y))_i = 1\{y=y\}$，并有 $E[(T(y))_i] = P(y = i) = \phi_i$。

然后下面证明二项分布也是指数家庭的一个成员：

$$
\begin{aligned}
p(y;\phi) &=\phi_1^{1\{y=1\}}\phi_2^{1\{y=2\}}\dots \phi_k^{1\{y=k\}} \\
          &=\phi_1^{1\{y=1\}}\phi_2^{1\{y=2\}}\dots \phi_k^{1-\sum_{i=1}^{k-1}1\{y=i\}} \\
          &=\phi_1^{(T(y))_1}\phi_2^{(T(y))_2}\dots \phi_k^{1-\sum_{i=1}^{k-1}(T(y))_i } \\
          &=\exp((T(y))_1 \log(\phi_1)+(T(y))_2 \log(\phi_2)+\\ &\qquad \dots+(1-\sum_{i=1}^{k-1}(T(y))_i)\log(\phi_k)) \\
          &= \exp((T(y))_1 \log(\frac{\phi_1}{\phi_k})+(T(y))_2 \log(\frac{\phi_2}{\phi_k})+\\ & \qquad\dots+(T(y))_{k-1}\log(\frac{\phi_{k-1}}{\phi_k})+\log(\phi_k)) \\
          &=b(y)\exp(\eta^T T(y)-a(\eta))
\end{aligned}
$$

其中：

$$
\begin{aligned}
&\eta =
    \begin{bmatrix}
      \log \frac {\phi _1}{\phi _k}\\
      \log \frac {\phi _2}{\phi _k}\\
	  \vdots \\
	  \log \frac {\phi _{k-1}}{\phi _k}\\
    \end{bmatrix} \\
&a(\eta) = -\log (\phi _k)\\
&b(y) = 1\\
\end{aligned}
$$

对于每个 $i \in [1, k]$ 链接函数为：

$$
\eta_i = \log \frac{\phi_i}{\phi_k}
$$

为了方便我们定义 $\eta_k = \log(\phi_k/\phi_k) = 0$，然后将函数取反：

$$
\begin{aligned}
e^{\eta_i} &= \frac{\phi_i}{\phi_k} \\
\phi_k e^{\phi_i} &= \phi_i \\
\phi_k \sum_{i=1}^{k} e^{\phi_i} &= \sum_{i=1}^{k}\phi_i = 1
\end{aligned}
$$

于是得到响应函数：

$$
\phi_i = \frac{e^{\eta_i}}{\sum_{j=1}^{k}e^{\eta_j}}
$$

这个函数从 $\eta$ 映射到了 $\phi$，称它为 softmax 函数。

为了完成模型，使用上面的假设 3，有 $\eta_i = \theta_i^T x$，这里 $\theta_i \in \mathbb{R}^{n+1}$ 是模型的参数。为了方便，令 $\theta_k = 0$，并有 $\eta_k = \theta_k^Tx = 0$。

可以得到：
$$
\begin{aligned}
p(y = i |x;\theta) &= \phi_i \\
&= \frac{e^{\eta_i}}{\sum_{j=1}^{k}e^{\eta_j}} \\
&= \frac{e^{\theta_i^Tx}}{\sum_{j=1}^{k}e^{\theta_j^Tx}}
\end{aligned}
$$
这种用于解决 $y \in \{1, \dots, k\}$ 的分类问题的方法叫做 softmax 回归，它是逻辑回归的一般化。

最后，系统模型的为：

$$
\begin{aligned}
h_\theta (x) &= E[T(y)|x;\theta]\\
&= \left[
    \begin{array}{c}
      \phi_1\\
      \phi_2\\
	  \vdots \\
	  \phi_{k-1}\\
    \end{array}
\right]\\
&= \left[
    \begin{array}{ccc}
      \exp(\eta_1) \\
      \exp(\eta_2) \\
	  \vdots \\
	    \exp(\eta_{k-1}) \\
    \end{array}
\right]\frac{1}{\sum^k_{j=1}\exp(\eta_j)}\\
&= \left[
    \begin{array}{ccc}
      \exp(\theta_1^Tx) \\
      \exp(\theta_2^Tx) \\
	  \vdots \\
	    \exp(\theta_{k-1}^Tx) \\
    \end{array}
\right]\frac{1}{\sum^k_{j=1}\exp(\theta_j^Tx)}\\
\end{aligned}
$$

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)

[3] [Generalized linear model](https://en.wikipedia.org/wiki/Generalized_linear_model)
