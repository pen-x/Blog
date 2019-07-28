---
title: pandas 索引方法
date: 2019-07-27 01:26:19
tags:
  - pandas
---

Pandas 

索引返回的是复制还是视图

## loc 方法

loc 方法是**纯基于轴标签**的索引方法，即使传入的是参数是数字类型，也会被用于和索引值进行比对，而不是索引位置。可以是以下：
- 单个标量，用于匹配轴标签，例如 5 或者 'a'（这里 5 对应索引中的标签 5，而不是第 5 个位置）
- 标签列表，例如 ['a', 'b', 'c']
- 切片对象，例如 'a': 'f'（与 python 中的切片对象不同，切片的起止标签都包含在输出中），仅：表示全部切片
- 布尔数组，数组的长度要和该维度的长度一致
- 可调用函数，函数的输入只有一个参数，代表当前 Series 或 DataFrame，函数的返回值需要是前面四种之一。

### Series

```python
In [1]: s = pd.Series(np.random.randn(6), index=list('abcdef'))

In [1]: print(s)
Out[1]:
a   -1.008989
b    0.480573
c   -0.806321
d   -1.471417
e    1.691925
f   -1.873039
dtype: float64
```

```python
In [1]: s.loc['b']
Out[1]: 0.4805730548396703
```

```python
In [1]: s.loc[['d', 'a']]
Out[1]: 
d   -1.471417
a   -1.008989
dtype: float64
```

```python
In [1]: s.loc['c':]
Out[1]:
c   -0.806321
d   -1.471417
e    1.691925
f   -1.873039
dtype: float64
```

```python
In [1]: s > 0
Out[1]:
a    False
b     True
c    False
d    False
e     True
f    False
dtype: bool

In [1]: s[s > 0]
Out[1]:
b    0.480573
e    1.691925
dtype: float64
```

### DataFrame

df.loc[row_indexer, column_indexer]
- df.loc[indexer] 表示对行进行索引
- df.loc[:, indexer] 表示对列进行索引
- df.loc[row_indexer,column_indexer] 表示同时对行列进行索引

```python
In [1]: df = pd.DataFrame(np.random.randn(6, 4), 
                        index=list('abcdef'), columns=list('ABCD'))

In [1]: print(df)
Out[1]:
          A         B         C         D
a  0.841807  0.286257  0.290228  0.070770
b  0.118998 -1.435618  0.553285 -0.554076
c -0.510557  1.125717  0.594134  0.564652
d -0.129981  0.022513 -1.743066 -0.735291
e -0.237622  0.336630 -1.006766 -0.844039
f  0.750835  0.437800  0.532366  0.145945
```

```python
In [1]: df.loc['a']
Out[1]:
A    0.841807
B    0.286257
C    0.290228
D    0.070770
Name: a, dtype: float64
```

```python
In [1]: df.loc[['a', 'c', 'f']]
Out[1]:
          A         B         C         D
a  0.841807  0.286257  0.290228  0.070770
c -0.510557  1.125717  0.594134  0.564652
f  0.750835  0.437800  0.532366  0.145945
```

```python
In [1]: df.loc[:, 'B': 'C']
Out[1]:
          B         C
a  0.286257  0.290228
b -1.435618  0.553285
c  1.125717  0.594134
d  0.022513 -1.743066
e  0.336630 -1.006766
f  0.437800  0.532366
```

```python
In [1]: df.loc['d':, 'A':'C']
Out[1]:
          A         B         C
d -0.129981  0.022513 -1.743066
e -0.237622  0.336630 -1.006766
f  0.750835  0.437800  0.532366
```

```python
In [1]: df.loc['b'] > 0
Out[1]:
A     True
B    False
C     True
D    False
Name: b, dtype: bool

In [1]: df.loc['b': 'c', df.loc['b'] > 0]
Out[1]:
          A         C
b  0.118998  0.553285
c -0.510557  0.594134
```

```python
In [1]: df.loc['a', 'A']
Out[1]: 0.8418070411036316
```

TODO: 此处添加切片超出范围时的处理

## iloc 方法

iloc 方法是**纯基于位置**的索引方法，和 python 类似。可以是以下：
- 单个整数，用于匹配轴位置，例如 5 
- 整数列表，例如 [4, 3, 0]
- 切片对象，例如 1：7（与 python 中的切片对象一致），仅：表示全部切片
- 布尔数组，数组的长度要和该维度的长度一致
- 可调用函数，函数的输入只有一个参数，代表当前 Series 或 DataFrame，函数的返回值需要是前面四种之一。

### Series

```python
In [1]: s = pd.Series(np.random.randn(5), index=list(range(0, 10, 2)))

In [1]: print(s)
Out[1]: 
0   -0.247860
2   -0.214236
4   -0.953676
6   -0.341041
8   -1.745646
dtype: float64
```

```python
In [1]: s.iloc[:3]
Out[1]: 
0   -0.247860
2   -0.214236
4   -0.953676
dtype: float64
```

```python
In [1]: s.iloc[3]
Out[1]: -0.3410412599942804
```

```python
In [1]: df = pd.DataFrame(np.random.randn(6, 4), 
                        index=list(range(0, 12, 2)), 
                        columns=list(range(0, 8, 2)))

In [1]: print(df)
Out[1]: 
           0         2         4         6
0  -0.201264 -0.108100  0.619799  0.454962
2  -0.073565  1.863290 -1.017726 -0.147788
4   0.510884 -0.095537 -1.391593  0.749990
6  -0.673233  0.691208 -0.187173 -0.184766
8  -1.005933 -0.763360 -0.770139  1.683957
10  0.325233 -0.843027 -1.349218  0.626582
```

```python
In [1]: df.iloc[:3]
Out[1]: 
          0         2         4         6
0 -0.201264 -0.108100  0.619799  0.454962
2 -0.073565  1.863290 -1.017726 -0.147788
4  0.510884 -0.095537 -1.391593  0.749990
```

```python
In [1]: df.iloc[1:5, 2:4]
Out[1]: 
          4         6
2 -1.017726 -0.147788
4 -1.391593  0.749990
6 -0.187173 -0.184766
8 -0.770139  1.683957
```

TODO：如果有索引不存在，会抛出异常

```python
In [1]: df.iloc[[1, 3, 5], [1, 3]]
Out[1]:
           2         6
2   1.863290 -0.147788
6   0.691208 -0.184766
10 -0.843027  0.626582
```

```python
In [1]: df.iloc[1:3, :]
Out[1]:
          0         2         4         6
2 -0.073565  1.863290 -1.017726 -0.147788
4  0.510884 -0.095537 -1.391593  0.749990
```

```python
In [1]: df.iloc[:, 1:3]
Out[1]:
           2         4
0  -0.108100  0.619799
2   1.863290 -1.017726
4  -0.095537 -1.391593
6   0.691208 -0.187173
8  -0.763360 -0.770139
10 -0.843027 -1.349218
```

```python
In [1]: df.iloc[1, 1]
Out[1]: 1.8632895439526078
```

值得注意的是，前两种方法中

## at/iat 方法

loc/iloc 的简化版，只用于返回单个值，其中 at 的参数只能是两个标签值，iat 的参数只能是两个整数值。

```python
In [1]: s = pd.Series([0, 1, 2, 3], index=['a', 'b', 'c', 'd'])

In [1]: print(s)
Out[1]: 
a    0
b    1
c    2
d    3
dtype: int64

In [1]: s.at['b']
Out[1]: 1

In [1]: s.iat[2]
Out[1]: 2
```

```python
In [1]: df = pd.DataFrame([[0, 2, 3], [0, 4, 1], [10, 20, 30]], columns=['A', 'B', 'C'])

In [1]: print(df)
Out[1]: 
    A   B   C
0   0   2   3
1   0   4   1
2  10  20  30

In [1]: df.at[4, 'B']
Out[1]: 2

In [1]: df.iat[1, 2]
Out[1]: 1
```

## 索引运算 []

Pandas 也支持 python 标准的索引运算 []（即调用 __getitem() 内置方法），

Object Type	Selection	Return Value Type
Series	series[label]	scalar value
DataFrame	frame[colname]	Series corresponding to colname

利用标签的切片运算与普通的Python切片运算不同，其末端是包含的：

```python
In [1]: s = pd.Series(np.arange(4.), index=['a', 'b', 'c', 'd'])

In [1]: print(s)
Out[1]: 
a    0.0
b    1.0
c    2.0
d    3.0
dtype: float64
```

```python
In [1]: s['b']
In [1]: 1.0

In [1]: s[1]
In [1]: 1.0

In [1]: s[2: 4]
In [1]: 
c    2.0
d    3.0
dtype: float64

In [1]: s[['b', 'a', 'd']]
In [1]: 
b    1.0
a    0.0
d    3.0
dtype: float64

In [1]: s[[1, 3]]
In [1]: 
b    1.0
d    3.0
dtype: float64

In [1]: s[s < 2]
In [1]: 
a    0.0
b    1.0
dtype: float64

In [1]: s['b': 'c']
In [1]: 
b    1.0
c    2.0
dtype: float64
```

```python
In [1]: df = pd.DataFrame(np.arange(16).reshape((4, 4)), 
                        index=['Ohio', 'Colorado', 'Utah', 'New York'], 
                        columns=['one', 'two', 'three', 'four'])

In [1]: print(df)
Out[1]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

```python
In [1]: df['two']
In [1]: 
Ohio         1
Colorado     5
Utah         9
New York    13
Name: two, dtype: int32

In [1]: df[['three', 'one']]
In [1]: 
          three  one
Ohio          2    0
Colorado      6    4
Utah         10    8
New York     14   12

In [1]: 
In [1]: 

In [1]: 
In [1]: 
```

行切片

```python
In [1]: df[: 2]
In [1]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
```

布尔数组

```python
In [1]: df[df['three'] > 5
In [1]: 
          one  two  three  four
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

布尔型 DataFrame

```python
In [1]: df < 5
In [1]: 
            one    two  three   four
Ohio       True   True   True   True
Colorado   True  False  False  False
Utah      False  False  False  False
New York  False  False  False  False

In [1]: df[df < 5]
In [1]: 
          one  two  three  four
Ohio      0.0  1.0    2.0   3.0
Colorado  4.0  NaN    NaN   NaN
Utah      NaN  NaN    NaN   NaN
New York  NaN  NaN    NaN   NaN
```

## 属性运算 .

Pandas 支持使用属性运算 . 来直接访问 Series 的索引或 DataFram 的列。

You can use this access only if the index element is a valid Python identifier, e.g. s.1 is not allowed. See here for an explanation of valid identifiers.
The attribute will not be available if it conflicts with an existing method name, e.g. s.min is not allowed.
Similarly, the attribute will not be available if it conflicts with any of the following list: index, major_axis, minor_axis, items.
In any of these cases, standard indexing will still work, e.g. s['1'], s['min'], and s['index'] will access the corresponding element or column.

```python
In [1]: s.b
In [1]: 1.0

In [1]: df.two
In [1]: Ohio         1
Colorado     5
Utah         9
New York    13
Name: two, dtype: int32
```