---
title: python pandas 使用指南
date: 2018-08-24 10:14:58
tags: 
    - 工具箱
---

# Series 操作

``` python 
# 获取所有的值
In [27]: ser2.values
Out[27]: array([0, 1, 2, 3])
# 获取所有的索引
In [28]: ser2.index
Out[28]: Index([u'a', u'b', u'c', u'd'], dtype='object')
```

# 按行遍历DataFrame

## 不修改内容

要以 Pandas 的方式迭代遍历DataFrame的行，可以使用：


``` python
# 1、DataFrame.iterrows()
for index, row in df.iterrows():  # 返回（索引， Series）元组
    print row["c1"], row["c2"]

# 2 DataFrame.itertuples()
for row in df.itertuples(index=True, name='Pandas'):
    print getattr(row, "c1"), getattr(row, "c2")
```

itertuples()应该比iterrows()快

但请注意，根据文档(目前 Pandas 0.19.1)：

df.iterrows：数据的dtype可能不是按行匹配的，因为iterrows返回一个系列的每一行，它不会保留行的dtypes(dtypes跨DataFrames列保留)*

iterrows：不要修改行，你不应该修改你正在迭代的东西。这不能保证在所有情况下都能正常工作。根据数据类型的不同，迭代器返回一个副本而不是一个视图，写入它将不起作用。

## 第二种方案: apply()

DataFrame.apply(),对每行的内容进行修改

``` python
def valuation_formula(x, y):
    return x * y * 0.5
 
df['price'] = df.apply(lambda row: valuation_formula(row['x'], row['y']), axis=1)
```

## 方案3：df.iloc函数

``` python 
for i in range(0, len(df)):
    print df.iloc[i]['c1'], df.iloc[i]['c2']
```

## 第四种方案：略麻烦，但是更高效，将DataFrame转为List
可以编写自己的实现namedtuple的迭代器,这相当于pd.DataFrame.itertuples，但是效率更高。

``` python
from collections import namedtuple
 
def myiter(d, cols=None):
    if cols is None:
        v = d.values.tolist()
        cols = d.columns.values.tolist()
    else:
        j = [d.columns.get_loc(c) for c in cols]
        v = d.values[:, j].tolist()
 
    n = namedtuple('MyTuple', cols)
 
    for line in iter(v):
        yield n(*line)
```

``` python 
# 将自定义函数用于给定的DataFrame：
list(myiter(df))
### [MyTuple(c1=10, c2=100), MyTuple(c1=11, c2=110), MyTuple(c1=12, c2=120)]

# 或与pd.DataFrame.itertuples：
list(df.itertuples(index=False))
### [Pandas(c1=10, c2=100), Pandas(c1=11, c2=110), Pandas(c1=12, c2=120)]
```