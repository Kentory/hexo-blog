---
title: 正则表达式（一）--元字符
date: 2019-06-04 18:43:14
tags: 正则
---

## 元字符
元字符是一些在正则表达式中有特殊用途、不代表它本身字符意义的一组字符。利用元字符，我们可以控制字符串匹配的方式。如果要在正则表达式中使用元字符本身的意义，例如：如果想搜索字符串中的'?'，那么需要对元字符进行转义，使用的方法是把一个反斜杠（\）放在元字符前面，这样元字符就失去了特殊的意义，还会代表它本身的字符意义。

### 常用的元字符

| 元字符 | 含义 |
| :---: | :-----: |
| \d | 匹配一个数字字符，等价于[0-9] |
| \w | 匹配字母、数字、下划线，等价于[A-Za-z0-9_] |
| ^	| 匹配输入字符串的开始位置 |
| $	| 匹配输入字符串的结束位置 |
| *	| 匹配前面的子表达式零次或多次，* 等价于{0,} |
| + | 匹配前面的子表达式一次或多次，+ 等价于{1,} |
| ?	| 匹配前面的子表达式零次或一次，? 等价于{0,1} |
| {n} | 匹配n次 |
| {n,} | 匹配n次或多次 |
| {n,m} | 匹配n到m次 |

更多元字符文档可以查看[RUNOOB](https://www.runoob.com/regexp/regexp-metachar.html)和[CSDN](https://blog.csdn.net/feiying008/article/details/52886304)。

### 推荐几个正则练习网站

一个好玩的正则闯关网站：[Regex Golf](https://alf.nu/RegexGolf)
![Regex Golf](RegexGolf.png)
一个类似数独的正则拼图网站：[Regex Cross­word](https://regexcrossword.com/)
![Regex Cross­word](RegexCross­word.png)
一个在线学习正则表达式网站：[regexr](https://regexr.com/)
![regexr](regexr.png)

