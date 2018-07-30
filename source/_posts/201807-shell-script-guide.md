---
title: shell 脚本note
date: 2018-07-30 10:30:49
tags:
---

# Linux Shell中三种引号的用法及区别

Linux Shell中有三种引号，分别为双引号（" "）、单引号(' ')以及反引号(\` \`)。

单引号不进行替换，将字符串中所有字符作为普通字符输出，而反引号中字符串作为shell命令执行，并返回执行结果。具体含义如下：

- 双引号（" "）：对字符串中出现的 $, '', `和\ 进行替换，除此以外所有的字符都解释成字符本身。
- 单引号（' '）：不替换特殊字符（$,'',`和\），将字符串中所有字符作为普通字符输出。
- 反引号（\` \`）：字符串作为shell命令执行，并返回执行结果。

# Shell的for循环语句

第一类：数字性循环
``` shell
# 1
for((i=1;i<=10;i++));  
do   
echo $(expr $i \* 3 + 1);  
done  

# 2 
for i in $(seq 1 10)  
do   
echo $(expr $i \* 3 + 1);  
done   

# 3
for i in {1..10}  
do  
echo $(expr $i \* 3 + 1);  
done  

# 4
awk 'BEGIN{for(i=1; i<=10; i++) print i}'  
```

第二类：字符性循环

``` bash
# 1
for i in `ls`;  
do   
echo $i is file name\! ;  
done 

# 2
for i in $* ;  
do  
echo $i is input chart\! ;  
done  

# 3
for i in f1 f2 f3 ;  
do  
echo $i is appoint ;  
done  

# 4
list="rootfs usr data data2"  
for i in $list;  
do  
echo $i is appoint ;  
done  
```

第三类：路径查找
``` shell
for file in /proc/*;  
do  
echo $file is file path \! ;  
done  

for file in $(ls *.sh)  
do  
echo $file is file path \! ;  
done  
```
现在一般都使用for in结构，for in结构后面可以使用函数来构造范围，比如$()、这些，里面写一些查找的语法，比如ls test*，那么遍历之后就是输出文件名了。