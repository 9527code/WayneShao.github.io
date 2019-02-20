---
title: 查看当前IP和归属地的方法
tags:
  - 软件心得
  - IP
  - 爬虫
abbrlink: 2316
date: 2018-02-19 21:45:56
---
可以通过http协议进行get请求来获得当前IP及归属地信息。
<!--more-->
### [ip138](http://2017.ip138.com/ic.asp)
Xpath获取文字信息
```
//center
```
![](http://qiniucdn.wayneshao.com/20180219214549906/20180219095338344.png)
### [whatismyip](http://www.whatismyip.com.tw/)
Xpath获取信息:
1. IP地址
```
//body/span[1]
```
2. 来源地区
```
//body/span[2]
```
![](http://qiniucdn.wayneshao.com/20180219214549906/20180219095925145.png)

### 优劣
二者网页大小差不多，均为300+KB。ip138 为国内服务商，对国内IP地址可以精确到市级，并且会包含运营商信息，不过信息包含在一个标签中，需要获取之后自行截取。whatismyip 为台湾网站，只能精确到国家，但是比较方便的是IP地址和来源是包含在两个标签中的，获取更为方便。