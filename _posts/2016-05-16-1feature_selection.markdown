---
layout:     post
title:      "Feature selection"
subtitle:   "One startpoint of ml"
date:       2016-05-16 10:07:00 +0800
author:     "Roach"
catalog: true
tags:
    - 机器学习
---

## 维数灾难

[The curse of dimensionality refers to various phenomena that arise when analyzing and organizing data in high-dimensional spaces (often with hundreds or thousands of dimensions) that do not occur in low-dimensional settings such as the three-dimensional physical space of everyday experience.](https://en.wikipedia.org/wiki/Curse_of_dimensionality)

就是说，在对高维数据进行分析时会出现一些低维数据分析中没有的问题。但并不是说，处理数据的所有算法都会遇到维数灾难（these algorithms which are scalable）。比如在高维数据中，样本更多的分布于空间的边、角；单位超立方体内切超球体，在维度非常高的情况下体积趋近于0。并且，给定一个样本点，样本中与其距最远样本之间距离为 $$max$$，最近样本距离为 $$min$$，那么在维数 $$d$$ 足够大的情况下，$$max - min$$ 是 $$min$$ 的高阶无穷小，即,

$$\lim _{d\to \infty }E\left({\frac {\operatorname {dist} _{\max }(d)-\operatorname {dist} _{\min }(d)}{\operatorname {dist} _{\min }(d)}}\right)\to 0$$

在机器学习中，会需要处理高维小样本的数据，样本个数数十至百，而特征个数成千上万。而机器之所以可以学习就是在有足够的样本数下（样本复杂度），可以使学习算法以任意期望的概率将**泛化误差**与**经验误差**的差距缩小至任意给定范围——**PAC可学习** (Probably Approximately Correct)。实际上，在给定概率之后，**泛化误差**与**经验误差**的差距与样本成反比而与 $$VC$$维成正比。$$VC$$维与算法的hypothesis相关，而在线性分类器的情况下，若数据特征维度为 $$d$$，那么$$VC$$维是 $$d+1$$。

#### 为什么进行特征选择？

我们要保证学习算法得到的结果是可靠的，那么就要足够的样本量和较低的特征数。而对于高维小样本数据，就类似于在样本抽样不够的情况下，来估计样本的分布，就容易过拟合，算法得到的参数是错误的，**泛化误差**与**经验误差**的差距很大。

并且考虑这样的情况，现在得到的样本在三维情况下因为一个样本不可分，但是增加了一维变成四维之后可分了。但是，这个样本数据有错误（噪音），就有可能导致对下一个需要预测类别的样本得到错误的结果。

所以，就需要在进行学习之前，对数据进行特征选择，数据中特征减少了就相对增加了样本采样的密度。那么，特征选择就需要**选择和学习问题相关的特征（子集）、减少冗余特征（增大采样）并且删除噪音特征**。
