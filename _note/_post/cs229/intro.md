---
mathjax: true
title: 机器学习引言
date: 2019-12-26 10:23:38
tags: [cs229, 机器学习]
categories: 机器学习
---

> 机器学习分类：

1. 监督学习（supervised learning）
2. 无监督学习（unsupervised learning）
3. 强化学习（reinforcement learning）

<!-- more -->

> 常用记号

$m$ ：表示训练的样本的数量

$x$ ：表示输入变量/特征变量

$y$ ：表示输出变量/目标变量

$(x^{(i)}, y^{(i)})$ ：第 $i$ 个训练样本

### 矩阵计算

#### 梯度

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

也就是：

$$
(\nabla_Af(A))_{ij} = \frac{\partial  f(A)}{\partial A_{ij}}
$$

注意 $\nabla_Af(A)$ 的形状和 $A$ 的形状一样。

> 函数的梯度只有当函数是实值函数的时候才有意义。$Ax, A\in \mathbb{R}^{n\times n}$ 对 $x$ 不能求梯度，因为这个函数是向量值函数。

#### 海森矩阵

如果 $f: \mathbb{R}^n \to \mathbb{R}$，那么对于 $x$ 的海森矩阵（写成 $\nabla^2_xf(x)$ 或者 $H$）：

$$
(\nabla_x^2f(x))_{ij} = \frac{\partial^2 f(x)}{\partial x_i \partial x_j}
$$

注意：海森矩阵永远是对阵矩阵。

对于实值函数来说，二阶导数就是一阶导数的导数，但是对于一个关于向量的函数来说，这个变得不一样。

下面这种表达式是没有定义的：

$$
\nabla_x\nabla_xf(x) = \nabla_x \begin{bmatrix}
    \frac{\partial f(x)}{\partial x_1} \\
    \frac{\partial f(x)}{\partial x_2} \\
    \vdots \\
    \frac{\partial f(x)}{\partial x_n} \\
\end{bmatrix}
$$

但是这并不说海森不是梯度的梯度，对于梯度的第 $i$ 个元素 $(\nabla_x f(x))_i = \frac{\partial f(x)}{\partial x_i}$ 对它求关于 $x$ 的梯度，就可以得到：

$$
\nabla_x \frac{\partial f(x)}{\partial x_i} = \begin{bmatrix}
    \frac{\partial^2 f(x)}{\partial x_i\partial x_1} \\
    \frac{\partial^2 f(x)}{\partial x_i\partial x_2} \\
    \vdots \\
    \frac{\partial^2 f(x)}{\partial x_i\partial x_n} \\
\end{bmatrix}
$$

上面是海森矩阵的第 $i$ 行（列），于是有：

$$
\nabla^2_xf(x) =
\begin{bmatrix}
\nabla_x(\nabla_x f(x))_1 & \nabla_x(\nabla_x f(x))_2 & \cdots & \nabla_x(\nabla_x f(x))_n
\end{bmatrix}
$$

如果不介意这个不严格的话，可以说海森矩阵是梯度的梯度。

#### 二次线性函数的梯度和海森矩阵

对于 $x \in \mathbb{R}^n$，令 $f(x) = b^Tx$ 对于已知 $b \in \mathbb{R}^n$，那么就有：

$$
f(x) = \sum_{i=1}^{n} b_ix_i
$$

得到梯度为：

$$
\nabla_x b^Tx = b
$$

对于二次型：

$$
f(x) = x^TAx \qquad A \in \mathbb{S}^n
$$

梯度和海森矩阵就是：

$$
\begin{aligned}
\nabla_x x^TAx &= 2Ax \\
\nabla^2_x &= 2A
\end{aligned}
$$

#### 最小二乘法

对于最小二乘法需要最小化欧几里德范式 $\| Ax - b\|_2^2$：

$$
\begin{aligned}
\| Ax - b\|_2^2 &= (Ax - b)^T(Ax - b) \\
&= x^TA^TAx - 2b^TAx + b^Tb
\end{aligned}
$$

对其求梯度得到：

$$
\nabla_x(x^TA^TAx - 2b^TAx + b^Tb) = 2A^TAx - 2A^Tb
$$

令其等于 $0$ 得到：$x = (A^TA)^{-1}A^Tb$

#### 行列式的梯度

对于每个 $j \in (1, \dots, n)$，$A$ 的行列式

$$
|A| = \sum_{i=1}^{n} (-1)^{i+j}A_{ij} |A_{\backslash i, \backslash j}|
$$

所以有：

$$
\begin{aligned}
    \frac{\partial }{\partial A_{kl}} |A| &= \frac{\partial }{\partial A_{kl}} \sum_{i=1}^{n} (-1)^{i+j}A_{ij} |A_{\backslash i, \backslash j}| \\
    &= (-1)^{k+l} |A_{\backslash k, \backslash l}| \\
    &= (\text{adj}(A))_{lk}
\end{aligned}
$$

根据伴随矩阵的性质 $\text{adj}(A) = C^T$，其中 $C$ 是余子式矩阵，就有：

$$
\nabla_A |A| = (\text{adj}(A))^T = |A| A^{-T}
$$

例子：如果有 $f(A) = log |A|, \quad f:\mathbb{S}^n_{++} \to \mathbb{R}$，求 $f(A)$ 的梯度。

$$
\frac{\partial log |A|}{\partial A_{ij}} = \frac{\partial log |A|}{\partial |A|} \frac{\partial |A|}{\partial A_{ij}} = \frac{1}{|A|} \frac{\partial |A|}{\partial A_{ij}}
$$

然后可以得到：

$$
\nabla_A \log |A| = \frac{1}{|A|} \nabla_A |A| = A^{-T}
$$

#### 用特征值优化二次型

有条件约束的优化问题：

$$
\max_{x\in \mathbb{R}^n} x^TAx \quad \text{s.t.} \| x \|_2^2 = 1
$$

利用拉格朗日乘数法，得到目标函数：

$$
\mathcal{L}(x, \lambda) = x^TAx - \lambda x^Tx
$$

对 $x$ 求梯度，令其等于 $0$ 得到：

$$
\nabla_x \mathcal{L}(x, \lambda) = 2A^T x - 2\lambda x = 0
$$

只要让 $Ax = \lambda x$ 就能得到极值，特征向量正好满足这个条件。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
