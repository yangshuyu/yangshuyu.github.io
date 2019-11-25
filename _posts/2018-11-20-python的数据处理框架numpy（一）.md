---
layout:     post
title:      python的数据处理框架numpy（一）
subtitle:   numpy的简单使用
date:       2018-11-20
author:     Ysy
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - python
    - 数据处理
    - numpy
---
##前言
>最近在做python的数据处理,其中pandas、numpy、matplotlib这三个库被称为数据处理三剑客，我先看的那就是numpy。

#### 简介
numpy是python语言的一个扩充程序,他用c实现，所以操作起来速度更快，他支持了高级大量的数据运算，也提供了大量的数据函数库（eg：std、sum、min、max等），这些方便我们数据处理(eg:使用numpy的内置函数画正态分布图)

#### numpy基本使用
创建numpy

```
numpy.array(object, dtype = None, copy = True, order = None, subok = False, ndmin = 0)
```

参数说明

| 参数 | 描述 | 例 |
| --- | --- | --- | 
| object    | 要生成的数据的内容    |  [1, 2, 3]   |
| dtype     | 返回数组所需的数据类型 |  str、int    |          
| copy     | 默认为true，对象是否被复制 |           |                    
| order       | 表示数组在内存的存放次序是以行(C)为主还是以列(F)为主 |     ‘A'、'C':行优先 、'F':列优先、'K'      |          
| subok        | 默认情况下，返回的数组被强制为基类数组。 如果为true，则返回子类 |           |          
| ndmin         | 返回数组的最小维数 |     3      |          
          

```
import numpy as np

d1 = np.array([1, 2, 3])
print('d1:', d1)
d2 = np.array([1, 2, '3'])          #这里会将所有的数据全部转化为字符串
print('d2:', d2)
d3 = np.array([1, 2, 3], ndmin=2)   #指定维度为2
print('d3:', d3)
d4 = np.array([1, 2, 3], dtype=str) #指定数据类型为字符串
print('d4:', d4)
```
输出

```
d1: [1 2 3]
d2: ['1' '2' '3']
d3: [[1 2 3]]
d4: ['1' '2' '3']
```

#### 基础运算符操作

```
mean = d1.mean()        #求平均值
print('平均值:', mean)

s = d1.sum()            #求和
print('和:', s)

m = d1.max()            #求最大值
print('最大值:', m)

m = d1.min()            #求最小值
print('最小值:', m)

p = d1.argmax()         #求最大值所在位置
print('最大值位置:', p)

d = d1.std()            #求标准差  主要是求离散程度
print('标准差', d)
```

输出

```
平均值: 2.0
和: 6
最大值: 3
最小值: 1
标准差 0.816496580927726
最大值位置: 2
```

#### 向量操作

```
d1 = np.array([1, 2, 3])
d2 = np.array([4, 5, 6])

vector_sum = d1 + d2             #向量和
print('向量和:', vector_sum)

vector_sub = d1 - d2             #向量差
print('向量差:', vector_sub)

vector_mul = d1 * d2             #向量积
print('向量积:', vector_mul)

#向量还可以与常数操作
vector_constant_sum = d1 + 2     #与常数求和
print('与常数求和:', vector_constant_sum)

vector_constant_mul = d1 * 2     #与常数乘积
print('与常数乘积:', vector_constant_mul)
					.
					.
					.
```

输出

```
向量和: [5 7 9]
向量差: [-3 -3 -3]
向量积: [ 4 10 18]
与常数求和: [3 4 5]
与常数乘积: [2 4 6]
```

#### 原地非原地

```
d1 = np.array([1, 2, 3, 4])
d2 = d1
d1 += np.array([1, 1, 1, 1])
print('原地', d2)

d1 = np.array([1, 2, 3, 4])
d2 = d1
d1 = d1 + np.array([1, 1, 1, 1])
print('非原地', d2)
```
输出

```
原地: [2 3 4 5]
非原地: [1 2 3 4]
```
从上面结果可以看出来,+=改变了原来数组,而+没有。这是因为:

+=:它是原地计算,不会创建一个新的数组,在原始数组中更改元素 
+:它是非原地计算,会创建一个新的数组,不会修改原始数组中的元素 
这和python本身的数组操作是一样的

#### 切片操作

```
l1 = [1, 2, 3, 5]
l2 = l1[0:2]
l2[0] = 5
print('list:', l1)

d1 = np.array([1, 2, 3, 5])
d2 = d1[0:2]
d2[0] = 5
print('numpy:', d1)
```
输出

```
list: [1, 2, 3, 5]
numpy: [5 2 3 5]

```
从输出可以看出,list中改变切片中的元素,不会影响原来的数组.而numpy改变切片中的元素,原来的数组也会跟着改变了。这是因为numpy的切片编程不会创建一个新数组出来,当修改对应的切片也会更改原始的数据。这样的机制,可以让numpy的效率更高

#### 二维数组

```
d1 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

print('第一个数组:', d1[0])             #获取其中一维数组

print('第一个数组的第一个元素:', d1[0][0])
print('第一个数组的第一个元素:', d1[0, 0])

print('所有元素的和:', d1.sum())         #求和是求所有元素的和
print('所有元素的和:', d1.std())         #求标准差是求所有元素的标准差

#那怎么获取某一列或者某一行的求和那,这就用到axis这个参数了，等于0时是按列，等于1是按行
print('按列求和:', d1.sum(axis=0))         #等于0是按列求和
print('按行求和:', d1.sum(axis=1))         #等于1是按照行求和
				.
				.
				.
```
输出

```
第一个数组: [1 2 3]
第一个数组的第一个元素: 1
第一个数组的第一个元素: 1
所有元素的和: 45
所有元素的和: 2.581988897471611
按列求和: [12 15 18]
按行求和: [ 6 15 24]
```
这些就是我认为numpy的简单使用。
