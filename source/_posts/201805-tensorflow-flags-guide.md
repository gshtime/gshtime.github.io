---
title: tensorflow 中的 tf.app.flags
date: 2018-05-21 20:29:51
tags:
---

tf定义了tf.app.flags，用于支持接受命令行传递参数，相当于接受argv

``` python
import tensorflow as tf
flags = tf.app.flags #flags是一个文件：flags.py，用于处理命令行参数的解析工作

# 第一个是参数名称，第二个参数是默认值，第三个是参数描述
flags.DEFINE_string("para_string", "default_val", "description") # 定义一个 string
flags.DEFINE_integer("pare_int", 1000, "this is an integer") # 定义一个integer
flags.DEFINE_bool("para_bool", True, "description") # 定义一个 bool

# FLAGS是一个对象，保存了解析后的命令行参数
FLAGS = flags.FLAGS
def main(_):
    print FLAGS.para_string #调用命令行输入的参数

if __name__ = "__main__": # 使用这种方式保证了，如果此文件被其它文件import的时候，不会执行main中的代码
    tf.app.run() # 解析命令行参数，调用main函数 main(sys.argv)
```

传入参数方法：

``` bash
$ python script.py --para_1=value1 --para_2=value2
# 不传的话，会使用默认值,注意等号左右没有空格
```