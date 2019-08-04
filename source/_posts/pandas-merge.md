---
title: Pandas 数据集合并（2）
date: 2019-08-02 10:47:54
tags:
  - pandas
---

## pandas.merge 方法

关系型数据库（基于 SQL）的核心运算 join 是通过一个或多个键将多个数据表中的行连接起来的运算，pandas 中通过 merge 函数来实现这种数据库风格的合并。

```python
pd.merge(left, right, how='inner', on=None, left_on=None, right_on=None,
         left_index=False, right_index=False, sort=True,
         suffixes=('_x', '_y'), copy=True, indicator=False,
         validate=None)
```

- left：一个 DataFrame 或带名字的 Series 对象。
- right：另一个 DataFrame 或带名字的 Series 对象。
- on：用于连接的列名，必须存在于左右两个 DataFrame 对象中。如果没有指定，且其他连接键参数也未指定，则以左右两个 DataFrame 列名的交集作为连接键。
- left_on：左侧 DataFrame 中用作连接键的列。
- right_on：右侧 DataFrame 中用作连接键的列。
- left_index：将左侧的行索引用作其连接键。
- right_index：将右侧的行索引用作其连接键。
- sort：根据连接键对合并后的数据进行排序，默认为 True。有时候在处理大数据集时，禁用该选项可以获得更好的性能。
- suffixes：字符串元组，用于追加到重叠列名的末尾，默认为 ('_x', '_y')。例如，如果左右两个 DataFrame 对象都有 'data' 列，则结果中就会出现 'data_x' 和 'data_y'。
- copy：设置为 False 可以在某些特殊情况下避免将数据复制到结果数据结构中。默认为 True，即总是复制。
- indicator：添加特殊的列 _merge，它可以指明每个行的来源，包括 left_only、right_only 和 both 三种。
- validate：

### 连接方式

首先是最简单的单键合并：

```python
In [1]: left = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                             'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3']})

In [2]: right = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                              'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']})

In [3]: result = pd.merge(left, right, on='key')
```

![](/images/merging_merge_on_key.png)

更复杂的

```python
In [1]: left = pd.DataFrame({'key1': ['K0', 'K0', 'K1', 'K2'],
                             'key2': ['K0', 'K1', 'K0', 'K1'],
                             'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3']})

In [2]: right = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                              'key2': ['K0', 'K0', 'K0', 'K0'],
                              'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']})

In [3]: result = pd.merge(left, right, on=['key1', 'key2'])
```

![](/images/merging_merge_on_key_multiple.png)

how 参数

| 选项  | SQL 语法         | 说明                 |
|-------|------------------|----------------------|
| left  | LEFT OUTER JOIN  | 使用左表中所有的键   |
| right | RIGHT OUTER JOIN | 使用右表中所有的键   |
| outer | FULL OUTER JOIN  | 使用两个表中所有的键 |
| inner | INNER JOIN       | 使用两个表都有的键   |

```python
In [1]: pd.merge(left, right, how='left', on=['key1', 'key2'])
```

![](/images/merging_merge_on_key_left.png)

```python
In [1]: pd.merge(left, right, how='right', on=['key1', 'key2'])
```

![](/images/merging_merge_on_key_right.png)

```python
In [1]: pd.merge(left, right, how='outer', on=['key1', 'key2'])
```

![](/images/merging_merge_on_key_outer.png)

```python
In [1]: pd.merge(left, right, how='inner', on=['key1', 'key2'])
```

![](/images/merging_merge_on_key_inner.png)

### 重复键检查

可以使用 validate 参数来自动检查

```python
In [1]: left = pd.DataFrame({'A' : [1,2], 'B' : [1, 2]})

In [2]: right = pd.DataFrame({'A' : [4,5,6], 'B': [2, 2, 2]})

In [3]: result = pd.merge(left, right, on='B', how='outer', validate="one_to_one")

...
MergeError: Merge keys are not unique in right dataset; not a one-to-one merge
```

```python
In [1]: pd.merge(left, right, on='B', how='outer', validate="one_to_many")
Out[1]:
   A_x  B  A_y
0    1  1  NaN
1    2  2  4.0
2    2  2  5.0
3    2  2  6.0
```

### 合并指示符

```python
In [1]: df1 = pd.DataFrame({'col1': [0, 1], 'col_left': ['a', 'b']})

In [2]: df2 = pd.DataFrame({'col1': [1, 2, 2], 'col_right': [2, 2, 2]})

In [3]: pd.merge(df1, df2, on='col1', how='outer', indicator=True)
Out [3]:
   col1 col_left  col_right      _merge
0     0        a        NaN   left_only
1     1        b        2.0        both
2     2      NaN        2.0  right_only
3     2      NaN        2.0  right_only
```

```python
In [1]: pd.merge(df1, df2, on='col1', how='outer', indicator='indicator_column')
Out[1]:
   col1 col_left  col_right indicator_column
0     0        a        NaN        left_only
1     1        b        2.0             both
2     2      NaN        2.0       right_only
3     2      NaN        2.0       right_only
```

### 使用 Index 合并

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                             'B': ['B0', 'B1', 'B2']},
                             index=['K0', 'K1', 'K2'])

In [2]: right = pd.DataFrame({'C': ['C0', 'C2', 'C3'],
                              'D': ['D0', 'D2', 'D3']},
                              index=['K0', 'K2', 'K3'])

In [3]: pd.merge(left, right, left_index=True, right_index=True, how='outer')
```

![](/images/merging_merge_index_outer.png)

```python
In [1]: pd.merge(left, right, left_index=True, right_index=True, how='inner')
```

![](/images/merging_merge_index_inner.png)

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3'],
                             'key': ['K0', 'K1', 'K0', 'K1']})

In [2]: right = pd.DataFrame({'C': ['C0', 'C1'],
                              'D': ['D0', 'D1']},
                              index=['K0', 'K1'])
    
In [3]: pd.merge(left, right, left_on='key', right_index=True, how='left')
```

![](/images/merging_merge_key_columns.png)

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3'],
                             'key1': ['K0', 'K0', 'K1', 'K2'],
                             'key2': ['K0', 'K1', 'K0', 'K1']})

In [2]: index = pd.MultiIndex.from_tuples([('K0', 'K0'), ('K1', 'K0'),
                                           ('K2', 'K0'), ('K2', 'K1')])

In [3]: right = pd.DataFrame({'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']},
                              index=index)
In [4]: pd.merge(left, right, left_on=['key1', 'key2'],
                 right_index=True, how='inner')
```

![](/images/merging_join_multikeys_inner.png)

普通索引也可以和层次化索引合并：

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                             'B': ['B0', 'B1', 'B2']},
                             index=pd.Index(['K0', 'K1', 'K2'], name='key'))
 
In [2]: index = pd.MultiIndex.from_tuples([('K0', 'Y0'), ('K1', 'Y1'),
                                           ('K2', 'Y2'), ('K2', 'Y3')],
                                           names=['key', 'Y'])

In [3]: right = pd.DataFrame({'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']},
                              index=index)

In [4]: pd.merge(left, right, left_index=True, right_index=True, how='inner')
```

![](/images/merging_merge_multiindex_alternative.png)

```
In [1]: leftindex = pd.MultiIndex.from_tuples([('K0', 'X0'), ('K0', 'X1'),
                                               ('K1', 'X2')],
                                               names=['key', 'X'])

In [2]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                             'B': ['B0', 'B1', 'B2']},
                             index=leftindex)

In [3]: rightindex = pd.MultiIndex.from_tuples([('K0', 'Y0'), ('K1', 'Y1'),
                                                ('K2', 'Y2'), ('K2', 'Y3')],
                                                names=['key', 'Y'])

In [4]: right = pd.DataFrame({'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']},
                              index=rightindex)

In [5]: pd.merge(left, right, left_index=True, right_index=True, how='inner')
```

![](/images/merging_merge_two_multiindex.png)

### 混合 Index 和 Column

传递给 on、left_on 和 right_on 的参数既可以是列名，也可以是行名，因此允许混合 Index 和 Column 合并：

### 覆盖重复列名

```python
In [1]: left = pd.DataFrame({'k': ['K0', 'K1', 'K2'], 'v': [1, 2, 3]})

In [2]: right = pd.DataFrame({'k': ['K0', 'K0', 'K3'], 'v': [4, 5, 6]})

In [3]: pd.merge(left, right, on='k')
```

```python
In [1]: pd.merge(left, right, on='k', suffixes=['_l', '_r'])
```

![](/images/merging_merge_overlapped_suffix.png)

## join 方法

DataFrame.join() 函数

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                             'B': ['B0', 'B1', 'B2']},
                             index=['K0', 'K1', 'K2'])

In [2]: right = pd.DataFrame({'C': ['C0', 'C2', 'C3'],
                              'D': ['D0', 'D2', 'D3']},
                              index=['K0', 'K2', 'K3'])

In [3]: result = left.join(right)
```

![](/images/merging_join.png)

```python
In [1]: result = left.join(right, how='outer')
```

![](/images/merging_join_outer.png)

```python
In [1]: result = left.join(right, how='inner')
```

![](/images/merging_join_inner.png)