---
title: 【下载】C# 调用迅雷、IDM下载方法汇总
tags:
  - 'C#'
  - 下载
  - IDM
  - 迅雷
  - 爬虫
abbrlink: 46213
date: 2018-03-07 09:36:14
categories:
  - 踩坑笔记
---
在开发桌面软件时常常有下载文件的需求，小文件、少文件可以自己做相应的网络请求，但是当文件的大小或者数量达到一定规模时，自己封装网络请求就不是很划算的事情了，这时我们可以采取调用迅雷或者IDM、aria2c之类的专业下载软件来进行下载。
<!--more-->
## 迅雷
### 直接调用迅雷
1.  安装迅雷后可以再引用的com组件中找到名为 “ThunderAgent 1.0 Type Library” 的com组件，勾选引用之后，把类库属性中的嵌入互操作类型修改为false。
![](http://qiniucdn.wayneshao.com/20180307172636630/20180307054659587.png)
2. 使用代码调用AgentLib直接在迅雷中增加新任务
```csharp
new ThunderAgentLib.AgentClass().AddTask("http://s1.static.haoke258.info/attach_2/2018-1-30-5ZD6R0BZZ.rar");
```
    以上为最简单的调用方式，只是传入了下载地址，其他的属性为默认值，传入后续的几个参数或者才用AddTask2方法就可以手动修改任务属性。
```csharp
public virtual extern void AddTask([MarshalAs(UnmanagedType.BStr), In] string bstrUrl, [MarshalAs(UnmanagedType.BStr), In] string bstrFileName = "", [MarshalAs(UnmanagedType.BStr), In] string bstrPath = "", [MarshalAs(UnmanagedType.BStr), In] string bstrComments = "", [MarshalAs(UnmanagedType.BStr), In] string bstrReferUrl = "", [In] int nStartMode = -1, [In] int nOnlyFromOrigin = 0, [In] int nOriginThreadCount = -1);

public virtual extern void AddTask2([MarshalAs(UnmanagedType.BStr), In] string bstrUrl, [MarshalAs(UnmanagedType.BStr), In] string bstrFileName = "", [MarshalAs(UnmanagedType.BStr), In] string bstrPath = "", [MarshalAs(UnmanagedType.BStr), In] string bstrComments = "", [MarshalAs(UnmanagedType.BStr), In] string bstrReferUrl = "", [In] int nStartMode = -1, [In] int nOnlyFromOrigin = 0, [In] int nOriginThreadCount = -1, [MarshalAs(UnmanagedType.BStr), In] string bstrCookie = "");
```
### 迅雷开放平台
迅雷官方曾经开放过一个mini版下载SDK，相当于一个无界面版本的mini迅雷，可以供程序直接调用，现在虽然已经停止服务，但是网上仍然有它的传说，
优点是不用下载迅雷，软件+dll总共只有1mb+，缺点就是没有迅雷完整版的那些加速功能，没有那么快。
我按照自己的理解把SDK封装了一下：https://github.com/WayneShao/ThunderSdk
下面是调用代码：
```csharp
using System;
using ThunderSdk;

namespace ThunderSdkDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            var manager = new DownloadManager(1, @"D:\");
            manager.CreateNewTask("https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=770549720,375505130&fm=173&s=612A66F94AA394CE4A84E71B030050D7&w=218&h=146&img.JPEG",
                "2018-1-30-5ZD6R0BZZ.jpg");

            manager.TaskDownload += (s, e) =>
            {
                if (!(s is DownFileInfo info)) return;
                Console.WriteLine(info.TaskInfo.Percent);
            };

            manager.TaskCompleted += (s, e) =>
              {
                  if (!(s is DownFileInfo info)) return;
                  Console.WriteLine(info.FileName + "下载完成");
              };

            manager.StartAllTask();
            Console.ReadKey();
        }
    }
}
```
## Internet Download Manager
这个是我用过之后体验最好的，推荐使用。
使用方法如下：
### 获取DLL文件
1. 下载 [IDMCOMAPI.zip](http://www.internetdownloadmanager.com/support/download/IDMCOMAPI.zip)。
2. 解压 IDManTypeInfo.tlb 文件到任意位置。
3. 打开 Visual Studio安装时附带的命令行工具。（任选一个即可，推荐第一个）
![](http://qiniucdn.wayneshao.com/20180308000415303/20180308121452182.png)
4. 使用 类型库导入程序 (Tlbimp.exe)将tlb文件转换成dll文件。
```shell
TlbImp IDManTypeInfo.tlb
```
    > Microsoft (R) .NET Framework Type Library to Assembly Converter 3.5.30729.1
    Copyright (C) Microsoft Corporation.  All rights reserved.

    >Type library imported to ![IDManLib.dll]
This will create an IDManLib.dll

如果懒得完成上面的步骤，也可以直接下载我已经导入好的文件[IDManLib.7z](http://qiniucdn.wayneshao.com/20180308000415303/IDManLib.7z)
## 引用DLL文件
手动选择 IDManLib.dll 文件进行引用，引用后同样也要把类库属性中的嵌入互操作类型修改为false，编译平台最好选择x86。 
## 调用方法
和迅雷的类似，将下载地址、文件路劲、文件名等参数传入方法中即可成功在IDM中建立新任务。
```csharp
new IDManLib.CIDMLinkTransmitterClass().SendLinkToIDM("http://s1.static.haoke258.info/attach_2/2018-1-30-5ZD6R0BZZ.rar", "http://ailushe95.info/", "", "", "", "", @"D:\Backup\Downloads\LULULU", "【感謝擼友投稿】無名小姐姐 2V.rar", 0);
```
