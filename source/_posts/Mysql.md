---
title: Mysql日期函数
date: 2019-05-31 21:30:10
tags: SQL
---

### Mysql日期和时间类型
表示时间值的日期和时间类型为YEAR、DATE、TIME、DATETIME和TIMESTAMP。

每个时间类型有一个有效值范围和一个"零"值，当指定MySQL不能表示的值时使用"零"值。

| 类型 | 占用空间 | 范围 | 格式 | "零"值 |
| :---: | :-----: | :-----: | :----: | :----: |
| YEAR | 1字节 | 1901/2155 | YYYY | 0000 |
| DATE | 3字节 | 1000-01-01/9999-12-31 | YYYY-MM-DD | '0000-00-00' |
| TIME | 3字节 | -838:59:59 ~ 838:59:59 | HH:MM:SS | '00:00:00' |
| DATETIME | 8字节 | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | '0000-00-00 00:00:00' |
| TIMESTAMP | 4字节 | 1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07  UTC | YYYY-MM-DD HH:MM:SS | '0000-00-00 00:00:00' |


mysql5.7之后版本的sql_mode默认使用：
``` bash
sql-mode="STRICT_ALL_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_AUTO_CREATE_USER"
```
其中NO_ZERO_IN_DATE, NO_ZERO_DATE两个选项禁止了日期和时间的"零"值。

也可以手动调整MySQL的sql_mode变量：
``` bash
set @@global.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_DATE,NO_ENGINE_SUBSTITUTION';
set @@session.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_DATE,NO_ENGINE_SUBSTITUTION';
```

### Mysql日期时间函数
#### 获取日期时间
![get](get_date_time.png)

#### 比较日期时间
1. 日期差函数：datediff
DATEDIFF(date1,date2)，计算两个日期相差天数（date1-date2），参数类型可以是DATE、DATETIME和TIMESTAMP。
![datediff](datediff.png)
2. 时间差函数：timediff
TIMEDIFF(time1,time2)，两个时间相减得到的差值（time1-time2），参数类型可以是DATE、TIME、DATETIME和TIMESTAMP,但time1和time2必须参数类型相同；两个date计算得到00:00:00。
![timediff](timediff.png)
3. 时间差函数：timestampdiff
TIMESTAMPDIFF(interval, datetime1,datetime2)，返回（datetime2-datetime1）的时间差，结果单位由interval参数给出，参数类型可以是DATE、DATETIME和TIMESTAMP。interval可选择：
 * frac_second 毫秒（MySQL5.6后支持）
 * second 秒
 * minute 分钟
 * hour 小时
 * day 天
 * week 周
 * month 月
 * quarter 季度
 * year 年
![timestampdiff](timestampdiff.png)
