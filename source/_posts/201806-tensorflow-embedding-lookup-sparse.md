---
title: 机器学习中的embedding原理及tf.nn.embedding_lookup_sparse等API的理解
date: 2018-06-01 14:57:00
mathjax: true
tags: embedding_lookup, tf.gather, embedding_lookup_sparse
---

# 概述

本文主要讲解tensorflow中涉及embedding的API。之前看了一些文章，写的云山雾绕，花了好长时间才搞懂，太笨了。

embedding 算法主要用于处理稀疏特征，应用于NLP、推荐、广告等领域。所以word2vec 只是embbeding 思想的一个应用，而不是全部。

代码地址：git@github.com:gshtime/tensorflow-api.git

# embedding原理

常见的特征降维方法主要有PCA、SVD等。

而embedding的主要目的也是对（稀疏）特征进行降维，它降维的方式可以类比为一个全连接层（没有激活函数），通过 embedding 层的权重矩阵计算来降低维度。

假设：
- feature_num : 原始特征数
- embedding_size: embedding之后的特征数
- [feature_num, embedding_size]  权重矩阵shape
- [m, feature_num]  输入矩阵shape，m为样本数
- [m, embedding_size]  输出矩阵shape，m为样本数

![稀疏向量的选择](http://p8vrqzrnj.bkt.clouddn.com/WX20180601-155344.png)

应用中一般将物体嵌入到一个低维空间 $R^{embedding-size} (embedding-size << feature_num)$ ，只需要再compose 上一个从 $R^{feature-num}$ 到 $R^{embedding-size}$ 的线性映射就好了。每一个shape为 $feature-num \times embedding-size$ 的矩阵M(embedding矩阵) 都定义了 $R^{feature-num}$ 到 $R^{embedding-size}$ 的一个线性映射: $x \mapsto Mx$ 。当 $x$ 是一个标准基向量的时候，$Mx$对应矩阵 $M$ 中的一列，这就是对应 $id$ 的向量表示。

![连接层](http://p8vrqzrnj.bkt.clouddn.com/20170814220528829.jpg)

从id(索引)找到对应的 One-hot encoding ，然后红色的weight就直接对应了输出节点的值(注意这里没有 activation function)，也就是对应的embedding向量。

# tensorflow API

## 基础： tf.SparseTensor

构造稀疏向量矩阵，【未求证】使用上每一行为一个样本

SparseTensor(indices, values, dense_shape)

params:

- indices: A 2-D int64 tensor of dense_shape [N, ndims], which specifies the indices of the elements in the sparse tensor that contain nonzero values (elements are zero-indexed). For example, indices=[[1,3], [2,4]] specifies that the elements with indexes of [1,3] and [2,4] have nonzero values. 是dense_shape这个矩阵中所有values值的位置，与values一一对应。
- values: A 1-D tensor of any type and dense_shape [N], which supplies the values for each element in indices. For example, given indices=[[1,3], [2,4]], the parameter values=[18, 3.6] specifies that element [1,3] of the sparse tensor has a value of 18, and element [2,4] of the tensor has a value of 3.6. 每一个稀疏值，与其位置indices一一对应。
- dense_shape: A 1-D int64 tensor of dense_shape [ndims], which specifies the dense_shape of the sparse tensor. Takes a list indicating the number of elements in each dimension. For example, dense_shape=[3,6] specifies a two-dimensional 3x6 tensor, dense_shape=[2,3,4] specifies a three-dimensional 2x3x4 tensor, and dense_shape=[9] specifies a one-dimensional tensor with 9 elements. 稀疏向量矩阵的shape

Example: The sparse tensor

``` python
SparseTensor(indices=[[0, 0], [1, 2]], values=[1, 2], dense_shape=[3, 4])

# represents the dense tensor

# [[1, 0, 0, 0]
#  [0, 0, 2, 0]
#  [0, 0, 0, 0]]
```
## tf.nn.embedding_lookup 和 partition_strategy 参数

``` python
# Signature:
tf.nn.embedding_lookup(params, ids, partition_strategy='mod', name=None, validate_indices=True, max_norm=None)
# Docstring:
# Looks up `ids` in a list of embedding tensors.
```

是根据 `ids` 中的id，寻找 `params` 中的第id行。比如 `ids=[1,3,5]`，则找出`params`中第1，3，5行，组成一个tensor返回。

embedding_lookup不是简单的查表，`params` 对应的向量是可以训练的，训练参数个数应该是 feature_num * embedding_size，即前文表述的embedding层权重矩阵，就是说 lookup 的是一种全连接层。

此外，以下要记录的几个API里，都有参数 partition_strategy （切分方式）, 这个参数是当len(params) > 1 时，才生效，即当params 以list [a, b, c] (a,b,c都是tensor) 输入多个tensor时，对params的选择顺序进行切分，（而不是对ids进行切分，ids只有选择的作用，当然也决定了在return中次序）。

### 输入为单个tensor时

``` python 
# 当输入单个tensor时，partition_strategy不起作用，不做 id（编号） 的切分
a = np.arange(20).reshape(5,4)
print (a)

# 前面的编号是我手动加的，意思是不做切分的时候就顺序编号就行
# 0#[[ 0  1  2  3]
# 1# [ 4  5  6  7]
# 2# [ 8  9 10 11]
# 3# [12 13 14 15]
# 4# [16 17 18 19]]

tensor_a = tf.Variable(a)
embedded_tensor = tf.nn.embedding_lookup(params=tensor_a, ids=[0,3,2,1])
init = tf.global_variables_initializer()
with tf.Session() as sess:
    sess.run(init)
    embedded_tensor = sess.run(embedded_tensor)
    print(embedded_tensor)
# 根据 ids 参数做选择
#[[ 0  1  2  3]  选择了 id 0
# [12 13 14 15]  选择了 id 3
# [ 8  9 10 11]  选择了 id 2
# [ 4  5  6  7]] 选择了 id 1

```
### 输入为多个tensor时

partition_strategy 开始起作用，开始对多个tensor 的第 0 维上的项进行编号，编号的方式有两种，"mod"（默认） 和 "div"。

假设：一共有三个tensor [a,b,c] 作为params 参数，所有`tensor`的第 0 维上一共有 10 个项目（id 0 ~ 9）。

- "mod" : (id) mod len(params) 得到多少就把 id 分到第几个tensor里面
    - a 依次分到id： 0 3 6 9
    - b 依次分到id： 1 4 7
    - c 依次分到id： 2 5 8

- "div" : (id) div len(params) 可以理解为依次排序，但是这两种切分方式在无法均匀切分的情况下都是将前(max_id+1)%len(params)个 partition 多分配一个元素.
    - a 依次分到id： 0 1 2 3 
    - b 依次分到id： 4 5 6
    - c 依次分到id： 7 8 9


``` python
# partition_strategy='div' 的情况
a = tf.Variable(np.arange(8).reshape(2,4))
b = tf.Variable(np.arange(8,12).reshape(1,4))
c = tf.Variable(np.arange(12, 20).reshape(2,4))

embedded_tensor = tf.nn.embedding_lookup(params=[a,b,c], ids=[1,2,4], partition_strategy='div', name="embedding")

init = tf.global_variables_initializer()
with tf.Session() as sess:
    sess.run(init)
    print (sess.run(a))
    print (sess.run(b))
    print (sess.run(c))
    
    print ("########## embedded_tensor #########")
    print(sess.run(embedded_tensor))
# [[0 1 2 3]    a分到 id 0
#  [4 5 6 7]]   a分到 id 1

# [[ 8  9 10 11]] b分到 id 2
#                 b分到 id 3

# [[12 13 14 15]  c分到 id 4
#  [16 17 18 19]] (c 没分到id)
# ########## embedded_tensor ######### 注：ids=[1,2,4]
#[[ 4  5  6  7]   按 ids 选的 1
# [ 8  9 10 11]   按 ids 选的 2
# [12 13 14 15]]  按 ids 选的 4
```
注：如果ids 中 有 3，这个id 虽然被分给 b 这个tensor了，但是b没有，会报一个错误
``` python
InvalidArgumentError: indices[0] = 1 is not in [0, 1)
	 [[Node: embedding_5/GatherV2_1 = GatherV2[Taxis=DT_INT32, Tindices=DT_INT32, Tparams=DT_INT64, _device="/job:localhost/replica:0/task:0/device:CPU:0"](Variable_154/read, ConstantFolding/embedding_5/DynamicPartition-folded-1, embedding_5/GatherV2_1/axis)]]
```

## tf.gather()
函数签名如下：
``` python 
tf.gather(
    params,
    indices,
    validate_indices=None,
    name=None
```
参数说明：
- params是一个tensor，
- indices是个值为int的tensor用来指定要从params取得元素的第0维的index。

该函数可看成是tf.nn.embedding_lookup()的特殊形式，所以功能与其类似，即将其看成是embedding_lookup函数的params参数内只有一个tensor时的情形。

## tf.nn.embedding_lookup_sparse

``` python
tf.nn.embedding_lookup_sparse(
    params,
    sp_ids,
    sp_weights,
    partition_strategy='mod',
    name=None,
    combiner=None,
    max_norm=None
)
```
### 参数

见官网API，不贴了：[python API](https://www.tensorflow.org/api_docs/python/tf/nn/embedding_lookup_sparse)

- params: A single tensor representing the complete embedding tensor, or a list of P tensors all of same shape except for the first dimension, representing sharded embedding tensors. Alternatively, a PartitionedVariable, created by partitioning along dimension 0. Each element must be appropriately sized for the given partition_strategy.

Returns:
A dense tensor representing the combined embeddings for the sparse ids. For each row in the dense tensor represented by sp_ids, the op looks up the embeddings for all ids in that row, multiplies them by the corresponding weight, and combines these embeddings as specified.

In other words, if

- shape(combined params) = [p0, p1, ..., pm]

and

- shape(sp_ids) = shape(sp_weights) = [d0, d1, ..., dn]

then

- shape(output) = [d0, d1, ..., dn-1, p1, ..., pm].

For instance, if params is a 10x20 matrix, and sp_ids / sp_weights are

[0, 0]: id 1, weight 2.0 [0, 1]: id 3, weight 0.5 [1, 0]: id 0, weight 1.0 [2, 3]: id 1, weight 3.0

with combiner="mean", then the output will be a 3x20 matrix where

output[0, :] = (params[1, :] * 2.0 + params[3, :] * 0.5) / (2.0 + 0.5) output[1, :] = (params[0, :] * 1.0) / 1.0 output[2, :] = (params[1, :] * 3.0) / 3.0

### 示例：

``` python
a = np.arange(8).reshape(2, 4)
b = np.arange(8, 16).reshape(2, 4)
c = np.arange(12, 20).reshape(2, 4)
print ("a :")
print (a)
print ("b :")
print (b)
print ("c :")
print (c)
a = tf.Variable(a, dtype=tf.float32)
b = tf.Variable(b, dtype=tf.float32)
c = tf.Variable(c, dtype=tf.float32)
idx = tf.SparseTensor(indices=[[0,0], [0,2], [1,0], [1, 1]], values=[1,2,2,0], dense_shape=(2,3))
result = tf.nn.embedding_lookup_sparse([a,c,b], idx, None, partition_strategy='mod', combiner="sum")

init = tf.global_variables_initializer()
with tf.Session() as sess:
    sess.run(init)
    res = sess.run(result)
    print ("\n# result here")
    print(res)

# a :
# [[0 1 2 3]    id 0
#  [4 5 6 7]]   id 3
# b :
# [[ 8  9 10 11]  id 1
#  [12 13 14 15]] id 4
# c :
# [[16 17 18 19] id 2
#  [20 21 22 23]] id 5

# result here
# [[24. 26. 28. 30.]  
#  [16. 18. 20. 22.]]
```
解释：
``` python
#idx = tf.SparseTensor(indices=[[0,0], [0,2], [1,0], [1, 1]], values=[1,2,2,0], dense_shape=(2,3))
构造如下稀疏向量矩阵，每一行视为一个样本的特征向量
[
    [[1], [], [2]],
    [[2], [0], []]
]
以第一个样本 [[1], [], [2]] 为例：选择id 1 和id 2：
# [[ 8  9 10 11]  id 1
# [[16 17 18 19] id 2
根据 combiner="sum" ，把上面两个向量 按 axis=0 加到一起，得到：
# [24. 26. 28. 30.]
即 为第一个样本的 dense 向量
```

如果len(params) == 1, 就是不存在 partition 了，比较好理解，不再赘述。

喝最烈的果粒橙，钻最深的牛角尖。
end.