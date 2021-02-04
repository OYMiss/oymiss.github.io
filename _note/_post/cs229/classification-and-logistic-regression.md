---
mathjax: true
title: 分类和逻辑回归
date: 2019-12-28 19:00:32
tags: [cs229, 机器学习]
categories: 机器学习
---

> 分类问题

和回归问题一样，只不过预测的值是离散的。

对于二元分类，$1$ 代表正类别（positive class），$0$ 代表负类别（negative class）。

给出一个 $x^{(i)}$ 其对应的 $y^{(i)}$ 叫做标签（label）。

<!--more-->


### 逻辑回归

已知 $y\in \{0, 1\}$，那么如果用连续方法去做，那么 $h(x)$ 大于零或者小于一都是没用的。为了更加有效，有下面的函数：
$$
h_{\theta}(x) = g(\theta^Tx) = \frac{1}{1+e^{-\theta^Tx}}
$$
其中 $g(z)$ 叫做逻辑函数或者 sigmoid​ 函数。
$$
g(z) = \frac{1}{1+e^{-z}}
$$
$g(z)$ 的导数有：
$$
g'(z) = g(z)(1-g(z))
$$
利用最大似然估计进行对 $\theta$ 的估计。先假设：
$$
\begin{aligned}
P(y = 1 | x; \theta) &= h_{\theta}(x) \\
P(y = 0 | x; \theta) &= 1 - h_{\theta}(x)
\end{aligned}
$$
于是有：
$$
p(y | x; \theta) = (h_{\theta}(x))^y(1-h_{\theta}(x))^{1-y}
$$
然后得到似然函数：
$$
\begin{aligned}
L(\theta) &= p({\bf y} | {\bf X}; \theta) \\
&= \prod_{i=1}^{m} p(y^{(i)} | x^{(i)}; \theta) \\
&= \prod_{i=1}^{m} (h_{\theta}(x))^{y^{(i)}}(1-h_{\theta}(x))^{1- y^{(i)}}
\end{aligned}
$$
取对数得到：
$$
\begin{aligned}
l(\theta) &= \log L(\theta) \\
&= \sum_{i=1}^{m}y^{(i)} \log h(x^{(i)}) + (1- y^{(i)})\log (1-h(x^{(i)}))
\end{aligned}
$$
对于一个训练数据 $(x, y)$，求出 $l(\theta) = y \log h(x) + (1-y) \log(1-h(x))$ 的梯度，为了最大化似然函数，接下来进行梯度上升（gradient ascent ）。
$$
\begin{aligned}
\frac{\partial}{\partial \theta_j} l(\theta) &= \left( y \frac{1}{g(\theta^Tx)} - (1-y) \frac{1}{1-g(\theta^Tx)} \right) \frac{\partial}{\partial\theta_j} g(\theta^Tx) \\
&= \left( y \frac{1}{g(\theta^Tx)} - (1-y) \frac{1}{1-g(\theta^Tx)} \right) g(\theta^Tx)(1-g(\theta^Tx))\frac{\partial}{\partial\theta_j} \theta^Tx \\
&= (y - h_{\theta}(x)) x_j
\end{aligned}
$$
与上面线性规划的结果是非常类似的，但是这里的 $h_{\theta}(x)$ 是一个非线性函数。
$$
\theta_j = \theta_j + \alpha(y^{(i)} - h_{\theta}(x^{(i)}))x_j^{(i)}
$$
如果把 $g(z)$ 变成阶跃函数，继续使用上面的更新规则，就得到了感知器学习算法（perceptron learning algorithm）。

### 另一种最大化 $l(\theta)$ 的方法

> 牛顿迭代法

如果要求一个函数 $f(\theta)$ 的零点，可以用牛顿迭代法进行计算，更新方法是：

$$
\theta = \theta - \frac{f(\theta)}{f'(\theta)}
$$

如果要求一个函数的极大值，可以求 $f'(\theta) = 0$ 的点。

更新法则中将 $f(\theta)$ 换成 $l'(\theta)$。

当 $\theta$ 是一个向量时，更新方法变为：
$$
\theta = \theta - H^{-1}\nabla_{\theta} l(\theta)
$$
其中 $H$ 表示海森矩阵，
$$
H_{ij} = \frac{\partial^2l(\theta)}{\partial\theta_i\partial \theta_j}
$$
根据海森矩阵可以判断出 $l(\theta)$ 在 $l'(\theta)=0$ 处取得极大值，这种利用牛顿方法最大化似然函数的方法叫做 Fisher scoring。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
