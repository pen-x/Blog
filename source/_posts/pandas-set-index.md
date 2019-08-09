---
title: Pandas 重设索引
date: 2019-08-09 15:12:02
tags:
  - pandas
---

Pandas 在创建 DataFrame 和通过 IO 工具导入数据时可以设置索引，然而有时候你可能已经创建好了 DataFrame，希望重新设置索引。

## set_index 方法

```python
DataFrame.set_index(self, keys, drop=True, append=False, inplace=False, 
                    verify_integrity=False)
```

- keys: 
- drop:
- append:
- inplace:
- verify_integrity:

```python
In [1]: data = pd.DataFrame({'a': ['bar', 'bar', 'foo', 'foo'],
                             'b': ['one', 'two', 'one', 'two'],
                             'c': ['z', 'y', 'x', 'w'],
                             'd': [1.0, 2.0, 3.0, 4.0]})

In [2]: data
Out[2]:
     a    b  c    d
0  bar  one  z  1.0
1  bar  two  y  2.0
2  foo  one  x  3.0
3  foo  two  w  4.0

In [3]: indexed1 = data.set_index('c')
In [4]: indexed1
Out[4]:
     a    b    d
c               
z  bar  one  1.0
y  bar  two  2.0
x  foo  one  3.0
w  foo  two  4.0

In [5]: indexed2 = data.set_index(['a', 'b'])
In [6]: indexed2
Out[6]:
         c    d
a   b          
bar one  z  1.0
    two  y  2.0
foo one  x  3.0
    two  w  4.0
```

设置 drop 参数为 False 可以在将某一列设置为索引的同时依然保留该列：

```python
In [1]: frame = data.set_index('c', drop=False)

In [2]: frame
Out[2]:
     a    b  c    d
c                  
z  bar  one  z  1.0
y  bar  two  y  2.0
x  foo  one  x  3.0
w  foo  two  w  4.0

In [1]: frame = data.set_index('c', drop=True)

In [2]: frame
Out[2]:
     a    b    d
c               
z  bar  one  1.0
y  bar  two  2.0
x  foo  one  3.0
w  foo  two  4.0
```

设置 append 参数为 True 则会保留已有的索引，形成层次化索引：

```python
In [1]: frame = data.set_index(['a', 'b'], append=True)

In [2]: frame
Out[2]:
           c    d
  a   b          
0 bar one  z  1.0
1 bar two  y  2.0
2 foo one  x  3.0
3 foo two  w  4.0

In [1]: frame = data.set_index(['a', 'b'], append=False)

In [2]: frame
Out[2]:
         c    d
a   b          
bar one  z  1.0
    two  y  2.0
foo one  x  3.0
    two  w  4.0
```

设置 inplace 参数为 True 将原地重设索引，而不是创建一个新对象：

```python
In [1]: data.set_index(['a', 'b'], inplace=True)

In [2]: data
Out[2]:
     a    b  c    d
0  bar  one  z  1.0
1  bar  two  y  2.0
2  foo  one  x  3.0
3  foo  two  w  4.0

In [1]: data.set_index(['a', 'b'], inplace=False)

In [2]: data
Out[2]:
         c    d
a   b          
bar one  z  1.0
    two  y  2.0
foo one  x  3.0
    two  w  4.0
```

## reset_index 方法

DataFrame 提供了一个便利的方法 reset_index，它是 set_index 的反向操作，即将索引转化成新的列，同时根据需要创建默认索引 [0,1,...,n]。

```python
DataFrame.reset_index(self, level=None, drop=False, inplace=False, col_level=0, col_fill='')
```

- level：指定需要转变为列的索引，可以是索引级别，也可以是索引的标签，以及它们的列表。
- drop：是否丢弃索引，即重设索引的同时不产生新的列，默认为 False。
- inplace：是否原地修改 DataFrame，不产生新对象，默认为 False。
- col_level：
- col_fill：

## reindex 方法

## reindex_like 方法