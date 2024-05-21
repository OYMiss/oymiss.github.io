---
author: Chongzhuo Yang
pubDatetime: 2019-08-04T01:37:37.816Z
title: 矩阵的广义逆
slug: "pseudo-inverse"
# featured: false
tags:
  - math
description: 介绍矩阵的广义逆及其几何意义。
---

## 定义

对于一个 $m \times n$ 的实矩阵 $A$，$A^+$ 应该满足下面四个条件：

1. $AA^+A = A$
2. $A^+AA^= A^+$
3. $(AA^+)^T = AA^+$
4. $(A^+A)^T = A^+A$

如果 $A$ 列线性无关，则有：

$$
A^+ = (A^TA)^{-1}A^T
$$

称为左逆，并有 $A^+A = I$。

<!--more-->

如果 $A$ 行线性无关，则有：

$$
A^+ = A^T(AA^T)^{-1}
$$

称为右逆，并有 $AA^+ = I$。

## 几何解释

对于一个线性方程组 $Ax = y$，$A_{m \times n}$ 矩阵是一个 $\mathbb{R}^n \to \mathbb{R}^m$ 的线性变换。需要解出 $x^* \in \mathbb{R}^n$ 使得 $Ax^* = y$，可以通过广义逆求出 $x^* = A^+y$，相当于 $x^* = A^+Ax^*$，这里的 $Ax^*$ 也一定在 $A$ 的列空间中。

如果方程 $Ax = y$ 无解，即找不到一个 $x^*$ 使得 $Ax^* = y$，将 $y$ 投影到 $A$ 的列空间上并得到 $y'$，这样方程才有解，即 $Ax^* = y'$。

根据 $A$ 的向量子空间的性质，$C(A)$ 与 $N(A^T)$ 是正交的，可以用这两个空间里面的向量张成整个 $\mathbb{R}^m$。一定能将 $y_m$ 分解为 $y_m = P_{C(A)}(y) + P_{N(A^T)}(y) = y' + y_0$。可以得到

$$
A^Ty = A^Ty' + A^Ty_0 = A^Ty'
$$

其中 $y'$ 可以写成 $Ax^*$，可以得到 $A^Ty = A^TAx^*$，如果 $A^TA$ 可以逆，即 $R(A) = n$，则有

$$
x^* = (A^TA)^{-1}A^Ty
$$

其中，$(A^TA)^{-1}A^T$ 叫做 Moore-Penrose 广义逆。

## 求解

> 如果 $A^{-1}$ 存在，那么 $A^+ = A^{-1}$。

将 $A$ 进行 SVD 分解即 $A = U\Sigma V^T$，就有 $A^+ = V\Sigma^+U^T$。

其中 $\Sigma^+$ 就是将 $\Sigma$ 转置一下然后将 $\sigma_i$ 变为 $1/\sigma_i$。

[1] [Matrix Methods in Data Analysis, Signal Processing, and Machine Learning](https://ocw.mit.edu/courses/mathematics/18-065-matrix-methods-in-data-analysis-signal-processing-and-machine-learning-spring-2018/)

[2] [Moore–Penrose inverse](https://en.wikipedia.org/wiki/Moore%E2%80%93Penrose_inverse)

[3] [Geometrical meaning of the Moore-Penrose pseudo inverse](http://massimozanetti.altervista.org/files/mydocs/geometricalMeaningMoorePenrose.pdf)
