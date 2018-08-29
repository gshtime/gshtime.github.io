---
title: spark 原理资料
date: 2018-08-21 12:17:29
tags:
---

《Spark学习: 简述总结》
https://blog.csdn.net/databatman/article/details/53023818

2 Spark 系统架构
首先明确相关术语:

- 应用程序(Application): 基于Spark的用户程序，包含了一个Driver Program 和集群中多个的Executor；
驱动(Driver): 运行Application的main()函数并且创建SparkContext;
- 执行单元(Executor): 是为某Application运行在Worker Node上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上，每个Application都有各自独立的Executors;
- 集群管理程序(Cluster Manager): 在集群上获取资源的外部服务(例如：Local、Standalone、Mesos或Yarn等集群管理系统)；
- 操作(Operation): 作用于RDD的各种操作分为Transformation和Action.

整个 Spark 集群中,分为 Master 节点与 worker 节点,,其中 Master 节点上常驻 Master 守护进程和 Driver 进程, Master 负责将串行任务变成可并行执行的任务集Tasks, 同时还负责出错问题处理等,而 Worker 节点上常驻 Worker 守护进程, Master 节点与 Worker 节点分工不同, Master 负载管理全部的 Worker 节点,而 Worker 节点负责执行任务. 

Driver 的功能是创建 SparkContext, 负责执行用户写的 Application 的 main 函数进程,Application 就是用户写的程序. 
Spark 支持不同的运行模式,包括Local, Standalone,Mesoses,Yarn 模式.不同的模式可能会将 Driver 调度到不同的节点上执行.集群管理模式里, local 一般用于本地调试. 

每个 Worker 上存在一个或多个 Executor 进程,该对象拥有一个线程池,每个线程负责一个 Task 任务的执行.根据 Executor 上 CPU-core 的数量,其每个时间可以并行多个 跟 core 一样数量的 Task4.Task 任务即为具体执行的 Spark 程序的任务. 

![](http://p8vrqzrnj.bkt.clouddn.com/20161103175047811)

在实际编程中,我们不需关心以上调度细节.只需使用 Spark 提供的指定语言的编程接口调用相应的 API 即可. 

在 Spark API 中, 一个 应用(Application) 对应一个 SparkContext 的实例。一个 应用 可以用于单个 Job，或者分开的多个 Job 的 session，或者响应请求的长时间生存的服务器。与 MapReduce 不同的是，一个 应用 的进程（我们称之为 Executor)，会一直在集群上运行，即使当时没有 Job 在上面运行。 

而调用一个Spark内部的 Action 会产生一个 Spark job 来完成它。 为了确定这些job实际的内容，Spark 检查 RDD 的DAG再计算出执行 plan 。这个 plan 以最远端的 RDD 为起点（最远端指的是对外没有依赖的 RDD 或者 数据已经缓存下来的 RDD），产生结果 RDD 的 Action 为结束 。并根据是否发生 shuffle 划分 DAG 的 stage.

``` scala
// parameter
val appName = "RetailLocAdjust"
val master = "local"   // 选择模式
val conf = new SparkConf().setMaster(master).setAppName(appName)
// 启动一个 SparkContext Application
val sc = new SparkContext(conf)
val rdd = sc.textFile("path/...")
```

# 参考文献

文献:大数据分析平台建设与应用综述
Spark学习手册（三）：Spark模块摘读
Spark入门实战系列–3.Spark编程模型（上）–编程模型及SparkShell实战 
文献: 基于 spark 平台推荐系统研究.
Apache Spark源码走读之7 – Standalone部署方式分析
Spark性能优化指南——基础篇
Apache Spark Jobs 性能调优（一）
Spark性能优化指南——基础篇
Apache Spark Jobs 性能调优（一）
Apache Spark Jobs 性能调优（一）
Apache Spark Jobs 性能调优（一）
Spark性能优化指南——基础篇