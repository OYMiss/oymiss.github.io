---
mathjax: true
title: 模型选择和正则化
date: 2020-01-15 15:53:44
tags: [cs229, 机器学习]
categories: 机器学习
---

### 交叉验证（Cross validation）

如果一味追求训练误差最小化就会总是选择参数复杂的模型，结果会造成过拟合。利用保留交叉验证（hold-out cross validation）可以这么做：

1. 随机将 $S$ 分为 $S_{\text{train}}$ （大概占 70%）和 $S_{\text{cv}}$（大概 30%），这里 $S_{\text{cv}}$ 被称作保留交叉验证集（hold-out cross validation set）。
2. 只利用 $S_{\text{train}}$ 对 $M_i$ 进行训练，得到假设（hypothesis） $h_i$。
3. 选择在保留交叉验证集上 $\hat\epsilon_{S_{cv}}(h_i)$ 最小的假设 $h_i$ 作为输出。

这个算法的缺点是，它“浪费”了 30% 的数据。只用了 70% 的数据进行训练。

<!-- more -->

还有一个算法叫做 k 重交叉验证（k-fold cross validation），这个算法每次保留很少的数据。

1. 随机将 $S$ 分为 $k$ 个不相交的集合 $S_1, \dots, S_k$，且大小都为 $m/k$。
2. 对于模型 $M_i$，对于每个 $j = 1, \dots, k$，利用除了 $S_j$ 的训练集 $S_i$ 去进行训练，然后将 $k$ 次的 $\hat\epsilon_{S_{j}}(h_{ij})$ 做平均值作为模型的泛化误差。
3. 选取泛化误差最低的模型作为输出。

通常选择 $k = 10$，叫做 10 重交叉验证。但是还有一种方法是选择 $k = m$ 每次只留出来一种数据用来测试，这种方法叫做留一交叉验证（leave-one-out cross validation）。

### 特征选择（Feature Selection）

如果特征很多，但是只有少量的特征与输出有关，那么也有可能造成过拟合。

这种情况可以选择将特征减少，筛选出和结果相关性强的特征。假设现在有 $d$ 个特征，如果要选出最优的特征集合，需要对 $2^d$ 进行判别。

所以这里介绍一种特征选择算法，叫做前向搜索（forward search）：

1. 初始化 $\mathcal{F} = \emptyset$。
2. 重复
   1. 对于 $i = 1, \dots, d$，如果 $i \notin \mathcal{F}$ 令 $\mathcal{F}_i = \mathcal{F} \cup \{i\}$，然后特征为 $\mathcal{F}$ 时的泛化误差。
   2. 将 $\mathcal{F}$ 设为上面 $\mathcal{F}_i$ 中泛化误差最小的。
3. 输出上面过程中泛化误差最小的集合。

也可以选择在所有特征中每次剔除一个，然后计算泛化误差，这种方法叫做反向搜索（backward search）。

除了上面的方法，还可以采用过滤特征选择（filter feature selection）的方法，这种方法通过计算用来衡量 $x_i$ 这个特征为标签 $y$ 的提供的信息的多少的值 $S(i)$，然后选择出来 $S(i)$ 尽量大的 $k$ 个特征。

$S(i)$ 可以是 $x_i$ 与 $y$ 之间的互信息（mutual information）$\text{MI}(x_i, y)$：

$$
{\text{MI}}(x_i, y)=\sum_{x_i\in\{0, 1\}}\sum_{y\in\{0,1\}}p(x_i,y)\log\frac{p(x_i,y)}{p(x_i)p(y)}
$$

这里假设 $x_i$ 和 $y$ 都是 $0$ 或 $1$。上面的互信息可以表示成 Kullback-Leibler 散度（divergence）：

$$
{\text{MI}}(x_i,y)={\text{KL}}(p(x_i,y) \| p(x_i)p(y))
$$

### 贝叶斯统计和正则化（Bayesian statistics and regularization）

下面介绍另一种避免过拟合的工具。

前面已经使用过最大似然法（maximum likelihood estimation 缩写 MLE）进行参数的估计，利用下面的公式：

$$
\theta_{\text{ML}}=\arg \max_{\theta}\prod_{i=1}^{m}p(y^{(i)}|x^{(i)};\theta)
$$

把这个参数 $\theta$ 看作是一个未知的参数，频率派的统计学家认为 $\theta$ 是常量并且未知的。在他们眼中，$\theta$ 不是一个随机变量，只是一个常量。于是工作就变成利用统计方法（例如最大似然估计）去估计这个参数。

另一种参数优化问题的方法是根据贝叶斯派（Bayesian）的观点把 $\theta$ 作为一个随机变量。这里指定一个先验分布（prior distribution）$p(\theta)$ 来表示关于参数的“先验信心”。给一个训练集 $S = \{(x^{(i)},y^{(i)})\}^m_{i=1}$，对于一个新的 $x$ 来说，可以计算参数的后验概率。

$$
\begin{aligned}
p(\theta|S)
&=\frac{p(S|\theta)p(\theta)}{p(S)}\\
&=\frac{(\prod_{i=1}^{m}p(y^{(i)}|x^{(i)},\theta))p(\theta)}{\int_{\theta} {\left(\prod_{i=1}^{m}p(y^{(i)}|x^{(i)},\theta)p(\theta)\right)}d\theta}
\end{aligned}
$$

上面公式中的 $p(y(i)|x(i),\theta)$ 依赖于正在使用什么模型，如果正在使用的是贝叶斯逻辑回归（Bayesian logistic regression），那么 $p(y^{(i)}|x^{(i)},\theta)=h_\theta(x^{(i)})^{y^{(i)}} (1-h_\theta(x^{(i)}))^{(1-y^{(i)})}$ 其中 $h_\theta(x^{(i)})=1/(1+\exp(-\theta^Tx^{(i)}))$。

> 这里的 $\theta$ 是一个随机变量，概率可以写成 $p(y(i)|x(i),\theta)$ 而不是之前的 $p(y(i)|x(i);\theta)$。

当给出一个新的测试样本 $x$，可以利用 $\theta$ 的后验概率去计算标签 $y$ 的后验概率：

$$
p(y|x,S)=\int_\theta p(y|x,\theta)p(\theta|S)d\theta
$$

如果目标是为了求给定 $x$ 的情况下 $y$ 的期望值的话，可以输出：

$$
E[y|x,S]=\int_y y p(y|x,S)dy
$$

上面这个过程被称作“完全贝叶斯（fully Bayesian）”预测，这种方法的不容易计算。

实际上，一般的做法是近似估计 $\theta$ 后验概率，一种常用的做法是用点估计代替后验分布。对于 $\theta$ 的 MAP（maximum a posteriori）估计为：

$$
\theta_{\text{MAP}}=\arg \max_\theta \prod_{i=1}^{m} p(y^{(i)}|x^{(i)})p(\theta)
$$

注意这个式子和 MLE（maximum likelihood）除了后面多了一个 $p(\theta)$ 都一模一样。

在实际的应用中，一个常用的选择时先验概率 $p(\theta)$ 服从 $\theta \sim \mathcal{N}(0, \tau^2 I)$，这比利用最大似然进行选择有更小的范数（norm）。通常情况下，贝叶斯 MAP 估计相对于 ML 估计更不容易过拟合。

> 可以把上面最大化的函数 $\prod_{i=1}^{m} p(y^{(i)}|x^{(i)})p(\theta)$ 取一个对数，$\log p(\theta)$ 也就可以写成 $C\theta$。这个对线性规划的正则化给了直观的解释。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
