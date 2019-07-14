---
title: pandas series
tags:
  - Pandas
date: 2019-07-13 11:45:38
---


Pandas.Series 是一个一维的带标签数组，可以保存任何类型的数据 (整数，字符串，浮点数，Python对象等)，轴标签统称为索引。

Series 的索引既可以是基于数字的，如 [1, 2, 3, 4, 5]， 也可以是基于标记的，如 ['a', 'b', 'c', 'd']，同时重复的索引值也是合理的，如 ['a', 'a', 'b']。

## 构造函数

```python
pandas.Series(data=None, index=None, dtype=None, name=None, copy=False, fastpath=False)
```

| 编号 | 参数 | 描述 |
|------|----------|------------------------------------------------------------------------|
| 1 | data | Series 中要存储的数据，可以是类 array 类型、可迭代类型、字典类型、标量 |
| 2 | index | Series 中数据的索引，需要和数据的长度相同，默认为 np.arange(n) |
| 3 | dtype | Series 的数据类型，如果没有，将推断数据类型 |
| 4 | copy | 是否从输入中复制数据，默认为 false |
| 5 | name | Series 名称 |
| 6 | fastpath |  |