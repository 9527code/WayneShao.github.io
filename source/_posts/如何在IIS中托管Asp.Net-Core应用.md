---
title: 如何在IIS中托管.NET Core应用
tags:
  - .NET Core
  - IIS
abbrlink: 19877
date: 2018-03-05 01:36:13
categories:
  - 踩坑笔记
---
Asp.NET Core 应用如果需要托管在IIS下，需要为IIS[下载](https://www.microsoft.com/net/download/all)安装 AspNetCoreModule 模块。


<!--more-->
下面以最新的.NET Core Runtime 2.1.0-preview1版本为例：
## 安装 Server Hosting Installer
首先访问微软的[.Net下载中心](https://www.microsoft.com/net/download/all)
，并找到我们要下载的版本。
![](http://qiniucdn.wayneshao.com/20180305013606660/20180305014343191.png)
点击进入详情页后，找到 Windows 分类下的 **Server Hosting Installer** 链接，并点击下载
![](http://qiniucdn.wayneshao.com/20180305013606660/20180305014516178.png)
下载安装完成以后即可在IIS的模块中找到托管.NET Core 应用所需的 AspNetCoreModule 模块。
![](http://qiniucdn.wayneshao.com/20180305013606660/20180305014647982.png)

## 发布程序
```shell
dotnet publish -o D:\Web\aspnetcoredemo
```