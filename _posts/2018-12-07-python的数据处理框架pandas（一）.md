---
layout:     post
title:      python的数据处理框架pandas（二）
subtitle:   pandas
date:       2018-12-07
author:     Yancy
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - python
    - 数据处理
    - pandas
---

##前言
>在前面提到的是数据处理三剑客中的numpy，这一偏来说一下三剑客另外一剑-pandas

#### 简介
pandas是基于numpy的一种工具，他也是为了解决数据分析来创建的，pandas的series拥有numpy的所有功能，而且pandas拥有dataframe拥有更强大的功能

#### pandas中的数据类型
pandas有两种数据类型:series与dataframe

series是一种一维的数据类型,其功能与numpy相同, 可以把它当作一个由带标签的元素组成的 numpy 数组。标签可以是数字或者字符

dataframe是一个二维的数据类型，就像是csv与excel存储的数据一样

#### pandas的数据定义
定义个series

```
s1 = pd.Series([1, 2, 3])
print(s1)
print('获取第一个值:', s1[0])

s2 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])     #自定义series的索引
print(s2)
print('索引为a的值:', s2['a'])

```
输出

```
0    1
1    2
2    3
dtype: int64
获取第一个值: 1
a    1
b    2
c    3
dtype: int64
索引为a的值: 1
```
定义一个dataframe

```
data = [[0, 150], [0, 150], [0, 150], [0, 300]]
index = ['语文', '数学', '英语', '理综']
columns = ['李四', '张三']
df = DataFrame(data=data, index=index, columns=columns)

print(df)                    #定义的dataframe数据
```
输出

```
    李四   张三
语文   0  150
数学   0  150
英语   0  150
理综   0  300

```
#### pandas的常用方法
##### 查看前几行的数据

```
df.head(3)
```
输出

```
    李四   张三
语文   0  150
数学   0  150
英语   0  150
```
head方法还可以传入负数,传入负数可以看做从倒数第几位往前取值.

##### 查看后几行的数据

```
print(df.tail(3))
```
输出

```
    李四   张三
数学   0  150
英语   0  150
理综   0  300
```

##### dataframe重命名

```
df.columns = [2, 1]
# or
df.rename(columns={
    '李四': 2,
    '张三': 1
}, inplace=True)
print(df)
```
输出

```
    2    1
语文  0  150
数学  0  150
英语  0  150
理综  0  300
```

这两种方法都可以将张三和李四改成2和1，我是使用第二种方法，其中第二种方法的inplace参数，它为True，则在原有df上更改，为false则copy一个新的df

##### 求dataframe的长度

```
print('dataframe有多少行:', len(df))
print('datafram行数和列数:', df.shape)
```
输出

```
dataframe有多少行: 4
datafram行数和列数: (4, 2)
```

##### 去除重复行

```
df.drop_duplicates('张三', keep='last', inplace=True) #去重,并保留最后一个
print(df)
```

输出

```
    李四   张三   类型
数学   0  150   主修
英语   0  100  非主修
理综   0  300  非主修
```

##### 获取基本统计数据

```
pd.options.display.float_format = '{:,.3f}'.format #只小数点后三位
```

输出

```
         李四      张三
count 4.000   4.000
mean  0.000 187.500
std   0.000  75.000
min   0.000 150.000
25%   0.000 150.000
50%   0.000 150.000
75%   0.000 187.500
max   0.000 300.000
```

##### dataframe的筛选
获取某一列
```
print(df.张三)    #获取张三那一列
print(df['张三'])
```
输出

```
语文    150
数学    150
英语    150
理综    300
Name: 张三, dtype: int64

```
当我们提取列的时候，提取的是series，我们可以吧dataframe看做是series的一个字典，所以在获取列的时候，我们就会得到一个 series

按条件获取某一列(布尔过滤（boolean masking))

```
print(df[df['张三'] > 150])   #和numpy的使用方法类似
```
输出

```
    李四   张三
理综   0  300
```
复合条件过滤

```
d = df[(df['张三'] > 100) & (df['张三'] < 300)] 
print(d)
```
输出

```
    李四   张三
语文   0  150
数学   0  150
英语   0  150
```

这里不能用 and 关键字，因为会引发操作顺序的问题，而且使用and会报错

按照字符串开头过滤

```
df['李四'] = '1000'
d = df[df['李四'].str.startswith('100')]
print(d)
```
输出

```
      李四   张三
语文  1000  150
数学  1000  150
英语  1000  150
理综  1000  300
```

#### 索引
刚才是获取某一列的数据,我们获取行数据的话是有专门的方法

```
print(df.iloc[30])       #iloc是获取第多少行的值
print(df.loc['理综'])    #loc是根据索引名来获取值
```
输出
```
李四      0
张三    300
Name: 理综, dtype: int64
```

#### 对数据集应用函数
有时需要根据数据的不同来对数据进行不同的处理，这时候我们就需要用到apply或者applymap函数了

```
def level(scroll):
    l = 'A' if scroll > 150 else 'B'
    return l

df['level'] = df['张三'].apply(level)
print(df)

#直接用dataframe调用apply方法
def level(data):
    if data['张三'] - data['李四'] >= 0:
        return 1
    return 0
df['level'] = df.apply(level, axis=1)
print(df)
```

输出

```
    李四   张三 level
语文   0  150     B
数学   0  150     B
英语   0  150     B
理综   0  300     A
```
applymap

```
def level(scroll):
    l = 'A' if scroll > 150 else 'B'
    return l

df = df.applymap(level)
print(df)
```
输出

```
   李四 张三
语文  B  B
数学  B  B
英语  B  B
理综  B  A
```

#### 排序操作
按照索引排序
```
data = [[0, 150], [0, 150], [0, 150], [0, 300]]
index = [1, 3, 2, 4]
columns = ['李四', '张三']
df = DataFrame(data=data, index=index, columns=columns)

df = df.sort_index()
print(df)
```
输出

```
   李四   张三
1   0  150
2   0  150
3   0  150
4   0  300
```

按照value进行排序

```
df = df.sort_values(by='张三', ascending=False)   #ascending为True是从小到大，默认为True
print(df)
```
输出

```
   李四   张三
4   0  300
1   0  150
3   0  150
2   0  150
```

#### 数据集操作
数据集操作就是重新建立数据结构，使数据集呈现一种更方便并且有用的形式

首先，是<font color=#b73559> grouypby </font>

```
d_max = df['张三'].groupby(df['类型']).max()             #按照类型聚合张三并取最大值
print(d_max)
d_min = df['张三'].groupby(df['类型']).min()             #按照类型聚合张三并取最小值
print(d_min)
d_cumsum = df['张三'].groupby(df['类型']).cumsum()             #按照类型聚合张三并按照挨着的同一类型从上到下累加（一般先根据某一类型排序，整合到一起，在累加）
print(d_cumsum)

#还可以直接用整体的dataframe来groupby，与上面使用方法相同
all_max = df.groupby(df['类型']).max()
print(all_max)
all_cumsum = df.groupby(df['类型']).cumsum()
print(all_cumsum)

```
groupby也可以按照多列进行分组

#### 合并数据集
在进行数据处理的时候，经常需要用到数据合并，然后统一进行来操作。

首先，是<font color=#b73559> merge </font>

```
pd.merge(left, right, how='inner', on=None, left_on=None, right_on=None,
          left_index=False, right_index=False, sort=False,
          suffixes=('_x', '_y'), copy=True, indicator=False,
          validate=None)
```
| 缩略| 含义 |
| --- | --- |
| left    | 基类dataframe |
| right    | 想要合并的frame |
| how    | 合并方式，支持 left、right、outer、inner(默认) |
| on    | 指定需要合并的列 |
| left_on    | left用作连接键的列，与right_on一起使用 |
| right_on    | right用作连接键的列，与left_on一起使用 |
| suffixes    | 用于追加到重叠列名的末尾，默认为(‘_x’,‘_y’)。如果左右两个DataFrame对象都有“data”，则结果就会出现“data_x”和“data_y” |

实现起来很简单

```
data = [['001', 80, 70], ['002', 90, 60], ['003', 100, 50]]
columns = ['id', '语文', '数学']
df1 = DataFrame(data=data,columns=columns)


data = [['001', 80, 70], ['002', 90, 60], ['004', 100, 50]]
columns = ['id', '英语', '理综']
df2 = DataFrame(data=data, columns=columns)

all_df = pd.merge(df1, df2, how='left', on='id')

print(all_df)
```

输出

```
    id   语文  数学    英语    理综
0  001   80  70  80.0  70.0
1  002   90  60  90.0  60.0
2  003  100  50   NaN   NaN
```
outer、right、inner与left同理

#### 数据空值处理

在合并的时候，我们发现会有一些值是NaN，这些值可能需要我们手动处理一下

首先，发现空值<font color=#b73559> isnull() </font>和<font color=#b73559> notnull() </font>

```
print(all_df.isnull())
```

输出

```
      id     语文     数学     英语     理综
0  False  False  False  False  False
1  False  False  False  False  False
2  False  False  False   True   True
```
我个人是没怎么用过这个方法的，感觉没什么用

删除有空值的行或列 <font color=#b73559> dropna(axis=0, how='any', inplace=False) </font>

```
print(all_df.dropna(axis=0))

```
输出

```
    id  语文  数学    英语    理综
0  001  80  70  80.0  70.0
1  002  90  60  90.0  60.0
```

很重要的一个方法，填充NaN值<font color=#b73559> fillna()</font>

```
print(all_df.fillna(value=0))
```

输出

```
    id   语文  数学    英语    理综
0  001   80  70  80.0  70.0
1  002   90  60  90.0  60.0
2  003  100  50   0.0   0.0
```
