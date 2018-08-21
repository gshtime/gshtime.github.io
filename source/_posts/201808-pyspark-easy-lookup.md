---
title: 【Spark 使用教程】SparkSession
date: 2018-08-02 19:24:25
categories: spark
tags:
    - pyspark
    - 大数据
---
# SparkSession的功能
Spark2.0中引入了SparkSession的概念，它为用户提供了一个统一的切入点来使用Spark的各项功能，用户不但可以使用DataFrame和Dataset的各种API，学习Spark的难度也会大大降低。

Spark REPL和Databricks Notebook中的SparkSession对象
在之前的Spark版本中，Spark shell会自动创建一个SparkContext对象sc。2.0中Spark shell则会自动创建一个SparkSession对象（spark），在输入spark时就会发现它已经存在了。

![](http://p8vrqzrnj.bkt.clouddn.com/20160823134504487.png)

在Databricks notebook中创建集群时也会自动生成一个SparkSession，这里用的名字也是spark。

# SparkContext

![](http://p8vrqzrnj.bkt.clouddn.com/20160823134731334.png)

SparkContext起到的是一个中介的作用，通过它来使用Spark其他的功能。每一个JVM都有一个对应的SparkContext，driver program通过SparkContext连接到集群管理器来实现对集群中任务的控制。Spark配置参数的设置以及对SQLContext、HiveContext和StreamingContext的控制也要通过SparkContext进行。

不过在Spark2.0中上述的一切功能都是通过SparkSession来完成的，同时SparkSession也简化了DataFrame/Dataset API的使用和对数据的操作。

# pyspark.sql.SparkSession

https://blog.csdn.net/cjhnbls/article/details/79254188

``` python
ctx = SparkSession.builder.appName("ApplicationName").config("spark.driver.memory", "6G").master('local[7]').getOrCreate()
```

# Scala API

## 创建SparkSession

在2.0版本之前，使用Spark必须先创建SparkConf和SparkContext，代码如下：

``` scala
//set up the spark configuration and create contexts
val sparkConf = new SparkConf().setAppName("SparkSessionZipsExample").setMaster("local")
// your handle to SparkContext to access other context like SQLContext
val sc = new SparkContext(sparkConf).set("spark.some.config.option", "some-value")
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
```

不过在Spark2.0中只要创建一个SparkSession就够了，SparkConf、SparkContext和SQLContext都已经被封装在SparkSession当中。下面的代码创建了一个SparkSession对象并设置了一些参数。这里使用了生成器模式，只有此“spark”对象不存在时才会创建一个新对象。

``` scala
// Create a SparkSession. No need to create SparkContext
// You automatically get it as part of the SparkSession
val warehouseLocation = "file:${system:user.dir}/spark-warehouse"
val spark = SparkSession
   .builder()
   .appName("SparkSessionZipsExample")
   .config("spark.sql.warehouse.dir", warehouseLocation)
   .enableHiveSupport()
   .getOrCreate()
```
执行完上面的代码就可以使用spark对象了。

## 设置运行参数

创建SparkSession之后可以设置运行参数，代码如下, 也可以使用Scala的迭代器来读取configMap中的数据。

``` scala
//set new runtime options
spark.conf.set("spark.sql.shuffle.partitions", 6)
spark.conf.set("spark.executor.memory", "2g")
//get all settings
val configMap:Map[String, String] = spark.conf.getAll()
```

## 读取元数据

如果需要读取元数据（catalog），可以通过SparkSession来获取。
``` scala
//fetch metadata data from the catalog
spark.catalog.listDatabases.show(false)
spark.catalog.listTables.show(false)
```
这里返回的都是Dataset，所以可以根据需要再使用Dataset API来读取。 ???

## 创建Dataset和Dataframe

通过SparkSession来创建Dataset和Dataframe有多种方法。其中最简单的就是使用spark.range方法来生成Dataset，在摸索Dataset API的时候这个办法尤其有用。

``` scala
// create a Dataset using spark.range starting from 5 to 100, with increments of 5
val numDS = spark.range(5, 100, 5)
// reverse the order and display first 5 items
numDS.orderBy(desc("id")).show(5)
//compute descriptive stats and display them
numDs.describe().show()
// create a DataFrame using spark.createDataFrame from a List or Seq
val langPercentDF = spark.createDataFrame(List(("Scala", 35), ("Python", 30), ("R", 15), ("Java", 20)))
//rename the columns
val lpDF = langPercentDF.withColumnRenamed("_1", "language").withColumnRenamed("_2", "percent")
// order the DataFrame in descending order of percentage
lpDF.orderBy(desc("percent")).show(false)
```

## 读取JSON数据

此外，还可以用SparkSession读取JSON、CSV、TXT和parquet表。下面的代码中读取了一个JSON文件，返回的是一个DataFrame。

``` scala
// read the json file and create the dataframe
val jsonFile = args(0)
val zipsDF = spark.read.json(jsonFile)
//filter all cities whose population > 40K
zipsDF.filter(zipsDF.col("pop") > 40000).show(10)
```

## 使用SparkSQL

借助SparkSession用户可以像SQLContext一样使用Spark SQL的全部功能。下面的代码中先创建了一个表然后对此表进行查询。

``` scala
// Now create an SQL table and issue SQL queries against it without
// using the sqlContext but through the SparkSession object.
// Creates a temporary view of the DataFrame
zipsDF.createOrReplaceTempView("zips_table")
zipsDF.cache()
val resultsDF = spark.sql("SELECT city, pop, state, zip FROM zips_table")
resultsDF.show(10)
```

## 存储/读取Hive表

下面的代码演示了通过SparkSession来创建Hive表并进行查询的方法。

``` scala
//drop the table if exists to get around existing table error
spark.sql("DROP TABLE IF EXISTS zips_hive_table")
//save as a hive table
spark.table("zips_table").write.saveAsTable("zips_hive_table")
//make a similar query against the hive table 
val resultsHiveDF = spark.sql("SELECT city, pop, state, zip FROM zips_hive_table WHERE pop > 40000")
resultsHiveDF.show(10)
```

这里可以看到从DataFrame API、Spark SQL和Hive语句返回的结果是完全相同的。