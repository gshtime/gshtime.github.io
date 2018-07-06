---
title: tensorflow 和 在线学习 onlinelearning
date: 2018-06-23 00:50:14
tags: tensorflow onlinelearning 在线学习
---

# 阿里巴巴 TFRS

[阿里妈妈基于TensorFlow做了哪些深度优化？TensorFlowRS架构解析](https://mp.weixin.qq.com/s/yuHavuGTYMH5JDC_1fnjcg)

亮点：

1. ps-plus框架重构，解决了水平扩展问题，支持增量更新，（grpc，lock，graph-engine）方面，Failover机制。

2. 在线学习

问题：

1、tensorflow的worker与ps-plus的对接，是重构worker还是对接口进行了修改？


## 综述

场景：搜索、广告、推荐
场景特点: 样本规模和特征空间通常非常巨大，千亿样本、百亿特征并不罕见，同时存在大量的稀疏特征作为Embedding输入

TFRS主要成果：

1. 解决了原生TF水平扩展能力不足的问题。在我们的测试中，绝大多数搜索广告模型的训练性能提升在十倍以上，某些模型的极限性能最高可提升百倍。
2. 支持完备的在线学习语义，模型变更实时写出；稀疏特征无需做连续ID化，可以直接使用原始特征表征进行训练，大幅简化了特征工程的复杂度。
3. 异步训练的梯度修正优化器（grad-compensation optimizer），有效减少了异步大规模并发引起的训练效果损失。
4. 集成了高效的Graph Embedding、Memory Network、Cross Media等多种高级训练模式。
5. 模型可视化系统DeepInSight提供深度模型训练的多维度可视化分析。

## TensorFlowRS分布式架构

在使用TensorFlow的过程中我们发现TF作为一个分布式训练系统有两个主要的问题：

1. 水平扩展能力差：在大部分模型的性能测试中,我们发现随着数据并行度的增加，单个worker的样本处理QPS急剧下降。当worker数量增大到一定规模的时候，系统整体QPS不再有增长甚至有所下降。
2. 缺乏完备的分布式Failover机制。

TensorFlowRS采取的解决方案包括：

- 通过对接独立参数服务器提升水平扩展能力

在对TF做过细致的profiling之后，我们发现TF原生的PS由于设计和实现方面的多种原因（grpc，lock，graph-engine），很难达良好的水平扩展能力。于是我们决定丢掉TF-PS的包袱，重新实现一个高性能的参数服务器：PS-Plus。此外我们提供了完整的TF on PS-Plus方案，可以支持用户在Native-PS和PS-Plus之间自由切换，并且完全兼容TensorFlow的Graph语义和所有API。用户可以在深度网络代码一行不改的情况下，将参数分布和运行在PS-Plus上，享受高性能的参数交换和良好的水平扩展能力。 

- 重新设计Failover机制，支持动态组网和Exactly-Once的Failover

TensorFlowRS引入了worker state，在checkpoint中存储了worker的状态信息，worker重启后，会从接着上次的进度继续训练。此外TensorFlowRS通过zk生成cluster配置，支持了动态组网的Failover。新的Failover机制可以保证任意角色挂掉的情况下，系统都能在分钟级完成Failover，并且不多算和漏算数据。

TensorFlowRS的整体架构

![TensorFlowRS的整体架构](http://p8vrqzrnj.bkt.clouddn.com/tfrs_instruction.jpg)

## PS-Plus

PS-Plus相对于传统的ParameterServer有如下特点：

(1)高性能：PS-Plus通过智能参数分配，零拷贝，seastar等多项技术，进一步提升了单台server的服务能力和系统整体的水平扩展能力。在实测中，在64core的机器上单个server能轻松用满55+的核心，在dense场景下io能打满双25G网卡，系统整体在 1~4000 worker 的范围内都具有近似线性的水平扩展能力

(2)高度灵活：PS-Plus拥有完善的UDF接口，用户可使用SDK开发定制化的UDF插件，并且可以通过简单的C++以及Python接口进行调用。

(3)完备的在线学习支持：PS-Plus支持非ID化特征训练，特征动态增删，以及模型增量实时导出等支撑在线学习的重要特性。

下面从中选取几点做比较详细的介绍：

1. 智能参数分配

参数分配策略(variable placement)，决定了如何将一个参数切分并放置到不同的server上。placement策略的好坏在高并发的情况下对PS的整体性能有着重大的影响。传统ParameterServer的placement方案是由系统预先实现几种常见的placement算法（比如平均切分+roundrobin），或者由用户在创建参数的时候手工划分，往往没有综合考虑全局的参数规模、Server的负载等。

PS-Plus实现了基于模拟退火算法的启发式参数分配策略，后续也在考虑实现基于运行时负载，动态rebalance的placement策略。PS-Plus的placement设计有如下优点：

- 综合考虑了全局参数的shape信息，在cpu，内存，网络带宽等限制条件下给出了近似最优的placement方案，避免了手工分配造成的不均匀、热点等问题。
- 整个参数分配过程由系统内部自动完成，用户无需配置即可获得接近最优的性能，用户无需了解PS底层实现的具体细节。
- Partition由框架自动完成，在上层算法代码，如TF代码中，不需要额外使用PartitionedVariable等机制，使用简单方便。

2. 去ID化特征支持

目前主流的深度学习框架都是以连续的内存来存储训练参数，通过偏移量（ID值）来寻址到具体的权重。为了避免内存的浪费，需要对特征做从0开始的连续ID化编码，这一过程我们称之为特征ID化。特征ID化是一个非常复杂的过程，尤其是当样本和特征数量非常庞大的时候，特征ID化会占用大量的时间和机器资源，给样本构建带来了很大的复杂度。

PS-Plus内部实现了一个定制化的hashmap，针对参数交换场景做了专门的优化，在支持特征动态增删的同时提供了超高的性能。通过hashmap，PS-Plus直接实现了对非ID特征的支持，极大的简化了样本构建的复杂度。

3. 通信层优化

对于Parameter Server架构，延迟是影响整体性能的重要原因。尤其是在模型复杂度不高的情况下，模型计算部分往往在10~100ms量级，那么总体通信的延迟就成为一个关键因素。

在传统的pipeline线程模型，高并发情况下中断和线程上下文切换会导致很大的开销，同时会引起大量的cache-line miss。此外，高频的锁竞争是带来延迟的最主要原因之一，即便是各类SpinLock、读写锁等优化也并不能有效消除这个问题。我们认为polling + run to completion是一个正确的选择，并且设计了我们的整体通信层架构。在新的通信层中，我们使用了Seastar作为底层的框架。对于Server、Worker上的connection，都严格保证connection绑定到固定的线程，同时线程与CPU核心绑定。Request、response直接采用run to completion的方式在当前线程处理。

整体架构如下图所示：

![ps-plus架构](http://p8vrqzrnj.bkt.clouddn.com/ps-plus.jpg)

在Seastar的基础上，我们做了很多功能、性能的改进和优化，这里做一些简要的介绍。

外部线程交互队列。我们借鉴Seastar核心之间的交互机制，提供了一个 M:N 无锁生产者消费者队列，用于外部线程与Seastar内部线程进行交互。相比传统队列性能有极大的提升。

写请求顺序调度。从外部线程poll到的写请求，如果直接调用Seastar的写接口，会导致写buffer无法保证有序。我们通过队列机制的改造，自动保证了写顺序，同时基本不损失多connection的并发写的性能。

灵活的编解码层。我们提供了一套编解码层的抽象接口，方便用户使用，从而不需要借助protobuf等传统的序列化、反序列化的第三方库，同时也避免了protobuf的一些性能问题。

## 在线学习

1. 非ID化特征支持
2. 特征动态增删
3. 模型增量实时导出
4. AUC Decay

## 大规模训练场景下的收敛效果优化

boost 方法解决分布式并行训练中梯度和模型不一致的问题。

## 高级训练模式

TFRS中集成了多种高阶训练模式，例如Graph Embedding，Memory Network，Cross Media Training等。

## 可视化模型分析系统DeepInsight

DeepInsight是一个深度学习可视化质量评估系统，支持训练阶段模型内部数据的全面透出与可视化分析，用以解决模型评估、分析、调试等一系列问题，提高深度模型的可解释性。

# tensorflow issue
API: sparse_column_with_hash_bucket.
params: hash_keys
source: https://github.com/tensorflow/tensorflow/issues/19324#issuecomment-394597155
