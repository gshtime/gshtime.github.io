---
title: google-wide-and-deep-network
date: 2018-06-02 13:32:55
mathjax: true
tags:
---

# 第0章

wide是指高维特征+特征组合的LR。LR高效、容易规模化（scalable）、可解释性强。但是泛化性需要在特征工程上下功夫
deep就是deep learning了。特征工程省力，但是容易过度泛化over-generalize。

## 参考阅读

- 2016 《Wide & Deep Learning for Recommender Systems》 [blog]()
- 2017 《Deep & Cross Network for Ad Click Predictions》 [blog]()
- [google research blog](https://research.googleblog.com/2016/06/wide-deep-learning-better-together-with.html)
- [wide & deep github code](https://github.com/tensorflow/models/tree/master/official/wide_deep)
- 《2016-Deep Crossing: Web-Scale Modeling without Manually Crafted Combinatorial Features》 

链接：本文参考：https://blog.csdn.net/yujianmin1990/article/details/78989099

# paper

Wide & Deep前者是用来给用户推荐潜在喜欢的APP；Deep & Cross是用来预测用户可能点击的广告排序。

## Memorization 和 Generalization

paper 的两个重要概念：

Memorization（对应wide）: 特征关系的 Memorization ，是 cross-product transformation 实现的一个wide set，它有效也便于理解。*Memorization* 可以宽泛地定义成学到items或features的共现率，并利用（exploiting）这种在历史数据中的相关关系（correlation）。

Generalization （对应deep)： 相关性的传递（transitivity），新特征组合,多样性（diversity）好一些。是基于相关关系的转移，并探索（explores）在过往很少或从不出现的新的特征组合。

基于Memorization的推荐系统通常更局部化(topical)，将items与执行相应动作的users直接相关。而基于Generalization的推荐则更趋向于推荐多样化的items。

## 本文的主要贡献

Wide & Deep 学习框架，可以用于联合训练带embeddings的feed-forward神经网络，以及对于稀疏输入的常用推荐系统所使用的带特征转换的线性模型。
Wide & Deep推荐系统的实现和评估在Google Play上已经产品化，这个app store具有数十亿的活跃用户、以及上百万的app。
开源，在Tensorflow上提供了一个高级API。

## 特征输入

W&D的特征包括三方面： 
- User-Feature：contry, language, demographics. 
- Contextual-Feature：device, hour of the day, day of the week. 像余额宝、阿里音乐那个比赛都用了时间特征
- Impression-Feature：app age, historical statistics of an app. 

1. Wide部分的输入特征

    - raw input features and transformed features [手挑的交叉特征]. 

notice: Wide_Deep 这里的 cross-product transformation：只在离散特征（稀疏）之间做组合，不管是文本策略型的，还是离散值的；没有连续值特征的啥事，至少在W&D的paper里面是这样使用的。 

2. Deep部分的输入特征

    - raw input + embeding处理 

对非连续值之外的特征做 embedding 处理，这里都是策略特征，就是乘以个 embedding_matrix 。在TensorFlow里面的接口是：tf.feature_column.embedding_column，默认trainable=True. 

对连续值特征的处理是：将其按照累积分布函数P(X≤x)，压缩至[0,1]内。 

### notice

Wide部分用FTRL+L1来训练；Deep部分用AdaGrad来训练。 

Wide&Deep在TensorFlow里面的API接口为：tf.estimator.DNNLinearCombinedClassifier 

![](http://p8vrqzrnj.bkt.clouddn.com/wide-and-deep-1.png)

## 3. Wide & Deep Learning

### 3.1 Wide组件

wide组件是一个泛化的线性模型，形式为：$y=w^Tx+b$，如图1(左）所示。y是预测，$x = [x_1, x_2, …, x_d]$ 是d维的特征向量， $w = [w_1, w_2,…, w_d]$ 是模型参数，其中b为bias。特征集包括原始的输入特征和转换后的特征，一个最重要的转换是，cross-product transformation。它可以定义成：

$\phi_k(x)=\prod_{i=1}^{d}x_{i}^{c_{ki}}, c_{ki} \in \{0, 1\}$ (paper 公式1)

其中 $c_{ki}$ 为一个boolean变量，如果第i个特征是第k个变换ϕk的一部分，那么为1; 否则为0.对于二值特征，一个cross-product transformation（比如：”AND(gender=female, language=en)”）只能当组成特征（“gender=female” 和 “language=en”）都为1时才会为1, 否则为0. 这会捕获二值特征间的交叉，为通用的线性模型添加非线性。

### 3.2 Deep组件

Deep组件是一个前馈神经网络(feed-forward NN)，如图1(右）所示。对于类别型特征，原始的输入是特征字符串（比如：”language=en”）。这些稀疏的，高维的类别型特征会首先被转换成一个低维的、dense的、real-valued的向量，通常叫做“embedding vector”。embedding的维度通常是O(10)到O(100)的阶。该 embedding vectors 被随机初始化，接着最小化最终的loss的方式训练得到该值。这些低维的dense embedding vectors接着通过前向传递被feed给神经网络的隐层。特别地，每个隐层都会执行以下的计算：

$a^{l+1}=f(W^{(l)}a^{(l)}+b^{(l)})$ (公式2)

其中，l是层数，f是激活函数（通常为ReLUs），$a^{(l)}$，$b^{(l)}$ 和$W^{(l)}$分别是第l层的activations, bias，以及weights。

![](http://p8vrqzrnj.bkt.clouddn.com/DX-20180603@2x.png)

### 3.3 Wide & Deep模型的联合训练

Wide组件和Deep组件组合在一起，对它们的输入日志进行一个加权求和来做为预测，它会被feed给一个常见的logistic loss function来进行联合训练。注意，联合训练（joint training）和集成训练（ensemble）有明显的区别。在ensemble中，每个独立的模型会单独训练，相互并不知道，只有在预测时会组合在一起。相反地，**联合训练（joint training）会同时优化所有参数，通过将wide组件和deep组件在训练时进行加权求和的方式进行。**这也暗示了模型的size：对于一个ensemble，由于训练是不联合的（disjoint），每个单独的模型size通常需要更大些（例如：更多的特征和转换）来达到合理的精度。相比之下，对于联合训练（joint training）来说，wide组件只需要补充deep组件的缺点，使用一小部分的cross-product特征转换即可，而非使用一个full-size的wide模型。

一个Wide&Deep模型的联合训练，通过对梯度进行后向传播算法、SGD优化来完成。在试验中，我们使用FTRL算法，使用L1正则做为Wide组件的优化器，对Deep组件使用AdaGrad。

组合模型如图一（中）所示。对于一个logistic regression问题，模型的预测为：

$P(Y = 1 | x) = \sigma(w_{wide}^{T} [x, \phi(x)] + w_{deep}^{T} a^{(l_f)} + b)$  (paper 公式3)

其中Y是二分类的label，$\sigma(·)$是sigmoid function， $\phi(x)$ 是对原始特征x做cross product transformations，b是bias项。$w_{wide}$ 是所有wide模型权重向量，$w_{deep}$ 是应用在最终激活函数 $a^{(lf)}$ 上的权重。

## 4 系统实践

连续值先用累计分布函数CDF归一化到[0,1]，再划档离散化。

# 论文翻译

[基于Wide & Deep Learning的推荐系统](http://d0evi1.com/widedeep-recsys/)

# 附

## Tensorflow

只需要3步，即可以使用tf.estimator API来配置一个wide，deep或者Wide&Deep：

1. 选中wide组件的特征：选中你想用的稀疏的base特征列和交叉列特征列
2. 选择deep组件的特征：选择连续型的列，对于每个类别型列的embedding维，以及隐层的size。

将它们放置到一个Wide&Deep模型中（DNNLinearCombinedClassifier）

关于更详细的操作，示例代码在：/tensorflow/tensorflow/examples/learn/wide_n_deep_tutorial.py，具体详见tensorflow tutorial。

## tf.nn.embedding_lookup_sparse

如何处理不定长的字符串的embedding问题

``` python
import tensorflow as tf

# 输入数据如下
csv = [
    "1,oscars|brad-pitt|awards",
    "2,oscars|film|reviews",
    "3,matt-damon|bourne",
]

# 第二列是不定长的特征。处理如下
# Purposefully omitting "bourne" to demonstrate OOV mappings.
TAG_SET = ["oscars", "brad-pitt", "awards", "film", "reviews", "matt-damon"]
NUM_OOV = 1

def sparse_from_csv(csv):
    ids, post_tags_str = tf.decode_csv(csv, [[-1], [""]])
    table = tf.contrib.lookup.index_table_from_tensor(
        mapping=TAG_SET, num_oov_buckets=NUM_OOV, default_value=-1)  # 构造查找表
    split_tags = tf.string_split(post_tags_str, "|")
    return ids, tf.SparseTensor(
            indices=split_tags.indices,
            values=table.lookup(split_tags.values),  # 不同值通过表查到的index
            dense_shape=split_tags.dense_shape)

# Optionally create an embedding for this.
TAG_EMBEDDING_DIM = 3

ids, tags = sparse_from_csv(csv)

embedding_params = tf.Variable(tf.truncated_normal([len(TAG_SET) + NUM_OOV, TAG_EMBEDDING_DIM]))
embedded_tags = tf.nn.embedding_lookup_sparse(embedding_params, sp_ids=tags, sp_weights=None)

# Test it out
with tf.Session() as sess:
    sess.run([tf.global_variables_initializer(), tf.tables_initializer()])
    print(s.run([ids, embedded_tags]))
```

## 继续学习

[TensorFlow Wide And Deep 模型详解与应用](https://blog.csdn.net/kwame211/article/details/78015498)
