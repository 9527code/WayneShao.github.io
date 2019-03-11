---
title: 'bash脚本错误 Syntax error: else unexpected (expecting then)解决'
abbrlink: 15091
date: 2019-03-06 01:16:56
author: 玮仔Wayne
tags:
  - 踩坑
  - 编码
  - dos2unix
categories:
  - 踩坑笔记
---
今天在使用 bash 脚本发布程序时，发生了这样的错误
```bash
xxx.sh: Syntax error: "else" unexpected (expecting "then")
```
到处搜索资料尝试无果之后想到会不会是编码问题导致，
dos2unix 处理后测试果然可以
<!--more-->

```bash
sudo apt-get -y install dos2unix
```
处理
```bash
dos2unix xxx.sh
dos2unix: converting file xxx.sh to Unix format ...
```