---
title: scala array list tuple and so on
date: 2018-05-20 10:11:31
tags:
---

# Array 类型参数化数组

scala Array 的初始化

``` java
// 简洁的方法创造和初始化：
val numNames = Array("zero", "one", "two")

// 更罗嗦的调用 apply 方法：
val numNames2 = Array.apply("zero", "one", "two") 
```

``` java
object HelloWorld 
{

    def main(args: Array[String]) 
    {
      val greetStrings = new Array[String](3)  

      greetStrings(0) = "Scala: Hello" 
      greetStrings(1) = ", " 
      greetStrings(2) = "world!\n" 
      for (i <- 0 to 2) 
           print(greetStrings(i)) 
    }

}
// 输出结果：Scala: Hello, world!
```

Scala里的数组是通过把索引放在圆括号里面访问的，而不是像Java那样放在方括号里。所以数组的第零个元素是greetStrings(0)，不是greetStrings[0]。

`val` 的概念: 当你用val定义一个变量，那么这个变量就不能重新赋值，但它指向的对象却仍可以改变。

> 在本例中，你不能把greetStrings重新赋值成不同的数组；greetStrings将永远指向那个它被初始化时候指向的同一个Array[String]实例。但是你能一遍遍修改那个Array[String]的元素，因此数组本身是可变的。

# 使用列表【List】

Scala `Array` 数组是一个所有对象都共享相同类型的可变序列。比方说Array[String]仅包含String。尽管实例化之后你无法改变Array的长度，它的元素值却是可变的。因此，Array是可变的对象。

Scala的List类是共享相同类型的不可变对象序列。

和数组一样，List[String]包含的仅仅是String。 
Scala的List不同于Java的java.util.List，总是不可变的（而Java的List可变）。

``` java
// 创建一个Scala的List很简单
val oneTwoThree = List(1, 2, 3)

```
上述代码完成了一个新的叫做oneTwoThree的val，并已经用带有整数元素值1，2和3的新List[Int]初始化。

## List 操作

这个List可以这么用：

``` java
val oneTwo = List(1, 2)  
val threeFour = List(3, 4) 
val oneTwoThreeFour = oneTwo ::: threeFour  // ::: 拼接List

println(oneTwo + " 和 " + threeFour + " 是不可变的")  
println(oneTwoThreeFour + " 是个新列表了")

// 运行结果:
// List(1, 2) 和 List(3, 4) 是不可变的
// List(1, 2, 3, 4) 是个新列表了
```

List最常用的操作符是发音为”cons”的” :: “. 例如，

``` java
val twoThree = List(2, 3)
val oneTwoThree = 1 :: twoThree  // 拼接元素 和 List
println(oneTwoThree) 

// 结果为：List(1, 2, 3) 
```

类List没有提供append操作。 
如果你想通过添加元素来构造列表： 
- 前缀进去，完成之后再调用reverse； 
- 使用ListBuffer，一种提供append操作的可变列表，完成之后调用toList。

# 使用元组【Tuple】

另一种有用的容器对象是元组：tuple。与列表一样，元组也是不可变的，但与列表不同，元组可以包含不同类型的元素。

列表应该是List[Int]或List[String]的样子，元组可以同时拥有Int和String。

Scala里你可以简单地返回一个元组。 
而且这么做的确简单：实例化一个装有一些对象的新元组，只要把这些对象放在括号里，并用逗号分隔即可。 
一旦你已经实例化了一个元组，你可以用点号，下划线和一个基于1的元素索引访问它。

一个例子：

``` java
val pair = (99, "Luftballons")  //Scala推断元组类型为Tuple2[Int, String]，并把它赋给变量pair。
println(pair._1)                //访问_1字段，从而输出第一个元素，99。
println(pair._2)                

// 运行结果
// 99
// Luftballons
```

元组第一个元素是以99为值的Int，第二个是”luftballons”为值的String。

元组的实际类型取决于它含有的元素数量和这些元素的类型。 
因此，(99, “Luftballons”)的类型是Tuple2[Int, String]。

类似地，(‘u’, ‘r’, ‘the’, 1, 4, “me”)是Tuple6[Char, Char, String, Int, Int, String]。

## 访问元组的元素

为什么你不能像访问List里的元素那样访问元组的，就像pair(0)？ 
因为List的apply方法始终返回同样的类型，但是元组里的或许类型不同。 
_1可以有一个结果类型，_2是另外一个。 

> 另：元组元素编号从1开始。

# 使用Set和Map

当问题讨论到集和映射，Scala同样提供了可变和不可变的替代品，不过用了不同的办法。

对于集和映射，Scala把可变性建模在类继承中。

例如，Scala的API包含了集的一个基本特质：trait，特质这个概念接近于Java的接口。

Scala于是提供了两个子特质，一个是可变的集，另一个是不可变的集。这三个特质都共享同样的简化名，Set。

如果你想要使用HashSet，你可以根据你的需要选择可变的或不可变的变体。

创造集的缺省方法实例：

``` java
var jetSet = Set("Boeing", "Airbus")  //定义了名为jetSet的新var，包含两个字串
jetSet += "Lear"                      // jetSet = jetSet + "Lear" 
println(jetSet.contains("Cessna"))    //打印输出集是否包含字串"Cessna"。
println(jetSet.contains("Lear"))      //打印输出集是否包含字串"Lear"。

// 运行结果：
// false
// true
```

需要不可变集，就需要使用一个引用：import，如下所示：

``` java
import scala.collection.mutable.Set  

val movieSet = Set("Hitch", "Poltergeist")  
movieSet += "Shrek" 
println(movieSet)  

// 运行结果：
// Set(Poltergeist, Shrek, Hitch)
```

需要一个不可变的HashSet，你可以这么做：

``` java
import scala.collection.immutable.HashSet  
val hashSet = HashSet("Tomatoes", "Chilies")  
println(hashSet + "Coriander") 

// 运行结果
// Set(Chilies, Tomatoes, Coriander)
```

Map是Scala里另一种有用的集合类。 
和集一样，Scala采用了类继承机制提供了可变的和不可变的两种版本的Map。

`scala.collection` 包里面有一个基础Map特质和两个子特质Map： 
可变的Map在scala.collection.mutable里，不可变的在scala.collection.immutable里。

可变映射的创造过程：

``` java
import scala.collection.mutable.Map  

val treasureMap = Map[Int, String]()  
treasureMap += (1 -> "我在")  
treasureMap += (2 -> "学习")  
treasureMap += (3 -> "Scala")  
println(treasureMap(1) + treasureMap(2) + treasureMap(3)) 

// 运行结果：
// 我在学习Scala.
```

至于不可变映射，就不用引用任何类了，因为不可变映射是缺省的，代码例子：

``` java
val romanNumeral = Map(      
        1 -> "我", 2 -> "是", 3 -> "缺", 4 -> "省", 5 -> "的" )  
println(romanNumeral(1) + romanNumeral(2) + romanNumeral(3) + romanNumeral(4) + romanNumeral(5))  

// 运行结果：
// 我是缺省的
```