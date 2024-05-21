---
author: Chongzhuo Yang
pubDatetime: 2019-08-05T12:07:12.816Z
title: 奇异值分解与主成分分析
slug: "svd-pca"
# featured: false
tags:
  - math
description: 介绍奇异值分解，并将其应用在主成分分析中。
---

## 奇异值分解（SVD）

如果矩阵 $A_{m\times n}$ 不能够进行特征值分解，要考虑使用**奇异值分解（singular value decomposition，SVD）** 化成：

$$
A = U\Sigma V^T
$$

其中，$U_{m\times m}$ 和 $V_{n\times n}$ 都是正交矩阵，$\Sigma_{m\times n}$ 是一个矩形对角矩阵 （rectangular diagonal matrices），并且 $\Sigma$ 主对角线上的元素都是非负的，记作 $\sigma_i$。

$$
AA^T = U\Sigma V^T V\Sigma^T U^T = U \Sigma\Sigma^T U^T
$$

其中 $AA^T$ 是对称矩阵，一定能够对角化。

<!--more-->

$U$ 被称作 $A$ 的**左奇异向量 （left-singular vectors）**，它是 $AA^T$ 的特征向量的一个集合。

$V$ 被称作 $A$ 的**右奇异向量 （right-singular vectors）**，它是 $A^TA$ 的特征向量的一个集合。

$AA^T$ 与 $A^TA$ 拥有同样的特征值，于是 $\sigma_i^2$ 是 $AA^T$ 的特征值。

相对于特征值分解中 $Sv_i = \lambda v_i$，奇异值分解有：

$$
Av_i = \sigma_i u_i \qquad i = 1, \ldots, \min(m, n)
$$

可以把矩阵 $A$ 看作是一个变换，将 $\mathbb{R}^m$ 变换到 $\mathbb{R}^n$，其中 $\mathbb{R}^m$ 的基向量为 $u_i$，$\mathbb{R}^n$ 的基向量为 $v_i$。

## 主成分分析（PCA）

**主成分分析（Principal components analysis，PCA）** 是一种统计分析、简化数据集的方法。它利用正交变换来对一系列可能相关的变量的观测值进行线性变换，从而投影为一系列线性不相关变量的值，这些不相关变量称为主成分（Principal Components）。

用矩阵表示就是

$$
T = XW
$$

把数据变换到新的坐标系统后，使得第一主成分描述的是最大方差的方向，第二主成分是第二大方差的方向，以此类推。

对于每个主成分有：$t_{k(i)} = x_{i} \cdot w_{(k)}$。

为了使得方差最大化，第一个权重向量 $w_1$ 应该满足：

$$
\begin{align*} \\
 w_{(1)}
 & = \underset{\Vert \mathbf{w} \Vert = 1}{\arg \max} \{ \sum(t_1)^2_{(i)}\} \\
 & = \underset{\Vert \mathbf{w} \Vert = 1}{\arg \max} \{ \sum(x_{(i)}\cdot w)^2\} \\
 & = \underset{\Vert \mathbf{w} \Vert = 1}{\arg \max} \{{\Vert Xw \Vert}^2\} \\
 & = \underset{\Vert \mathbf{w} \Vert = 1}{\arg \max} \{ w^TX^TXw \} \\\\
\end{align*}
$$

当 $w_{(1)}$ 为单位向量时，

$$
w_{(1)} = \arg \max \left\{ \frac{w^TX^TXw}{w^Tw} \right\}
$$

**瑞利商（Rayleigh quotient）**的定义是：

$$
R(M, x) = \frac{x^*Mx} {x^*x}
$$

如果 $M$ 矩阵是一个 Hermitian 矩阵，实数范围内就是对称矩阵，那么有 $R(M, x) \in [\lambda_{min}, \lambda_{max}]$，如果取最大值，则 $x$ 为最大的特征值对应的一个特征向量。

所以 $w_{(1)}$ 的就是 $X^TX$ 的最大特征值所对应的特征向量，也就是 $X$ 的最大奇异值在 $V$ 中所对应的右奇异向量。

从 $X$ 中减去前 $k - 1$ 个主成分得到 $\hat{X_k}$。

$$
\hat{X_k} = X - \sum_{s = 1}^{k - 1}Xw_{(s)}w_{(s)}^T
$$

$$
w_{(k)} = \arg \max \left\{ \frac{w^T\hat{X_k}^T\hat{X_k}w}{w^Tw} \right\}
$$

经过计算可以得出 $\hat{X_k}^T\hat{X_k}$ 与 $X^TX$ 的特征值及特征向量相同，可以得到 $w_{(k)}$ 是 $X^TX$ 第 $k$ 大的特征值对应的特征向量。

最后可以写成：

$$
T = XW
$$

其中 $W_{p \times p}$ 是权重矩阵，它的列向量是 $X^TX$ 的特征向量，$X$ 是一个 $n \times p$ 的矩阵，$n$ 可以代表实验次数，$p$ 可以代表特征的个数。

可以只保留前 $L$ 主成分，得到 $T_L = X W_L$，其中 $T_L$ 是一个 $n \times L$ 的矩阵，$W_L$ 是一个 $p \times L$ 的矩阵。

## 将 SVD 用于 PCA

将 $X$ 做 SVD 得到 $X = U\Sigma W^T$，于是 $T$ 可以写成：

$$
\begin{align*}
T
&= XW \\
&= U\Sigma W^TW \\
&= U\Sigma
\end{align*}
$$

$T$ 矩阵可以变成左奇异向量乘以对应的奇异值得到，这个叫做**极分解（Polar decomposition）**。

对于矩阵 $A$，如果取出 $k$ 个最大的奇异值，得到 $A_k$。

$$
A = U \Sigma V^T = \sigma_1u_1v_1^T + \dots + \sigma_ru_rv_r^T \\
A_k = U_k \Sigma_k V_k^T = \sigma_1u_1v_1^T + \dots + \sigma_ku_kv_k^T
$$

**Eckart–Young 定理**：如果 $B$ 矩阵的秩为 $k$，那么 $\Vert A - B \Vert \ge \Vert A - A_k \Vert$。

被截取的 $n\times L$ 矩阵 $T_L$ 可以通过获得 $L$ 个最大的奇异值及其对应的奇异向量组成。

$$
T_L = U_L\Sigma_L = X W_L
$$

根据 Eckart–Young 定理，通过这种截断的奇异值分解得到的矩阵，$T_L$ 或者 $A_L$ 是最接近的原来矩阵的秩为 $L$ 的矩阵。

[1] [Matrix Methods in Data Analysis, Signal Processing, and Machine Learning](https://ocw.mit.edu/courses/mathematics/18-065-matrix-methods-in-data-analysis-signal-processing-and-machine-learning-spring-2018/)

[2] [Principal component analysis](https://en.wikipedia.org/wiki/Principal_component_analysis)

[3] [Singular value decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition#Geometric_meaning)

[4] [瑞利商与极值计算](https://seanwangjs.github.io/2017/11/27/rayleigh-quotient-maximum.html)

[5] [Rayleigh quotient](https://en.wikipedia.org/wiki/Rayleigh_quotient)
