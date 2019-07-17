---
title: Pandas 数据结构
date: 2019-07-17 17:29:03
tags:
  - pandas
---

Pandas 中最基本的数据类型包括 Series、DataFrame 和 Index：

- Series是一种类似于一维数组的对象，它由一组数据（各种NumPy数据类型）以及一组与之相关的数据标签（即索引）组成。
- DataFrame是一个表格型的数据结构，它含有一组有序的列，每列可以是不同的值类型（数值、字符串、布尔值等），但同一列中的数据类型相同。DataFrame既有行索引也有列索引，它可以被看做由Series组成的字典（共用同一个索引）。DataFrame中的数据是以一个或多个二维块存放的（而不是列表、字典或别的一维数据结构）。
- Index 对象负责管理轴标签和其他元数据（比如轴名称等）。构建Series或DataFrame时，所用到的任何数组或其他序列的标签都会被转换成一个Index。

## Series

### 构造函数

```python
pandas.Series(data=None, index=None, dtype=None, name=None, copy=False, fastpath=False)
```

| 参数 | 描述 |
|----------|------------------------------------------------------------------------|
| data | Series 中要存储的数据，可以是类数组类型、可迭代类型、字典类型、标量 |
| index | Series 中数据的索引，需要和数据的长度相同，默认为 np.arange(n) |
| dtype | Series 的数据类型，如果没有，将推断数据类型 |
| copy | 是否从输入中复制数据，默认为 false |
| name | Series 名称 |

#### 创建空的 Series

不提供任何参数，则可以创建一个空的 Series。

```python
In [1]: s = pd.Series()

In [2]: s
Out[2]: Series([], dtype: float64)
```

#### 从列表创建 Series

如果 data 是列表类型的，那么 index 参数需要具有相同的长度，否认会默认创建 [0, 1, 2, ..., len(data) - 1] 的默认索引。

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

#### 从字典创建 Series

如果 data 是字典类型的，则默认以字典的键作为索引。

```python
In [7]: s = pd.Series({'b': 1, 'a': 0, 'c': 2})

In [8]: s
Out[8]:
b    1
a    0
c    2
dtype: int64
```

对于 python 版本 >= 3.6 且 pandas 版本 >= 0.23 的情况，index 会按照字典的插入顺序排序，反之则按照键值的词汇顺序排序。在上述的例子中，低版本情况下 Series 的索引会显示为 ['a', 'b', 'c']。

如果设置了 index 参数，则索引中与 index 对应的数据中的值将被拉出，不存在的设置值为 NaN（即非数字，用于表示缺失或NA值）。和列表不同，这里 index 的长度并不需要和字典的长度一致。

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

#### 从标量创建 Series

如果 data 是标量值，则每个索引标签都对应这个标量值。

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

### 基本属性

我们以下面的 Series 为例：

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

## DataFrame

### 构造函数

```python
pandas.DataFrame(data=None, index=None, columns=None, dtype=None, copy=False)
```

| 参数 | 描述 |
|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| data | DataFrame 中要存储的数据，可以是类数组类型、可迭代类型、字典类型、其它 DataFrame。对于字典类型，其中可以包含 Series、数组、常量等 |
| index | DataFrame 中数据的行索引，可以是 Index 或数组类型，如果 data 参数无法推断出索引信息，同时也没有指定 index 参数，则默认使用 RangeIndex (0, 1, 2, …, n) |
| columns | DataFrame 中数据的列索引，要求与 index 参数一致 |
| dtype | DataFrame 的数据类型，如果没有，将推断数据类型 |
| copy | 是否从输入中复制数据，默认为 false |

#### 创建空的 DataFrame

如果不提供任何参数，则可以创建一个空的 DataFrame。

```python
In [1]: df = pd.DataFrame()

In [2]: print(df)
Out[2]:
Empty DataFrame
Columns: []
Index: []
```

#### 从列表创建 DataFrame

通过列表来创建 DataFrame 时，如果列表只有一维，则会创建只有一列的 DataFrame。其它情况下，例如列表中的各项为列表、字典或 Series，则各项会分别成为 DataFrame 的一行。

一维列表：

```python
In [1]: df = pd.DataFrame([1, 2, 3, 4, 5])

In [2]: print(df)
Out[2]:
    0
0   1
1   2
2   3
3   4
4   5
```

列表中的各项为列表时：

```python
In [1]: df = pd.DataFrame(np.arange(20).reshape((5, 4)))

In [2]: print(df)
Out[2]:
    0   1   2   3
0   0   1   2   3
1   4   5   6   7
2   8   9  10  11
3  12  13  14  15
4  16  17  18  19
```

一般需要用户指定行、列标签：

```python
In [1]: df = pd.DataFrame(np.arange(20).reshape((5, 4)),
                          index=['a', 'b', 'c', 'd', 'e'],
                          columns=['A', 'B', 'C', 'D'])

In [2]: print(df)
Out[2]:
    A   B   C   D
a   0   1   2   3
b   4   5   6   7
c   8   9  10  11
d  12  13  14  15
e  16  17  18  19
```

列表中的各项为字典时，字典的键会被合并成为 DataFrame 的列标签。

```python
In [1]: data = [{'a': 1, 'b': 2},{'a': 5, 'b': 10, 'c': 20}]
        df = pd.DataFrame(data)

In [2]: print(df)
Out[2]:
   a   b     c
0  1   2   NaN
1  5  10  20.0
```

当列表中的各项为 Series 时，Series 的索引会被合并成结果的列标签。

```python
In [1]: data = [pd.Series([1, 2, 3], index=['a', 'b', 'c']),
                pd.Series([4, 5, 6], index=['a', 'b', 'c'])]
        df = pd.DataFrame(data)

In [2]: print(df)
Out[2]:
   a  b  c
0  1  2  3
1  4  5  6
```

#### 从字典创建 DataFrame

通过字典来创建 DataFrame 时，字典中的各项会成为 DataFrame 的一列，字典的键构成 DataFrame 的列标签。

如果字典是由数组、列表或元组组成的，那么每个序列的长度要求必须相同。

```python
In [1]: data = {'state': ['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada', 'Nevada'],
                'year': [2000, 2001, 2002, 2001, 2002, 2003],
                'pop': [1.5, 1.7, 3.6, 2.4, 2.9, 3.2]}
        df = pd.DataFrame(data)

In [2]: print(df)
Out[2]:
    state  year  pop
0    Ohio  2000  1.5
1    Ohio  2001  1.7
2    Ohio  2002  3.6
3  Nevada  2001  2.4
4  Nevada  2002  2.9
5  Nevada  2003  3.2
```

如果字典是由字典组成的，那么内层字典的键会被用作行标签。

```python
In [1]: data = {'Nevada': {2001: 2.4, 2002: 2.9},
                'Ohio': {2000: 1.5, 2001: 1.7, 2002: 3.6}}
        df = pd.DataFrame(data)

In [2]: print(df)
Out[2]:
      Nevada  Ohio
2000     NaN   1.5
2001     2.4   1.7
2002     2.9   3.6
```

如果字典是由 Series 组成的，那么各 Series 的索引会被合并成结果的行索引。

```python
In [1]: data = {'one' : pd.Series([1, 2, 3], index=['a', 'b', 'c']),
                'two' : pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])}
        df = pd.DataFrame(data)

In [2]: print(df)
Out[2]:
   one  two
a  1.0    1
b  2.0    2
c  3.0    3
d  NaN    4
```

### 基本属性

我们以下面的 Series 为例：

```python
In [15]: df = pd.DataFrame([('parrot',   24.0, 'second'),
                    ('lion',     80.5, 1),
                    ('monkey', np.nan, None)], columns=('name', 'max_speed', 'rank'))

In [15]: dfprint(df)
Out[15]:
     name  max_speed    rank
0  parrot       24.0  second
1    lion       80.5       1
2  monkey        NaN    None
```

| 属性 | 描述 | 示例 |
|---------|---------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| index | DataFrame 行索引 | RangeIndex(start=0, stop=3, step=1) |
| columns | DataFrame 列索引 | Index(['name', 'max_speed', 'rank'], dtype='object') |
| dtypes | DataFrame 中各列的数据类型 | name          object<br> max_speed    float64<br> rank          object<br> dtype: object |
| values | DataFrame 底层数据的 numpy 表示，建议使用 to_numpy() 方法替代 | array([['parrot', 24.0, 'second'],<br>        ['lion', 80.5, 1],<br>        ['monkey', nan, None]], dtype=object) |
| ndim | DataFrame 的底层数据维度，始终为 2 | 2 |
| size | DataFrame 的底层数据个数 | 9 |
| shape | DataFrame 的底层数据形状 | (3, 3) |

## Index

Pandas 中的索引类型有很多，包括单调序列索引 RangeIndex、类别索引 CategoricalIndex、多层索引 MultiIndex、间隔索引 IntervalIndex 等，它们都继承自 Index 类型。

Index 对象的一大特点在于它是不可变的，任何修改 Index 的操作都会返回一个新的 Index 对象。

### 构造函数

```python
pandas.Index(data=None, dtype=None, copy=False, name=None, fastpath=None, tupleize_cols=True)
```

通过列表构造 Index：

```python
In [1]: idx = pd.Index([1, 2, 3])

In [2]: print(idx)
Out[2]:
Int64Index([1, 2, 3], dtype='int64')

In [1]: idx = pd.Index(list('abc'))

In [2]: print(idx)
Out[2]:
Index(['a', 'b', 'c'], dtype='object')
```

Series 的行标签，DataFrame 的行标签和列标签都是 Index 类型:

```python
In [1]: data = {'Nevada': {2001: 2.4, 2002: 2.9},
                'Ohio': {2000: 1.5, 2001: 1.7, 2002: 3.6}}
        df = pd.DataFrame(data)

In [2]: print(df.index)
Out[2]:
Int64Index([2000, 2001, 2002], dtype='int64')

In [2]: print(df.columns)
Out[2]:
Index(['Nevada', 'Ohio'], dtype='object')
```

与python的集合不同，pandas的Index可以包含重复的标签：

```python
In [1]: dup_labels = pd.Index(['foo', 'foo', 'bar', 'bar'])

In [2]: print(dup_labels)
Out[2]:
Index(['foo', 'foo', 'bar', 'bar'], dtype='object')
```