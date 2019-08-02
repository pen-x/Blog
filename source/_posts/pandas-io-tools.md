---
title: Pandas IO 工具
date: 2019-07-23 16:26:52
tags:
  - pandas
toc: true
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
ValueError: Specified a delimiter with both sep and delim_whitespace=True; 
you can only specify one.
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

```python
In [1]: data = ('a,b,a\n'
                '0,1,2\n'
                '3,4,5')

In [1]: pd.read_csv(io.StringIO(data))
Out[1]:
   a  b  a.1
0  0  1    2
1  3  4    5

In [3]: pd.read_csv(io.StringIO(data), mangle_dupe_cols=False)
...
ValueError: Setting mangle_dupe_cols=False is not supported yet
```

## 解析设置

- dtype: 如果不想采用 pandas 自动推断的列数据类型，可以通过设置 dtype 参数来改变：
  - 单个类型名，将所有列数据都转化为指定类型，存在不满足列时会抛出异常
    - python 基本类型，如 int、float 等
    - numpy 数据类型，如 np.float64, np.int32 等
    - 类型字符串，如 'float', 'int32' 等
  - 列名 -> 类型字典，将指定列转化为对应类型，其它列使用默认推断
- true_values: 列表类型，解析时可以被当作 True 的所有值
- false_values: 列表类型，解析时可以被当作 False 的所有值
- **na_values**: 额外被解析为 NAN（非数字）的值，默认被认作 NaN 的有 ‘’, ‘#N/A’, ‘#N/A N/A’, ‘#NA’, ‘-1.#IND’, ‘-1.#QNAN’, ‘-NaN’, ‘-nan’, ‘1.#IND’, ‘1.#QNAN’, ‘N/A’, ‘NA’, ‘NULL’, ‘NaN’, ‘n/a’, ‘nan’, ‘null’.
  - 单个值、字符串
  - 列表
  - 字典：对每一列设置不同的 NAN 值
- keep_default_na: 布尔类型，是否包含默认被解析为 NAN 的值，默认为 True
  - 如果 keep_default_na=True，则解析时既考虑默认 NAN 值，也考虑用户通过 na_value 设置的值
  - 如果 keep_default_na=False，则解析时只考虑用户通过 na_value 设置的值，如果没有设置 na_value，则没有值会被解析为 NAN
- **na_filter**: 布尔类型，是否检测 NAN 值，默认为 True，如果确认原始数据中没有 NAN 值则可以设置为 False 来提高解析速度
- skiprows: 输入文件开头需要跳过的行
  - 数字列表类型：选取对应位置的行，如 [0, 1, 2]
  - 函数类型：对每一行的行号调用该函数，如果函数返回 True 则跳过该行
- skip_blank_lines: 布尔类型，默认为 True，跳过空数据行。
- **nrows**: 整数类型，读取的数据行数，当源文件很大时可以用来读取部分数据。
- comment：如果一行中出现了 comment 标志，则该标志后面的字段都被认为是注释，不参与数据处理。
- lineterminator：换行符。

```python
In [1]: data = ('a,b,c,d\n'
                '1,2,3,4\n'
                '5,6,7,8\n'
                '9,10,11')

In [2]: df = pd.read_csv(io.StringIO(data))
In [3]: print(df.dtypes)
Out[3]:
a      int64
b      int64
c      int64
d    float64
dtype: object
```

```python
In [2]: df = pd.read_csv(io.StringIO(data), dtype=object)
In [3]: print(df.dtypes)
Out[3]:
a    object
b    object
c    object
d    object
dtype: object

In [3]: df['a'][0]
Out[3]: '1'
```

```python
In [2]: df = pd.read_csv(io.StringIO(data), 
                        dtype={'b': object, 'c': np.float64, 'd': 'Int64'})
In [3]: print(df.dtypes)
Out[3]:
a      int64
b     object
c    float64
d      Int64
dtype: object
```

```python
In [1]: data = ('a,b,c\n'
                '1,Yes,2\n'
                '3,No,4')

In [2]: pd.read_csv(io.StringIO(data))
Out[1]:
   a    b  c
0  1  Yes  2
1  3   No  4
```

```python
In [2]: pd.read_csv(io.StringIO(data), true_values=['Yes'], false_values=['No'])
Out[1]:
   a      b  c
0  1   True  2
1  3  False  4
```

```python
In [1]: data = ('something,a,b,c,d,message\n'
                'one,1,2,3,4,NA\n'
                'two,5,6,,8,world\n'
                'three,9,10,11,12,foo\n')

In [2]: pd.read_csv(io.StringIO(data))
Out[1]:
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     foo
```

可以看出默认情况下空字符串 和字符串 NA 都被识别为了 NAN 值，如果我们手动设置 na_values，那么 foo 也会被认作 NAN。

```python
In [1]: pd.read_csv(io.StringIO(data), na_values=['foo'])
Out[1]:
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       two  5   6   NaN   8   world
2     three  9  10  11.0  12     NaN
```

禁用默认 NAN 值，此时只有 foo 会被解析为 NAN，空字符串和字符串 NA 都保留了原值。

```python
In [1]: pd.read_csv(io.StringIO(data), na_values=['foo'], keep_default_na=False)
Out[1]:
  something  a   b   c   d message
0       one  1   2   3   4      NA
1       two  5   6       8   world
2     three  9  10  11  12     NaN
```

还可以为不同的列指定不同的 NAN 值。

```python
In [1]: sentinels = {'message': ['foo', 'NA'], 'something': ['two']}
In [1]: pd.read_csv(io.StringIO(data), na_values=sentinels)
Out[1]:
  something  a   b     c   d message
0       one  1   2   3.0   4     NaN
1       NaN  5   6   NaN   8   world
2     three  9  10  11.0  12     NaN
```

```python
In [1]: data = ('# hey!\n'
                'a,b,c,d,message\n'
                '# just wanted to make things more difficult for you\n'
                '# who reads CSV files with computers, anyway?\n'
                '1,2,3,4,hello\n'
                '5,6,7,8,world\n'
                '9,10,11,12,foo\n')

In [2]: pd.read_csv(io.StringIO(data), skiprows=[0, 2, 3])
Out[1]:
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

```python
In [2]: pd.read_csv(io.StringIO(data), comment='#')
Out[1]:
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

```python
In [1]: data = 'a,b,c~1,2,3~4,5,6'

In [2]: pd.read_csv(io.StringIO(data), lineterminator='~')
Out[1]:
   a  b  c
0  1  2  3
1  4  5  6
```

## 迭代设置

- chunksize
- iterator：布尔类型，设置为 True 将返回 TextFileReader 对象，用于 python 的 for 循环迭代，或直接调用 get_chunk() 方法

```python
In [1]: data = ('|0|1|2|3\n'
'0|0.4691122999071863|-0.2828633443286633|-1.5090585031735124|-1.1356323710171934\n'
'1|1.2121120250208506|-0.17321464905330858|0.11920871129693428|-1.0442359662799567\n'
'2|-0.8618489633477999|-2.1045692188948086|-0.4949292740687813|1.071803807037338\n'
'3|0.7215551622443669|-0.7067711336300845|-1.0395749851146963|0.27185988554282986\n'
'4|-0.42497232978883753|0.567020349793672|0.27623201927771873|-1.0874006912859915\n'
'5|-0.6736897080883706|0.1136484096888855|-1.4784265524372235|0.5249876671147047\n'
'6|0.4047052186802365|0.5770459859204836|-1.7150020161146375|-1.0392684835147725\n'
'7|-0.3706468582364464|-1.1578922506419993|-1.344311812731667|0.8448851414248841\n'
'8|1.0757697837155533|-0.10904997528022223|1.6435630703622064|-1.4693879595399115\n'
'9|0.35702056413309086|-0.6746001037299882|-1.776903716971867|-0.9689138124473498\n')

In [2]: reader = pd.read_csv(io.StringIO(data), sep='|', chunksize=4)
In [2]: print(reader)
Out[1]: <pandas.io.parsers.TextFileReader object at 0x0000027B441839B0>
```

```python
In [1]: for chunk in reader:
            print(chunk)

Out[1]:
   Unnamed: 0         0         1         2         3
0           0  0.469112 -0.282863 -1.509059 -1.135632
1           1  1.212112 -0.173215  0.119209 -1.044236
2           2 -0.861849 -2.104569 -0.494929  1.071804
3           3  0.721555 -0.706771 -1.039575  0.271860
   Unnamed: 0         0         1         2         3
4           4 -0.424972  0.567020  0.276232 -1.087401
5           5 -0.673690  0.113648 -1.478427  0.524988
6           6  0.404705  0.577046 -1.715002 -1.039268
7           7 -0.370647 -1.157892 -1.344312  0.844885
   Unnamed: 0         0        1         2         3
8           8  1.075770 -0.10905  1.643563 -1.469388
9           9  0.357021 -0.67460 -1.776904 -0.968914
```

```python
In [1]: reader = pd.read_csv(io.StringIO(data), sep='|', iterator=True)

In [1]: print(reader.get_chunk(1))
Out[1]:
   Unnamed: 0         0         1         2         3
0           0  0.469112 -0.282863 -1.509059 -1.135632

In [1]: print(reader.get_chunk(2))
Out[1]:
   Unnamed: 0         0         1         2         3
1           1  1.212112 -0.173215  0.119209 -1.044236
2           2 -0.861849 -2.104569 -0.494929  1.071804
```

## 引用设置

- quotechar：csv 中的字段如果包含分隔符，为了避免解析时出错，会使用引用符合将字段包围起来，引用符号可以通过 quotechar 参数设置
- quoting：控制字段的引用行为，包括 
  - csv.QUOTE_MINIMAL (0)：默认值，仅在字段包含特殊字符时引用，如分隔符、引用符号、换行符
  - csv.QUOTE_ALL (1)：给所有字段添加引用符号
  - csv.QUOTE_NONNUMERIC (2)：仅给非数字字段添加引用符号
  - csv.QUOTE_NONE (3)：不添加引用符号，如果字段中存在特殊符号会报错
- doublequote：是否将字段中的连续两个引用符号解析为单个引用符号，默认为 True。csv 文件的字段中如果出现了引用符号，通常会用双引用符号替代来避免歧义
- escapechar：转义字符标记，当 quoting=csv.QUOTE_NONE 时用来转义特殊符号

```python
In [1]: data = ('label1,label2,label3\n'
                'index1,"a","c""g",e\n'
                'index2,b,d,f\n')

In [1]: pd.read_csv(io.StringIO(data))
Out[1]:
       label1 label2 label3
index1      a    c"g      e
index2      b      d      f

In [1]: pd.read_csv(io.StringIO(data), quoting=csv.QUOTE_NONE)
Out[1]:
       label1  label2 label3
index1    "a"  "c""g"      e
index2      b       d      f
```

## 错误处理

- error_bad_lines：布尔类型，默认为 True，如果源数据中的某一行出现了过多的字段，则解析出错，抛出异常。设置为 False 可以忽略错误行，返回其它正确解析的结果。
- warn_bad_lines：布尔类型，默认为 True，为每个错误行输出警告信息。

```python
In [1]: data = ('a,b,c\n'
                '1,2,3\n'
                '4,5,6,7\n'
                '8,9,10')

In [1]: pd.read_csv(io.StringIO(data))
Out[1]:
...
ParserError: Error tokenizing data. C error: Expected 3 fields in line 3, saw 4

In [1]: pd.read_csv(io.StringIO(data), error_bad_lines=False)
Out[1]:
   a  b   c
0  1  2   3
1  8  9  10

b'Skipping line 3: expected 3 fields, saw 4\n'
```
