---
layout:     post
title:      "近端梯度下降——Proximal Method"
subtitle:   "L1 范数问题求解"
date:       2016-08-04 00:03:00 +0800
author:     "Roach"
catalog: true
tags:
    - 机器学习
    - 高等数学
---

> 从头越，苍山如海，残阳如血。

## Subgradient

定义 (subgradient; subdifferential). 对于在 $p$ 维欧式空间中的凸开子集 $U$ 上定义的实值函数 $f:U \Rightarrow \mathbb{R}$，一个 $p$ 维向量 $v$ 称为 $f$ 在一点 $x_0 \in U$ 处的 $\mbox{subgradient}$，如果对于任意 $x \in U$，满足

$$f(x)-f(x_0) \geq v \cdot (x-x_0) \qquad (1)$$

且由在点 $x_0$ 处的所有 `subgradient` 所组成的集合称为 $x_0$ 的 `subdifferential`，记为 $\partial f(x_0)$。**上面的`x`是对定义域内任意的 `x` 来说的。**

注意 `subgradient` 和 `subdifferential` 只是对凸函数定义的。例如一维的情况，$f(x)=\mid x \mid$，在 $x=0$ 处的 $\mbox{subdifferential}$ 就是 $[-1,1]$ 这个区间（集合）。

注意在 $f$ 的 $\mbox{gradient}$ 存在的点，$\mbox{subdifferential}$ 将是由 $\mbox{gradient}$ 构成的一个单点集合。这样就将 $\mbox{gradient}$ 的概念加以推广了。这个推广有一个很好的性质。

#### 性质

`性质` (condition for global minimizer). 点 $x_0$ 是凸函数 $f$ 的一个全局最小值点，当且仅当 $0 \in \partial f(x_0)$.

证明很简单，将 $0 \in \partial f(x_0)$ 带入到式 $(1)$ 中就可以了。

---

##  proximity operator

对于一个凸函数 $\varphi : \mathcal{H} \Rightarrow \mathbb{R}$，`proximity operator` $\mbox{prox}_{\varphi}:\mathcal{H}\Rightarrow\mathcal{H}$ 有如下定义：

$$\mbox{prox}_{\varphi}(x)=\operatorname {arg} \min_{u\in {\mathcal {H}}} \varphi (u) + \frac{1}{2} \mid\mid u-x \mid\mid_2^2$$

添加一个参数：

$$\mbox{prox}_{\gamma\varphi}(x)=\operatorname {arg} \min_{u\in {\mathcal {H}}} \varphi (u) + \frac{1}{2\gamma} \mid\mid u-x \mid\mid_2^2$$

其中 $\gamma>0$，表示对于给定点 $x$，找到最优的点 $u=\mbox{prox}_{\gamma\varphi} (x)$，使得 $\varphi(u)+\frac{1}{2\gamma}\mid\mid u-x \mid\mid_2^2$ 最小。这里 $\gamma$ 表示一个权衡。这个解是肯定存在的，对于凸函数：`well-defined because of the strict convexity of the` $l_2$ `norm`.

栗子：

+ $\varphi(x)=0: \mbox{prox}_{\gamma\varphi}(x)=x$
+ $\varphi(x)=I_C(x) \; \mbox{(indicator function of C): } \mbox{prox}_{\varphi} \mbox{ is projection on C }$

$$\mbox{prox}_{\gamma\varphi}(x) =P_C(x) = \operatorname{arg} \min_{u \in C} \mid\mid u−x \mid\mid_2^2$$

+ $\varphi(x)= \mid\mid x\mid\mid_1 : \mbox{prox}_{\varphi} \mbox{is shrinkage (soft threshold) operation}$

$$(\mbox{prox}_{\gamma\varphi}(x))_i =
\begin{cases}
x_i - \gamma &\mbox{ } x_i \geq \gamma \\
0 &\mbox{ } \mid x_i \mid \leq \gamma \\
x_i + \gamma &\mbox{ } x_i \leq -\gamma
\end{cases}$$

$I_C(x)$ 表示如果 $x$ 在集合中则函数值为 $0$，否则为正无穷；$\varphi(x)= \mid\mid x\mid\mid_1$的情况下一节进行进一步讨论。

并且，当 $x = \mbox{prox}_{\gamma\varphi}(x)$ 时，得到最优解。也就是`不动点`是最优解（通过求不动点的方式迭代求解）。

#### soft threshold

$R(x)=\varphi(x)= \mid\mid x\mid\mid_1$，根据性质：若 $x_0$ 是凸函数 $f(x)$ 的极值点，那么 $x_0 \in \partial f(x)$，得，

$$\begin{aligned}x^+ = u=\operatorname {prox} _{R}(x) &\iff 0\in \partial \left(R(u)+{\frac {1}{2 \gamma}}\mid\mid u-x\mid\mid_{2}^{2}\right)\\
&\iff 0\in \gamma \partial R(u)+u-x\\
&\iff x-u\in \gamma\partial\mid u\mid\\
&\iff u\in x-\gamma\partial\mid u\mid \end{aligned}$$

对于绝对值函数，仅在 $0$ 处不可导，即，

$$\partial |x_{i}|={\begin{cases}1, &x_i > 0 \\
-1, &x_i <0 \\
[-1,1], &x_i=0 \end{cases}}$$

那么，
$$(\mbox{prox}_{\gamma R}(x))_i =
\begin{cases}
x_i - \gamma &\mbox{ } x_i \geq \gamma \\
0 &\mbox{ } \mid x_i \mid \leq \gamma \\
x_i + \gamma &\mbox{ } x_i \leq -\gamma
\end{cases}$$

这里仅对$u=0$时进行讨论：

$\because u\in x-\gamma\partial\mid u\mid \And\ \partial \mid 0\mid=[-1,1]\therefore u=0 \in x-[-\gamma, \gamma] \Rightarrow 0 \in x+[-\gamma, \gamma]$，也就是说`0`在区间$x+[-\gamma, \gamma]$内，则$x \in [-\gamma, \gamma]$。

记 `soft thresholding operator` 为 $S_{\gamma }(x)= \mbox{prox} _{\gamma \mid\mid \cdot \mid\mid_1}(x)$

那么如果在 $l_1$ 范数前面加上正则化的系数 $\lambda$，即 $R(x)=\lambda \mid\mid x \mid\mid$，又会发生生么变化呢？

$$(\mbox{prox}_{\gamma R}(x))_i =
\begin{cases}
x_i - \lambda \gamma &\mbox{ } x_i \geq \lambda \gamma \\
0 &\mbox{ } \mid x_i \mid \leq \lambda \gamma \\
x_i + \lambda \gamma &\mbox{ } x_i \leq -\lambda \gamma
\end{cases}$$

可以发现，如果你待求得机器学习问题确定了（损失+正则化及其系数），那么 $\lambda$ 就是一个定值，而 $\gamma$ 看你如何取了。**从这里可以导出近端梯度下降方法**。

---

## Proximal Gradient Method

待求解问题：

$$\min_{x\in \mathcal{H}} F(x) + R(x)$$

其中，$F(x)$ 凸、可导，$R(X)$ 凸；

那么在 $x$ 出取得极值的条件是：$0 \in \partial(F+R)(x)$.

那么，$x$ 的递推公式是：$x^{k+1}=\mbox{prox}_{\gamma R}(x^k-\gamma \nabla F(x^k))$.

当 $\hat x=\mbox{prox}_{\gamma R}(\hat x-\gamma \nabla F(\hat x))$ 时， $\hat x$ 使得 $F(x)+R(x)$ 最小化。

#### 证明

另 $x^+=\mbox{prox}_{\gamma h}(x-\gamma \nabla g(x))$，那么需证明

$$x^+= \mbox{arg}\min_u\left(g(u)+h(u)\right)$$

证，

$$\begin{align}x^+ &= \mbox{prox}_{\gamma h}\Big(x-\gamma \nabla g(x)\Big) \\
&= \mbox{arg}\min_u \left(h(u)+\frac{1}{2\gamma}\mid\mid u-x+ \gamma \nabla g(x) \mid\mid_2^2\right) \\
&= \mbox{arg}\min_u \left(h(u)+ \frac{\gamma}{2}\mid\mid \nabla g(x)\mid\mid_2^2 + \nabla g(x)^T(u-x)+ \frac{1}{2\gamma}\mid\mid u-x \mid\mid_2^2\right) \\
&= \mbox{arg}\min_u \left(h(u)+ g(x) + \nabla g(x)^T(u-x) +\frac{1}{2\gamma}\mid\mid u-x \mid\mid_2^2\right) \\
& \approx \mbox{arm}\min_u \Big(g(u)+h(u)\Big)
\end{align}$$

后两式的变化是因为，变化的两项都与 $u$ 无关，然后通过**二阶泰勒展开近似**得到目标函数。

也就是说此类问题的通解公式为：$x^{k+1}=\mbox{prox}_{\gamma R}(x^k-\gamma \nabla F(x^k))$.

下面证明，**二阶泰勒近似**成立：

###### 凸函数下界

`affine lower bound from convexity` 凸函数的仿射下界：

$$g(u) \geq g(x)+ \triangledown g(x) ^T(u-x)  \quad \forall x,u$$

证明：

带有拉格朗日余项的泰勒展开：

$$g(u)=g(x)+\triangledown g(x)^T(u-x)+\frac{1}{2}(u-x)^T\triangledown ^2g(\xi)(u-x)$$

其中，$\xi \in (x,u)$.因为凸函数的二阶导大于 $0$ （或者正定矩阵），得证。

###### 凸函数的二次上界

$$g(u) \leq g(x)+ \triangledown g(x)^T(u-x) +\frac{L}{2}\mid\mid u-x \mid\mid_2^2  \quad \forall x,u$$

证明：

如果函数 $\triangledown g(x)$ 是利普希茨连续，其中的常量用 $L$ 表示：

$$\mid \mid \triangledown g(x \prime) -\triangledown g(x)\mid \mid _2 \leq L \mid \mid x \prime -x\mid \mid _2 \quad \forall x \prime, x$$

那么，拿一维的情况来说，

$$ \mid \triangledown g(x \prime) -\triangledown g(x)\mid  \leq L \mid x \prime-x\mid $$

则，

$$ \frac{\mid \triangledown g(x \prime) -\triangledown g(x)\mid }{\mid x \prime-x\mid } \leq L$$

取极限得：$\mid \triangledown^2g(x \prime) \mid \leq L$，且凸函数二阶导大于零，所以：

$$g(u) \leq g(x)+ \triangledown g(x)^T(u-x) +\frac{L}{2}\mid \mid u-x\mid \mid _2^2  \quad \forall x,u$$

也就是说之前的 $\gamma \in [0, \frac{1}{L}]$.

---

## 线性搜索

**近端梯度下降**

梯度下降的时候，一般设置的步长都较小，因为害怕震荡。在我之前`ML中的数学`一文中也试图讨论梯度下降步长的阈值，大于阈值就会引起震荡，但是不好确定。

那是不是就没有其他方法来，得到合适的步长求加快收敛的速度呢？最简单的方法就是`线性搜索`。这里先做一个通俗的解释：得到负梯度之后（函数值减小的方向），先令步长为 $t=1$，计算一下走完这一步之后的函数值，如果函数值增大了，说明步子大了，令步长 $t = \beta t,\beta \in (0,0.5)$，直到在某一次测试的时候函数值首次减小。这是才按照当前步长，进行一次梯度下降。

那么，这和我们当前与解决的问题有什么联系呢？

首先，对于函数 $\triangledown F(x)$ 其利普希茨常量`L`，不一定知道。那也就是说式 $(1)$ 中的 $\gamma$ 不知道，那就没法继续了。假如，有幸你遇到的问题对应的该常量已知，从其`泰勒展开二阶近似`的证明我们知道，只是求得了每一步近似函数的最小值点。也就是式 $(1)$ 给出的解，我们不好得到，或者得到有些许偏差。

注意，`近端算法`肯定有类似式 $(1)$ 用到`次梯度`的一步，那么我们可以将其结果进行转化。在进行近端梯度下降，用前一步的结果（这里说成立足点`x`）减去当前的结果，那就得到了一个方向。这个方向是式 $(1)$ 求得的 $R(x)$ 降低最快的方向。那么，在这个方向上走多远呢，使用`线性搜索`使得 $F(x)$ 的值首次下降，而前一个立足点`x`减去步长之后就成了新的立足点`x+1`，迭代直到收敛。可以看到每一次函数值都在降低，那么就一定会收敛。

其实，本文一直将最小化的问题表示为 $\min_x g(x) + h(x)$ 并非`F(x)`和`R(x)`将就一下吧。

---

![standpoint](/img/convex/standpoint.png)

每一个黑点都是可能被选则的立足点，但是上图这是没有 $F(x)$ 的情况下。有 $F(x)$ 结果类似，但会有不同。

---

#### 符号化

将 $\gamma$ 替换为 $t$，我们的原始问题是最小化`g(x)+h(x)`，那么迭代下面步骤就可以，只不过每步的 $t_k$ 不好确定。

$$x^{(k)}=\mbox{prox}_{\gamma R} \Big(x^{(k-1)}-\gamma \nabla F(x^{(k-1)}) \Big)$$

对公式进行如下的变换：

$$x^{(k)}=x^{(k-1)}-t_k G_{t_k}(x^{(k-1)})$$

其中，

$$G_t(x)=\frac{1}{t}(x-\mathbf {prox}_{th}(x-t \triangledown g(x)))$$

那么，线面就是线性搜索来确定 $x^+ = x - tG_t(x)$ 中的步长`t`：

`start at some,` $t:= \hat t$`,repeat,`$t:=\beta t$ `(with 0<β<1), until:`

$$g(x-tG_t(x)) \leq g(x)- t\triangledown g(x)^TG_t(x)+\frac{t}{2}||G_t(x)||_2^2$$

## LASSO

求解 $\mbox{L}1$`norm` 正则化的问题，

$$\min_{x\in \mathcal{H}} F(x) + \lambda \mid\mid x \mid\mid_1$$

则只需将公式变成，

$$x^{k+1}=\mbox{prox}_{\lambda \mid\mid\cdot\mid\mid_1}(x^k-\gamma \nabla F(x^k))$$

当然，还是要使用`线性搜索`，毕竟费的这么长时间才总结完。(╯°Д°)╯︵ ┻━┻ 摆好摆好不然下次没得掀了 ┬─┬ ノ('-'ノ) .

---

## 小结

~~本文尚未完成问题：~~

+ ~~二阶泰勒近似，使用了 利普希茨 条件，但是这一部分尚没有搞清楚~~
+ ~~有一个参考文献中，设置了类似于 `Proximal Gradient Method` 的步长，个人偏向不用设置步长~~

上面两个深坑，均已填 T_T 2016.08.21 更新。

待学习：[Regularization and VariableSelection via the Elastic Net
](https//web.stanford.edu/%7Ehastie/TALKS/enet_talk.pdf)

这个就是一范数、二范数一起用，但是这样的优点还没有了解，但是求解方法就是以上的方法了。看了也会另开一篇再写了~~~

---

## 附加

$\mbox{l}1$`norm` 稀疏化的一个解释（非明了的图形），待解决问题即 $LASSO$ 部分所列问题，若在 $0$ 处， $F'(x)<\gamma \mid_{x=0}$，则 $0 \in \partial \left(F(x)+\gamma \mid\mid x \mid\mid_1 \right) \mid_{x=0}$，则得**稀疏解**。

$\mbox{l}2$`norm` 则更倾向于平滑，待解决问题：

$$\min_w \sum_{n=1}^N \mid\mid w_1A_i+w_2B_i-y_i \mid\mid_2^2 + \lambda \mid\mid w \mid\mid^2_2$$

如果，变量 $A$ 和 $B$ 完全相关，则最后 $w_1=w_2$，因为此时 $\mbox{l}2$`norm` 才最小。

至于 $l1$ 中的多重共线性问题，究竟什么情况那个变量系数压缩为 $0$，学习之后再填吧。。。

![soft-thresholding](/img/machinelearning/soft-thresholding.svg)

---

## References

1. [3. Proximal gradient method - EECS at UC Berkeley](https://www.eecs.berkeley.edu/~elghaoui/Teaching/EE227A/lecture18.pdf)

2. [Proximal Gradient Method近端梯度算法](http://blog.csdn.net/lanyanchenxi/article/details/50448640)

3. [Proximal gradient methods for learning](https://en.wikipedia.org/wiki/Proximal_gradient_methods_for_learning#cite_note-12)

4. [Sparsity and Some Basics of L1 Regularization](http://freemind.pluskid.org/machine-learning/sparsity-and-some-basics-of-l1-regularization/#4cc0a68f3d787f62f6f765b11d409770c6584dd1)

二次更新添加：

1. [Proximal gradient method](https://web.stanford.edu/~boyd/papers/pdf/prox_algs.pdf)
2. [Proximal Algorithms--proximal gradient algorithm](http://blog.csdn.net/raby_gyl/article/details/51985908)
