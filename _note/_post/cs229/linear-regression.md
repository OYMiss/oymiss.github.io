---
mathjax: true
title: 线性回归
date: 2019-12-27 9:30:41
tags: [cs229, 机器学习]
categories: 机器学习
---

假设 $y$ 是关于 $x$ 的线性函数：

$$
h_{\theta}(x) = \theta_0 + \theta_1 x_1 + \theta_2 x_2
$$

如果令 $x_0 = 1$ 得到：

$$
h(x) = \theta^T x
$$

定义一个损失函数（cost function）：

$$
J(\theta) = \frac{1}{2} \sum^{m}_{i=1}(h(x^{(i)}) - y_{(i)})^2
$$

找到一个 $\theta$ 使得 $J(\theta)$ 值最小，这个就是线性回归问题。

<!--more-->

### 梯度下降（gradient descent）

找到函数下降最快的方向，进行参数的更新，这就是梯度下降。
$$
\theta_j = \theta_j - \alpha \frac{\partial}{\partial\theta_j}J(\theta)
$$
这里 $\alpha$ 称作学习速率。

对于一个训练样本：
$$
\begin{aligned}
\frac{\partial}{\partial\theta_j}J(\theta)
&= \frac{\partial}{\partial\theta_j}\frac{1}{2} \sum^{m}_{i=1}(h(x) - y)^2 \\
&= (h(x) - y) \cdot \frac{\partial}{\partial\theta_j}(h(x) - y) \\
&= (h(x) - y) x_j
\end{aligned}
$$

$$
\begin{aligned}
&Loop \ \{ \\
 &\quad   \theta_j = \theta_j - \alpha\sum_{i=1}^{m} (h(x^{(i)}) - y^{(i)}) x_j^{(i)} \\
&\}
\end{aligned}
$$

这个方法叫做批处理梯度下降（batch gradient descent）。

$$
\begin{aligned}
&\text{Loop} \ \{ \\
 & \quad \text{for i = 1 to m }\ \{ \\
 &\quad \quad   \theta_j = \theta_j - \alpha(h(x^{(i)}) - y^{(i)}) x_j^{(i)} \\
 & \quad \} \\
&\}
\end{aligned}
$$

这种每次只处理一个数据的方法，叫做随机梯度下降（stochastic gradient descent ）。每次更新参数的方向不一定是最优的方向，但是速度会比批处理梯度下降的速度要快。

### 正规方程组（the normal equations）

#### 矩阵微分

##### 定义

函数 $f$ 满足：

$$
\begin{aligned}
f: \mathbb{R}^{m\times n} \to \mathbb{R} \\
A \in \mathbb{R}^{m\times n} \quad f(A) \in \mathbb{R}
\end{aligned}
$$

可以得到矩阵微分的定义：

$$
\nabla_Af(A) =
\begin{bmatrix}
\frac{\partial{f}}{\partial A_{11}} & \cdots & \frac{\partial{f}}{\partial A_{1n}} \\
\vdots & \ddots & \vdots \\
\frac{\partial{f}}{\partial A_{m1}} & \cdots & \frac{\partial{f}}{\partial A_{mn}}
\end{bmatrix}
$$

##### 微分性质

基本微分法则：
$$
\begin{aligned}
&\nabla_{x} b^TAx = b^TA \\
&\nabla_{x} x^TAx = Ax + A^Tx \\
&\nabla_{A^T} f(A) = {\nabla_{A} f(A)}^T \\
\end{aligned}
$$

矩阵的迹（trace）的性质：
$$
\begin{aligned}
&tr AB = tr BA \\
&tr A = tr A^T \\
&tr (A + B) = tr A + tr B \\
&tr ABCD = tr DABC = tr CDAB = tr BCDA
\end{aligned}
$$

矩阵的迹的微分性质：
$$
\begin{aligned}
&\nabla_A tr AB = B^T \quad if \quad f(A) = tr AB \\
&\nabla_{A} trABA^TC = CAB + C^TAB^T \\
\end{aligned}
$$

#### 最小二乘法

$$
\begin{aligned}
X &= \begin{bmatrix}
(x^{(1)})^T \\
(x^{(2)})^T \\
\vdots \\
(x^{(m)})^T
\end{bmatrix} \\
y &= \begin{bmatrix}
y^{(1)} \\
y^{(2)} \\
\vdots \\
y^{(m)}
\end{bmatrix} \\
J(\theta) &= \frac 12 (X\theta - y)^T(X\theta - y)
\end{aligned}
$$

对损失函数进行微分：

$$
\begin{aligned}
\nabla_{\theta} J(\theta) &= \nabla_{\theta} \frac12(X\theta - y)^T(X\theta - y) \\
&=\frac12 \nabla_{\theta}(\theta^T X^TX\theta - \theta^T X^T y - y^TX\theta + y^Ty) \\
&=\frac12 \nabla_{\theta} tr(\theta^T X^TX\theta - \theta^T X^T y - y^TX\theta + y^Ty) \\
&=\frac12 (X^TX\theta + X^TX\theta - 2X^Ty + 0) \\
&=X^TX\theta - X^Ty
\end{aligned}
$$

令其等于零，得到正规方程组：

$$
\begin{aligned}
X^T X\theta &= X^Ty \\
\hat \theta &= (X^TX)^{-1}X^T y
\end{aligned}
$$

利用概率论的知识，设 $Y = \theta^T X + \epsilon$ ，其中 $Y$，$X$，$\epsilon$ 是随机变量，并且 $\epsilon \sim \mathcal{N}(0, \sigma^2)$。可以利用最大似然估计得到 $\theta$ 的最优解。

### 局部权重线性规划（Locally weighted linear regression）

对于普通的线性规划来说，预测一个点 $x$ 的值，我们需要做：

1. 最小化 $\sum_i (y^{(i)} - \theta^Tx^{(i)})^2$ 得到 $\theta$。
2. 输出 $\hat{y} = \theta^Tx$。

对于局部线性规划，损失函数变为了 $\sum_i w^{(i)}(y^{(i)} - \theta^Tx^{(i)})^2$，$w^{(i)}$ 可以表示为：
$$
w^{(i)} =\exp(-\frac{(x^{(i)} - x)^2}{2\tau^2})
$$
这使得离 $x$ 越远的数据点对损失的贡献越小。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
