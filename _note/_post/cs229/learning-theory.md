---
mathjax: true
title: 学习理论
date: 2020-01-10 10:27:58
tags: [cs229, 机器学习]
categories: 机器学习
---

### 偏差/方差的权衡（Bias/variance tradeoff）

<img src="/asset/learning-theory/linear-regression.png" width = "100%" height = "100%" alt=""/>

最左边的图片中情况属于欠拟合，最右边的情况属于过拟合。这两种情况的泛化误差（generalization error）都很大。

我们粗略定义偏差（Bias）为即使我们有非常大的训练集的情况下的期望泛化误差（expected generalization error）。除了偏差以外，还有方差（variance）构成泛化误差。

<!-- more -->

如果模型过于简单，偏差可能就会很大，方差可能很小。相反，如果过于复杂，偏差可能会很小，方差会很大。

### 预先准备（Preliminaries）

（The union bound 定理）令 $A_1,A_2, \dots, A_k$ 为 $k$ 个不是事件（可能不独立），那么：

$$
P(A_1\cup...\cup A_k)\leq P(A_1)+...+P(A_k)
$$

（Hoeffding 不等式）令 $Z_1,...,Z_n$ 为 $n$ 个独立同分布的随机变量，它们都来自 $\text{Bernoulli}(\phi)$。也就是 $P(Z_i =1)=\phi$ 和 $P(Z_i =0)= 1 - \phi$。令 $\hat\phi=(\frac1n)\sum^n_{i=1}Z_i$，对于任意定 值 $\gamma \geq 0$，那么：

$$
P(|\phi-\hat\phi|>\gamma)\leq 2\exp (-2\gamma^2n)
$$

为了简化问题，将注意力放在二元分类，也就是 $y\in\{0, 1\}$。

给定训练集 $S = \{(x^{(i)}, y^{(i)}); i = 1, \dots, m\}$，假设所有 $(x^{(i)}, y^{(i)})$ 对来自概率分布 $\mathcal{D}$。对于一个假设 $h$，训练误差（training error）（也被叫做 empirical risk 和 empirical error）为：

$$
\hat\epsilon(h) =\frac1m\sum^m_{i=1}1\{h(x^{(i)})\neq y^{(i)}\}
$$

定义泛化误差（generalization error）为：

$$
\epsilon(h) =P_{(x,y)\sim D}(h(x)\neq y)
$$

这是一个概率，也就是从 $\mathcal{D}$ 生成一个 $(x, y)$，然后 $h$ 会把它分类错的概率。

考虑到线性分类的情况，令 $h_{\theta}(x) = 1\{\theta^Tx \ge 0\}$，可以用什么方式来拟合 $\theta$ 呢？一种方式是最小化训练误差，然后求得

$$
\hat\theta=\arg\min_\theta\hat\epsilon(h_\theta)
$$

我们把这个过程叫做经验风险最小化（empirical risk minimization 简写 ERM），得到的假设为 $\hat h = h_{\hat\theta}$。这是一种最“基本”的学习算法。

定义一个假设类别（hypothesis class）$\mathcal{H}$ 来表示所有分类器的集合。对于线性分类问题 $\mathcal{H} = \{h_\theta : h_\theta(x) = 1\{\theta^T x \geq 0\}, \theta \in R^{n+1}\}$。

然后 ERM 可以被看作，在集合 $\mathcal{H}$ 中挑选出使得训练误差最小的那个假设：

$$
\hat h=\arg\min_{h\in \mathcal{H}}\hat\epsilon(h)
$$

### 有限 $\mathcal{H}$ 的情况

取任意一个 $h_i \in \mathcal{H}$。已知 $(x, y) \in \mathcal{D}$，定义 $Z = 1\{h_i(x) \ne y\}$。类似的，可以定义 $Z_j  = 1\{h_i(x^{(i)}) \ne y^{(i)}\}$（$Z$ 为伯努利分布），因为训练集也是来自 $\mathcal{D}$，$Z$ 和 $Z_j$ 有同样的分布。

对于一个随机的样本，分错类的概率也就是 $Z$（和 $Z_j$）的期望值，除此之外，训练误差可以写成：

$$
\hat\epsilon(h_i)=\frac 1m\sum_{j=1}^mZ_j
$$

然后利用 Hoeffding 不等式可以得到：

$$
P(|\epsilon(h_i)-\hat\epsilon(h_i)|>\gamma)\leq 2\exp (-2\gamma^2m)
$$

假设 $m$ 很大，那么训练误差将很大概率和泛化误差很接近。下面考虑所有 $h \in \mathcal{H}$ 同时满足的情况。设 $A_i$ 表示 $|\epsilon(h_i) - \hat\epsilon(h_i)| > \gamma$，对于任意给定 $A_i$，都有 $P(A_i) \le 2\exp (-2\gamma^2m)$。利用 union bound 可以得到：

$$
\begin{aligned}
P(\exists h\in \mathcal{H}.|\epsilon(h_i)-\hat\epsilon(h_i)|>\gamma)
& = P(A_1\cup...\cup A_k) \\
& \le \sum_{i=1}^k P(A_i) \\
& \le \sum_{i=1}^k 2\exp (-2\gamma^2m) \\
& = 2k\exp (-2\gamma^2m)
\end{aligned}
$$

同时用一减去左右两边，我们得到：

$$
\begin{aligned}
P(\forall h\in H.|\epsilon(h_i)-\hat\epsilon(h_i)|\le\gamma)&
= P(\neg\exists h\in H.|\epsilon(h_i)-\hat\epsilon(h_i)|>\gamma) \\
& \ge 1-2k\exp (-2\gamma^2m)
\end{aligned}
$$

所以，至少有 $1-2k\exp (-2\gamma^2m)$ 的概率 $\epsilon(h)$ 与 $\hat \epsilon(h)$ 距离在 $\gamma$ 之间。这个结果被叫做一致收敛（uniform convergence）结果。

给出两个值 $m$ 和 $\gamma$ 就能知道 $|\epsilon(h)-\hat\epsilon(h)| > \gamma$ 的概率，这三个值中，任意知道两个都能够求出来另外一个。

给出 $\gamma$ 和 $\delta > 0$，$m$ 应该多大才能保证错误都小与等于 $\gamma$ 的概率至少为 $1 - \delta$，令 $\delta = 2k \exp (-2\gamma^2m)$，求得：

$$
m\ge \frac{1}{2\gamma^2}\log\frac{2k}{\delta}
$$

> 所需要的样本数量是假设集合 $\mathcal{H}$ 的大小 $k$ 的对数（logarithmic）。

为了达到某些方法或者算法一定级别的性能而需要的训练集的大小被称作样本复杂度（sample complexity）。

当给出 $m$ 和 $\delta$ 的情况下，求 $\gamma$，得到：

$$
|\hat\epsilon(h)-\epsilon(h)|\le \sqrt{\frac{1}{2m}log\frac{2k}{\delta}}
$$

先假设一致收敛，也就是对于所有 $h$ 有 $|\epsilon(h_i)-\hat\epsilon(h_i)| \le \gamma$。

能对利用 $\hat{h} = arg\min_{h\in \mathcal{H}} \hat\epsilon(h)$ 的算法进行哪些推论呢？

定义 $h^∗ = arg \min_{h\in \mathcal{H}} \epsilon(h)$ 是 $\mathcal{H}$ 中最好的假设。

$$
\begin{aligned}
\epsilon(\hat h)
& \le \hat\epsilon(\hat h)+\gamma \\
& \le \hat\epsilon(h^*)+\gamma \\
& \le \epsilon(h^*)+2\gamma
\end{aligned}
$$

第一行利用的是一致收敛的假设 $| \epsilon(\hat h) - \hat\epsilon (\hat h) | \le \gamma$。
对于第二行，利用的是对于所有 $h$ 都有 $\hat\epsilon(\hat{h}) \leq \hat\epsilon(h)$，因为这里 $\hat h$ 是对 $\hat\epsilon(h)$ 的最小化，特殊的就有 $\hat\epsilon(\hat{h}) \le \hat\epsilon(h^∗)$。
第三行则再次利用了一致收敛的假设 $\hat\epsilon(h^∗) \le \epsilon(h^∗) + \gamma$。

最后表明，$\hat h$ 的泛化误差比在 $\mathcal{H}$ 中最好的假设差最多 $2\gamma$。

将上面两个结论放在一起，设 $|H| = k$，然后设 $m$ 和 $\delta$ 为任意的固定值，并且概率至少为 $1 - \delta$，则有：

$$
\epsilon(\hat h)\le (\min_{h\in H}\epsilon(h))+2\sqrt{\frac{1}{2m}log\frac{2k}{\delta}}
$$

这里是令 $\gamma$ 等于式子中的 $\sqrt{\cdot}$ 项。

这里对模型选择中偏差（bias）和方差（variance）之间的权衡进行了量化。具体来说：如果假设集 $\mathcal{H}$ 变成更大的假设集 $\mathcal{H}'$，那么式子中的第一项 $\min_h \epsilon(h)$ 就会减小（或不变），但是 $2\sqrt{\cdot}$ 这一项就会变大（$k$ 变大了）。

> 当使用更大的假设集的时候，偏差（bias）只会减少，但是如果 $k$ 变大，方差（variance）会跟着 $k$ 一起变大。

通过固定 $\gamma$ 和 $\delta$，可以得到下面的样本复杂度（sample complexity bound）：

设 $|H| = k$ ，然后令 $\delta$ 和 $\gamma$ 为任意的固定值。对于满足 $\epsilon(\hat{h}) \le min_{h\in H} \epsilon(h) + 2\gamma$ 的概率最少为 $1 - \delta$ 的情况下，下面等式关系成立：

$$
\begin{aligned}
m &\ge \frac{1}{2\gamma^2}log\frac{2k}{\delta} \\
  & = O(\frac{1}{\gamma^2}log\frac{k}{\delta})
\end{aligned}
$$

### 无限 $\mathcal{H}$ 的情况

这里做一个并不“正确”的假设，利用这个假设获得一个直观的结论。

假设 $\mathcal{H}$ 有 $d$ 个实数，然后这些参数是通过计算机存储的，并且使用 $64$ 位字节去保存。那么，$\mathcal{H}$ 的大小 $k = 2^{64d}$。

利用有限集的结论，对于固定的 $\gamma$ 和 $\delta$ 有：

$$
n \ge O(\frac{1}{\gamma^2} \log \frac{2^{64d}}{\delta}) = O(\frac{d}{\gamma^2} \log \frac{1}{\delta}) = O_{\gamma, \delta} (d)
$$

给定一个集合 $S = \{ x^{(1)}, \dots , x^{(D)} \}$，如果 $\mathcal{H}$ 可以实现 $S$ 中任意一种结果就说 $\mathcal{H}$ 散点（shatters）$S$，也就是说，对于任意的标签 $\{y^{(1)}, \dots, y^{(D)}\}$ 都存在一些 $h\in \mathcal{H}$ 使得 $h(x^{i}) = y^{(i)}$。

给定一个假设集 $\mathcal{H}$，然后定义它的 Vapnik-Chervonenkis 维度（dimension）为 $\mathcal{H}$ 能够散点的最大的集合的大小，写成 $\text{VC}(\mathcal{H})$。

<img src="/asset/learning-theory/three-points.png" width = "60%" height = "60%" alt="three points"/>

对于二维的线性分类器 $h(x) = 1\{ \theta_0 + \theta_1x_1 + \theta_2x_2 \ge 0 \}$ 能否对上面三个点的集合进行分类呢？

答案是可以。

<img src="/asset/learning-theory/shatter-three-points.png" width = "80%" height = "80%" alt="shatter three points"/>

而对于四个点，二维的线性分类器就没有办法进行完美分类，可以得到这里 $\text{VC}(\mathcal{H}) = 3$。但是如果三点共线，二维的线性分类器也没有办法对它进行完美分类。

给定 $\mathcal{H}$ 令 $D = \mathcal{H}$，那么至少有 $1-\delta$ 的概率，对于所有 $h \in \mathcal{H}$：

$$
|\epsilon(h)-\hat\epsilon(h)|\le O(\sqrt{\frac{D}{m}\log\frac{D}{m}+\frac 1m\log\frac 1\delta})
$$

另外，至少有 $1 - \delta$ 的概率可以得到：

$$
\hat\epsilon(h)\le \epsilon(h^*)+O(\sqrt{\frac{D}{m}\log\frac{D}{m}+\frac 1m\log\frac 1\delta})
$$
另外可以得到：

对于所有的 $h \in H$ 成立的 $|\epsilon(h) - \hat\epsilon(h)| \le \gamma$ （因此也有 $\epsilon(\hat h) ≤ \epsilon(h^∗) + 2\gamma$），则有至少为 $1 – \delta$ 的概率，满足 $m = O_{\gamma,\delta}(d)$

也就是说，让模型训练良好的训练样本的数量和 $\mathcal{H}$ 的 $\text{VC}$ 维度成线性。而对于大部分假设集 $\text{VC}$ 维度和参数的数量是线性的。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
