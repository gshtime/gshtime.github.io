---
title: python 字符串处理
date: 2018-08-01 00:02:02
tags:
---
# Refs

http://www.runoob.com/python/python-strings.html

# 字符串转义

| 转义字符      | 描述   |
| ------------- | ------ |
| `\`(在行尾时) | 续行符 |

# Python字符串运算符
下表实例变量 a 值为字符串 "Hello"，b 变量值为 "Python"：

``` python
# + 字符串连接	
a + b
'HelloPython'
# * 重复输出字符串	
a * 2
'HelloHello'
# in 成员运算符:如果字符串中包含给定的字符返回 True	
"H" in a
True
# r/R 原始字符串 - 原始字符串：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母"r"（可以大小写）以外，与普通字符串有着几乎完全相同的语法。	
print r'\n'
\n
print R'\n'
\n
```

# 内建函数

## find()方法

``` python
str.find(str, beg=0, end=len(string))

```

参数
- str -- 指定检索的字符串
- beg -- 开始索引，默认为0。
- end -- 结束索引，默认为字符串的长度。

返回值
如果包含子字符串返回开始的索引值，否则返回-1。

## index()方法

描述

Python index() 方法检测字符串中是否包含子字符串 str ，如果指定 beg（开始） 和 end（结束） 范围，则检查是否包含在指定范围内，该方法与 python find()方法一样，只不过如果str不在 string中会报一个异常。

语法

``` python
str.index(str, beg=0, end=len(string))
```

参数
- str -- 指定检索的字符串
- beg -- 开始索引，默认为0。
- end -- 结束索引，默认为字符串的长度。
  
返回值

如果包含子字符串返回开始的索引值，否则抛出异常。

## 其他方法

| 方法                                      | 描述                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| string.rfind(str, beg=0,end=len(string) ) | 类似于 find()函数，不过是从右边开始查找.                     |
| string.zfill(width)                       | 返回长度为 width 的字符串，原字符串 string 右对齐，前面填充0 |