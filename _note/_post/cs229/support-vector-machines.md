---
mathjax: true
title: 支持向量机
date: 2020-01-04 15:49:37
tags: [cs229, 机器学习]
categories: 机器学习
---

### 核方法（Kernel Methods）

#### 特征图（Feature maps）

在线性回归中，$y$ 是关于 $x$ 的线性函数，如果 $y$ 表示成关于 $x$ 的非线性函数会怎么样？这时候需要比线性模型更复杂的模型。

考虑拟合一个三次函数 $y = \theta_3x^3 + \theta_2x^2 + \theta_1x + \theta_0$，可以把这个三次函数看成是不同特征变量的线性函数，具体做法是，定义 $\phi: \mathbb{R} \to \mathbb{R}^4$ 如下：

$$
\phi(x) = \begin{bmatrix}
    1 \\
    x \\
    x^2 \\
    x^3
\end{bmatrix} \in \mathbb{R}^4
$$

定义一个包含 $\theta_0, \theta_1, \theta_2, \theta_3$ 的参数变量 $\theta$，然后可以把三次函数写成：

$$
y = \theta_3x^3 + \theta_2x^2 + \theta_1x + \theta_0 = \theta^T \phi(x)
$$

为了区分这两种变量，把原始的输入值 $x$ 叫做属性（attributes），当原始的输入被映射到新量值（quantities）$\phi(x)$ 的时候，这个新的量值叫做特征变量，并且把 $\phi$ 叫做特征图（Feature map），它将属性（attributes）映射到特征（features）。

<!-- more -->

#### 带有特征的最小二乘法（LMS with features）

原始的最小二乘法问题要求拟合 $\theta^T x$，得到的批量梯度下降更新为：

$$
\theta = \theta + \alpha \sum_{i=1}^{n}(y^{(i)} - \theta^Tx^{(i)})x^{(i)}
$$

令 $\phi: \mathbb{R}^b \to \mathbb{R}^p$ 是将 $x$ 映射到 $\phi(x)$ 的特征图 （这里 $d = 1$，$p = 4$），然后可以得到新的更新公式：

$$
\theta = \theta + \alpha \sum_{i=1}^{n}(y^{(i)} - \theta^T\phi(x^{(i)}))\phi(x^{(i)})
$$

类似的，随机梯度下降的更新公式是：

$$
\theta = \theta + \alpha (y^{(i)} - \theta^T\phi(x^{(i)}))\phi(x^{(i)})
$$

#### 带有核方法的最小二乘法（LMS with the kernel trick）

特征 $\phi(x)$ 有可能会变得特别大，让 $\phi(x)$ 包含所有关于 $x$ 的 $\le 3$ 次的单项式，

$$
\phi(x) = \begin{bmatrix}
    1 \\
    x_1 \\
    x_2 \\
    \vdots \\
    x_1^2 \\
    x_1 x_2 \\
    \vdots \\
    x_1^3 \\
    x_1^2x_2 \\
    \vdots
\end{bmatrix}
$$

上面的例子中，假设 $x$ 的维度是 $d$，那么 $\phi(x)$ 的维度是 $d^3$ 数量级上的，如果 $d = 1000$，那么 $1000^3 = 10^9$，这样的计算是非常困难的。

为了方便，将 $\theta$ 初始化为 $0$，然后将注意力集中在每次的迭代更新。一个主要的发现就是，$\theta$ 可以表示成向量 $\phi(x^{(1)}), \dots, \phi(x^{(n)})$ 的线性组合。

首先，对于 $\theta = 0$ 可以表示成 $\theta = \sum_{i=1}^{n}0 \cdot \phi(x^{(i)})$。

假设 $\theta$ 可以表示成：

$\theta = \sum_{i=1}^{n}\beta_i \phi(x^{(i)})$

那么下一轮的更新之后，$\theta$ 依然是 $\phi(x^{(1)}), \dots, \phi(x^{(n)})$ 的线性组合，

$$
\begin{aligned}
\theta &= \theta + \alpha \sum_{i=1}^{n}(y^{(i)} - \theta^T\phi(x^{(i)}))\phi(x^{(i)}) \\
&= \sum_{i=1}^{n}\beta_i \phi(x^{(i)}) + \alpha \sum_{i=1}^{n}(y^{(i)} - \theta^T\phi(x^{(i)}))\phi(x^{(i)}) \\
&= \sum_{i=1}^{n}\underbrace{(\beta_i + \alpha(y^{(i)} - \theta^T\phi(x^{(i)})))}_{\text{new } \beta_i} \phi(x^{(i)})
\end{aligned}
$$

新的 $\beta_i$ 和旧的通过 $\beta_i = \beta_i + \alpha(y^{(i)} - \theta^T\phi(x^{(i)}))$ 建立起关系。

将 $\theta = \sum_{i=1}^{n}\beta_i \phi(x^{(i)})$ 带入方程里面得到：

$$
\beta_i = \beta_i + \alpha(y^{(i)} - \sum_{i=j}^{n}\beta_j \phi(x^{(j)})^T  \phi(x^{(i)}))
$$

经常把 $\phi(x^{(j)})^T  \phi(x^{(i)})$ 写成 $\langle  \phi(x^{(j)}), \phi(x^{(i)}) \rangle$，通过把 $\beta_i$ 写成新的形式，原来的梯度下降算法变成了一个迭代更新 $\beta$ 的算法。仍然需要计算计算 $\langle  \phi(x^{(j)}), \phi(x^{(i)}) \rangle$ 的值，过程也也是 $O(p)$ 的。但是还有两条重要的性质：

1. 在循环之前可以提前计算 $\langle  \phi(x^{(j)}), \phi(x^{(i)}) \rangle$ 所有组合的结果。
2. 计算 $\langle  \phi(x^{(j)}), \phi(x^{(i)}) \rangle$ 是效率高的，因为：

$$
\begin{aligned}
  \langle  \phi(x), \phi(z) \rangle &= 1 + \sum_{i = 1}^{d} x_i z_i  + \sum_{i, j \in \{1, \dots, d\}} x_i x_j z_i z_j + \sum_{i, j, k \in \{1, \dots, d\}} x_ix_jx_k z_iz_jz_k \\
  &= 1 + \sum_{i = 1}^{d} x_i z_i + \left( \sum_{i = 1}^{d} x_i z_i \right)^2 + \left( \sum_{i = 1}^{d} x_i z_i \right)^3 \\
  &= 1 + \langle x, z \rangle + \langle x, z \rangle^2 + \langle x, z \rangle^3
\end{aligned}
$$

只需要在 $O(d)$ 事件内，计算出 $\langle x, z \rangle$ 就可以用常数时间得到 $\langle  \phi(x), \phi(z) \rangle$。

可以看出来，特征之间的内积很重要。定义一个 $\mathcal{X} \times \mathcal{X} \to \mathbb{R}$ 核函数（Kernel）满足：

$$
K(x, z) \overset{def}{=} \langle  \phi(x), \phi(z) \rangle
$$

总结一下这个算法的步骤：

1. 计算出所有 $K(x^{(i)}, x^{(j)})$，令 $\beta = 0$。
2. 循环：
   $$
   \beta = \beta - \alpha(y - K\beta)
   $$

然后就可以根据 $\beta$ 进行预测，

$$
\beta^T\phi(x) = \sum_{i=1}^{n} \beta_i K(x^{(i)}, x)
$$

#### 核函数的性质

什么样的核函数有与之对应的特征图 $\phi$，或者说，是否有一个特征映射函数 $\phi$ 使得 $K(x, z) = \phi(x)^T\phi(z)$ 对于所有 $x$ 和 $z$。

考虑下面的定义的核函数：

$$
K(x, z) = (x^Tz)^2
$$

可以写成

$$
\begin{aligned}
    K(x, z) &= \left( \sum_{i=1}^{d}x_i z_i \right) \left( \sum_{j=1}^{d}x_j z_j \right) \\
    &= \sum_{i=1}^{d} \sum_{j=1}^{d}x_ix_jz_iz_j \\
    &= \sum_{i,j = 1}^{d} (x_ix_j) (z_iz_j)
\end{aligned}
$$

可以得到对应的特征映射函数 $\phi$：

$$
\phi(x) = \begin{bmatrix}
    x_1x_1 \\
    x_1x_2 \\
    \vdots \\
    x_3x_2 \\
    x_3x_3 \\
\end{bmatrix}
$$

可以看出来计算高维度 $\phi(x)$ 需要 $O(d^2)$ 的时间，但是计算 $K(x,z)$ 只需要 $O(d)$ 的时间。


然后对于 $K(x, z) = (x^Tz + c)^2$ 也可以写成：


$$
\begin{aligned}
K(x, z) &= (x^Tz + c)^2 \\
&= \sum_{i, j = 1}^{d}(x_ix_j)(z_iz_j) + \sum_{i = 1}^{d}(\sqrt{2c} x_i)(\sqrt{2c} x_i) + c^2
\end{aligned}
$$

然后可以得到：

$$
\phi(x) = \begin{bmatrix}
    x_1x_1 \\
    x_1x_2 \\
    \vdots \\
    x_3x_2 \\
    x_3x_3 \\
    \sqrt{2c} x_1 \\
    \sqrt{2c} x_2 \\
    \sqrt{2c} x_3 \\
    c
\end{bmatrix}
$$

> 核用来表示相似程度。

对于 $K(x, z) = \phi(x)^T\phi(z)$，如果两个变量相距很远，那么 $K(x,z)$ 会变得非常小，所以我们可以把核函数当成衡量的 $\phi(x)$ 和 $\phi(z)$ 的相似程度的函数。

既然这样，如果取下面的函数会发生什么？

$$
K(x, z) = \exp \left( -\frac{\|x - z\|^2}{2\sigma^2} \right)
$$

如果核函数选成这个，那么是否存在 $\phi(x)$ 满足 $K(x, z) = \phi(x)^T \phi(z)$，答案是存在。并且这个核叫做高斯核（Gaussian kernel）。

##### 有效核函数的条件

有效核函数的必要条件：如果核函数是有效的，那么核矩阵（kernel matrix）有一些性质：

1. 核矩阵是对称矩阵。
2. 核矩阵是半正定的（positive semi-definite）。

下面证明核矩阵是正定的，对于任意 $z$：

$$
\begin{aligned}
    z^T K z
    &= \sum_i \sum_jz_i K_{ij} z_j \\
    &= \sum_i \sum_jz_i \phi(x^{(i)})^T \phi(x^{(j)}) z_j \\
    &= \sum_i \sum_j \sum_k z_i \phi_k(x^{(i)}) \phi_k(x^{(j)}) z_j \\
    &= \sum_k \left( \sum_i z_i \phi_k(x^{(i)}) \right)^2 \\
    &\ge 0
\end{aligned}
$$

> 如果核函数是有效的，那它对应的核矩阵是对阵半正定矩阵的（symmetric positive semidefinite）。

上面的条件不仅仅是必要条件而且是充分条件。

Mercer 定理：给出 $K:\mathbb{R}^d \times \mathbb{R}^d \to \mathbb{R}$，$K$ 是有效核函数的充分必要条件是对于所有 $x^{(1)},\dots,x^{(n)}$ 所对应的核矩阵是对阵半正定矩阵的。

### 支持向量机（Support Vector Machines）

#### 间隔（Margins）：直观解释

<img src="/asset/support-vector-machines/margins1.png" width = "60%" height = "60%" alt="margins1"/>

间隔可以解释成对数据分类成功的信心。上图中的直线，被称作分割平面（separating hyperplane），$C$ 点非常接近于分割边界，相比 $C$ 来说，对 $A$ 的分类成功的信心比 $C$ 大，$B$ 介于两个之间。

#### 记号（Notation）

在 SVM 中，使用 $y \in \{-1, 1\}$ 来表示类别，定义分类器：

$$
h_{w, b} = g(w^Tx + b)
$$

这里如果 $z \ge 0$ 那么 $g(z) = 1$ 否则 $g(z) = -1$。

#### 函数间隔和几何间隔（Functional and geometric margins）

对于训练样本定义 $w$ 和 $b$ 的函数间隔（functional margin）：

$$
\hat{\gamma}^{(i)} = y^{(i)}(w^Tx^{(i) + b})
$$

$y^{(i)}$ 的绝对值越大，$\hat{\gamma}^{(i)}$ 也就越大。如果 $w$ 和 $b$ 都扩大二倍，并不影响 $g(w^Tx + b)$，但是会让函数间隔扩大二倍。

对于一个训练集 $S$，定义 $w$ 和 $b$ 的函数间隔是每个训练样本的函数间隔的最小值，即：

$$
\hat{\gamma} = \min_{i = 1, \dots, m} \hat{\gamma}^{(i)}
$$

接下来考虑几何间隔（geometric margins）。

<img src="/asset/support-vector-machines/margins2.png" width = "60%" height = "60%" alt="margins2"/>

图片中，$w$ 表示于分割复平面（separating hyperplane）正交的。

假设 $A$ 点的标签为 $y^{(i)} = 1$，它距离分割边界的距离 $\gamma^{(i)}$ 就是线段 $AB$ 的距离。

可以用 $x^{(i)} - \gamma^{(i)} \cdot w / {\|w\|}$，这里 $w / {\|w\|}$ 是 $w$ 所在方向的方向向量，又有 $B$ 点在直线 $w^Tx + b = 0$ 上：

$$
w^T(x^{(i)}-\gamma^{(i)}\frac{w}{\parallel w\parallel })+b=0
$$

可以得到：

$$
\gamma^{(i)}=\frac{w^Tx^{(i)}+b}{\| w\| }=\left(\frac{w}{\| w\| }\right)^Tx^{(i)}+\frac{b}{\| w\| }
$$

对于任意一个训练样本，它的几何间隔为：

$$
\gamma^{(i)}=y^{(i)}\left(\left(\frac{w}{\| w\| }\right)^Tx^{(i)}+\frac{b}{\| w\| }\right)
$$

注意到，如果 $\| w\| = 1$ 那么函数间隔就等于几何间隔。对于几何间隔来说，如果把 $w$ 和 $b$ 变为两倍，结果是不变的。

同样的一个训练集的几何间隔也是所有训练样本的几何间隔的最小值。

$$
\gamma = \min_{i = 1, \dots, m} {\gamma}^{(i)}
$$

#### 最优间隔分类器（The optimal margin classifier）

假设数据都是可以被一个复平面分割的，问题就变成最大化训练集的（几何）间隔：

$$
\begin{aligned}
\max_{\gamma,w,b} \quad& \gamma \\
s.t. \quad &y^{(i)}(w^Tx^{(i)}+b) \geq \gamma,\quad i=1,\dots,m\\
&\| w\| =1 \\
\end{aligned}
$$

这里 $\| w\| =1$ 保证边界间隔和函数间隔是相同的，所以可以保证边界间隔至少是 $\gamma$。

这个约束 $\| w\| = 1$ 导致问题不是一个凸的，不容易计算，改变一下：

$$
\begin{aligned}
\max_{\hat\gamma,w,b} \quad& \frac{\hat \gamma}{\| w\| } \\
s.t. \quad &y^{(i)}(w^Tx^{(i)}+b) \geq \gamma,\quad i=1,\dots,m \\
\end{aligned}
$$

这里打算最大化 $\hat \gamma / \|w\|$，对于所有函数间隔都至少为 $\hat \gamma$。因为 $\gamma = \hat \gamma / \|w\|$，这个算法可以给出我们想要的结果，避免了 $\|w\| = 1$。但是，目标函数还不是一个凸函数。

上面说了，任意缩放 $w$ 和 $b$ 不会改变最后的结果。然后我们缩放约束使得训练集的函数间隔变为 $1$：

$$
\hat \gamma = 1
$$

然后最大化 $\hat \gamma / \|w\| = 1 / \|w\|$ 就相当于最小化 $\|w\|^2$，优化问题就可以转化为：

$$
\begin{aligned}
\min_{\gamma,w,b} \quad& \frac{1}{2}\parallel w\parallel ^2 \\
s.t. \quad &y^{(i)}(w^Tx^{(i)}+b) \geq 1,\quad i=1,\dots,m\\
\end{aligned}
$$

现在目标函数转换为一个凸的二次函数，并且只有线性约束，解出这个问题就可以得到最优间隔分类器。

这个是我们要优化的主问题，可以把约束条件写成：

$$
g_i(w)=-y^{(i)}(w^Tx^{(i)}+b)+1\leq 0
$$

根据 KKT 对偶互补条件，只有在测试样本的函数间隔等于 $1$ （也就是 $g_i(w) = 0$ ）的情况下才会有 $a_i > 0$。

<img src="/asset/support-vector-machines/support-vector.png" width = "60%" height = "60%" alt="support-vector"/>

虚线上的点是距离决策边界最近的三个点，这三个点也叫做支持向量（support vectors），也只有这三个样本对应的 $\alpha_i$ 不是零。

构建出来一个拉格朗日函数（Lagrangian）：

$$
\mathcal{L}(w,b,\alpha )=\frac{1}{2}\| w\| ^2 - \sum^m_{i=1}\alpha_i [y^{(i)}(w^Tx^{(i)}+b)-1]
$$

当固定 $\alpha$ 的情况下，求出最小化 $\mathcal{L}$ 情况下的 $w$ 和 $b$，然后就可以得到 $\theta_\mathcal{D}$。

> $$
> \theta_{\mathcal{D}}(\alpha,\beta)=\min_w \mathcal{L}(w,\alpha,\beta)
> $$

分别对 $w$ 和 $b$ 求导，并令其等于 $0$：

$$
\begin{aligned}
\nabla_w \mathcal{L}(w,b,\alpha)&=w-\sum^m_{i=1}\alpha_i y^{(i)}x^{(i)} = 0 \\
\frac{\partial}{\partial b}L(w,b,\alpha)&=\sum^m_{i=1}\alpha_i y^{(i)}=0
\end{aligned}
$$

化简可以可以得到 $w$ 的另一种形式：

$$
\begin{aligned}
w=\sum^m_{i=1}\alpha_i y^{(i)}x^{(i)}
\end{aligned}
$$

$w=\sum^m_{i=1}\alpha_i y^{(i)}x^{(i)}$ 和 $\sum^m_{i=1}\alpha_i y^{(i)} = 0$ 带入拉格朗日函数里面可以得到：

$$
L(w,b,\alpha)=\sum^m_{i=1}\alpha_i-\frac12 \sum^m_{i,j=1} y^{(i)}y^{(j)}\alpha_i\alpha_j(x^{(i)})^Tx^{(j)}
$$

得到下面的对偶优化问题：

$$
\begin{aligned}
\max_\alpha \quad & W(\alpha) =\sum^m_{i=1}\alpha_i-\frac12\sum^m_{i,j=1}y^{(i)}y^{(j)}\alpha_i\alpha_j\langle	x^{(i)} x^{(j)}	\rangle\\
s.t. \quad &  \alpha_i \geq 0,\quad i=1,...,m\\
& \sum^m_{i=1} \alpha_iy^{(i)}=0
\end{aligned}
$$

找到 $\alpha$ 之后，就可以计算出 $w^*$，然后还可以计算出 $b$：

$$
b^\ast =- \frac{\max_{i:y^{(i)}=-1}w^{\ast T}x^{(i)}+\min_{i:y^{(i)}=1}w^{\ast T}x^{(i)}}{2}
$$

#### 正则化和不可分割的情况（Regularization and the non-separable case）

<img src="/asset/support-vector-machines/non-separable-case.png" width = "60%" height = "60%" alt="non-separable-case"/>

一开始就假设两组数据是可以被分割的，但是，对于上图来说，实线并不是最好的分割结果，虚线才是。为了减少极端数据对结果的影响，还需要考虑不可分割的情况。

使用 $l_1$ 正则化进行解决：

$$
\begin{aligned}
\max_\alpha \quad & W(\alpha) =\sum^m_{i=1}\alpha_i-\frac12\sum^m_{i,j=1}y^{(i)}y^{(j)}\alpha_i\alpha_j \langle  x^{(i)},x^{(j)} \rangle \\
 s.t. \quad & 0\leq \alpha_i \leq C,i=1,...,m\\
 & \sum^m_{i=1}\alpha_iy^{(i)}=0  \\
\end{aligned}
$$

#### SMO 算法

SMO（sequential minimal optimization）算法给出了一种解决 SVM 的对偶问题的高效方法。

##### 坐标上升算法（Coordinate ascent）

假设我们要解决下面这个无约束的优化问题：

$$
\max_\alpha W(\alpha_1,\alpha_2,...,\alpha_m)
$$

然后引出坐标上升算法。

$$
\begin{aligned}
&\text{Loop untile convergence: } \{  \\
&\qquad For\quad i=1,...,m, \{ \\
&\qquad\qquad\alpha_i:= \arg \max_{\hat \alpha_i}W(\alpha_1,...,\alpha_{i-1},\hat\alpha_i,\alpha_{i+1},...,\alpha_m) \\
&\qquad\} \\
&\}
\end{aligned}
$$

<img src="/asset/support-vector-machines/coordinate-ascent.png" width = "60%" height = "60%" alt="coordinate-ascent"/>

##### SMO

再写一遍要优化的问题：

$$
\begin{aligned}
\max_\alpha \quad & W(\alpha) =\sum^m_{i=1}\alpha_i-\frac12\sum^m_{i,j=1}y^{(i)}y^{(j)}\alpha_i\alpha_j \langle  x^{(i)},x^{(j)} \rangle \\
 s.t. \quad & 0\leq \alpha_i \leq C,i=1,...,m\\
 & \sum^m_{i=1}\alpha_iy^{(i)}=0  \\
\end{aligned}
$$

如果只改变 $\alpha_i$，然后固定 $\alpha_1, \dots, \alpha_i, \dots, \alpha_m$，这时候利用坐标下降是得不到结果的。假设当前 $i = 1$，则有：

$$
\alpha_1y^{(1)} = - \sum_{i=2}^{m}\alpha_iy^{(i)}
$$

然后左右同时乘以 $y^{(1)}$ 得到：

$$
\alpha_1 = - y^{(1)}  \sum_{i=2}^{m}\alpha_iy^{(i)}
$$

可以看出来，如果 $\alpha_2, \dots, \alpha_m$ 不变，$\alpha_1$ 也不会变。

这里需要选出来一对 $\alpha_i$ 和 $\alpha_j$ 来进行改变，其他的保持不变。

$$
\alpha_1y^{(1)} + \alpha_2y^{(2)} = -\sum^m_{i=3}\alpha_iy^{(i)}
$$

因为我们已经固定了 $\alpha_3, ..., \alpha_m$ 的值，等式右边可以写成常量 $\zeta$。

<img src="/asset/support-vector-machines/constraints-on-alpha.png" width = "60%" height = "60%" alt="constraints-on-alpha"/>

结合上面的约束条件，可以看出，$\alpha_1$ 和 $\alpha_2$ 被限制在图中的直线上。

可以得到 $\alpha_1$ 和 $\alpha_2$ 的关系：

$$
\alpha_1=(\zeta-\alpha_2y^{(2)})y^{(1)}
$$

目标函数其实就是：

$$
W(\alpha_1,\alpha_2,...,\alpha_m)=W((\zeta-\alpha_2y^{(2)})y^{(1),\alpha_2,...,\alpha_m})
$$

这时候 $\alpha_3, ..., \alpha_m$ 都是常量，函数就变成了关于 $\alpha_2$ 的二次函数，很容易可以求出驻点，结果用 $\alpha_2^{new, unclipped}$ 表示。并有约束条件 $L \leq \alpha_2 \leq H$，可以得到优化后的 $\alpha_2$：

$$
\alpha_2^{new}= \begin{cases}  H & \text{if} \quad\alpha_2^{new, unclipped}> H\\ \alpha_2^{new, unclipped} & \text{if} \quad L\leq \alpha_2^{new, unclipped}\leq H\\  L & \text{if} \quad \alpha_2^{new, unclipped} <L\\ \end{cases}
$$

#### 拉格朗日对偶性（Lagrange duality）

考虑下面的优化问题：

$$
\begin{aligned}
\min_w \quad & f(w)& \\
s.t. \quad &h_i(w) =0,\quad i=1,\dots,l\\
\end{aligned}
$$

可以利用拉格朗日乘数法（method of Lagrange multipliers）来解决，先得到拉格朗日函数，然后在对其求偏导，并令其等于 $0$：

$$
L(w,\beta)=f(w)+\sum^l_{i=1}\beta_i h_i(w) \\
\frac{\partial L }{\partial w_i} =0; \quad \frac{\partial L }{\partial \beta_i} =0;
$$

这样就可以求出 $w$ 和 $\beta$。

下面扩展到包含不等式和等式的约束问题。

考虑下面的优化问题，通常被叫做主优化问题（primal optimization problem）：

$$
\begin{aligned}
\min_w \quad & f(w)& \\
s.t. \quad & g_i(w) \le 0,\quad i=1,\dots,k\\
& h_i(w) =0,\quad i=1,\dots,l\\
\end{aligned}
$$

定义一个广义拉格朗日函数：

$$
L(w,\alpha,\beta)=f(w)+\sum^k_{i=1}\alpha_ig_i(w)+\sum^l_{i=1}\beta_ih_i(w)
$$

上面的式子中， $\alpha_i$ 和 $\beta_i$ 都是拉格朗日乘数（Lagrange multipliers）。考虑下面这样一个量（quantity）：

$$
\theta_{\mathcal{P}}(w)=\max_{\alpha,\beta:\alpha_i \geq 0}L(w,\alpha,\beta)
$$

给定一个 $w$，如果不满足约束条件（例如：$g_i(w) > 0$ 或者 $h_i(w) \neq 0$），这个量会变得成无穷大。

相反，如果 $w$ 满足约束条件，就有 $\theta_{\mathcal{P}}(w) = f(w)$。所以有：

$$
\theta_{\mathcal{P}}(w)= \begin{cases} f(w) & \text {if w satisfies primal constraints} \\
\infty & \text{otherwise} \end{cases}
$$

然后可以得到
$$
\min_{w}\theta_{\mathcal{P}}(w) = \min_w \max_{\alpha, \beta:\alpha_i \ge 0} \mathcal{L}(w, \alpha, \beta)
$$

这个问题和原来的问题是一样的。我们求出问题的最优解 $p^* = \min_{w}\theta_{\mathcal{P}}(w)$，也就是主问题的值（value）。

然后再去看它的对偶（dual）问题。

定义：

$$
\theta_{\mathcal{D}}(\alpha,\beta)=\min_w \mathcal{L}(w,\alpha,\beta)
$$

可以得到对偶优化问题：

$$
\max_{\alpha,\beta:\alpha_i\geq 0} \theta_{\mathcal{D}}(\alpha,\beta)  = \max_{\alpha,\beta:\alpha_i\geq 0} \min_w \mathcal{L}(w,\alpha,\beta)
$$

另外再定义，这个问题的最优解为 $d^* = \max_{\alpha,\beta:\alpha_i\geq 0} \theta_{\mathcal{D}}(w)$。

可以得到：

$$
d^* = \max_{\alpha,\beta:\alpha_i\geq 0} \min_w \mathcal{L}(w,\alpha,\beta)\le \min_w \max_{\alpha, \beta:\alpha_i \ge 0} \mathcal{L}(w, \alpha, \beta) = p^*
$$

当满足一定条件的时候 $d^* = d^*$。

假设 $f$ 和 $g$ 是凸函数、 $h_i$ 是仿射的（affine），并且对于约束条件是有解（存在 $w$ 使得对于所有 $i$ 有 $g_i(w) < 0$）。

如果上面的假设就一定存在 $w^*$，$\alpha^*$ 和 $\beta^*$ 使得 $w^*$ 是主问题的解，$\alpha^*$，$\beta^*$ 是对偶问题的解，并且有 $d^* = d^* = \mathcal{L}(w^*, \alpha^*, \beta^*)$。

除了这些，$w^*$，$\alpha^*$ 和 $\beta^*$ 还满足 Karush-Kuhn-Tucker (KKT) 条件：

$$
\begin{aligned}
\frac{\partial}{\partial w_i}\mathcal{L}(w^\ast,\alpha^\ast,\beta^\ast) &= 0,\quad i=1,\dots,d \\
\frac{\partial}{\partial \beta_i}\mathcal{L}(w^\ast,\alpha^\ast,\beta^\ast)&= 0 ,\quad i=1,\dots,l \\
\alpha_i^\ast g_i(w^\ast)&= 0,\quad i=1,\dots,k \\
g_i(w^\ast)&\leq 0,\quad i=1,\dots,k \\
\alpha_i^\ast &\geq 0,\quad i=1,\dots,k \\
\end{aligned}
$$

其中 $\alpha_i^\ast g_i(w^\ast)= 0$ 叫做 KKT 对偶互补（dual complementarity）条件。如果 $\alpha_i^\ast > 0$ 就说明满足的是等式约束而不是不等式。

[1] [CS229 - Machine Learning](https://see.stanford.edu/Course/CS229)

[2] [CS229 课程讲义中文翻译](https://kivy-cn.github.io/Stanford-CS-229-CN/)
