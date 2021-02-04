---
mathjax: true
title: 独立成分分析
date: 2020-01-20 21:41:44
tags: [cs229, 机器学习]
categories: 机器学习
---

独立成分分析（Independent Components Analysis，缩写 ICA）和主成分分析（PCA）类似，都是为了找到数据的新基（basis）来表示数据，但是两者的目标不一样。

例如：在一个酒会上，有 $n$ 个人同时在讲话，并且有 $n$ 个麦克风在不同的位置记录着这几个人的声音，能否利用这 $n$ 个麦克风的数据分离出这 $n$ 个人各自的讲话内容呢？

假设有一些数据 $s \in \mathbb{R}^n$ 通过 $n$ 个不同的来源生成，但是观测到的是：

$$
x = As
$$

这里 $A$ 是未知的矩阵，叫做混合矩阵（mixing matrix）。重复的观测得到数据集 $\{ x^{(i)}; i=1,\dots,m \}$，ICA 的目标是恢复产生数据 $x^{(i)} = As^{(i)}$ 的源数据（sources） $s^{(i)}$。

<!-- more -->

令 $W = A^{-1}$ 为解混合矩阵（unmixing matrix），目标是为了找到 $W$，这样就可以计算 $s^{(i)} = Wx^{(i)}$ 来恢复源数据（sources）。

> 源数据进行排列或者缩放大部分情况下不会影响结果。

> 需要保证 $s$ 不是高斯分布。

假设 $s^{(i)}$ 之间是独立的，则有：

$$
p(s) = \prod_{j = 1}^{n}p_s(s_j)
$$

因为 $x = As = W^{-1}s$，可以得到：

$$
p(x) = \prod_{j = 1}^{n}p_s(w_j^Tx) \cdot |W|
$$

> 对于线性变换，$s = Wx$，这个式子 $p_x(x) = p_s(Wx)$ 是错误的，需要在式子后面加上 $|W|$。

因为不能选取正态分布作为 $p_s$ 的分布，这里选择 sigmoid 函数作为 $s$ 的概率分布函数，于是有 $p_s(s) = g'(s)$，其中 $g(s) = 1/(1+e^{-s})$。

给定数据集 $\{ x^{(i)}; i=1,\dots,m \}$ 可以得到对数似然函数：

$$
p(x) = \sum_{i=1}^{m}\left(\sum_{j = 1}^{n}\log g'(w_j^Tx) + \log |W| \right)
$$

利用 $\nabla_W|W| = |W|W^{-T}$ 计算出微分之后，可以得到梯度上升算法的更新规则：

$$
W=W+\alpha\begin{pmatrix}
\begin{bmatrix}
1-2g(w_1^T x^{(i)}) \\
1-2g(w_2^T x^{(i)}) \\
\vdots \\
1-2g(w_n^T x^{(i)})
\end{bmatrix}x^{(i)T} + W^{-T}
\end{pmatrix}
$$

> 利用最大似然估计的时候就已经错误地假设了 $x^{(i)}$ 之间是独立的。当数据大的时候并不影响模型的性能，也可以在随机打乱的数据进行 SGD 来加快收敛的速度。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
