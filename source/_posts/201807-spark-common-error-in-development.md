---
title: spark 开发中遇到的一些问题总结
date: 2018-07-30 00:30:07
tags:
---

不知道spark怎么调bug，没有系统学习过，摸着石头过河吧。

# 卡在一个excutor上，没有报错

因为一台机器的内存分配给越多的executor，每个executor的内存就越小，以致出现过多的数据spill over甚至out of memory的情况。
把这个参数调大些试试:spark.shuffle.memoryFraction

参数说明：该参数用于设置shuffle过程中一个task拉取到上个stage的task的输出后，进行聚合操作时能够使用的Executor内存的比例，默认是0.2。也就是说，Executor默认只有20%的内存用来进行该操作。shuffle操作在进行聚合时，如果发现使用的内存超出了这个20%的限制，那么多余的数据就会溢写到磁盘文件中去，此时就会极大地降低性能。

参数调优建议：如果Spark作业中的RDD持久化操作较少，shuffle操作较多时，建议降低持久化操作的内存占比，提高shuffle操作的内存占比比例，避免shuffle过程中数据过多时内存不够用，必须溢写到磁盘上，降低了性能。此外，如果发现作业由于频繁的gc导致运行缓慢，意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值。

设置了参数，如下，可以跑完，但是变得很慢。

``` java
SparkConf sc = new SparkConf()
.setAppName("SparkCalculateSR")
.set("spark.storage.memoryFraction", "0.2")
.set("spark.default.parallelism", "20")
.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
.set("spark.shuffle.consolidateFiles", "true")  // consolidateFiles这个参数是hashshuffle的时候用的
.set("spark.reducer.maxSizeInFlight", "100m")
.set("spark.shuffle.file.buffer", "100k")
.set("spark.shuffle.io.maxRetries", "10")
.set("spark.shuffle.io.retryWait", "10s");
```

继续建议：

上边设置的参数可以提高shuffle的稳定性,所以是跑成功了.如果要增大shuffle使用executor内存可以调下边两个参数
- num-executors 100 --这个调小
- spark.shuffle.memoryFraction --这个调大 

不知道具体慢在哪了,所以没法给具体的优化建议.你采用的是hashshuffle吗? consolidateFiles这个参数是hashshuffle的时候用的,要不改成SortShuffle试试,一般慢都慢在shuffle上了

## ref

[spark 卡住](https://bbs.csdn.net/topics/392142267)

# spark 配置计算

``` s
--num-executors 100 \
--driver-memory 6g \
--executor-memory 6g \
--executor-cores 8 \
```

100个executors  一个executor-memory 6G内存  8核cpu   那得多少内存多少cpu啊

答案：600g内存， 800核

# 参考：[开发中遇到的一些问题](https://www.cnblogs.com/arachis/p/Spark_prog.html) 文中有很多之前碰到的问题

1.StackOverflowError

问题：简单代码记录 :

for (day <- days){

　　rdd = rdd.union(sc.textFile(/path/to/day) .... )

}

大概场景就是我想把数量比较多的文件合并成一个大rdd,从而导致了栈溢出；

解决：很明显是方法递归调用太多，我之后改成了几个小任务进行了合并；这里union也会造成最终rdd分区数过多

2.java.io.FileNotFoundException: /tmp/spark-90507c1d-e98 ..... temp_shuffle_98deadd9-f7c3-4a12(No such file or directory) 类似这种 

报错：Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 76.0 failed 4 times, most recent failure: Lost task 0.3 in stage 76.0 (TID 341, 10.5.0.90): java.io.FileNotFoundException: /tmp/spark-90507c1d-e983-422d-9e01-74ff0a5a2806/executor-360151d5-6b83-4e3e-a0c6-6ddc955cb16c/blockmgr-bca2bde9-212f-4219-af8b-ef0415d60bfa/26/temp_shuffle_98deadd9-f7c3-4a12-9a30-7749f097b5c8 (No such file or directory)

场景：大概代码和上面差不多：

for (day <- days){

　　rdd = rdd.union(sc.textFile(/path/to/day) .... )

}

rdd.map( ... )

解决：简单的map都会报错，怀疑是临时文件过多；查看一下rdd.partitions.length 果然有4k多个；基本思路就是减少分区数

可以在union的时候就进行重分区：

for (day <- days){

　　rdd = rdd.union(sc.textFile(/path/to/day,numPartitions) .... )

　　rdd = rdd.coalesce(numPartitions)

} //这里因为默认哈希分区，并且分区数相同；所有最终union的rdd的分区数不会增多,贴一下源码以防说错

``` scala
/** Build the union of a list of RDDs. */
 def union[T: ClassTag](rdds: Seq[RDD[T]]): RDD[T] = withScope {
   val partitioners = rdds.flatMap(_.partitioner).toSet
   if (rdds.forall(_.partitioner.isDefined) && partitioners.size == 1) {
     /*这里如果rdd的分区函数都相同则会构建一个PartitionerAwareUnionRDD：m RDDs with p partitions each
* will be unified to a single RDD with p partitions*/
     new PartitionerAwareUnionRDD(this, rdds)
   } else {
     new UnionRDD(this, rdds)
   }
 }
```

或者最后在重分区

for (day <- days){

　　rdd = rdd.union(sc.textFile(/path/to/day) .... )

} 

rdd.repartition(numPartitions)


# Spark Shuffle FetchFailedException解决方案

在大规模数据处理中，这是个比较常见的错误。

报错提示

SparkSQL shuffle操作带来的报错

``` java
org.apache.spark.shuffle.MetadataFetchFailedException: 
Missing an output location for shuffle 0

org.apache.spark.shuffle.FetchFailedException:
Failed to connect to hostname/192.168.xx.xxx:50268

RDD的shuffle操作带来的报错
WARN TaskSetManager: Lost task 17.1 in stage 4.1 (TID 1386, spark050013): java.io.FileNotFoundException: /data04/spark/tmp/blockmgr-817d372f-c359-4a00-96dd-8f6554aa19cd/2f/temp_shuffle_e22e013a-5392-4edb-9874-a196a1dad97c (没有那个文件或目录)

FetchFailed(BlockManagerId(6083b277-119a-49e8-8a49-3539690a2a3f-S155, spark050013, 8533), shuffleId=1, mapId=143, reduceId=3, message=
org.apache.spark.shuffle.FetchFailedException: Error in opening FileSegmentManagedBuffer{file=/data04/spark/tmp/blockmgr-817d372f-c359-4a00-96dd-8f6554aa19cd/0e/shuffle_1_143_0.data, offset=997061, length=112503}
```

原因

shuffle分为shuffle write和shuffle read两部分。 
shuffle write的分区数由上一阶段的RDD分区数控制，shuffle read的分区数则是由Spark提供的一些参数控制。

shuffle write可以简单理解为类似于saveAsLocalDiskFile的操作，将计算的中间结果按某种规则临时放到各个executor所在的本地磁盘上。

shuffle read的时候数据的分区数则是由spark提供的一些参数控制。可以想到的是，如果这个参数值设置的很小，同时shuffle read的量很大，那么将会导致一个task需要处理的数据非常大。结果导致JVM crash，从而导致取shuffle数据失败，同时executor也丢失了，看到Failed to connect to host的错误，也就是executor lost的意思。有时候即使不会导致JVM crash也会造成长时间的gc。

解决办法

知道原因后问题就好解决了，主要从shuffle的数据量和处理shuffle数据的分区数两个角度入手。

减少shuffle数据

思考是否可以使用map side join或是broadcast join来规避shuffle的产生。

将不必要的数据在shuffle前进行过滤，比如原始数据有20个字段，只要选取需要的字段进行处理即可，将会减少一定的shuffle数据。

SparkSQL和DataFrame的join,group by等操作

通过spark.sql.shuffle.partitions控制分区数，默认为200，根据shuffle的量以及计算的复杂度提高这个值。

Rdd的join,groupBy,reduceByKey等操作

通过spark.default.parallelism控制shuffle read与reduce处理的分区数，默认为运行任务的core的总数（mesos细粒度模式为8个，local模式为本地的core总数），官方建议为设置成运行任务的core的2-3倍。

提高executor的内存

通过spark.executor.memory适当提高executor的memory值。

是否存在数据倾斜的问题

空值是否已经过滤？异常数据（某个key数据特别大）是否可以单独处理？考虑改变数据分区规则。

spark 分区详解 shuffle过程


1、报错：ERROR storage.DiskBlockObjectWriter: Uncaught exception while reverting partial writes to file /hadoop/application_1415632483774_448143/spark-local-20141127115224-9ca8/04/shuffle_1_1562_27

java.io.FileNotFoundException: /hadoop/application_1415632483774_448143/spark-local-20141127115224-9ca8/04/shuffle_1_1562_27 (No such file or directory)

　　表面上看是因为shuffle没有地方写了，如果后面的stack是local space 的问题，那么清一下磁盘就好了。上面这种问题，是因为一个excutor给分配的内存不够，此时，减少excutor-core的数量，加大excutor-memory的值应该就没有问题。

2、报错：ERROR executor.CoarseGrainedExecutorBackend: Driver Disassociated [akka.tcp://sparkExecutor@pc-jfqdfx31:48586] -> [akka.tcp://sparkDriver@pc-jfqdfx30:41656] disassociated! Shutting down.
15/07/23 10:50:56 ERROR executor.CoarseGrainedExecutorBackend: RECEIVED SIGNAL 15: SIGTERM

　　这个错误比较隐晦，从信息上看来不知道是什么问题，但是归根结底还是内存的问题，有两个方法可以解决这个错误，一是，如上面所说，加大excutor-memory的值，减少executor-cores的数量，问题可以解决。二是，加大executor.overhead的值，但是这样其实并没有解决掉根本的问题。所以如果集群的资源是支持的话，就用1的办法吧。

　　另外，这个错误也出现在partitionBy(new HashPartition(partiton-num))时，如果partiton-num太大或者太小的时候会报这种错误，说白了也是内存的原因，不过这个时候增加内存和overhead没有什么用，得去调整这个partiton-num的值。

配置详解