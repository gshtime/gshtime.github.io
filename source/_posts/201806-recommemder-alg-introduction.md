---
title: 推荐算法方面的优秀文章
date: 2018-06-03 15:18:15
tags:
---

# 一些概念

推荐广告怎么做？

系统架构是什么样？ 
系统架构： bizer(特征抽取) → QR(用户理解 利用user id或上下文、位置等 → 相关的tag list) → Searcher(广告倒排 tag → ad list(后续ctr排序预选的ad)) → Ranker(根据cpm = ctr*bid，以及相关性等精排), 然后依次返回到用户做个性化推荐。

广告从投放到计费经过哪些主要流程？
投放到计费流程：点击付费(CPC), 展示付费(CPM), 按销售收入付费(CPS)等。

这个流程中算法策略关注什么？
算法策略关注：架构流程中每个步骤的数据的准确性，完整性对整个系统的准确度有重要影响。

# 综述 

[CTR预估算法之FM, FFM, DeepFM及实践](https://blog.csdn.net/John_xyz/article/details/78933253)

阿里巴巴

- [常用推荐算法（50页干货）](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247483811&idx=1&sn=fc3ee4ddfc4a8d6014a4cd90cdb5983c&scene=4#wechat_redirect)

腾讯

- [常见计算广告点击率预估算法总结](https://cloud.tencent.com/developer/article/1005915)
- 高航，[深度学习在 CTR 中应用](https://cloud.tencent.com/developer/article/1006667)

美团

- [第09章：深入浅出ML之Factorization家族](http://www.52caml.com/head_first_ml/ml-chapter9-factorization-family/)
# 时间线


[FM算法(一)：算法理论](https://www.cnblogs.com/AndyJee/p/7879765.html)
[FM算法(二)：工程实现](https://www.cnblogs.com/AndyJee/p/8032553.html)
[FM算法详解](https://blog.csdn.net/bitcarmanlee/article/details/52143909)

[深入FFM原理与实践](https://tech.meituan.com/deep-understanding-of-ffm-principles-and-practices.html)

# 2013 

[各大公司广泛使用的在线学习算法FTRL详解](http://www.cnblogs.com/EE-NovRain/p/3810737.html )


# code + action

[TensorFlow Estimator of Deep CTR --DeepFM/NFM/AFM/FNN/PNN](https://zhuanlan.zhihu.com/p/33699909)

# 协同过滤 待总结
https://blog.csdn.net/u010297828/article/details/51504952

# AFM

https://blog.csdn.net/jiangjiang_jian/article/details/80674250

# embedding
https://ask.hellobi.com/blog/wenwen/11885