---
title: pandas series
tags:
  - Pandas
date: 2019-07-13 11:45:38
---


Pandas.Series 是一个一维的带标签数组，可以保存任何类型的数据 (整数，字符串，浮点数，Python对象等)，轴标签统称为索引。

Series 的索引既可以是基于数字的，如 [1, 2, 3, 4, 5]， 也可以是基于标记的，如 ['a', 'b', 'c', 'd']，同时重复的索引值也是合理的，如 ['a', 'a', 'b']。

## 构造函数

```python
pandas.Series(data=None, index=None, dtype=None, name=None, copy=False, fastpath=False)
```

| 编号 | 参数 | 描述 |
|------|----------|------------------------------------------------------------------------|
| 1 | data | Series 中要存储的数据，可以是类数组类型、可迭代类型、字典类型、标量 |
| 2 | index | Series 中数据的索引，需要和数据的长度相同，默认为 np.arange(n) |
| 3 | dtype | Series 的数据类型，如果没有，将推断数据类型 |
| 4 | copy | 是否从输入中复制数据，默认为 false |
| 5 | name | Series 名称 |
| 6 | fastpath |  |

### 创建空的 Series

不提供任何参数，则可以创建一个空的 Series。

```python
In [1]: s = pd.Series()

In [2]: s
Out[2]: Series([], dtype: float64)
```

### 从数组创建 Series

如果 data 是数组类型的，那么 index 需要提供一个相同长度的数组，否认会默认创建 [0, 1, 2, ..., len(data) - 1] 的 index。

```python
In [3]: s = pd.Series(np.random.randn(5))

In [4]: s
Out[4]: 
0   -0.574951
1   -0.209498
2    0.057490
3    0.257507
4    0.309958
dtype: float64

In [5]: s = pd.Series(np.random.randn(5), index=['a', 'b', 'c', 'd', 'e'])

In [6]: s
Out[6]: 
a   -1.907402
b   -0.403695
c    0.862696
d   -0.126665
e   -1.162734
dtype: float64
```

### 从字典创建 Series

如果 data 是字典类型的，则默认以字典的键值作为索引。对于 python 版本 >= 3.6 且 pandas 版本 >= 0.23 的情况，index 会按照字典的插入顺序排序，反之则按照键值的词汇顺序排序。

```python
In [7]: s = pd.Series({'b': 1, 'a': 0, 'c': 2})

In [8]: s
Out[8]:
b    1
a    0
c    2
dtype: int64
```

在上述的例子中，如果 python 版本小于 3.6 或者 pandas 版本小于 0.23，Series 的索引会显示为 ['a', 'b', 'c']。

如果设置了 index 参数，则索引中与 index 对应的数据中的值将被拉出，不存在的设置值为 NaN（即“非数字”（not a number），在pandas中，它用于表示缺失或NA值）。这里 index 的长度不需要和字段长度一致。

```python
In [9]: s = pd.Series({'b': 1, 'a': 0, 'c': 2}, index=['b', 'c', 'd', 'a'])

In [10]: s
Out[10]: 
b    1.0
c    2.0
d    NaN
a    0.0
dtype: float64
```

### 从标量创建 Series

如果 data 是标量值，则必须要提供 index 参数，每个索引都对应这个标量值。

```python
In [11]: s = pd.Series(5)

In [12]: s
Out[12]:
0    5
dtype: int64

In [13]: s = pd.Series(5, index=['a', 'b', 'c', 'd', 'e'])

In [14]: s
Out[14]:
a    5
b    5
c    5
d    5
e    5
dtype: int64
```

## 基本属性

```python
In [15]: s = pd.Series(5, index=['a', 'b', 'c', 'd', 'e'], name='data')
```

| 属性名 | 描述                                | 示例                                                  |
|--------|-------------------------------------|-------------------------------------------------------|
| name   | Series 的名字                       | 'data'                                                |
| index  | Series 的索引                       | Index(['a', 'b', 'c', 'd', 'e'], dtype='object')      |
| array  | Series 的底层数据                   | <PandasArray> [5, 5, 5, 5, 5] Length: 5, dtype: int64 |
| dtype  | Series 的底层数据类型               | dtype('int64')                                        |
| shape  | Series 的底层数据形状，形式为 (n, ) | (5,)                                                  |
| ndim   | Series 的底层数据维度，始终为 1     | 1                                                     |
| size   | Series 的底层数据个数               | 5                                                     |