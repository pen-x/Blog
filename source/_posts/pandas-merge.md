---
title: Pandas 数据集合并（2）
date: 2019-08-02 10:47:54
tags:
  - pandas
---

## pandas.merge 方法

关系型数据库（基于 SQL）的核心运算 join 是通过一个或多个键将多个数据表中的行连接起来的运算，pandas 中通过 merge 函数高效地实现这种数据库风格的合并。

```python
pd.merge(left, right, how='inner', on=None, left_on=None, right_on=None,
         left_index=False, right_index=False, sort=True,
         suffixes=('_x', '_y'), copy=True, indicator=False,
         validate=None)
```

- left：一个 DataFrame 或带名字的 Series 对象。
- right：另一个 DataFrame 或带名字的 Series 对象。
- **on**：用于连接的列名，必须存在于左右两个 DataFrame 对象中。如果没有指定，且其他连接键参数也未指定，则以左右两个 DataFrame 列名的交集作为连接键。
- **left_on**：左侧 DataFrame 中用作连接键的列，既可以是列名，也可以是索引的名字。
- **right_on**：右侧 DataFrame 中用作连接键的列，既可以是列名，也可以是索引的名字。
- **left_index**：将左侧的行索引用作其连接键。如果是左侧的 DataFrame 或 Series 对象是层次化索引，则其索引的级别数要和右侧的连接键数目一致。
- **right_index**：将右侧的行索引用作其连接键，类似 left_index。
- **how**：连接方式，可以是 left、right、outer 或 inner 之一，**默认为 inner**。
- sort：根据连接键对合并后的数据进行排序，默认为 True。有时候在处理大数据集时，禁用该选项可以获得更好的性能。
- suffixes：字符串元组，用于追加到重叠列名的末尾，默认为 ('_x', '_y')。例如，如果左右两个 DataFrame 对象都有 'data' 列，则结果中就会出现 'data_x' 和 'data_y'。
- copy：设置为 False 可以在某些特殊情况下避免将数据复制到结果数据结构中。默认为 True，即总是复制。
- indicator：添加特殊的列 _merge，它可以指明每个行的来源，包括 left_only（数据仅来自左侧数据源）、right_only（数据仅来自右侧数据源） 和 both（数据同时来自左右两个数据源） 三种。
- validate：默认为 None，可以设置为如下值来对连接进行检查：
   - 'one_to_one' 或 '1:1'：检查连接键在左右两侧的数据集中是否都唯一。
   - 'one_to_many' 或 '1:m'：检查连接键在左侧数据集中是否唯一。
   - 'many_to_one' 或 'm:1'：检查连接键在右侧数据集中是否唯一。
   - 'many_to_many' 或 'm:m'：不进行检查。

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

稍复杂一些的多键合并：

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

TODO: 单对单、单对多、多对多

### 连接方式

how 参数用于指定哪些连接键可以出现在最终的结果中，对于非 inner 的连接方式，合并后的结果中都会缺失部分数据，pandas 默认这些值为 NA。下表列出了 how 参数的可选值以及对应的 SQL 语法：

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

可以使用 validate 参数来帮助检查连接键中是否存在重复项。

```python
In [1]: left = pd.DataFrame({'A' : [1,2], 'B' : [1, 2]})

In [2]: right = pd.DataFrame({'A' : [4,5,6], 'B': [2, 2, 2]})

In [3]: result = pd.merge(left, right, on='B', how='outer', validate="one_to_one")

...
MergeError: Merge keys are not unique in right dataset; not a one-to-one merge
```

在上面的例子中，右侧的 DataFrame 的连接键 B 中存在重复值，由于设置了 validate 值为 one_to_one，表明期望的是一对一连接，因此重复的连接键会抛出异常。

如果用户了解右侧 DataFrame 连接键的重复情况，但是需要确保左侧 DataFrame 没有重复连接键，则可以设置 validate 为 one_to_many，如下所示：

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

可以设置 indicator 参数为 True 来显示合并指示符，结果中会产生单独的 _merge 列，指明每个行的来源，包括 left_only（数据仅来自左侧数据源）、right_only（数据仅来自右侧数据源） 和 both（数据同时来自左右两个数据源） 三种。

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

indicator 参数也可以设置为字符串类型值，此时输入值代表新产生列的列名：

```python
In [1]: pd.merge(df1, df2, on='col1', how='outer', indicator='indicator_column')
Out[1]:
   col1 col_left  col_right indicator_column
0     0        a        NaN        left_only
1     1        b        2.0             both
2     2      NaN        2.0       right_only
3     2      NaN        2.0       right_only
```

### 索引合并

如果想要利用索引而不是某一列作为连接键进行合并，则可以利用 left_index 和 right_index 参数：

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

当然，对于有名字的索引，设置 on 参数可以达到同样的效果：

### 索引与列合并

结合 left_on、right_index 或 left_index、right_on，就可以实现索引和列的合并：

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

### 层次化索引合并

单层索引和层次化索引合并的结果是

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

传递给 on、left_on 和 right_on 的参数既可以是列名，也可以是索引名，甚至是列名和索引名的组合，这就帮助我们省去了合并前重设索引的麻烦：

```python
left_index = pd.Index(['K0', 'K0', 'K1', 'K2'], name='key1')

In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3'],
                             'key2': ['K0', 'K1', 'K0', 'K1']},
                             index=left_index)

In [2]: right_index = pd.Index(['K0', 'K1', 'K2', 'K2'], name='key1')

In [3]: right = pd.DataFrame({'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3'],
                              'key2': ['K0', 'K0', 'K0', 'K1']},
                              index=right_index) 

In [4]: pd.merge(left, right, on=['key1', 'key2'])
```

![](/images/merge_on_index_and_column.png)

### 覆盖重复列名

suffixes 参数可以接受一个字符串元组，用于追加到重叠列名的末尾：

```python
In [1]: left = pd.DataFrame({'k': ['K0', 'K1', 'K2'], 'v': [1, 2, 3]})

In [2]: right = pd.DataFrame({'k': ['K0', 'K0', 'K3'], 'v': [4, 5, 6]})

In [3]: pd.merge(left, right, on='k')
```

![](/images/merging_merge_overlapped.png)

```python
In [1]: pd.merge(left, right, on='k', suffixes=['_l', '_r'])
```

![](/images/merging_merge_overlapped_suffix.png)

## DataFrame 实例的 merge 方法

DataFrame 的实例对象也有一个 merge 方法，被调用的 DataFrame 会被当做合并的左对象，其它参数和 pandas.merge() 方法一样，功能也完全一致：

```python
In [1]: left = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                             'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3']})

In [2]: right = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                              'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']})

In [3]: left.merge(right, on='key')
```

![](/images/merging_merge_on_key.png)

## DataFrame 实例的 join 方法

DataFrame 的实例对象还有一个 join 方法，它是 merge 方法的简化版，简化之处在于**右侧 DataFrame 只有索引参与合并**，左侧 DataFrame 的连接键通过 on 参数指定。join 方法的参数包括：

- other：右侧 DataFrame 或带名字的 Series 对象。
- on：被调用 DataFrame 中用于连接的列名或索引名，用于和右侧 DataFrame 的索引进行合并。如果设置为多个值，则右侧 DataFrame 必须包含层次化索引。
- hower：连接方式，可以是 left、right、outer 或 inner 之一，**与 merge 方法不同，这里默认为 left**。
- lsuffix：字符串，用于追加到左侧重叠列名的末尾。
- rsuffix：字符串，用于追加到右侧重叠列名的末尾。
- sort：根据连接键对合并后的数据进行排序，默认为 True。

join 方法可以和 merge 方法互相转化，参照下面例子中的注释：

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                             'B': ['B0', 'B1', 'B2']},
                             index=['K0', 'K1', 'K2'])

In [2]: right = pd.DataFrame({'C': ['C0', 'C2', 'C3'],
                              'D': ['D0', 'D2', 'D3']},
                              index=['K0', 'K2', 'K3'])

In [3]: result = left.join(right)
# result = pd.merge(left, right, left_index=True, right_index=True, how='left')
```

![](/images/merging_join.png)

```python
In [1]: result = left.join(right, how='outer')
# result = pd.merge(left, right, left_index=True, right_index=True, how='outer')
```

![](/images/merging_join_outer.png)

```python
In [1]: result = left.join(right, how='inner')
# result = pd.merge(left, right, left_index=True, right_index=True, how='inner')
```

![](/images/merging_join_inner.png)

通过 on 参数让列参与合并：

```python
In [1]: left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3'],
                             'key': ['K0', 'K1', 'K0', 'K1']})

In [2]: right = pd.DataFrame({'C': ['C0', 'C1'],
                              'D': ['D0', 'D1']},
                              index=['K0', 'K1'])
    
In [3]: result = left.join(right, on='key')
# pd.merge(left, right, left_on='key', right_index=True, how='left')
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
In [4]: result = left.join(right, on=['key1', 'key2'])
# result = pd.merge(left, right, left_on=['key1', 'key2'], right_index=True, how='left')
```

![](/images/merging_join_multikeys.png)

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

In [4]: result = left.join(right, how='inner')
# pd.merge(left, right, left_index=True, right_index=True, how='inner')
```

![](/images/merging_merge_multiindex_alternative.png)