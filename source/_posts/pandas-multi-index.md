---
title: pandas 层次化索引
date: 2019-07-29 20:31:09
tags:
  - pandas
toc: false
---

层次化索引（Hierarchical / Multi-level Indexing）是pandas的一项重要功能，它使你能在一个轴上拥有多个（两个以上）索引级别。抽象点说，它使你能以低维度形式处理高维度数据。

以下图为例，这里有两个索引级别，level 0 表示第一个索引级别，level 1 表示第二个索引级别。结合各个索引级别上的标签则可以构成一个索引项，例如 ('bar', 'one')、('foo', 'two') 等。

![](/images/pandas-multi-index.png)

## 基本属性

层次化索引的类型是 pandas.MultiIndex，它继承自 pandas.Index，有如下一些基本属性：

- **names**: 索引中各级别的名称，各级别名称的默认值是 None。
- **levels**: 标签数组，表示各级别的唯一标签。在上面的例子中，第一个级别有 'bar'、'baz'、'foo'、'qux' 4 种标签，第二个级别只有 'one' 和 'two' 两种标签，因此 levels 的值是 [['bar', 'baz', 'foo', 'qux'], ['one', 'two']]。
- **codes**: 整数数组，和 levels 配合使用，数组的每一项代表索引一个级别的全部取值。由于都是整数，它们代表在标签数组中的坐标。例如 [0, 0, 1, 1, 2, 2, 3, 3] 代表第一级索引标签为 ['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux']。
- nlevels: 索引的总级别数。
- levshape: 索引中各级别标签的数目。

```python
In [1]: arrays = [['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux'],
                  ['one', 'two', 'one', 'two', 'one', 'two', 'one', 'two']]

In [2]: index = pd.MultiIndex.from_arrays(arrays)
In [3]: index
Out[3]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])

In [4]: index.levels
Out[4]: FrozenList([['bar', 'baz', 'foo', 'qux'], ['one', 'two']])

In [5]: index.codes
Out[5]: FrozenList([[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])

In [6]: index.names
Out[6]: FrozenList([None, None])

In [7]: index.nlevels
Out[7]: 2

In [8]: index.levshape
Out[8]: (4, 2)
```

## 构造函数

有多种创建 MultiIndex 的方式，分别包括：

- MultiIndex.from_arrays()：通过数组列表创建层次化索引，每个数组代表一个索引级别上的标签取值。
- MultiIndex.from_tuples()：通过元组列表创建层次化索引，每个元组代表一个索引项，即当前索引项在各个索引级别上的取值。
- MultiIndex.from_product()：通过元素的乘积创建层次化索引，。
- MultiIndex.from_frame()：通过 DataFrame 创建层次化索引。

这些方法都可以接受一个 **names** 参数来指定索引各级别的名字，如不指定，则默认为 None。

通过数组列表创建时，要求各数组的长度一致：

```python
In [1]: arrays = [['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux'],
                  ['one', 'two', 'one', 'two', 'one', 'two', 'one', 'two']]

In [2]: index = pd.MultiIndex.from_arrays(arrays)
In [3]: index
Out[3]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])
```

也可以通过元组列表创建，元组列表可以非常直观地查看各索引项：

```python
In [1]: tuples = list(zip(*arrays))

In [2]: tuples
Out[2]: 
[('bar', 'one'),
 ('bar', 'two'),
 ('baz', 'one'),
 ('baz', 'two'),
 ('foo', 'one'),
 ('foo', 'two'),
 ('qux', 'one'),
 ('qux', 'two')]

In [3]: index = pd.MultiIndex.from_tuples(tuples, names=['first', 'second'])
In [4]: index
Out[4]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]],
           names=['first', 'second'])
```

from_product() 方法则会计算标签数组的笛卡尔积作为最终的索引项，例如一个 4 维数组和一个 2 维数组会产生 4 * 2 = 8 个索引项：

```python
In [1]: iterables = [['bar', 'baz', 'foo', 'qux'], ['one', 'two']]

In [2]: index = pd.MultiIndex.from_product(iterables)
In [3]: index
Out[3]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])
```

还可以将 DataFrame 直接转成 MultiIndex，每一列代表索引的一个级别，如果没有显式地设置 names 参数，则默认使用列名作为各级别名称：

```python
In [1]: df = pd.DataFrame([['bar', 'one'], ['bar', 'two'],
                   ['foo', 'one'], ['foo', 'two']],
                  columns=['first', 'second'])

In [2]: index = pd.MultiIndex.from_frame(df)
In [3]: index
Out[3]: 
MultiIndex(levels=[['bar', 'foo'], ['one', 'two']],
           codes=[[0, 0, 1, 1], [0, 1, 0, 1]],
           names=['first', 'second'])
```

## 创建带层次化索引的 Series/DataFrame

创建带层次化索引的 Series 和 DataFrame 非常简单，只需要将 MultiIndex 类型或对应的标签数组列表传入 index 参数即可：

```python
In [1]: arrays = [['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux'],
                  ['one', 'two', 'one', 'two', 'one', 'two', 'one', 'two']]

In [2]: s = pd.Series(np.random.randn(8), index=arrays)
In [3]: s
Out[3]: 
bar  one    1.263382
     two    0.602905
baz  one   -1.256620
     two   -0.465676
foo  one   -0.945741
     two    0.539336
qux  one   -0.799361
     two    0.862340
dtype: float64

In [4]: df = pd.DataFrame(np.random.randn(8, 4), index=arrays)
In [5]: df
Out[5]: 
                0         1         2         3
bar one -1.236381 -0.530875  0.077212 -0.470537
    two -0.026897  0.054784  0.722662 -0.453915
baz one  1.105476  0.844287  1.335848  0.214336
    two  0.121810 -0.177061 -2.461339 -3.835201
foo one  0.435589  0.201775  0.629469  1.526154
    two  0.189863 -1.225097 -0.427277  1.715833
qux one  0.802831  0.136600  1.747222  1.458064
    two  0.016336 -0.982178 -0.220142 -0.613992
```

层次化所以不仅可以用于行索引，也可以用于列索引：

```python
In [1]: df = pd.DataFrame(np.random.randn(3, 8), index=['A', 'B', 'C'], columns=index)
In [2]: df
Out[2]: 
first        bar                 baz                 foo                 qux          
second       one       two       one       two       one       two       one       two
A       0.895717  0.805244 -1.206412  2.565646  1.431256  1.340309 -1.170299 -0.226169
B       0.410835  0.813850  0.132003 -0.827317 -0.076467 -1.187678  1.130127 -1.436737
C      -1.413681  1.607920  1.024180  0.569605  0.875906 -2.211372  0.974466 -2.006747

In [3]: df = pd.DataFrame(np.random.randn(6, 6), index=index[:6], columns=index[:6])
In [4]: df
Out[4]: 
first              bar                 baz                 foo          
second             one       two       one       two       one       two
first second                                                            
bar   one    -0.514887 -0.003622 -0.215498  0.242727 -1.140779 -1.726628
      two     0.746819 -0.087636  0.702180 -1.051544  1.562155 -0.617531
baz   one    -0.730083 -1.252182  1.876346 -0.138117 -0.431828  1.478820
      two    -0.498563  1.059249  1.784043 -0.893297  0.226484 -0.751087
foo   one    -0.835002  0.225263 -0.414389  0.093658 -0.877045  1.107530
      two    -0.107460 -0.955070 -1.556112 -0.887133  0.165047 -1.435674
```

值得注意的是，如果传入的是元组列表（上面讲到的第二种构造层次化索引的方式），pandas 并不会自动将其解析为层次化索引，而是将整个元组当作一个索引级别。例如在下面的例子中，只有一个索引级别：

```python
In [1]: s = pd.Series(np.random.randn(8), index=tuples)
In [1]: s
Out[1]: 
(qux, two)   -0.317093
(foo, two)   -0.915911
(foo, one)    1.195680
(baz, one)   -2.420424
(baz, two)   -0.885700
(bar, one)    1.003974
(qux, one)    0.370383
(bar, two)    1.457808
dtype: float64
```

## 检索层次化索引

和普通索引一样，层次化索引可以通过 iloc 方法进行基于位置的检索、通过 loc 方法进行基于轴标签的检索。其中 loc 方法的输入参数略有不同，需要用标签元组来替代单个轴标签：

```python
In [1]: df = df.T
In [2]: df
Out[2]:
                     A         B         C
first second                              
bar   one    -0.155753 -0.655474  0.243651
      two     0.013189 -0.599820 -0.447043
baz   one     1.735876 -0.429916  0.325410
      two     1.728114 -2.251222  0.930528
foo   one     1.245292 -0.442700 -0.585784
      two     1.358895  0.854176 -1.289802
qux   one    -1.503507  0.737687  1.673752
      two    -2.085532  1.124874 -0.325829

In [3: df.loc[('bar', 'two')]
Out[3]:
A    0.013189
B   -0.599820
C   -0.447043
Name: (bar, two), dtype: float64

In [4]: df.loc[[('bar', 'two'), ('qux', 'one')]]
Out[4]:
                     A         B         C
first second                              
bar   two     0.013189 -0.599820 -0.447043
qux   one    -1.503507  0.737687  1.673752

In [5]: df.loc[('baz', 'two'):('qux', 'one')]
Out[5]:
                     A         B         C
first second                              
baz   two     1.728114 -2.251222  0.930528
foo   one     1.245292 -0.442700 -0.585784
      two     1.358895  0.854176 -1.289802
qux   one    -1.503507  0.737687  1.673752

In [6]: df['A'] > 0
Out[6]:
first  second
bar    one       False
       two        True
baz    one        True
       two        True
foo    one        True
       two        True
qux    one       False
       two       False
Name: A, dtype: bool

In [7]: df.loc[df['A'] > 0]
Out[7]:
                     A         B         C
first second                              
bar   two     0.013189 -0.599820 -0.447043
baz   one     1.735876 -0.429916  0.325410
      two     1.728114 -2.251222  0.930528
foo   one     1.245292 -0.442700 -0.585784
      two     1.358895  0.854176 -1.289802
```

同时对行、列进行索引的方法也类似：

```python
In [1]: df.loc[('bar', 'two'), 'A']
Out[1]: 0.09833600046171068

In [2]: df.loc[('bar', 'two'): ('foo', 'one'), 'A']
Out[2]:
first  second
bar    two       0.013189
baz    one       1.735876
       two       1.728114
foo    one       1.245292
Name: A, dtype: float64
```

层次化索引的一个重要特性就是**部分索引**，即只提供索引的部分标签，选取包含该标签的全部层次化索引：

```python
In [1]: df.loc[('bar',)]
Out[1]:
               A         B         C
second                              
one    -0.155753 -0.655474  0.243651
two     0.013189 -0.599820 -0.447043

In [2]: df.loc[('bar', 'two'): ('baz',)]
Out[2]:
                     A         B         C
first second                              
bar   two     0.013189 -0.599820 -0.447043
baz   one     1.735876 -0.429916  0.325410
      two     1.728114 -2.251222  0.930528

In [3]: df.loc[('bar',): ('baz',)]
Out[3]:
                     A         B         C
first second                              
bar   one    -0.155753 -0.655474  0.243651
      two     0.013189 -0.599820 -0.447043
baz   one     1.735876 -0.429916  0.325410
      two     1.728114 -2.251222  0.930528
```

进行部分索引时，可以使用简略的写法，效果时一样的：

```python
In [1]: df.loc['bar']
Out[1]:
               A         B         C
second                              
one    -0.155753 -0.655474  0.243651
two     0.013189 -0.599820 -0.447043

In [2]: df.loc[('bar', 'two'): 'baz']
Out[2]:
                     A         B         C
first second                              
bar   two     0.013189 -0.599820 -0.447043
baz   one     1.735876 -0.429916  0.325410
      two     1.728114 -2.251222  0.930528

In [3]: df.loc['bar': 'baz']
Out[3]:
                     A         B         C
first second                              
bar   one    -0.155753 -0.655474  0.243651
      two     0.013189 -0.599820 -0.447043
baz   one     1.735876 -0.429916  0.325410
      two     1.728114 -2.251222  0.930528
```

索引运算符 [] 也支持简单的部分索引：

```python
In [1]: df = df.T
In [2]: df
Out[2]:
first        bar                 baz                 foo                 qux          
second       one       two       one       two       one       two       one       two
A      -0.155753  0.013189  1.735876  1.728114  1.245292  1.358895 -1.503507 -2.085532
B      -0.655474 -0.599820 -0.429916 -2.251222 -0.442700  0.854176  0.737687  1.124874
C       0.243651 -0.447043  0.325410  0.930528 -0.585784 -1.289802  1.673752 -0.325829

In [3]: df['bar']
Out[3]:
second       one       two
A      -0.155753  0.013189
B      -0.655474 -0.599820
C       0.243651 -0.447043

In [4]: df['bar', 'one']
Out[4]:
A   -0.155753
B   -0.655474
C    0.243651
Name: (bar, one), dtype: float64

In [5]: df['bar']['one']
Out[5]:
A   -0.155753
B   -0.655474
C    0.243651
Name: one, dtype: float64
```