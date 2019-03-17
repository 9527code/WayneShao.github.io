---
title: '缺少google api密钥,因此chromium的部分功能将无法使用”的解决办法'
tags:
  - 软件心得
abbrlink: 37840
date: 2016-10-01 14:27:20
categories:
  - 踩坑笔记
---
 使用Chromium时会遇到 “缺少google api密钥,因此chromium的部分功能将无法使用”提示，google了一下 setx Google_API_KEY 和 chromium portable google api keys are missing 找到了解决办法。
 <!-- more -->
 打开windows的cmd命令提示符，依次输入以下命令： 
```cmd
setx GOOGLE_API_KEY "no" 
setx GOOGLE_DEFAULT_CLIENT_ID "no" 
setx GOOGLE_DEFAULT_CLIENT_SECRET "no"
```

![](http://qiniucdn.wayneshao.com/20180218230720/20180218110847654.png)
当然,这样只是可以去掉chromium开启时的提示,如果需要使用Google API 服务,还是推荐去谷歌申请,然后重新setx为真实api key。