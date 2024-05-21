---
author: Chongzhuo Yang
pubDatetime: 2019-08-31 10:01:29
title: 用线性代数的眼光来看离散傅立叶变换
slug: "explain-dft-by-linear-algebra"
# featured: false
tags:
  - math
description: 使用线性代数的知识来推导出卷积公式。
---

如果 $x$ 和 $c$ 都是长度为 $n$ 的向量，那么它们循环卷积的结果 $y$ 就有

$$
y_k = (x \otimes c)_k = \sum_{i=0}^{n-1} x_i c_{k-i}
$$

> 其中 $c_{k-i}$ 代表循环移位。

将 $c$ 的循环移位写成矩阵的形式

$$
C=
\begin{bmatrix}
c_0     & c_{n-1} & \dots  & c_{2} & c_{1}  \\
c_{1} & c_0    & c_{n-1} &         & c_{2}  \\
\vdots  & c_{1}& c_0    & \ddots  & \vdots   \\
c_{n-2}  &        & \ddots & \ddots  & c_{n-1}   \\
c_{n-1}  & c_{n-2} & \dots  & c_{1} & c_0 \\
\end{bmatrix}
$$

<!--more-->

于是就有

$$
y = Cx
$$

举个例子：

$$
\begin{bmatrix}
16 \\
14 \\
16 \\
14 \\
\end{bmatrix}
=\begin{bmatrix}
1 & 4 & 3 & 2 \\
2 & 1 & 4 & 3 \\
3 & 2 & 1 & 4 \\
4 & 3 & 2 & 1 \\
\end{bmatrix}
\begin{bmatrix}
1 \\
2 \\
1 \\
2 \\
\end{bmatrix}
$$

其中 $C$ 可以用另一种方式表示：

$$
C=f(P)=c_0I+c_1P+c_2P^2+\ldots+c_{n-1}P^{n-1}
$$

$P$ 矩阵是一个置换矩阵，也被称作循环矩阵。

$$
P=
\begin{bmatrix}
 0&0&\ldots&0&1\\
 1&0&\ldots&0&0\\
 0&\ddots&\ddots&\vdots&\vdots\\
 \vdots&\ddots&\ddots&0&0\\
 0&\ldots&0&1&0
\end{bmatrix}
$$

它可以表示为

$$
\begin{pmatrix}
   0 & I_{n-k} \\
   I_k & 0
\end{pmatrix}
$$

可以得到 $\vert P - \lambda I_n\vert = \lambda^n - 1$，也可以通过 $P^n = I$ 来得到 $\lambda^n = 1$。根据 $e^{-j2\pi k} = 1$，解出来得到 $\lambda_k = e^{\frac{-j2\pi k}{n}} = \omega^k$，计算可以得到 $P\alpha_k = \omega^k(\alpha_{k(n-1)}, \alpha_{k(0)}, \cdots, \alpha_{k(n-3)}, \alpha_{k(n-2)})^T$。

> 这里不明白 $\omega = e^{-j2\pi/n}$，还是 $\omega = e^{j2\pi/n}$，感觉是一样的。

从而有 $\alpha_{k(i)} = \omega^k\alpha_{k(i-1)}$ 前一项乘以 $\omega^{k}$ 等于后一项，最后一项乘以 $\omega^{k}$ 得到第一项。如果 $\alpha_{k} = (1, \omega^{k}, \omega^{2k}, \cdots, \omega^{(n-1)k})^T$ 就正好满足，其中有 $\omega^{kn}=\omega^{k0}=1$ 。

可以得到 $P$ 的特征向量组成的矩阵：

$$
F = \begin{bmatrix}
1&1&1&1&\cdots &1 \\
1&\omega&\omega^2&\omega^3&\cdots&\omega^{n-1} \\
1&\omega^2&\omega^4&\omega^6&\cdots&\omega^{2(n-1)}\\ 1&\omega^3&\omega^6&\omega^9&\cdots&\omega^{3(n-1)}\\
\vdots&\vdots&\vdots&\vdots&\ddots&\vdots\\
1&\omega^{n-1}&\omega^{2(n-1)}&\omega^{3(n-1)}&\cdots&\omega^{(n-1)(n-1)}
\end{bmatrix}
$$

$C$ 矩阵的特征向量也是 $\alpha_k$，但是特征值变为

$$
\begin{aligned}
\lambda_k^* &= c_0 + c_1\lambda_k + c_2\lambda_k^2 + \cdots + c_{n-1}\lambda_k^{n-1} \\
&= (c_0, c_1, \cdots, c_{n-1})^T(1, \lambda_k^1, \lambda_k^2, \cdots, \lambda_k^{n-1}) \\
&= (c_0, c_1, \cdots, c_{n-1})^T(1, \omega^{k}, \omega^{2k}, \cdots, \omega^{(n-1)k}) \\
&= (c_0, c_1, \cdots, c_{n-1})^T\alpha_{k}
\end{aligned}
$$

可以得到特征值组成的向量为 $Fc$，将 $C$ 对角化，得到：

$$
C = F^{-1}diag(Fc)F
$$

这时候如果将 $y = Cx$ 中的 $x$ 投影到 $F$ 列（行）空间里，也就是 $C$ 的特征向量张成的空间，可以得到 $y$ 在这个空间中的坐标。

如果将 $x$ 和 $y$ 也写成类似于 $C$ 的循环矩阵，可以得到：

$$
Y = CX
$$

然后做对角化可以得到：

$$
\begin{aligned}
F^{-1}diag(Fy)F
&= F^{-1}diag(Fc)F F^{-1}diag(Fx)F \\
&= F^{-1}diag(Fc)diag(Fx)F
\end{aligned}
$$

于是可以得到卷积定理：

$$
\begin{aligned}
Fy
&= Fc \times Fx \\
&= F(c \otimes x)
\end{aligned}
$$

> 这里 $\times$ 表示 element-wise multiplication。

这就是著名的卷积定理，时域卷积对应着频域的乘积，而频域是向量构成的循环矩阵的特征值。

[1] [Matrix Methods in Data Analysis, Signal Processing, and Machine Learning](https://ocw.mit.edu/courses/mathematics/18-065-matrix-methods-in-data-analysis-signal-processing-and-machine-learning-spring-2018/)

[2] [Circulant matrix](https://en.wikipedia.org/wiki/Circulant_matrix)

[3] [Discrete Fourier transform](https://en.wikipedia.org/wiki/Discrete_Fourier_transform)

[4] [Circular convolution](https://en.wikipedia.org/wiki/Circular_convolution)
