---
mathjax: true
title: 生成学习算法
date: 2020-01-01 16:44:38
tags: [cs229, 机器学习]
categories: 机器学习
---

试图直接学习 $p(y|x)$ 的算法（比如：逻辑回归）或者直接学习从输入到标签的映射的算法（比如：感知器算法）这些算法被称作判别式学习方法（discriminative learning algorithms）。那些对 $p(x|y)$ 和 $p(y)$ 建模的算法被称作生成学习方法（generative learning algorithms）。

例如：如果 $y$ 表示一个样本是小狗（$y = 0$）还是大象（$y = 1$），那么 $p(x | y = 0)$ 建立小狗特征的模型，$p(x | y = 1)$ 建立大象特征的模型。

对 $p(y)$ （class priors）和 $p(x | y)$ 建模之后可以通过贝叶斯公式（Bayes rule）来得到在 $x$ 条件下的 $y$ 的后验概率分布（posterior distribution）：

$$
p(y|x) = \frac{p(x|y) p(y)}{p(x)}
$$

<!-- more -->

实际上，如果计算 $p(y|x)$ 是用来进行预测的时候，不需要计算分母 $p(x)$，因为：

$$
\begin{aligned}
\arg\max_y p(y|x) &= \arg \max_y \frac{p(x|y) p(y)}{p(x)} \\
&= \arg \max_y p(x|y) p(y)
\end{aligned}
$$

### 高斯判别分析（Gaussian discriminant analysis）

高斯判别方法属于生成学习方法，假设 $p(x|y)$ 是一个多元正态分布。

#### 多元正态分布（The multivariate normal distribution）

$n$ 维正态分布的参数由一个均值向量（mean vector） $\mu \in \mathbb{R}^n$ 和协方差矩阵（covariance matrix） $\Sigma \in \mathbb{R}^{n\times n}$ 表示，这里 $\Sigma$ 是一个对阵矩阵并且是半正定矩阵。$\mathcal{N}(\mu, \Sigma)$ 的概率密度函数为：

$$
p(x;\mu,\Sigma)=\frac{1}{(2\pi)^{n/2}|\Sigma|^{1/2}} \exp(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu))
$$

其中 $|\Sigma|$ 表示行列式。向量类型的随机变量的协方差为 $Cov(Z) = E[(Z - E[Z])(Z - E[Z])^T]$。如果 $X \sim \mathcal{N}(\mu, \Sigma)$ 那么：

$$
Cov(X) = \Sigma
$$

当 $\mu = 0$ 和 $\Sigma = I$ 时，正态分布被称为标准正态分布。

根据 $\Sigma$ 的不同，正态分布的形状也会发生变化。

#### 高斯判别分析模型

对于一个分类问题，如果输入变量为连续随机变量，就可以使用高斯分析（GDA）模型，这里 $p(x|y)$ 使用多元正态分布。

$$
\begin{aligned}
    y &\sim \text{Bernoulli}(\phi) \\
    x | y = 0 &\sim \mathcal{N}(\mu_0, \Sigma) \\
    x | y = 1 &\sim \mathcal{N}(\mu_1, \Sigma) \\
\end{aligned}
$$

这里使用了同一个协方差矩阵 $\Sigma$。

$$
\begin{aligned}
p(y) & =\phi^y (1-\phi)^{1-y}\\
p(x|y=0) & = \frac{1}{(2\pi)^{n/2}|\Sigma|^{1/2}} \exp ( - \frac{1}{2}(x-\mu_0)^T\Sigma^{-1}(x-\mu_0)  )\\
p(x|y=1) & = \frac{1}{(2\pi)^{n/2}|\Sigma|^{1/2}} \exp ( - \frac{1}{2}(x-\mu_1)^T\Sigma^{-1}(x-\mu_1)  )\\
\end{aligned}
$$

得到模型的似然函数：

$$
\begin{aligned}
l(\phi,\mu_0,\mu_1,\Sigma) &= \log \prod^m_{i=1}p(x^{(i)},y^{(i)};\phi,\mu_0,\mu_1,\Sigma)\\
&= \log \prod^m_{i=1}p(x^{(i)}|y^{(i)};\mu_0,\mu_1,\Sigma)p(y^{(i)};\phi)\\
\end{aligned}
$$

化简可以得到：

$$
\begin{aligned}
    l(\phi,  \mu_0, \mu_1, \Sigma)
    &= \sum_{i=1}^{m} \log p(x^{(i)}|y^{(i)}; \mu_0, \mu_1, \Sigma) + \sum_{i=1}^{m}\log p(y^{(i)};\phi) \\
    &= \sum_{i=1}^{m}\left( (-\frac{1}{2}(x^{(i)}-\mu_{y^{(i)}})^T\Sigma^{-1}(x^{(i)}-\mu_{y^{(i)}})) - \frac 12 \log |\Sigma| +  \log \frac{1}{(2\pi)^{n/2}}\right) \\& \qquad + \sum_{i=1}^{m}y^{(i)} \log \phi + (1 - y^{(i)})\log (1 - \phi)
\end{aligned}
$$

下面只考虑对 $\Sigma$ 求导的情况。

首先有下面两个公式：

$$
\begin{aligned}
  \frac{\partial x^TA^{-1}x}{\partial A} &= -A^{-T} x x^TA^{-T} \\
\frac{\partial \log |A| }{\partial A} &= A^{-T} \\
\end{aligned}
$$

令 $a = x^{(i)}-\mu_{y^{(i)}}$ 然后求导得到：

$$
\begin{aligned}
    \Sigma^{-T} - \Sigma^{-T} a a^T\Sigma^{-T} &= 0 \\
    \Sigma = \Sigma^T &= aa^T
\end{aligned}
$$

于是就得到了：

$$
\Sigma = \frac{1}{m} \sum_{i=1}^{m} (x^{(i)}-\mu_{y^{(i)}})(x^{(i)}-\mu_{y^{(i)}})^T
$$

然后分别对 $\phi$，$\mu_0$ 和 $\mu_1$ 求偏导数，令其等 $0$ 得到：

$$
\begin{aligned}
    \phi &= \frac{1}{m} \sum_{i=1}^{m} 1\{ y^{(i)} = 1 \} \\
    \mu_0 &= \frac{\sum_{i=1}^{m} 1\{ y^{(i)} = 0 \}x^{(i)}}{\sum_{i=1}^{m} 1\{ y^{(i)} = 0 \}} \\
    \mu_1 &= \frac{\sum_{i=1}^{m} 1\{ y^{(i)} = 1 \}x^{(i)}}{\sum_{i=1}^{m} 1\{ y^{(i)} = 1 \}}
\end{aligned}
$$

### 朴素贝叶斯方法（Naive Bayes）

如果要判断邮件是否为一个垃圾邮件，可以建立一个词汇表，统计一个邮件中这些词有没有出现，比如：

$$
x=\begin{bmatrix}1\\0\\0\\\vdots \\1\\ \vdots \\0\end{bmatrix} \begin{matrix}\text{a}\\ \text{aardvark}\\ \text{aardwolf}\\ \vdots\\ \text{buy}\\ \vdots\\ \text{zygmurgy}\\ \end{matrix}
$$

现在如果想要建立一个生成模型，必须对 $p(x|y)$ 进行建模，但是如果词汇表中有 $50000$ 个单词，然后 $x \in \{0, 1\}^{50000}$。现在就需要建立一个 $2^{50000} - 1$ 维的参数向量，显然这太多了。

为了对 $p(x|y)$ 进行建模，需要做一个非常强的假设。假设如果给定 $y$ 那么 $x_i$ 都是不相关的，这个假设就是朴素贝叶斯假设（Naive Bayes (NB) assumption），衍生出来的算法叫做朴素贝叶斯分类器（Naive Bayes classifier）。这里不是说 $x_i$ 不相关，而是条件不相关（conditionally independent）。

于是有：

$$
\begin{aligned}
p(x|y) &= p(x_1, ..., x_{50000}|y) \\
& = p(x_1|y)p(x_2|y,x_1)p(x_3|y,x_1,x_2) ... p(x_{50000}|y,x_1,x_2,...,x_{49999})\\
& = p(x_1|y)p(x_2|y)p(x_3|y) ... p(x_{50000}|y)\\
& = \prod^n_{i=1}p(x_i|y)\\
\end{aligned}
$$

模型的参数为 $\phi_{i|y=1} = p (x_i = 1|y = 1)$ ，$\phi_{i|y=0} = p (x_i = 1|y = 0)$ 和 $\phi_y = p (y = 1)$，然后可以写出联合似然函数：

$$
\begin{aligned}
    \mathcal{L}(\phi_y, \phi_{j | y = 0}, \phi_{j | y = 1}) &= \prod^m_{i=1}p(x^{(i)},y^{(i)}) \\
    &= \prod^m_{i=1}p(x^{(i)}|y^{(i)}) p(y^{(i)}) \\
    &= \prod^m_{i=1}p(y^{(i)}) \prod^n_{j=1}p(x^{(i)}_j|y^{(i)})
\end{aligned}
$$

> 利用下面的公式对似然函数进行化简
> $$
> \begin{aligned}
> p(y^{(i)}) &= \phi_y^{y^{(i)}} (1-\phi_y)^{1-y^{(i)}} \\
> \log p(y^{(i)}) &= y^{(i)} \log \phi_y + (1-y^{(i)}) \log (1-\phi_y) \\
> p(x|y^{(i)}) &= \prod^n_{j=1}p(x_j|y) = \prod^n_{j=1}(\phi_{j|y=y^{(i)}})^{x_j}(1-\phi_{j|y=y^{(i)}})^{1-x_j} \\
> \log p(x^{(i)}|y^{(i)}) &= \sum^n_{j=1} x^{(i)}_j \log (\phi_{j|y=y^{(i)}}) + (1-x^{(i)}_j) \log (1-\phi_{j|y=y^{(i)}}) \\
> \end{aligned}
> $$


$$
\begin{aligned}
    &\mathcal{l}(\phi_y, \phi_{j | y = 0}, \phi_{j | y = 1}) \\
    =& \sum^m_{i=1}\left(\log p(y^{(i)}) + \sum^n_{j=1}\log p(x^{(i)}_j|y^{(i)}) \right) \\
    =& \sum^m_{i=1}\left( y^{(i)} \log \phi_y + (1-y^{(i)}) \log (1-\phi_y)+ \sum^n_{j=1} x^{(i)}_j \log (\phi_{j|y=y^{(i)}}) + (1-x^{(i)}_j) \log (1-\phi_{j|y=y^{(i)}}) \right) \\
\end{aligned}
$$

对 $\phi_{j | y = 0}$ 求偏导数，得到：

$$
\begin{aligned}
\frac{\partial }{\partial \phi_{j | y = 0}} \mathcal{l}(\phi_y, \phi_{j | y = 0}, \phi_{j | y = 1})
&= \sum_{i=1}^{m} 1\{ y^{(i)} = 0\} (\frac {x^{(i)}_j} {\phi_{j|y=0}} + \frac{(1-x^{(i)}_j) } {(1-\phi_{j|y=0})}) \\
&= \sum_{i=1}^{m} 1\{ y^{(i)} = 0\} \frac{x^{(i)}_j - \phi_{j|y=0} } {\phi_{j|y=0}(1-\phi_{j|y=0})}
\end{aligned}
$$

令其偏导数等于 $0$ 得到：

$$
\phi_{j|y=0} = \frac{\sum^m_{i=1}1\{x_j^{(i)} =1 \wedge y^{(i)} =0\} }{\sum^m_{i=1}1\{y^{(i)} =0\}}
$$

同理可以得出：

$$
\begin{aligned}
\phi_{j|y=1} &=\frac{\sum^m_{i=1}1\{x_j^{(i)} =1 \wedge y^{(i)} =1\} }{\sum^m_{i=1}1\{y^{(i)} =1\}} \\
\phi_{y} &= \frac{\sum^m_{i=1}1\{y^{(i)} =1\}}{m} \\
\end{aligned}
$$

拟合好所有参数之后就可以对新样本进行预测，只需要计算：

$$
\begin{aligned}
p(y=1|x)&=  \frac{p(x|y=1)p(y=1)}{p(x)}\\
&= \frac{(\prod^n_{i=1}p(x_i|y=1))p(y=1)}{(\prod^n_{i=1}p(x_i|y=1))p(y=1)+  (\prod^n_{i=1}p(x_i|y=0))p(y=0)}  \\
\end{aligned}
$$

#### 拉普拉斯平滑（Laplace smoothing）

如果一个词汇表中的单词（假设是 $x_{35000}$ ）在之前的训练集中都没有出现过，根据最大似然估计得到的参数就是，

$$
\begin{aligned}
\phi_{35000|y=1} &=  \frac{\sum^m_{i=1}1\{x^{(i)}_{35000}=1 \wedge y^{(i)}=1  \}}{\sum^m_{i=1}1\{y^{(i)}=0\}}  &=0 \\
\phi_{35000|y=0} &=  \frac{\sum^m_{i=1}1\{x^{(i)}_{35000}=1 \wedge y^{(i)}=0  \}}{\sum^m_{i=1}1\{y^{(i)}=0\}}  &=0 \\
\end{aligned}
$$

计算出来的后验概率就是：

$$
\begin{aligned}
p(y=1|x) &= \frac{ \prod^n_{i=1} p(x_i|y=1)p(y=1) }   {\prod^n_{i=1} p(x_i|y=1)p(y=1) +\prod^n_{i=1} p(x_i|y=1)p(y=0)    }\\
&= \frac00\\
\end{aligned}
$$

因为包含了 $p(x_{35000} | y) = 0$ 所以概率变成了 $0/0$ 模型不知道怎么去预测。

把这个问题推广可以取 $\{1, \dots, k\}$ 的多项随机变量 $z$，模型的参数可以是 $\phi_j = p(z = j)$，如果给出 $n$ 个独立观测值 $\{z^{(1)}, \dots, z^{(n)}\}$，利用最大似然估计可以得到：

$$
\phi_j = \frac{\sum^n_{i=1}1\{z^{(i)}=j\}}n
$$

就和前面问题一样，这里的 $\phi$ 可能会等于零，为了避免这种情况，可以使用拉普拉斯平滑（Laplace smoothing），将上面的估计结果变为：

$$
\phi_j=\frac{\sum^n_{i=1}1\{z^{(i)}=j\}+1}{k+n}
$$

这里的 $\phi_j$ 加起来也是 $1$，并且对于所有 $j$，$\phi_j$ 都不等于 $0$。

回到原来的朴素贝叶斯分类器，利用拉普拉斯平滑可以参数的估计可以变为：

$$
\begin{aligned}
\phi_{j|y=1} & =\frac{1+\sum^m_{i=1}1\{x_j^{(i)}=1\wedge y ^{(i)}=1\}}{2+\sum^m_{i=1}1{\{y^{(i)}=1\}}}\\
\phi_{j|y=0} & =\frac{1+\sum^m_{i=1}1\{x_j^{(i)}=1\wedge y ^{(i)}=0\}}{2+\sum^m_{i=1}1{\{y^{(i)}=0\}}}\\
\end{aligned}
$$

> 在实际应用中，通常是否对$\phi_y$ 使用拉普拉斯并没有太大影响，因为通常我们会对每个垃圾邮件和非垃圾邮件都有一个合适的划分比例，所以$\phi_y$ 会是对$p(y = 1)$ 的一个合理估计，无论如何都会与零点有一定距离。

#### 针对文本分类的事件模型（Event models for text classification）

上面使用的模型被叫做伯努利事件模型（Bernoulli event model），在这个模型中，邮件生成的方式这样的：

1. 根据先验概率 $p(y)$，确定这个邮件是不是垃圾邮件。
2. 根据概率 $p(x_j = 1| y) = \phi_{j|y}$，确定词汇表中哪些单词要选。

这样一封邮件的概率就是 $p(y)\prod^n_{i=1}p(x_i|y)$

除此之外，还有另外一种模型，它被称作多项事件模型（Multinomial event model）。为了描述这个算法，令 $x_j$ 表示邮件中第 $j$ 个单词的身份或 ID（identity），这里 $x_j \in \{1, \dots, |V|\}$，$|V|$ 表示词汇表的容量。一个邮件可以表示成长度为 $d$ 的向量 $(x_1, x_2, \dots, x_d)$ （注意：每个邮件的 $d$ 可能不同）。例如：“A NeurIPS . . .”（如果 a 在词汇表的第 $1$ 位，NeurIPS 在第 $35000$ 位）可以表示成 $(1, 35000, \dots)$。

在这个模型中，邮件生成的方式是：

1. 根据先验概率 $p(y)$，确定这个邮件是不是垃圾邮件。
2. 根据同一个多项分布的概率 $p(x_j|y)$ 依次选出 $x_1, x_2, \dots, x_d$，其中 $x_i$ 独立同分布。

这样一封邮件的概率就是 $p(y)\prod^d_{i=1}p(x_i|y)$，这个与上面的十分类似，但是，这里的 $x_j|y$ 是一个多项分布而不是伯努利分布。

如果给定一个训练集$\{(x^{(i)},y^{(i)}); i = 1, ..., m\}$，其中 $x^{(i)}  = ( x^{(i)}_{1} , x^{(i)}_{2} ,..., x^{(i)}_{d_i})$（这里的 $d_i$ 是在第$i$个训练样本中的单词数目），那么这个数据的似然函数如下所示：

$$
\begin{aligned}
\mathcal{L}(\phi,\phi_{k|y=0},\phi_{k|y=1})& = \prod^m_{i=1}p( x^{(i)},y^{(i)}) \\
& = \prod^m_{i=1}(\prod^{d_i}_{j=1}p(x_j^{(i)}|y;\phi_{k|y=0},\phi_{k|y=1}))p( y^{(i)};\phi_y) \\
\end{aligned}
$$

让上面的这个函数最大化就可以产生对参数的最大似然估计：

$$
\begin{aligned}
\phi_{k|y=1} &= \frac{\sum^m_{i=1}\sum^{d_i}_{j=1}1\{x_j^{(i)}=k\wedge y^{(i)}=1\}}{\sum^m_{i=1}1\{y^{(i)}=1\}d_i} \\
\phi_{k|y=0} &= \frac{\sum^m_{i=1}\sum^{d_i}_{j=1}1\{x_j^{(i)}=k\wedge y^{(i)}=0\}}{\sum^m_{i=1}1\{y^{(i)}=0\}d_i} \\
\phi_y&=   \frac{\sum^m_{i=1}1\{y^{(i)}=1\}}{m} \\
\end{aligned}
$$

如果使用拉普拉斯平滑（实践中会用这个方法来提高性能）来估计$\phi_{k|y=0}$ 和 $\phi_{k|y=1}$，就在分子上加1，然后分母上加$|V|$，就得到了下面的等式：

$$
\begin{aligned}
\phi_{k|y=1} &= \frac{\sum^m_{i=1}\sum^{d_i}_{j=1}1\{x_j^{(i)}=k\wedge y^{(i)}=1\}+1}{\sum^m_{i=1}1\{y^{(i)}=1\}d_i+|V|} \\
\phi_{k|y=0} &= \frac{\sum^m_{i=1}\sum^{d_i}_{j=1}1\{x_j^{(i)}=k\wedge y^{(i)}=0\}+1}{\sum^m_{i=1}1\{y^{(i)}=0\}d_i+|V|} \\
\end{aligned}
$$

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)






