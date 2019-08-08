---
title: Pandas 数据集合并（3）
date: 2019-08-08 22:24:25
tags:
  - pandas
---

## 实例 combine_first 方法

实例方法 **combine_first** 可以将重复数据拼接在一起，用一个对象中的值填充另一个对象中的缺失值，并返回一个新对象。

```python
In [1]: df1 = pd.DataFrame({'A': [None, 0], 'B': [None, 4]})

In [2]: df1
Out[2]:
     A    B
0  NaN  NaN
1  0.0  4.0

In [3]: df2 = pd.DataFrame({'A': [1, 1], 'B': [3, 3]})

In [4]: df2
Out[4]: 
   A  B
0  1  3
1  1  3

In [5]: result = df1.combine_first(df2)
In [6]: result
Out[6]:
     A    B
0  1.0  3.0
1  0.0  4.0
```

如果两个对象的行或列不同，则返回的对象会包含它们各自的并集：

```python
In [1]: df3 = pd.DataFrame({'B': [3, 3], 'C': [1, 1]}, index=[1, 2])

In [2]: df3
Out[2]:
   B  C
1  3  1
2  3  1

In [3]: result = df1.combine_first(df3)
In [4]: result
Out[4]:
     A    B    C
0  NaN  NaN  NaN
1  0.0  4.0  1.0
2  NaN  3.0  1.0
```

## 实例的 update 方法

update 则使用另一个对象中的非空值原地替换当前对象中对应的值：

```python
In [1]: df = pd.DataFrame({'A': [1, 2, 3], 'B': [400, 500, 600]})

In [2]: new_df = pd.DataFrame({'B': [4, 5, 6], 'C': [7, 8, 9]})

In [3]: df.update(new_df)

In [4]: df
Out[4]:
   A  B
0  1  4
1  2  5
2  3  6
```

在上面的例子中，可以看出对象 df 中的 B 列全部被替换了，而并不仅替换掉非空值，这是和 combine_fist 方法的区别。

空值则会被忽略：

```python
In [1]: df = pd.DataFrame({'A': [1, 2, 3], 'B': [400, 500, 600]})

In [2]: new_df = pd.DataFrame({'B': [4, np.nan, 6]})

In [3]: df.update(new_df)

In [4]: df
Out[4]:
   A      B
0  1    4.0
1  2  500.0
2  3    6.0
```

由于 update 方法只是对被调用对象进行修改，因此另一个对象中多余的行或列会被直接丢弃：

```python
In [1]: df = pd.DataFrame({'A': ['a', 'b', 'c'], 'B': ['x', 'y', 'z']})

In [2]: new_df = pd.DataFrame({'B': ['d', 'e', 'f', 'g', 'h', 'i']})

In [3]: df.update(new_df)

In [4]: df
Out[4]:
   A  B
0  a  d
1  b  e
2  c  f
```

Series 同样可以参与：

```python
In [1]: df = pd.DataFrame({'A': ['a', 'b', 'c'], 'B': ['x', 'y', 'z']})

In [2]: new_column = pd.Series(['d', 'e'], name='B', index=[0, 2])

In [3]: df.update(new_column)

In [4]: df
Out[4]:
   A  B
0  a  d
1  b  y
2  c  e
```

