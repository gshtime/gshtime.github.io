---
title: 推荐系统及广告系统相关算法综述
date: 2018-06-24 00:01:42
categories: 推荐系统算法
tags:
    - 推荐系统
    - 计算广告
mathjax: true
---

目前，推荐系统领域主流的算法主要包括：

1. ftrl, 2013年, Google公司, 《Ad click prediction: a view from the trenches》

# 在线学习 ftrl

传统的批量算法的每次迭代是对全体训练数据集进行计算（例如计算全局梯度），优点是精度和收敛还可以，缺点是无法有效处理大数据集（此时全局梯度计算代价太大），且没法应用于数据流做在线学习。

而在线学习算法的特点是：每来一个训练样本，就用该样本产生的loss和梯度对模型迭代一次，一个一个数据地进行训练，因此可以处理大数据量训练和在线训练。准确地说，Online Learning并不是一种模型，而是一种模型的训练方法，Online Learning能够根据线上反馈数据，实时快速地进行模型调整，使得模型及时反映线上的变化，提高线上预测的准确率。

Online Learning的流程包括：将模型的预测结果展现给用户，然后收集用户的反馈数据，再用来训练模型，形成闭环的系统。如下图所示：

![](http://p8vrqzrnj.bkt.clouddn.com/onlinelearning1.png)

## CTR预测算法

这篇论文提出的基于FTRL的在线CTR预测算法，就是一种Online Learning算法。

即，针对每一个训练样本，首先通过一种方式进行预测，然后再利用一种损失函数进行误差评估，最后再通过所评估的误差值对参数进行更新迭代。直到所有样本全部遍历完，则结束。

那么，如何选择模型预测方法、评估指标以及模型更新公式就是该算法的重点所在。下面将介绍论文中这三部分内容：

1.预测方法：在每一轮 $t$ 中，针对特征样本 ${x}_{t}\in {R}^{d}$，以及迭代后(第一此则是给定初值)的模型参数 $w_t$，我们可以预测该样本的标记值：${p}_{t}=\sigma({w}_{t},{x}_{t})$ ，其中 $\sigma(a)=\frac{1}{1+exp(-a)}$ 是一个sigmoid函数。

2.损失函数：对一个特征样本 $x_t$，对应的标记为 $y_t \in 0, 1$， 则通过logistic loss 来作为损失函数，即： ${\mathit{l}}_{t}({w}_{t})= -{y}_{t}log{p}_{t}-(1-{y}_{t})log(1-{p}_{t})$ 

3.迭代公式： 目的是使得损失函数尽可能小，即可以采用极大似然估计来求解参数。首先求梯度：

$${g}_{t}=\frac{d\mathit{l}_{t}(w)}{dw}=(\sigma(w*{x}_{t})-{y}_{t}){x}_{t}=({p}_{t}-{y}_{t}){x}_{t}$$

暂时看不懂了
基于FTRL的在线CTR预测算法
https://blog.csdn.net/yz930618/article/details/75270869

## ftrl References
[@默一鸣](https://blog.csdn.net/yimingsilence/article/details/75123026) 标注了一下各篇paper的主要内容，感兴趣的可以有选择性地看一下，如果只关注工程实现，看标红的那篇就ok了：
[1] J. Langford, L. Li, and T. Zhang. Sparse online learning via truncated gradient.JMLR, 10, 2009. （截断梯度的paper）

[2] H. B. McMahan. Follow-the-regularized-leader and mirror descent: Equivalence theorems and L1 regularization. In AISTATS, 2011 （FOBOS、RDA、FTRL等各种方法对比的paper）

[3] L. Xiao. Dual averaging method for regularized stochastic learning and online optimization. In NIPS, 2009 （RDA方法）

[4] J. Duchi and Y. Singer. Efficient learning using forward-backward splitting. In Advances in Neural Information Processing Systems 22, pages 495{503. 2009. （FOBOS方法）

[5] H. Brendan McMahan, Gary Holt, D. Sculley, Michael Young, Dietmar Ebner, Julian Grady, Lan Nie, Todd Phillips, Eugene Davydov, Daniel Golovin, Sharat Chikkerur, Dan Liu, Martin Wattenberg, Arnar Mar Hrafnkelsson, Tom Boulos, Jeremy Kubica, Ad Click Prediction: a View from the Trenches, Proceedings of the 19th ACM SIGKDD International Conference on Knowledge Discovery and Data Mining (KDD) (2013) （这篇是那篇工程性的paper）

[6] H. Brendan McMahan. A unied analysis of regular-ized dual averaging and composite mirror descent with implicit updates. Submitted, 2011 （FTRL理论发展，regret bound和加入通用正则化项）

[7] H. Brendan McMahan and Matthew Streeter. Adap-tive bound optimization for online convex optimiza-tion. InCOLT, 2010 （开始的那篇理论性paper）

# embedding

https://www.cntk.ai/pythondocs/layerref.html#embedding

embedding层，是一种用连续值向量来表示离散值或者word的方法，与one-hot相比，推荐系统中embedding层减少了特征的维度。在cntk中，embedding vector用行向量的方式来表示，将一个word映射到它的embedding只需要向量相乘（选取索引对应的embedding行）。

梯度： embedding矩阵的梯度矩阵也是只在非0值的地方存在梯度，每次更新只要更新存在值的embedding层就行了（和推荐系统好像有点不同）

word2vec中的bp
http://www.claudiobellei.com/2018/01/06/backprop-word2vec/

