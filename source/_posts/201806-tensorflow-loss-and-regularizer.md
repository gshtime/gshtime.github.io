---
title: tensorflow中损失函数加正则项
date: 2018-06-03 16:01:26
tags:
---

# 概述

在损失函数上加上正则项（结构风险最小化）是防止过拟合的一个重要方法,下面介绍如何在TensorFlow中使用正则项.

tensorflow中对参数使用正则项分为两步: 

1. 创建一个正则方法(函数/对象) 
2. 将这个正则方法(函数/对象),应用到参数上

# 创建正则项

## l1 正则

`tf.contrib.layers.l1_regularizer(scale, scope=None)`

返回一个用来执行L1正则化的函数,函数的签名是func(weights). 
参数:

- scale: 正则项的系数.
- scope: 可选的scope name

## l2 正则
`tf.contrib.layers.l2_regularizer(scale, scope=None)`

返回一个执行L2正则化的函数.

## 多正则

`tf.contrib.layers.sum_regularizer(regularizer_list, scope=None)`

返回一个可以执行多种(个)正则化的函数.意思是,创建一个正则化方法,这个方法是多个正则化方法的混合体.

参数: 
- regularizer_list: regulizer的列表

# 应用正则方法

`tf.contrib.layers.apply_regularization(regularizer, weights_list=None)`

参数

- regularizer:就是我们上一步创建的正则化方法
- weights_list: 想要执行正则化方法的参数列表,如果为None的话,就取GraphKeys.WEIGHTS中的weights.

函数返回一个标量Tensor,同时,这个标量Tensor也会保存到GraphKeys.REGULARIZATION_LOSSES中.这个Tensor保存了计算正则项损失的方法.

现在,我们只需将这个正则项损失加到我们的损失函数上就可以了.

> 如果是自己手动定义weight的话,需要手动将weight保存到GraphKeys.WEIGHTS中,但是如果使用layer的话,就不用这么麻烦了,别人已经帮你考虑好了.(最好自己验证一下tf.GraphKeys.WEIGHTS中是否包含了所有的weights,防止被坑)

# 其它

在使用tf.get_variable()和tf.variable_scope()的时候,你会发现,它们俩中有regularizer形参.如果传入这个参数的话,那么variable_scope内的weights的正则化损失,或者weights的正则化损失就会被添加到GraphKeys.REGULARIZATION_LOSSES中. 
示例:

``` python
import tensorflow as tf
from tensorflow.contrib import layers

regularizer = layers.l1_regularizer(0.1)

with tf.variable_scope('var', initializer=tf.random_normal_initializer(), 
regularizer=regularizer):
    weight = tf.get_variable('weight', shape=[8], initializer=tf.ones_initializer())
with tf.variable_scope('var2', initializer=tf.random_normal_initializer(), 
regularizer=regularizer):
    weight2 = tf.get_variable('weight', shape=[8], initializer=tf.ones_initializer())

regularization_loss = tf.reduce_sum(tf.get_collection(tf.GraphKeys.REGULARIZATION_LOSSES))
```