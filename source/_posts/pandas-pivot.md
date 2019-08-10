---
title: Pandas 轴向旋转
date: 2019-08-10 16:01:11
tags:
  - pandas
---

有许多用于重新排列表格型数据的基础运算。这些函数也称作**重塑（reshape）**或**轴向旋转（pivot）**运算。

- pivot
- stack/unstack
- melt

## pivot 方法

![](/images/reshaping_pivot.png)

在日常生活中，我们常使用记录（Record）的格式来存储数据。例如在下面的例子中，一个日期 date 和一个变量 variable 构成一条记录 record，记录的值为 value。date 共有 '2000-01-03'、'2000-01-04' 和 '2000-01-05' 三种，variable 有 A、B、C、D 四种，因此一共有 12 条记录。

```python
In [1]: data = {
            'date': list(pd.date_range('2000-01-03', '2000-01-05')) * 4,
            'variable': list('AAABBBCCCDDD'),
            'value': np.random.randn(12)
        }

In [2]: df = pd.DataFrame(data)

In [3]: df
Out[3]:
         date variable     value
0  2000-01-03        A  0.109682
1  2000-01-04        A -0.760654
2  2000-01-05        A -0.221242
3  2000-01-03        B  0.238108
4  2000-01-04        B -1.464651
5  2000-01-05        B  0.237751
6  2000-01-03        C  1.315819
7  2000-01-04        C -0.396375
8  2000-01-05        C -0.092896
9  2000-01-03        D -0.165830
10 2000-01-04        D  0.577890
11 2000-01-05        D -0.888130
```

如果我们想要选出 variable 为 A 的所有记录，可以使用前面介绍过的布尔索引：

```python
In [1]: df[df['variable'] == 'A']
Out[1]:
        date variable     value
0 2000-01-03        A  0.109682
1 2000-01-04        A -0.760654
2 2000-01-05        A -0.221242
```

但是更好的做法是创建一个新的 DataFrame，索引为 date 的不同值、列为 variable 的不同值。这就需要轴向旋转操作，DataFrame.pivot() 方法实现了这一功能：

```python
DataFrame.pivot(self, index=None, columns=None, values=None)
```

- index：用来作为旋转后 DataFrame 行索引的列，如果未指定则使用当前行索引。
- columns：用来作为旋转后 DataFrame 列索引的列，可以是多个列名。
- values：用来填充旋转后 DataFrame 中值的列，多个列名会产生层次化列索引。

```python
In [1]: df.pivot(index='date', columns='variable', values='value')
Out[1]:
variable           A         B         C        D
date                                             
2000-01-03  0.109682  0.238108  1.315819 -0.16583
2000-01-04 -0.760654 -1.464651 -0.396375  0.57789
2000-01-05 -0.221242  0.237751 -0.092896 -0.88813
```

如果没有设置 values 值，那么 DataFrame 中没有被用作 index 和 columns 的列都将被用作 values 参数。如果此时 values 值多于一个，那么就会在旋转结果的列索引产生层次化索引，values 是层次化索引的最高级别：

```python
In [1]: df['value2'] = df['value']

In [2]: df
Out[2]:
         date variable     value    value2
0  2000-01-03        A  0.109682  0.109682
1  2000-01-04        A -0.760654 -0.760654
2  2000-01-05        A -0.221242 -0.221242
3  2000-01-03        B  0.238108  0.238108
4  2000-01-04        B -1.464651 -1.464651
5  2000-01-05        B  0.237751  0.237751
6  2000-01-03        C  1.315819  1.315819
7  2000-01-04        C -0.396375 -0.396375
8  2000-01-05        C -0.092896 -0.092896
9  2000-01-03        D -0.165830 -0.165830
10 2000-01-04        D  0.577890  0.577890
11 2000-01-05        D -0.888130 -0.888130

In [3]: pivoted = df.pivot(index='date', columns='variable')

In [4]: pivoted
Out[4]:
               value                                 value2            \
variable           A         B         C        D         A         B   
date                                                                    
2000-01-03  0.109682  0.238108  1.315819 -0.16583  0.109682  0.238108   
2000-01-04 -0.760654 -1.464651 -0.396375  0.57789 -0.760654 -1.464651   
2000-01-05 -0.221242  0.237751 -0.092896 -0.88813 -0.221242  0.237751   

                               
variable           C        D  
date                           
2000-01-03  1.315819 -0.16583  
2000-01-04 -0.396375  0.57789  
2000-01-05 -0.092896 -0.88813  
```

可以从中选出只包含 value2 的部分：

```python
In [1]: pivoted['value2']
Out[1]:
variable           A         B         C        D
date                                             
2000-01-03  0.109682  0.238108  1.315819 -0.16583
2000-01-04 -0.760654 -1.464651 -0.396375  0.57789
2000-01-05 -0.221242  0.237751 -0.092896 -0.88813
```

需要注意的是，如果（index，column）的元组在原始 DataFrame 中存在重复项，则会抛出异常。

## stack/unstack 方法

DataFrame 提供了 stack/unstack 方法来

- stack：将列索引的最内层进行轴向旋转，变成行索引的最内层。
    - level：索引的级别或标签，默认值为 -1，即最内层索引。
    - dropna：是否在结果中丢弃该列，默认为 True。
- unstack：将行索引的最内层进行轴向旋转，变成列索引的最内层。
    - level：索引的级别或标签，默认值为 -1，即最内层索引。
    - fill_value：用于替换 Nan 值。

这两个方法的输入参数相同：


![](/images/reshaping_stack.png)

![](/images/reshaping_unstack.png)

```python
In [1]: tuples = list(zip(*[['bar', 'bar', 'baz', 'baz', 'foo', 'foo', 'qux', 'qux'],
                            ['one', 'two', 'one', 'two', 'one', 'two', 'one', 'two']]))

In [2]: index = pd.MultiIndex.from_tuples(tuples, names=['first', 'second'])

In [3]: df = pd.DataFrame(np.random.randn(8, 2), index=index, columns=['A', 'B'])

In [4]: df2 = df[:4]

In [5]: df2
Out[5]:
                     A         B
first second                    
bar   one    -0.034398  0.415722
      two     0.244418 -0.100704
baz   one     0.094069  0.302489
      two     1.683473  0.084727
```

stack 方法将列索引的最内层转化成行索引的最内层，如果列索引是单层索引，那么生成的结果就是一个 Series：

```python
In [1]: stacked = df2.stack()

In [2]: stacked
Out[2]:

first  second   
bar    one     A   -0.034398
               B    0.415722
       two     A    0.244418
               B   -0.100704
baz    one     A    0.094069
               B    0.302489
       two     A    1.683473
               B    0.084727
dtype: float64
```

unstack 方法则是 stack 方法的逆方法：

```python
In [1]: stacked.unstack()

Out[1]:
                     A         B
first second                    
bar   one    -0.034398  0.415722
      two     0.244418 -0.100704
baz   one     0.094069  0.302489
      two     1.683473  0.084727
```

可以设置 level 参数来指定 unstack 时旋转不同的索引级别：

```python
In [1]: stacked.unstack(1)

Out[1]:
second        one       two
first                      
bar   A -0.034398  0.244418
      B  0.415722 -0.100704
baz   A  0.094069  1.683473
      B  0.302489  0.084727

In [2]: stacked.unstack(0)

Out[2]:
first          bar       baz
second                      
one    A -0.034398  0.094069
       B  0.415722  0.302489
two    A  0.244418  1.683473
       B -0.100704  0.084727
```

![](/images/reshaping_unstack_1.png)

![](/images/reshaping_unstack_0.png)

使用索引标签可以达到同样的效果：

```python
In [1]: stacked.unstack('second')

Out[1]:
second        one       two
first                      
bar   A -0.034398  0.244418
      B  0.415722 -0.100704
baz   A  0.094069  1.683473
      B  0.302489  0.084727
```

需要注意的是，stack 和 unstack 方法都会对索引进行排序：

```python
In [1]: index = pd.MultiIndex.from_product([[2, 1], ['a', 'b']])

In [2]: df = pd.DataFrame(np.random.randn(4), index=index, columns=['A'])

In [3]: df
Out[3]:
            A
2 a -0.070235
  b -2.225196
1 a -1.176649
  b -0.579985

In [4]: df.usntack()
Out[4]:
          A          
          a         b
1 -1.176649 -0.579985
2 -0.070235 -2.225196

In [5]: df.unstack().stack()
Out[5]:
            A
1 a -1.176649
  b -0.579985
2 a -0.070235
  b -2.225196

In [6]: all(df.unstack().stack() == df.sort_index())
Out[6]: True
```

### 处理多个索引级别

level 参数也可以设置为索引级别或标签列表：

```python
In [1]: columns = pd.MultiIndex.from_tuples([
            ('A', 'cat', 'long'), ('B', 'cat', 'long'),
            ('A', 'dog', 'short'), ('B', 'dog', 'short')
        ], names = ['exp', 'animal', 'hair_length'])

In [2]: df = pd.DataFrame(np.random.randn(4, 4), columns=columns)

In [3]: df
Out[3]:
exp                 A         B         A         B
animal            cat       cat       dog       dog
hair_length      long      long     short     short
0           -0.540375  1.577036 -0.060911  0.861272
1            0.130549  0.592937 -0.607023 -2.064989
2           -0.399434 -1.706509 -0.207034  1.399829
3           -0.019359 -0.478885 -0.011463 -1.273179

In [4]: df.stack(level=['animal', 'hair_length'])
Out[4]:
exp                          A         B
  animal hair_length                    
0 cat    long        -0.540375  1.577036
  dog    short       -0.060911  0.861272
1 cat    long         0.130549  0.592937
  dog    short       -0.607023 -2.064989
2 cat    long        -0.399434 -1.706509
  dog    short       -0.207034  1.399829
3 cat    long        -0.019359 -0.478885
  dog    short       -0.011463 -1.273179
```

级别数和级别标签效果一致：

```python
In [1]: df.stack(level=[1, 2])
Out[1]:

exp                          A         B
  animal hair_length                    
0 cat    long        -0.540375  1.577036
  dog    short       -0.060911  0.861272
1 cat    long         0.130549  0.592937
  dog    short       -0.607023 -2.064989
2 cat    long        -0.399434 -1.706509
  dog    short       -0.207034  1.399829
3 cat    long        -0.019359 -0.478885
  dog    short       -0.011463 -1.273179
```

### 处理缺失数据

在 stack/unstack 的过程中，会出现

```python
In [1]: columns = pd.MultiIndex.from_tuples([('A', 'cat'), ('B', 'dog'),
                                             ('B', 'cat'), ('A', 'dog')],
                                             names=['exp', 'animal'])

In [2]: index = pd.MultiIndex.from_product([('bar', 'baz', 'foo', 'qux'), 
                                            ('one', 'two')],
                                            names=['first', 'second'])

In [3]: df = pd.DataFrame(np.random.randn(8, 4), index=index, columns=columns)

In [4]: df
Out[4]:
exp                  A         B                   A
animal             cat       dog       cat       dog
first second                                        
bar   one    -0.693096  0.878046  1.091910  0.936811
      two     1.188222 -1.664456 -0.549990  1.051823
baz   one    -1.791475  0.994634  0.953791  0.676937
      two     0.527280 -0.243705  0.464124  0.358392
foo   one     1.185904 -2.313854  2.805139 -0.322814
      two     1.190213 -0.072086  1.194713  0.095381
qux   one     0.558042  0.093665 -0.778069 -0.225832
      two     2.225235 -0.230078 -0.599908  0.518119
```

在下面的例子中，没有 (bar, one, dog, A) 和 (bar, two, dog, A) 的记录，因此 stack() 后用 NaN 填充。

```python
In [1]: df2 = df.iloc[:2, [0,1,2]]

In [2]: df2
Out[2]:
exp                  A         B         
animal             cat       dog      cat
first second                             
bar   one    -0.693096  0.878046  1.09191
      two     1.188222 -1.664456 -0.54999
    
In [3]: df2.stack()
Out[3]:
exp                         A         B
first second animal                    
bar   one    cat    -0.693096  1.091910
             dog          NaN  0.878046
      two    cat     1.188222 -0.549990
             dog          NaN -1.664456
```

unstack() 方法也会遇到同样的问题：

```python
In [1]: df3 = df.iloc[[0, 1, 4, 7], [1, 2]]

In [2]: df3
exp                  B          
animal             dog       cat
first second                    
bar   one     0.878046  1.091910
      two    -1.664456 -0.549990
foo   one    -2.313854  2.805139
qux   two    -0.230078 -0.599908

In [3]: df3.unstack()
Out[3]:
exp            B                              
animal       dog                 cat          
second       one       two       one       two
first                                         
bar     0.878046 -1.664456  1.091910 -0.549990
foo    -2.313854       NaN  2.805139       NaN
qux          NaN -0.230078       NaN -0.599908
```

在 unstack() 方法中可以设置 fill_value 来设置填充值：

```python
In [1]: df3.unstack(fill_value=-1e9)
Out[1]:
exp                B                                          
animal           dog                         cat              
second           one           two           one           two
first                                                         
bar     8.780463e-01 -1.664456e+00  1.091910e+00 -5.499900e-01
foo    -2.313854e+00 -1.000000e+09  2.805139e+00 -1.000000e+09
qux    -1.000000e+09 -2.300780e-01 -1.000000e+09 -5.999080e-01
```

## melt 方法

![](/images/reshaping_melt.png)

```python
In [1]: cheese = pd.DataFrame({
            'first': ['John', 'Mary'],
            'last': ['Doe', 'Bo'],
            'height': [5.5, 6.0],
            'weight': [130, 150]
        })

In [2]: cheese
Out[2]:
  first last  height  weight
0  John  Doe     5.5     130
1  Mary   Bo     6.0     150

In [3]: cheese.melt(id_vars=['first', 'last'])
Out[3]:
  first last variable  value
0  John  Doe   height    5.5
1  Mary   Bo   height    6.0
2  John  Doe   weight  130.0
3  Mary   Bo   weight  150.0

In [4]: cheese.melt(id_vars=['first', 'last'], var_name='quantity')
Out[4]:
  first last quantity  value
0  John  Doe   height    5.5
1  Mary   Bo   height    6.0
2  John  Doe   weight  130.0
3  Mary   Bo   weight  150.0
```