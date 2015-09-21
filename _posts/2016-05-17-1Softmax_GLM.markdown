---
layout:     post
title:      "广义线性模型与Softmax"
subtitle:   "一点儿logistic回归——对数几率回归"
date:       2016-05-17 07:25:00 +0800
author:     "Roach"
catalog: true
tags:
    - 机器学习
    - 概率论
---

本文是Andrew NG在网易公开课第四讲的关于Softmax，以及[UFLDL Tutorial](http://ufldl.stanford.edu/wiki/index.php/UFLDL_Tutorial)中Softmax回归的学习笔记和理解。

## 欲解决的问题

逻辑回归是用来处理二分类问题，类标 $y^{(i)} \in \{0,1\}$，而Softmax回归是逻辑回归的扩展：同样将代测样本预测为某一类，但是类别 $y^{(i)} \in \{1,\ldots,K\}$，这里 $K$ 是类别个数。

对数据进行分类时，逻辑回归对应二项分布，softmax回归对应多项分布。

## GLM套路

1. $y \mid x; \theta \sim ExpFamily(\eta)$；
2. Given $X$, goal is a output $E\left[ T\left( y \right) \mid x \right]$, want $h \left( x \right) = E\left[ T\left( y \right) \mid x \right]$；
3. $\eta = \theta^\top x$；

    这么好的东西，似乎不应该说成套路！

    上面列表其实应该有三个补充：

4. $ExpFamily(\eta)=b(y)exp(\eta^TT(y)-a(\eta))$；
5. $y \sim Multinomial(\Phi) \longleftrightarrow ExpFamily(\eta)$；
6. $E\left[ T\left( y \right) \mid x \right]$ is a function of $\Phi$；

我们的目标是得到 $h(x)$，然后通过判断其得到的值，对样本分类。对于softmax，$h(x)$ 输出多个值，样本会分到概率得分最大的类别。

但是通过有序列表的$2$和$6$，知道 $h(x)$ 可以由 $\Phi$ 表示。而我们已知的只有 $\theta$ 和 $x$，也就是线性模型参数和样本。所以需要某种方法，来实现 $\Phi$ 与 $\eta=\theta^Tx$ 的**转换**。转换就是通过列表的$5$：多项分布也属于指数分布族。

也就是说，广义线性模型就是对一个事物两个角度的观察，使得将一个线性模型转换为一个分布。而转换的函数通常叫做**连接函数**。

---

## softmax回归

#### 模型

softmax回归用来处理多类别问题，$y^{(i)} \in \{1,\ldots,K\}$，$K$ 表示类别个数。$\Phi_i = P(y = i; \Phi)$，$\begin{matrix} \sum_{i=1}^K \Phi_i = 1 \end{matrix}$；且 $P(y = k; \Phi) = 1- \begin{matrix} \sum_{i=1}^{k-1} \Phi_i \end{matrix}$。

引入 $T(y)$：

$$T_{(1)} = \begin{bmatrix} 1 \\ 0 \\ 0 \\ \vdots \\ 0 \end{bmatrix} T_{(2)} = \begin{bmatrix} 0 \\ 1 \\ 0 \\ \vdots \\ 0 \end{bmatrix} \cdots T_{(k-1)} = \begin{bmatrix} 0 \\ 0 \\ 0 \\ \vdots \\ 1 \end{bmatrix} T_{(k)} = \begin{bmatrix} 0 \\ 0 \\ 0 \\ \vdots \\ 0 \end{bmatrix}$$

且 $1\{ y = k \} \leftrightarrow T(y)_i$，表示 $y = k$ 时 $T(y)$ 的第 $i$ 个分量的取值。那么：

$$\begin{align}
P(y:\Phi) &= \Phi^{1\{y=1\}}_1 \Phi^{1\{y=2\}}_2 \dots \Phi^{1\{y=k\}}_k \\
&= \Phi^{1\{y=1\}}_1 \Phi^{1\{y=2\}}_2 \dots \Phi^{1 - \begin{matrix} \sum_{i=1}^{k-1} \end{matrix}1\{y=i\}}_k \\
&= \Phi^{T(y)_1}_1 \Phi^{T(y)_2}_2 \dots \Phi^{1 - \begin{matrix} \sum_{i=1}^{k-1} \end{matrix} T(y)_i}_k \\
&= exp(T(y)_1log(\Phi_1) + T(y)_2log(\Phi_2) + \dots + (1 - \begin{matrix} \sum\nolimits_{i=1}^{k-1} \end{matrix} T(y)_i) log(\Phi_k)) \\
&= exp(T(y)_1log(\Phi_1/\Phi_k) + \dots + T(y)_{k-1}log(\Phi_{k-1}/\Phi_{k}) + log(\Phi_k)) \\
&= b(y)exp(\eta^TT(y)-a(\eta))
\end{align}$$

则：

$$\eta = \begin{bmatrix} log(\Phi_1/\Phi_k) \\
log(\Phi_2/\Phi_k) \\
\dots \\
log(\Phi_{k-1}/\Phi_k) \end{bmatrix} \in \mathbb{R}^{k-1}
\quad a(\eta)=-log(\Phi_k) \quad b(y)=1
$$

且，$\eta^T = \begin{bmatrix} \eta^1  \eta^2 \dots \eta^{k-1}\end{bmatrix}$，$\begin{matrix} \sum_{i=1}^k \Phi_i= 1 \end{matrix}$，则 $\Phi_i = e^{\eta_i}\Phi_k \Rightarrow$

$\Phi_i = \frac{e^{\eta_i}}{1+\begin{matrix}\sum_{j=1}^{k-1}e^{\eta_i}\end{matrix}} (i=1,2\dots,k-1) \qquad\qquad (1)$

$$\begin{align}
h_\theta(x) &= E\left[T(y)\mid x;\theta\right] \\
&= \left.
E\left[
\begin{matrix}
1\{ y=1 \} \\
1\{ y=2 \} \\
\vdots \\
1\{y=k-1\}
\end{matrix}
\right| x;\theta
\right] \\
&= \begin{bmatrix}
\Phi_1 \\
\Phi_2 \\
\vdots \\
\Phi_{k-1}
\end{bmatrix}
\end{align}$$

这样，就得到了hypothesis，然后可以通过最大似然估计求得所有参数的估计 $\Theta$.

$$\begin{align}
l(\theta) &= \prod_{i=1}^m p(y^{(i)} \mid x^{(i)}; \theta) \\
&= \prod_{i=1}^m \Phi_1^{1\{y^{(i)}=1} \dots \Phi_k^{1\{y^{(i)}=k}
\end{align}$$

将公式 $(1)$ 带入，使用梯度下降求得。

---

#### Ufldl

上一小节，已经算是给出了一个损失函数，并且求解的算法也给出。也就是这个问题已经解决了。这正是 Andrew Ng 网易公开课所讲的 softmax回归。下面提一下 ufldl 内容，不同的地方是 ufldl 的方法存在参数“冗余”，存在参数的多个解。为解决这个问题，在梯度下降时使用了”权重衰减——正则化”，较网易公开课的方式梯度下降更为简单。

ufldl 方法得到的 hypothesis 如下所示：

$$\begin{align}
h_\theta(x) =
\begin{bmatrix}
P(y = 1 | x; \theta) \\
P(y = 2 | x; \theta) \\
\vdots \\
P(y = K | x; \theta)
\end{bmatrix}
=
\frac{1}{ \sum_{j=1}^{K}{\exp(\theta^{(j)\top} x) }}
\begin{bmatrix}
\exp(\theta^{(1)\top} x ) \\
\exp(\theta^{(2)\top} x ) \\
\vdots \\
\exp(\theta^{(K)\top} x ) \\
\end{bmatrix}
\end{align}$$

与**模型**部分 $h_\theta(x) \in \Re^{k-1}$ 不同就是这里的 $h_\theta(x) \in \Re^{k}$，欲得到这样的结果只需另**模型**部分 $T(y)$ 维度加 $1$，且 $T(k)$ 的第 $k$ 个元素为 $1$，然后对 $h_\theta(x)$ 进行概率归一化。

$\theta$ 是 $n*K$ 的矩阵：

$$\theta = \left[\begin{array}{cccc}| & | & | & | \\
\theta^{(1)} & \theta^{(2)} & \cdots & \theta^{(K)} \\
| & | & | & |
\end{array}\right].$$

#### 损失函数

$$\begin{align}
J(\theta) = - \left[ \sum_{i=1}^{m} \sum_{k=1}^{K}  1\left\{y^{(i)} = k\right\} \log \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})}\right]
\end{align}$$

令：

$$P(y^{(i)} = k | x^{(i)} ; \theta) = \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)}) } \qquad \qquad (2)$$

$$\begin{align}
\nabla_{\theta^{(k)}} J(\theta) = - \sum_{i=1}^{m}{ \left[ x^{(i)} \left( 1\{ y^{(i)} = k\}  - P(y^{(i)} = k | x^{(i)}; \theta) \right) \right]  }
\end{align}$$

注意，对 $\theta^{(k)}$ 求偏导的时候，如果 $y^{i} = k$，$(2)$ 式分子分母均包含 $\theta^{(k)}$；如果$y^{i} \not = k$，$(2)$ 式分母求和项中存在 $\theta^{(k)}$。

###### 参数冗余：

对于每个 $\theta^{(k)}$ 减去相同的值，即 $\theta^{(k)} - \psi$ 之后，损失函数不变：

$$\begin{align}
P(y^{(i)} = k | x^{(i)} ; \theta)
&= \frac{\exp((\theta^{(k)}-\psi)^\top x^{(i)})}{\sum_{j=1}^K \exp( (\theta^{(j)}-\psi)^\top x^{(i)})}  \\
&= \frac{\exp(\theta^{(k)\top} x^{(i)}) \exp(-\psi^\top x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)}) \exp(-\psi^\top x^{(i)})} \\
&= \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})}.
\end{align}$$

#### 算法

使用权重衰减（正则化）方法，解决参数冗余问题：

$$\begin{align}
J(\theta) = - \left[ \sum_{i=1}^{m} \sum_{k=1}^{K}  1\left\{y^{(i)} = k\right\} \log \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})}\right]
+\frac{\lambda}{2} \sum_{k=1}^K \sum_{j=0}^n \theta_{kj}^2
\end{align}$$

$$\begin{align}
\nabla_{\theta^{(k)}} J(\theta) = - \sum_{i=1}^{m}{ \left[ x^{(i)} \left( 1\{ y^{(i)} = k\}  - P(y^{(i)} = k | x^{(i)}; \theta) \right) \right]}
+\lambda \theta_k
\end{align}$$

#### context

若类标之间没有关系，使用softmax回归；若一个样本可以归属多个类，使用多个二元分类器。

---

## 对数几率回归

#### 最大似然

$$\begin{align}
l(\theta) &= \left[ \sum_{i=1}^m   (1-y^{(i)}) \log (1-h_\theta(x^{(i)})) + y^{(i)} \log h_\theta(x^{(i)}) \right] \qquad \qquad (3)\\
&= - \left[ \sum_{i=1}^{m} \sum_{k=0}^{1} 1\left\{y^{(i)} = k\right\} \log P(y^{(i)} = k | x^{(i)} ; \theta) \right]  \qquad \qquad (4)\\
h_\theta(x^{(i)}) &= \frac{1}{1+e^{-\theta(x^{(i)})}} \\
g(z) &= \frac{1}{1+e^{-z}} \\
g'(z) &= g(z)(1-g(z)) \\
ln'(x) &= x^{-1} \\
h_\theta(x^{(i)}) &= g(\theta^Tx^{(i)}) \qquad (**)
\end{align}$$

$(4)$ 可以看出softmax回归是对数几率回归的拓展；

将 $(**)$ 带入 $(3)$，使用链式求导得到：

$$\nabla_{\theta} l(\theta) = (y^{(i)} - h_\theta(x^{(i)}))x^{(i)} \qquad \qquad (5)$$

因为是最大似然（负损失），使用梯度上升（对于参数的变化，结果是一样的）。

#### 拓展

从公式 $(5)$ 可以看出，当算法停止分类边界已经确定的情况下，如果新加入一个正类样本，梯度相对之前全部增加（正类样本梯度全为正，负类反之）。$w$ (分类超平面法向量)会向新加入正类方向偏转，这对分类边界会造成什么样的影响呢？

类似问题：[SVM和逻辑斯特回归对同一样本A进行训练，如果某类中增加一些数据点，那么原来的决策边界分别会怎么变化？](https://www.zhihu.com/question/30123068)

---

## 交叉熵

`Softmax`是最小化每一个样本估计分类概率和样本真实分布的交叉熵，即：

$$1\left\{y^{(i)} = k\right\} \log \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})}
$$

而`Softmax`的损失函数 $L_i= \log \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^K \exp(\theta^{(j)\top} x^{(i)})}$. 因为损失是对样本来说的，所以需要通过一个**指示函数**确定该样本对应的类别：$k$.

说交叉熵之前，先了解一下`KL散度`。

#### KL散度

`KL散度`是用来度量使用基于`Q`的编码来编码来自`P`的样本平均所需的额外的比特个数。所以，`KL散度`，虽然可以用于度量两个概率分布之间的差异，却不满足对称性度量。

$$\begin{align}
K L (P \mid \mid Q) &= \int_{-\infty}^{+\infty} p(x) \log \frac{p(x)}{q(x)} \, dx \\
&= \int_{-\infty}^{+\infty} p(x) \log p(x) \, dx - \int_{-\infty}^{+\infty} p(x) \log q(x) \, dx \\
&= -H(P) + H(P,Q)
\end{align}$$

其中，$H(P,Q)$ 就是**交叉熵**；在信息论中表示，使用基于 $Q$ 的编码对来自 $P$ 的变量进行编码所需的最小字节数。

所以，`Softmax`分类器所做的就是最小化在估计分类概率（就是 $L_i =e^{f_{y_i}}/\sum_je^{f_j}$）和“真实”分布之间的交叉熵.

比如，一个三分类问题：`1, 2, 3`三个类别，样本 $i$ 属于第一类，那么它的估计分类概率为：$L_1 \; L_2\; L_3$，而真实分布为 `p(x=1)=1, p(x=2)=P(x=3)=0`，所以`Softmax`对于样本 $i$ 就最小化：

$$
\sum_{k=1}^31\left\{y^{(i)} = k\right\} \log \frac{\exp(\theta^{(k)\top} x^{(i)})}{\sum_{j=1}^3 \exp(\theta^{(j)\top} x^{(i)})} \\
= \log \frac{\exp(\theta^{(1)\top} x^{(i)})}{\sum_{j=1}^3 \exp(\theta^{(j)\top} x^{(i)})}
$$

---

## 对数损失与hinge损失

注意，在`Softmax`的`hypothesis` $h_{\theta}(x)$ 中看似不像`LR`那样，它没有包含类标的信息，但是也是可以转化成那样的形式的。

包含类标信息指，**损失函数**形式是这样的：

$$\begin{align}
s &= \theta ^Tx +b \\
\hat{err}_{0/1} &= [ys \leq 0] \\
\hat{err}_{SVM} &= \max(1-ys,0) \\
\hat{err}_{SCE} &= \log_2 (1+\exp(-ys)) \\
\end{align}$$

其中，`error(SCE) is another upper bound loss function of` $err_{0/1}$.

且，

$$\hat{err}_{LR} = \ln2 \centerdot \hat{err}_{SCE}$$

当 $K=2$ 时：

$$\begin{align}
h_\theta(x) &=

\frac{1}{ \exp(\theta^{(1)\top}x)  + \exp( \theta^{(2)\top} x^{(i)} ) }
\begin{bmatrix}
\exp( \theta^{(1)\top} x ) \\
\exp( \theta^{(2)\top} x )
\end{bmatrix}
\end{align}$$

继而，

$$\begin{align}
h(x) &=

\frac{1}{ \exp( (\theta^{(1)}-\theta^{(2)})^\top x^{(i)} ) + \exp(\vec{0}^\top x) }
\begin{bmatrix}
\exp( (\theta^{(1)}-\theta^{(2)})^\top x )
\exp( \vec{0}^\top x ) \\
\end{bmatrix} \\

&=
\begin{bmatrix}
\frac{\exp( (\theta^{(1)}-\theta^{(2)})^\top x )}{ 1 + \exp( (\theta^{(1)}-\theta^{(2)})^\top x^{(i)} ) } \\
\frac{1}{ 1 + \exp( (\theta^{(1)}-\theta^{(2)})^\top x^{(i)} ) }
\end{bmatrix} \\

&=
\begin{bmatrix}
1 - \frac{1}{ 1  + \exp( (\theta^{(1)}-\theta^{(2)})^\top x^{(i)} ) } \\
\frac{1}{ 1  + \exp( (\theta^{(1)}-\theta^{(2)})^\top x^{(i)} ) } \\
\end{bmatrix}
\end{align}$$

#### 对比

|$-\infty$|$\longleftarrow$|$ys$|$\longrightarrow$|$+\infty$|
|:---:|:---:|:---:|:---:|:---:|
|$\approx-ys$||$\hat{err}_{SVM}(s,y)$||$=0$|
|$\approx-ys$||$\ln2\centerdot\hat{err}_{SCE}(s,y)$||$=0$|

非常像！但是，`LR`更能避免异常点的影响（`SVM`仅有几个支持向量确定超平面）。

---

一直想总结，一直没总结，还好，160517 -> 160707 -> 16年中秋，:(

/(ㄒoㄒ)/~~

---

## References

1. [Kernel Logistic Regression :: SVM versus Logistic Regression](https://www.youtube.com/watch?v=5K44AgZvcDk&list=PLXVfgk9fNX2IQOYPmqjqWsNUFl2kpk1U2&index=19)
