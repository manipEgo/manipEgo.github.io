---
title: Earth Mover's Distance / 动土距离
tags: AI LLM NLP math
type: article
pageview: true
key: earth-movers-distance

aside:
  toc: true
---

**Earth Mover's Distance (EMD)** 是一种用于衡量两个测度之间差异的指标。虽然它一般被应用于概率分布，但其计算方法和意义并不依赖于概率分布的部分特性，在性质更宽松（比如存在负值）的数据空间上也有应用价值。

<!--more-->

## EMD 的简介

EMD 也被称为 Wasserstein metric $W_1$, Kantorovich–Rubinstein metric, Mallows's distance。它是对 the optimal transport problem 的解。其非正式定义和问题定义参考其 [Wikipedia 页面](https://en.wikipedia.org/wiki/Earth_mover%27s_distance)及关联页面：

> In computer science, the earth mover's distance (EMD) is a measure of dissimilarity between two frequency distributions, densities, or measures, over a metric space $D$. Informally, if the distributions are interpreted as two different ways of piling up earth (dirt) over $D$, the EMD captures the minimum cost of building the smaller pile using dirt taken from the larger, where cost is defined as the amount of dirt moved multiplied by the distance over which it is moved.
> 
> The earth mover's distance is also known as Wasserstein metric $W_1$, Kantorovich–Rubinstein metric, or Mallows's distance. It is the solution of the optimal transport problem, which in turn is also known as the Monge-Kantorovich problem, or sometimes the Hitchcock–Koopmans transportation problem; when the measures are uniform over a set of discrete elements, the same optimization problem is known as minimum weight bipartite matching.

## EMD 的直观理解

其实英文维基的非正式定义便是对 EMD 的一种直观解释。从一个简单的一维分布开始，直观地理解 EMD 的含义：

我们可以将两个分布看作两个同质的“**土丘**”。这两个土丘总的 **质量/体积** 是相等的（[EMD 的变体](https://en.wikipedia.org/wiki/Earth_mover's_distance#Variants_and_extensions)可以处理总体积不同的情况），但是土丘的形状不同。我们希望将其中一个土丘变形成另一个土丘的形状。这个过程中，我们需要移动土丘中的“**土**”。移动一堆土需要做功，功的大小等于**土的质量**乘以**移动的距离**。那么完成这个变形的最小功就是 EMD。推广到高维空间，我们可以将 EMD 理解为将一个分布变形成另一个分布的最小代价。

## EMD 的计算

在应用中，EMD 需要衡量的两个分布往往不是已知的，且对分布的采样是离散且有限的，数据符合离散概率分布的形式。此时对 EMD 的计算等价于一个[线性优化问题](https://en.wikipedia.org/wiki/Earth_mover's_distance#EMD_between_signatures)：

> Represent a distribution $P$ as a signature, or collection clusters, where the $i$-th cluster represents a mass of $w_i$ centered at $p_i$. In this formulation, consider signatures $P = {(p_1, w_{p1}), (p_2, w_{p2}), ... , (p_m,w_{pm})}$ and $Q = {(q_1, w_{q1}), (q_2, w_{q2}), ... , (q_n,w_{qn})}$. Let $D = [d_{i,j}]$ be the ground distance between clusters $p_i$ and $q_j$. Then the EMD between $P$ and $Q$ is given by the optimal flow $F = [f_{i,j}]$, with $f_{i,j}$ the flow between $p_i$ and $q_j$, that minimizes the overall cost.
>
> $$\mathop{min}_{F} \sum^{m}_{i=1} \sum^{n}_{j=1} f_{i,j}d_{i,j}$$
>
> subject to the constraints:
> 
> $$f_{i,j} \geq 0, 1 \leq i \leq m, 1 \leq j \leq n$$
> 
> $$\sum^{n}_{j=1} f_{i,j} \leq w_{pi}, 1 \leq i \leq m$$
> 
> $$\sum^{m}_{i=1} f_{i,j} \leq w_{qj}, 1 \leq j \leq n$$
> 
> $$\sum^{m}_{i=1} \sum^{n}_{j=1} f_{i,j} = \mathop{min}\left\{\sum^{m}_{i=1} w_{pi}, \sum^{n}_{j=1} w_{qj}\right\}$$
>
> The optimal flow $F$ is found by solving this linear optimization problem. The earth mover's distance is defined as the work normalized by the total flow:
>
> $$EMD(P,Q) = \frac{\sum^m_{i=1} \sum^n_{j=1} f_{i,j}d_{i,j}}{\sum^m_{i=1}\sum^n_{j=1}f_{i,j}}$$

经过[证明](https://arxiv.org/abs/1509.02237)，EMD 有如下等价形式：

令 $U$ 和 $V$ 分别为 $P$ 和 $Q$ 的累积分布函数（CDF），则

$$l_1(u,v) = \int^{+\infty}_{-\infty} \vert U-V \vert$$

与 EMD 等价。

据此，计算 EMD 的代码实现变得十分简单。可以参考 `SciPy` 的 [wasserstein_distance 实现](https://github.com/scipy/scipy/blob/v1.11.4/scipy/stats/_stats_py.py#L9733-L9807)。

## EMD 的应用

此处以我的一个应用场景为例。

我需要分析两个数值序列在频域上的相似性。对两个序列应用 DFT 之后，会得到它们的频域表示。那么把这两个频域表示看作两个分布，将它们的 AUC 缩放到 $1$，它们之间的 EMD 便可一定程度上表征原始序列的值的频域相似性。当然，也可以不对 AUC 进行缩放，应用前文提到的 EMD 变体进行计算。因为幅值受到原始序列的值影响，所以在这个 case 上如何选择归一化还应另作讨论。但它们本质上都是在反映两个频域表示曲线被视作分布后的相似性，进而反映原始序列的频域相似性，非常直观且可解释。

值得一提的是，尽管这两个频域分布含有负值，但它们可以通过线性变换缩放到非负的、积分为 $1$ 的分布，因此直接对这两个包含负值的分布计算 EMD 显然也是具有意义的。事实上，私以为从 <a href="#emd-的计算">EMD 的计算</a>方法中可以看出，EMD 结果的意义表达并不依赖于分布的非负性，这将使得 EMD 更加便于应用。

当然，以上只是一个比较 naive 的应用思路，未经过严谨的证明，但能大致展现出应用 EMD 的想象空间即可。