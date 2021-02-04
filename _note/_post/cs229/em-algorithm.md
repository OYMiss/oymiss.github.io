---
mathjax: true
title: EM 算法
date: 2020-01-16 16:29:00
tags: [cs229, 机器学习]
categories: 机器学习
---

### Jensen 不等式（Jensen’s inequality）

如果 $f$ 的定义域为实数，当 $f''(x) \ge 0$ 时，$f$ 是凸函数。

当 $f$ 的输入为向量时，它是凸函数（convex）的条件就变成了它的海森矩阵是半正定矩阵。

> 严格凸函数的条件是 $f > 0$（海森矩阵是正定矩阵）。

如果 $f$ 是一个凸函数，令 $X$ 是一个随机变量，那么：

$$
\text{E}[f(X)] \ge f(\text{E}[X])
$$

<!-- more -->

另外，如果是严格凸函数的话 $\text{E}[f(X)] = f(\text{E}[X])$ 成立当且仅当 $X = \text{E}[X]$ 的概率为 $1$（也就是 $X$ 是一个常数）。

Jensen 不等式的结果可以用下图来进行直观的解释。

<img src="/asset/em-algorithm/jensen-inequality.png" width = "60%" height = "60%" alt="Jensen's inequality"/>

这里假设在 a 和 b 点的概率都为 $0.5$。

> $f$ 是凹函数（concave）的充要条件是 $-f$ 是凸函数。这时候 Jensen 不等式依然成立，不过不等号的方向发生了改变。

### EM 算法（The EM algorithm）

假设有 $m$ 个样本的训练集 $\{x^{(1)}, \dots, x^{(m)}\}$，要利用数据去拟合模型 $p(x,z)$，似然函数（likelihood）如下所示：

$$
\begin{aligned}
l(\theta) &= \sum_{i=1}^m\log p(x;\theta) \\
&= \sum_{i=1}^m\log\sum_z p(x,z;\theta)
\end{aligned}
$$

直接通过最大似然估计来对 $\theta$ 进行估计是比较困难的，因为这里的 $z^{(i)}$ 是潜在变量（latent random variables），但是如果 $z^{(i)}$ 被观测到了，那么这个最大似然估计就会变得简单。

EM 算法就是不断的构造 $l$ 的下界（E-step）然后优化这个下界（M-step）。

对于每个 $i$，令 $Q_i$ 是 $z$ 的分布，于是有：

$$
\begin{aligned}
\sum_i\log p(x^{(i)};\theta) &= \sum_i\log\sum_{z^{(i)}}p(x^{(i)},z^{(i)};\theta) \\
&= \sum_i\log\sum_{z^{(i)}}Q_i(z^{(i)})\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})} \\
&= \sum_i\log \text{E}_{z^{(i)}}\left[\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}\right] \\
&\ge \sum_i \text{E}_{z^{(i)}}\left[\log \frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}\right] \\
&= \sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
\end{aligned}
$$

> 上面的式子利用了 $\log$ 函数是凸函数的性质、 $z_i \sim Q_i$ 的期望的定义和 Jensen 不等式。

对于特定的 $\theta$，为了使这个界限更紧，需要 Jensen 不等式的等号成立，也就是使得：

$$
\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}=c
$$

只需要满足 $Q_i(z^{(i)})\propto p(x^{(i)},z^{(i)};\theta)$ 并且有 $\sum_z Q_i(z^{(i)}) = 1$，可以得到：

$$
\begin{aligned}
Q_i(z^{(i)}) &= \frac{p(x^{(i)},z^{(i)};\theta)}{\sum_z p(x^{(i)},z;\theta)} \\
&= \frac{p(x^{(i)},z^{(i)};\theta)}{p(x^{(i)};\theta)} \\
&= p(z^{(i)}|x^{(i)};\theta)
\end{aligned}
$$

只需要让 $Q_i$ 等于给定 $x^{(i)}$ 的条件下 $z^{(i)}$ 的后验概率分布，这里 $\theta$ 是已知的。

这样的 $Q_i$ 的选择给出了要去最大化的下界，这个步骤就是 E-step。在 M-step 中，最大化下界之后得到新的 $\theta$ 值。然后在利用新的 $\theta$ 值去计算出 $Q_i$。

下图是 EM 算法过程的直观解释：

<img src="/asset/em-algorithm/em.png" width = "80%" height = "80%" alt="em algorithm"/>

EM 算法分为两部分，一部分是 E-step 另一部分为 M-step。

（E-step）对每个 $i$，设

$$
Q_i(z^{(i)})=p(z^{(i)}|x^{(i)};\theta)
$$

（M-step） 设

$$
\theta = \arg\max_\theta\sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
$$

然后一直重复 E-step 和 M-step 直到收敛。

那么，能保证 EM 算法就一定收敛吗？

$$
\begin{aligned}
l(\theta^{(t+1)}) & \ge \sum_i\sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta^{(t+1)})}{Q_i^{(t)}(z^{(i)})}\\
& \ge \sum_i\sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta^{(t)})}{Q_i^{(t)}(z^{(i)})}\\
& = l(\theta^{(t)})
\end{aligned}
$$

上面第一个不等式是因为，对于所有 $Q_i$ 和 $\theta$ 下面的式子都成立：

$$
l(\theta)\ge \sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
$$

上面令 $Q_i = Q_i^{(t)}$ 和 $\theta = \theta^{(t+1)}$。

第二个不等式则是因为 $\theta^{(t+1)}$ 是最大化右边得到的：

$$
\theta^{(t+1)} = \arg\max_\theta \sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
$$

最后等式成立的原因是取 $Q_i$ 的方式使得 Jensen 不等式取到了等号。

如果定义：

$$
J(Q, \theta)=\sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
$$

有 $l(\theta) \ge J(Q, \theta)$，于是 EM 算法可以看作是对 $J$ 的坐标上升算法，在 E-step 过程中对 $Q$ 改变进行最大化，在 M-step 过程中改变 $\theta$ 以进行最大化。

### 混合高斯模型（Mixture of Gaussians）

不同于之前的高斯判别分析模型（Gaussian discriminant analysis model），这里的数据是没有标签的。给出数据集 $\{x^{(1)},...,x^{(m)}\}$，要对 $p(x^{(i)},z^{(i)}) = p(x^{(i)}|z^{(i)})p(z^{(i)})$ 进行建模，这里的 $z^{(i)}$ 叫做潜在变量或者隐变量。并且 $z^{(i)}$ 服从多项分布 $z^{(i)} \sim \text{Multinomial}(\phi)$，其中 $\phi_j$ 表示 $p(z^{(i)} = j)$。

E-step 如下：

$$
w_j^{(i)}=Q_i(z^{(i)}=j)=P(z^{(i)}=j|x^{(i)};\phi,\mu,\Sigma)
$$

M-step 中需要最大化的函数为：

$$
\begin{aligned}
\sum_{i=1}^m&\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\phi,\mu,\Sigma)}{Q_i(z^{(i)})}\\
&= \sum_{i=1}^m\sum_{j=1}^kQ_i(z^{(i)}=j)\log\frac{p(x^{(i)}|z^{(i)}=j;\mu,\Sigma)p(z^{(i)}=j;\phi)}{Q_i(z^{(i)}=j)} \\
&= \sum_{i=1}^m\sum_{j=1}^kw_j^{(i)}\log\frac{\frac{1}{(2\pi)^{n/2}|\Sigma_j|^{1/2}}\exp(-\frac 12(x^{(i)}-\mu_j)^T\Sigma_j^{-1}(x^{(i)}-\mu_j))\cdot\phi_j}{w_j^{(i)}}
\end{aligned}
$$

分别对 $\phi$、$\mu$ 和 $\Sigma$ 进行求导然后令其等零，分别得到：

> 对 $\phi$ 进行求导的时候需要加上约束条件 $\Sigma_{j=1}^{k}\phi_j = 1$ 并且 $\phi_j \ge 0$。

$$
\begin{aligned}
&\phi_l=\frac 1m\sum_{i=1}^m w_l^{(i)}, \\
&\mu_l=\frac{\sum_{i=1}^m w_l^{(i)}x^{(i)}}{\sum_{i=1}^m w_l^{(i)}}, \\
&\Sigma_l=\frac{\sum_{i=1}^m w_l^{(i)}(x^{(i)}-\mu_l)(x^{(i)}-\mu_l)^T}{\sum_{i=1}^m w_l^{(i)}}.
\end{aligned}
$$

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)

