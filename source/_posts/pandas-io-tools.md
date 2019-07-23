---
title: Pandas IO 工具
date: 2019-07-23 16:26:52
tags:
  - pandas
---

Pandas 提供了很多用于将表格型数据读取为 DataFrame 对象的函数，下表列举出了其中一些常用的函数及其说明。其中一些函数，比如pandas.read_csv，有类型推断功能，因为列数据的类型不属于数据类型。也就是说，你不需要指定列的类型到底是数值、整数、布尔值，还是字符串。其它的数据格式，如 HDF5、Feather 和 msgpack，会在格式中存储数据类型。

| 函数 | 说明 |
|-------------|-------------------------------------------------------------------------|
| read_csv | 从文件、URL、文件型对象中加载带分隔符的数据，默认分隔符为逗号 |
| read_table | 从文件、URL、文件型对象中加载带分隔符的数据，默认分隔符为制表符（'\t'） |
| read_fwf | 读取定宽列格式数据（也就是说，没有分隔符） |
| read_excel | 从 Excel XLS 或 XLSX 文件读取表格数据 |
| read_html | 读取 HTML 文档中的所有表格 |
| read_json | 读取 JSON 字符串中的数据 |
| read_pickle | 读取 Python pickle 格式中存储的任意对象 |

在这些函数中，read_csv 和 read_table 又是使用频率最高的，除了分隔符不同以外，这两个函数的功能及参数完全一致，并且也可以通过设置 sep 参数互相转换。

下面我们将以 read_csv 为例，详细介绍各项参数。

## 基本参数

- **filepath_or_buffer**：必填参数，用于指定数据的来源，既可以是文件路径、URL（包括 http、ftp 格式等），也可以是任何拥有 read() 函数的对象（例如 open() 函数打开的文件、StringIO() 创建的流数据）
- **sep**: 字符串类型，数据分隔符，默认为逗号，也可以是正则表达式的形式，例如 '\s+'。
- delimiter: 和 sep 参数功能一致，使用 sep 参数即可。
- delim_whitespace: 布尔类型，决定是否使用连续的空格作为分隔符，等同于设置 sep='\s+'，默认为 False。同一时刻只能设置 delim_whitespace 和 sep/delimiter 中的一个参数。

```python
In [1]: !cat example.csv
Out[1]:
a,b,c,d,message
1,2,3,4,hello
5,6,7,8,world
9,10,11,12,foo

In [1]: df = pd.read_csv('example.csv')
In [1]: print(df)
Out[1]:
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

```python
df.dtypes
```

上例演示了从文件读取数据的方式，为了方便，后文我们将直接读取经过 StringIO() 函数创建的流数据。

```python
In [1]: data = ('a,b,c,d,message\n'
                '1,2,3,4,hello\n'
                '5,6,7,8,world\n'
                '9,10,11,12,foo\n')

In [1]: df = pd.read_csv(io.StringIO(data))
In [1]: print(df)
Out[1]:
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

设置 sep='\s+' 或 delim_whitespace=True 都可以实现通过连续空格分隔数据：

```python
In [1]: data = 'a    b  c     d\n1  2 3    4\n'

In [1]: df = pd.read_csv(io.StringIO(data), sep='\s+')
In [1]: print(df)
Out[1]:
   a  b  c  d
0  1  2  3  4

In [1]: df = pd.read_csv(io.StringIO(data), delim_whitespace=True)
In [1]: print(df)
Out[1]:
   a  b  c  d
0  1  2  3  4
```

如果同时设置这两个参数，则会报错：

```python
In [1]: data = 'a    b  c     d\n1  2 3    4\n'

In [1]: df = pd.read_csv(io.StringIO(data), sep='\s+', delim_whitespace=True)

...
ValueError: Specified a delimiter with both sep and delim_whitespace=True; you can only specify one.
```

## Index/Column 设置

- **header**: 设定数据中的哪些行是 header 信息，可以被用作列名，其后的数据都是 data 信息。
  - "infer": 默认值，pandas 只能地推断列标签。如果用户没有设置 names 参数，则等同于 header=0，使用第一行推断列标签；如果用户设置了 names 参数，则等同于 header=None，原始数据中的所有行均为 data 信息。
  - None:
  - 单个数字：使用指定行生成列标签，如果设定了 skip_blank_lines=True，则注释行和空行并不会计算在内。
  - 数字列表：使用指定的多行来生成 MultiIndex 类型的列索引。
- **names**: 字符串列表，用于指定各列的列表前，一般当文件没有包含 header 时配合 header=None 使用。
- **index_col**: 使用哪些列来作为 DataFrame 的行索引：
  - None：默认值，如果列名和每行数据的数量一致，则会添加 [0, 1, 2, ...] 作为行索引；如果列名比每行数据的数量少，则使用数据行开头多出来的数据作为行索引。
  - 单个数字或数字列表：使用对应位置的列作为行索引，对于数字列表，行索引是 MultiIndex 类型的。
  - 单个字符串或字符串列表：使用对应列表前的列作为行索引，对于字符串列表，行索引是 MultiIndex 类型的。
- **usecols**: 最终返回的 DataFrame 中包含哪些列，可以接受多种数据类型：
  - 数字列表类型：选取对应位置的列，如 [0, 1, 2]，元素无先后顺序
  - 字符串列表类型：选取对应列标签的列，如 ['col1', 'col2', 'col3']，元素无先后顺序
  - 函数类型：对每一列的列标签调用该函数，如果函数返回 True 则选中该列
- squeeze: 布尔类型，如果解析后的数据只包含一列，那么就返回 Series 类型。
- prefix: 列标签前缀，仅在没有 header 信息时生效，例如 prefix='X' 则生成的列标签为 X0, X1, ...
- manglue_dupe_cols: 处理重复的列标签，默认为 True
  - False：对于重复的列标签，后面的值会覆盖前面的值
  - True：对于重复的列标签 X，各列会分别重命名为 X, X.1, ..., X.N

```python
In [1]: data = ('1,2,3,4,hello\n'
                '5,6,7,8,world\n'
                '9,10,11,12,foo\n')

In [1]: pd.read_csv(io.StringIO(data))
Out[1]:
   1   2   3   4  hello
0  5   6   7   8  world
1  9  10  11  12    foo
```

默认是以第一行推断列标签，所以第一行的各项成为了列名，这可能不是我们想要的结果，因此我们可以设置 header=None 来使用默认的

```python
In [1]: pd.read_csv(io.StringIO(data), header=None)
Out[1]:
   0   1   2   3      4
0  1   2   3   4  hello
1  5   6   7   8  world
2  9  10  11  12    foo
```

也可以自己定义列名：

```python
In [1]: pd.read_csv(io.StringIO(data), names=['a', 'b', 'c', 'd', 'message'])
Out[1]:
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

前面说过，header 的默认值为 “infer”，它会根据是否设置了 names 参数来决定 header=0 还是 header=None。如果手动设置了 header，然后又指定了 names 参数，那么第一行数据首先被选作为列标签，然后又被 names 所覆盖，最终的结果就是缺失第一行数据。因此设置 names 参数时一般使用 header 默认值或者设置 header=None。

```python
In [1]: pd.read_csv(io.StringIO(data), header=0, names=['a', 'b', 'c', 'd', 'message'])
Out[1]:
   a   b   c   d message
0  5   6   7   8   world
1  9  10  11  12     foo
```

设置 index_col 参数可以将对应列设置为行索引，下面的例子与 index_col='a' 等价：

```python
In [1]: pd.read_csv(io.StringIO(data), names=['a', 'b', 'c', 'd', 'msg'], index_col=0)
Out[1]:
    b   c   d    msg
a                    
1   2   3   4   hello
5   6   7   8   world
9  10  11  12     foo
```


```python
In [1]: pd.read_csv(io.StringIO(data), names=['a', 'b', 'c', 'd', 'msg'], 
                    index_col=['a', 'b'])
Out[1]:
       c   d    msg
a b                
1 2    3   4  hello
5 6    7   8  world
9 10  11  12    foo
```

与 usecols=['a', 'b', 'c'] 等价：

```python
In [1]: pd.read_csv(io.StringIO(data), names=['a', 'b', 'c', 'd', 'msg'],
                    usecols=[0, 1, 2])
Out[1]:
   a   b   c
0  1   2   3
1  5   6   7
2  9  10  11
```

```python
In [1]: pd.read_csv(io.StringIO(data), names=['a', 'b', 'c', 'd', 'msg'],
                    usecols=lambda x: x in ['a', 'b', 'c'])
Out[1]:
   a   b   c
0  1   2   3
1  5   6   7
2  9  10  11
```