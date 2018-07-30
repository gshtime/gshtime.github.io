---
title: scons - cpp编译工具学习
date: 2018-07-10 00:05:12
tags:
---

# scons是什么 

[office guide](https://scons.org/)

SCons 是一个开放源代码、以 Python 语言编写的下一代的程序建造工具。功能上类似于make。 

一个单个文件的程序是不需要scons和make之类的构建工具的，只要用gcc或者g++编译就好。但是一些相对较大的项目有多个文件，各个文件之间的依赖关系复杂，如果用g++来编译就会非常复杂，不仅要写的命令多，而且容易出错，所以就出现了make，但是make可能也存在某些问题，就出现了scons。

总之，这两种工具解决构建的方法就是用一个配置文件来记录下各个文件之间的依赖关系，用到了那些库，配置好环境变量等等，然后直接构建。scons并不是和g++一样的编译工具，而是在g++的基础上的工具。

# scons的优势

- 使用 Python 脚本做为配置文件
- 对于 C,C++ 和 Fortran, 内建支持可靠自动依赖分析 . 不用像 make 工具那样需要 执行”make depends”和”make clean”就可以获得所有的依赖关系。
- 内建支持 C, C++, D, Java, Fortran, Yacc, Lex, Qt，SWIG 以及 Tex/Latex。 用户还可以根据自己的需要进行扩展以获得对需要编程语言的支持。
- 支持 make -j 风格的并行建造。相比 make -j, SCons 可以同时运行 N 个工作，而 不用担心代码的层次结构。
- 使用 Autoconf 风格查找头文件，函数库，函数和类型定义。 
- 良好的夸平台性。SCons 可以运行在 Linux, AIX, BSD, HP/UX, IRIX, Solaris, Windows, Mac OS X 和 OS/2 上。

# scons的配置文件

SConstruct是scons的配置文件，是用python编写的，自然要遵守python语法

``` bash 
# 指定SConstruct
scons -f <SConstruct file>

```

# scons命令

``` bash
scons -Q # 减少编译时的由 scons 产生的冗余信息，关闭一些输出提示
scons -c # scons -clean，根据SConstruct文件，执行清理任务
```

# SConstruct 文件常用函数

## Program()：生成可执行文件

``` python
Program(target, source, libs) 
"""
编译hello.c并生成.o文件和可执行文件
target:编译的目标文件名 
source:需要编译的文件组 
libs:  需要的所有库 
"""
Program('hello.c')  # 编译hello.c为可执行文件,文件名为第一个文件名
Program('main', 'hello.c')  # 编译hello.c为可执行文件,文件名mian
Program('program', ['prog.c', 'file1.c', 'file2.c'])
# 也可以利用Glob函数获得名字列表
# Golb('*.c')返回规则匹配的string列表
# 就是类似上面的'prog.c', 'file1.c', 'file2.c'
Program('program', Glob('*.c') )

Program(target = 'main', source = 'hello.c')  # 使用关键字指明编译对象
```
## Object()：生成目标文件。如：

``` python
Object('hello.c')          # 编译hello.c，但只生成生成.o文件
Object('main', 'hello.c')  # 编译hello.c 为目标文件，文件名为main.o
```

## 生成库文件

``` python
Library()        # 生成库，默认为静态库。

StaticLibrary()  # 生成静态库 .a

ShareLibrary()   # 生成动态库 .so

```

# scons 关键字

编译相关关键字说明

## 基本的编译关键字

| 关键字     | 说明                                            |
| ---------- | ----------------------------------------------- |
| CXXFLAGS   | 编译参数                                        |
| CPPDEFINES | 指定预编译器 CPPDEFINES={'RELEASE_BUILD' : '1'} |
| LIBS       | 指定所需要链接的库文件                          |
| LIBPATH    | 指定库文件(.lib)的搜索目录                      |
| CPPPATH    | 指定[.h, .c, .cpp]等文件搜索路径，头文件路径    |

## 指定编译选项

``` python 
Program(target = 'bind_test1',
    source = [“bind_test1.cc”],
    LIBS = ['boost_system','boost_filesystem', 'boost_thread'],
    LIBPATH = ['./', '/usr/local/lib/' ],
    CPPPATH = ['./', '/usr/local/include/'],
    CCFLAGS = ['-g','-O3′] ,
    CPPDEFINES={'RELEASE_BUILD' : '1'}
)
```

> 注：LIBS和LIBPATH若为一个可以使用字符串，若为多个则使用列表

# scons进阶

对于scons来说，环境变量(Environment)是异常重要的，只有会用Environment，才能写出真正可以实际使用的scons脚本。

## 构造环境变量

由自己创建的环境变量，这个环境变量可以用来控制整个目标文件编译过程。

一种构造环境变量的目的就是交叉编译器的设置，由于scons默认用的是gcc来编译系统，而嵌入式开发要使用指定的交叉编译器来编译目标文件，所以可通过构造环境变量来修改编译器。

a) 创建

``` python
env = Environment(CC = 'arm-linux-gcc')
```

b) 使用

用arm-linux-gcc编译执行文件

`env.Program('hello.c')`

c) 从构造环境变量获取值：

`print "CC is:", env['CC']`

d) 拷贝

我们想使用gcc编译2个不同版本的debug/realse版本的程序，就可以通过拷贝创建2个环境变量

``` python
debug = env.Clone(CCFLAGS = '-g')

Realse = env.Clone(CCFLAGS = '-O2')

debug.Program('debug','hello.c')

debug.Program('realse','hello.c')
```
 
e) 追加到末尾：Append()

`env.Append(CCFLAGS = '-DLAST')`

f) 追加到开始位置：Prepend()

`env.Prepend(CCLFAGS = '-DFIRST')`

## Python的os模块使用

使用os模块，需要在每个脚本文件添加`import os`

``` python
import os
os.name      # 获取当前正在使用的平台，linux显示为'posix',windows返回的是'nt'
os.getcwd()  # 获取当前工作目录
os.getenv()  # 读取环境变量
os.putenv()  # 设置环境变量
os.listdir() # 返回指定目录下的所有文件和目录
os.remove()  # 删除一个文件
os.system()  # 运行shell命令
```

# ARGUMENTS.get()

Values of variables to be passed to the SConscript file(s) may be specified on the command line:

``` bash
scons debug=1 .
```

These variables are available in SConscript files through the ARGUMENTS dictionary, and can be used in the SConscript file(s) to modify the build in any way:

``` python
if ARGUMENTS.get('debug', 0):
    env = Environment(CCFLAGS = '-g')
else:
    env = Environment()
```

The command-line variable arguments are also available in the ARGLIST list, indexed by their order on the command line. This allows you to process them in order rather than by name, if necessary. ARGLIST[0] returns a tuple containing (argname, argvalue). A Python exception is thrown if you try to access a list member that does not exist.

