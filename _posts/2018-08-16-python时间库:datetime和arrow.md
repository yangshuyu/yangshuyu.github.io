---
layout:     post
title:      python常用时间库
subtitle:   datetime和arrow
date:       2018-08-16
author:     Yancy
header-img: img/post-bg-desk.jpg
catalog: true
tags:
   - python
---

##前言
>python常用的时间库有很多，功能也很多，例如时间与时间戳之间的转换、时间的前移、增加等，这篇记录常用功能，以防遗忘



###一、datetime
#### datetime.date(日期)
```
In [1]: import datetime

In [2]: datetime.date.today() 返回一个表示当前本地日期的date对象；
Out[3]: datetime.date(2018, 8, 17)

方法
In [4]: datetime.date.today().weekday() 返回weekday，如果是星期一，返回0；是星期2，返回1，以此类推；
Out[5]: 4

In [6]: datetime.date.today().strftime("%Y-%m-%d")
Out[7]: '2018-08-17'

```

#### datetime.time(时间)
```
In [1]: import datetime

In [2]: datetime.time(10, 22, 16)
Out[3]: datetime.time(10, 22, 16)

方法
In [4]: datetime.time(10, 22, 16).hour  获取时分秒
Out[5]: 10

In [6]: datetime.time(10, 22, 16).strftime("%X") 格式化
Out[7]: '10:22:16'

```


#### datetime.datetime(日期+时间)
```
In [1]: import datetime

In [2]: datetime.datetime.now()   获取本地当前时间
Out[3]: datetime.datetime(2018, 8, 17, 0, 53, 32, 325028)

In [4]: datetime.datetime.utcnow()   获取utc时间
Out[5]: datetime.datetime(2018, 8, 16, 16, 53, 39, 185611)

In [6]: datetime.datetime(2018, 2, 28, 0, 0, 0)  获取指定年月日日期
Out[7]: datetime.datetime(2018, 2, 28, 0, 0)

In [8]: datetime.datetime.now().month - datetime.datetime(2018, 2, 28, 0, 0, 0).month
Out[9]: 6

In [10]: datetime.datetime(2018, 2, 28, 0, 0, 0)  获取指定年月日照片
Out[11]: datetime.datetime(2018, 2, 28, 0, 0)

In [12]: datetime.datetime(2018, 2, 28, 0, 0, 0).timestamp()  获取时间戳
Out[13]: 1519747200.0

In [14]: datetime.datetime.now().strftime('%Y-%m-%d')  格式化时间
Out[15]: '2018-08-17'

```


#### datetime.timedelta(时间间隔，可以用来做时间加减)
```
In [1]: import datetime

In [2]: datetime.datetime.now() - datetime.timedelta(days=3)  当前时间减去三天
Out[3]: datetime.datetime(2018, 8, 14, 1, 13, 26, 431932)

In [4]: datetime.datetime.now() - datetime.timedelta(seconds=3)  当前时间减去三秒
Out[5]: datetime.datetime(2018, 8, 16, 16, 53, 39, 185611)

```


###一、arrow
#### arrow获取时间

```
In [13]: import arrow

In [14]: arrow.utcnow()
Out[15]: <Arrow [2017-02-01T08:30:37.627622+00:00]>

In [16]: arrow.now()
Out[17]: <Arrow [2017-02-01T16:32:02.857411+08:00]>

In [18]: arrow.now('US/Pacific') 获取指定时区的时间
Out[19]: <Arrow [2018-08-16T22:40:31.118807-07:00]>

```

#### arrow转换成时间戳

```
In [13]: import arrow

In [15]: arrow.utcnow().timestamp
Out[15]: 1534477823

In [19]: arrow.now()
Out[19]: <Arrow [2017-02-01T16:32:02.857411+08:00]>

```

#### arrow时间格式化

```
In [13]: import arrow

In [15]: arrow.now().format()
Out[15]: '2018-08-17 12:23:34+08:00'

In [19]: arrow.now().format("YYYY-MM-DD HH:mm")
Out[19]: '2018-08-17 12:24'

```

#### 字符串生成arrow对象

```
In [20]: arrow.get("2017-01-20 11:30", "YYYY-MM-DD HH:mm")
Out[21]: <Arrow [2017-01-20T11:30:00+00:00]>

```

#### 时间戳生成arrow对象
这里string、integer、float都可已生成
```
In [20]: arrow.get("1534477823")
Out[21]: <Arrow [2018-08-17T03:50:23+00:00]>

In [27]: arrow.get(1485937858.659424)
Out[28]: <Arrow [2017-02-01T08:30:58.659424+00:00]>

```

#### 直接生成Arrow对象

```
In [20]: arrow.Arrow(2017, 2, 1)
Out[20]: <Arrow [2017-02-01T00:00:00+00:00]>

In [20]: arrow.get(2018, 8, 15, 1, 0, 0)
Out[20]: <Arrow [2018-08-15T01:00:00+00:00]>

```

#### arrow时间加减
arrow有两个时间加减方法replace与shift
```
In [30]: t = arrow.now()
In [31]: t
Out[31]: <Arrow [2017-02-01T17:19:19.933507+08:00]>

In [33]: t.shift(days=-1)  # 前一天
Out[33]: <Arrow [2018-08-16T13:35:07.384647+08:00]>

In [34]: t.shift(weeks=-1)  # 前一周
Out[34]: <Arrow [2018-08-10T13:35:44.943040+08:00]>

In [35]: t.shift(months=-1) # 前一个月
Out[35]: <Arrow [2018-07-17T13:36:13.989258+08:00]>

In [37]: t.replace(days=1)  # 明年
Out[37]: <Arrow [2018-08-18T13:36:46.715196+08:00]>

```
