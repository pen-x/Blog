---
title: pandas 层次化索引
date: 2019-07-29 20:31:09
tags:
  - pandas
toc: true
---

层次化索引（hierarchical indexing）是pandas的一项重要功能，它使你能在一个轴上拥有多个（两个以上）索引级别。抽象点说，它使你能以低维度形式处理高维度数据。

## 构造函数

names 用于指定 index 各 level 的名字。

```python
In [1]: arrays = [['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux'],
                  ['one', 'two', 'one', 'two', 'one', 'two', 'one', 'two']]

In [1]: index = pd.MultiIndex.from_arrays(arrays)
In [1]: index
Out[1]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])
```

```python
In [1]: tuples = list(zip(*arrays))

In [1]: tuples
Out[1]: 
[('bar', 'one'),
 ('bar', 'two'),
 ('baz', 'one'),
 ('baz', 'two'),
 ('foo', 'one'),
 ('foo', 'two'),
 ('qux', 'one'),
 ('qux', 'two')]

In [1]: index = pd.MultiIndex.from_tuples(tuples, names=['first', 'second'])
In [1]: index
Out[1]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]],
           names=['first', 'second'])
```

```python
In [1]: iterables = [['bar', 'baz', 'foo', 'qux'], ['one', 'two']]

In [1]: index = pd.MultiIndex.from_product(iterables)
In [1]: index
Out[1]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])
```

```python
In [1]: df = pd.DataFrame([['bar', 'one'], ['bar', 'two'],
                   ['foo', 'one'], ['foo', 'two']],
                  columns=['first', 'second'])

In [1]: pd.MultiIndex.from_frame(df)
Out[1]: 
MultiIndex(levels=[['bar', 'foo'], ['one', 'two']],
           codes=[[0, 0, 1, 1], [0, 1, 0, 1]],
           names=['first', 'second'])
```

## MultiIndex 属性

- names:
- levels:
- codes:
- nlevels:
- levshape:

```python
In [1]: index
Out[1]: 
MultiIndex(levels=[['bar', 'baz', 'foo', 'qux'], ['one', 'two']],
           codes=[[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])

In [1]: index.levels
Out[1]: FrozenList([['bar', 'baz', 'foo', 'qux'], ['one', 'two']])

In [1]: index.codes
Out[1]: FrozenList([[0, 0, 1, 1, 2, 2, 3, 3], [0, 1, 0, 1, 0, 1, 0, 1]])

In [1]: index.names
Out[1]: FrozenList([None, None])

In [1]: index.nlevels
Out[1]: 2

In [1]: index.levshape
Out[1]: (4, 2)
```

## 创建带层次化索引的 Series/DataFrame


```python
In [1]: arrays = [['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux'],
                  ['one', 'two', 'one', 'two', 'one', 'two', 'one', 'two']]

In [1]: s = pd.Series(np.random.randn(8), index=arrays)
In [1]: s
Out[1]: 
bar  one    1.263382
     two    0.602905
baz  one   -1.256620
     two   -0.465676
foo  one   -0.945741
     two    0.539336
qux  one   -0.799361
     two    0.862340
dtype: float64

In [1]: df = pd.DataFrame(np.random.randn(8, 4), index=arrays)
In [1]: df
Out[1]: 
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

```python
In [1]: df = pd.DataFrame(np.random.randn(3, 8), index=['A', 'B', 'C'], columns=index)
In [1]: df
Out[1]: 
first        bar                 baz                 foo                 qux          
second       one       two       one       two       one       two       one       two
A       0.895717  0.805244 -1.206412  2.565646  1.431256  1.340309 -1.170299 -0.226169
B       0.410835  0.813850  0.132003 -0.827317 -0.076467 -1.187678  1.130127 -1.436737
C      -1.413681  1.607920  1.024180  0.569605  0.875906 -2.211372  0.974466 -2.006747

In [1]: df = pd.DataFrame(np.random.randn(6, 6), index=index[:6], columns=index[:6])
In [1]: df
Out[1]: 
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

## MultiIndex 索引

```python
In [1]: df['bar']
Out[1]: 
        one       two
A -0.253498  0.224826
B  2.025727  0.295492
C  0.186436 -0.678404

In [1]: df['bar', 'one']
Out[1]: 
A   -2.622886
B    0.780131
C    0.940327
Name: (bar, one), dtype: float64

In [1]: df['bar']['one']
Out[1]: 
A   -2.622886
B    0.780131
C    0.940327
Name: one, dtype: float64

In [1]: s['qux']
Out[1]: 
one   -0.799361
two    0.862340
dtype: float64
```

```python
In [1]: df.loc[('bar', 'two')]
Out[1]: 
A    0.098336
B    1.001876
C   -0.208750
Name: (bar, two), dtype: float64

In [1]: df.loc[('bar', 'two'), 'A']
Out[1]: 0.09833600046171068

In [1]: df.loc[('bar',),]
Out[1]: 
```