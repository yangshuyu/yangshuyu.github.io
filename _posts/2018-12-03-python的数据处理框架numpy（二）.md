---
layout:     post
title:      python的数据处理框架numpy（二）
subtitle:   numpy的进阶使用之dtype
date:       2018-12-03
author:     Ysy
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - python
    - 数据处理
    - numpy
---

##前言
>上一篇提到的是numpy的简单使用，包括一些基础操作以及向量操作等等，这次来了解一下numpy的自定义数据类型dtype

#### 简介
numpy支持比python更多种类的数值类型, 这些数据类型在某些情况下对我们的使用很有帮助，例如numpy的数据类型与tensorflow的数据类型无缝对接

```
import numpy as np
import tensorflow as tf

print(np.int32 == tf.int32)  #True  numpy与tensflow完美契合
```
输出

```
True
```
所以numpy的数据类型的了解与处理也是很重要的

#### 数据类型对象 (dtype)
数据类型对象描述了对应于数组的固定内存块的解释，取决于以下方面：

- 数据类型（整数、浮点或者 Python 对象）

- 数据大小

- 字节序（小端或大端）

- 在结构化类型的情况下，字段的名称，每个字段的数据类型，和每个字段占用的内存块部分。

如果数据类型是子序列，它的形状和数据类型。

字节顺序取决于数据类型的前缀<或>。 <意味着编码是小端（最小有效字节存储在最小地址中，LITTLE-ENDIAN）。 >意味着编码是大端（最大有效字节存储在最小地址中， BIG-ENDIAN）。

#### 数据类型的定义

```
numpy.dtype(object, align, copy)
```
定义简单的数据类型

```
import numpy as np
dt = np.dtype(np.int32)   #int32为numpy的内置数据类型
print(dt)
```
输出

```
int32
```
定义一个big-endian int 4*8=32位的数据类型

```
import numpy as np
dt = np.dtype('>i4')
print(dt)
print('字节顺序:', dt.byteorder)         #字节顺序
print('字节大小:', dt.itemsize)          #字节大小
print('字节名:', dt.name)                #字节名
```
输出

```
>i4
字节顺序: >
字节大小: 4
字节名: int32
```

#### 数据类型的自定义与使用

定义一个数据类型，其中name为长度最长为4的字符串，grades为2个float64的子数组

```
np.dtype([('name', 'U4', 1), ('grades', 'f8', 2)])
print(dt['name'])
print(dt['grades'])
```
输出
```
<U16
('<f8', (2,))
```
使用自定义数据类型

```
d = np.array([('Ysy', (8.1, 9.0)), ('Yu', 6.0)], dtype=dt)
print('d:', d)
print('gardes:', d['grades'][0])
```
输出

```
d: [('Ysy', [8.1, 9. ]) ('Yu', [6. , 6. ])]
gardes: [8.1 9. ]
```

缩写字符参数说明

| 缩略| 含义 |
| --- | --- | 
| 'b'    | boolean |
| 'i'    | (signed) integer |
| 'u'    | unsigned integer |
| 'f'    | floating-point |
| 'c'    | complex-floating point |
| 'm'    | timedelta |
| 'M'    | datetime |
| 'O'    | (Python) objects |
| 'S'    | (byte-)string |
| 'U'    | Unicode |
| 'V'    | raw data (void) |

#### 类型切换
类型转换有两种方式，一种是使用方法astype，一直是直接修改元数据数据类型，我们可以看一下区别

```
d = np.array([1., 2., 3., 4.])
print('原数据类型:', d.dtype)
print('数据长度:', d.shape)
m = d.astype('i8')
print('astype修改后数据类型:', m.dtype)
print('astype修改后数据长度:', m.shape)
d.dtype = 'i4'
print('dtype修改后数据类型:', d.dtype)
print('dtype修改后数据长度:', d.shape)
```
输出

```
原数据类型: float64
数据长度: (4,)
astype修改后数据类型: int64
astype修改后数据长度: (4,)
dtype修改后数据类型: int32
dtype修改后数据长度: (8,)
``` 
可以看出astype是copy了一个新对象，并成功修改了类型，而直接修改元数据类型会发生输出改变的错误，所以修改类型的时候要使用astype，不要直接修改元数据的dtype.       
