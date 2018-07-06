---
title: 模块化InnoSetup依赖项安装
tags:
  - InnoSetup
  - 安装包
abbrlink: 24425
date: 2016-11-23 11:07:33
---
原文在这里:http://www.codeproject.com/Articles/20868/NET-Framework-Installer-for-InnoSetup

源文件地址:http://www.codeproject.com/KB/install/dotnetfx_innosetup_instal/innodependencyinstaller.zip

源文件需要注册登录CodeProject才能下载
<!-- more -->
![](http://p4au3q1y8.bkt.clouddn.com/20180218232333/20180218112438669.png)
![](http://p4au3q1y8.bkt.clouddn.com/20180218232333/20180218112442373.png)

**说明:
通过添加模块化innosetup脚本来自动下载和安装各种依赖项 如.NET Framework 、VC++运行环境等。**

源代码是模块化的，结构如下：
![](http://p4au3q1y8.bkt.clouddn.com/20180218232333/20180218112524045.png)
* setup.iss - 包含基本设置，其中包含所需的模块（产品）。 
   你需要把所需的模块在顶部使用#include命令包含在源代码中,例如：
   #include "scripts\products\dotnetfx11.iss"
   然后你只需要在[Code]段调用它们的main函数,如：
   dotnetfx11();

* bin - 包含安装程序的最终输出
* src - 包含您的程序的应用程序文件
* scripts
    * products.iss - 包含产品脚本的共享代码。 您只需要更改[CustomMessages]部分和[Files]部分（包括isxdl语言文件）
    * isxdl - 包含用于设置（如果有要下载的内容）及其语言文件（例如german.ini）的下载器DLL。 这是您可以放置​​isxdldownloader的语言文件的地方。
    * products - 包含应用程序所需的产品的脚本（例如.NET Framework 2.0）
        * dotnetfx11.iss - .NET Framework 1.1
        * dotnetfx11lp.iss - .NET Framework 1.1 Language Pack
        * dotnetfx11sp1.iss - .NET Framework 1.1 + Service Pack 1
        * dotnetfx20.iss - .NET Framework 2.0
        * dotnetfx20lp.iss - .NET Framework 2.0 Language Pack
        * dotnetfx20sp1.iss - .NET Framework 2.0 + Service Pack 1
        * dotnetfx20sp1lp.iss - .NET Framework 2.0 Service Pack 1 Language Pack
        * dotnetfx20sp2.iss - .NET Framework 2.0 + Service Pack 2
        * dotnetfx20sp2lp.iss - .NET Framework 2.0 Service Pack 2 Language Pack
        * dotnetfx35.iss - .NET Framework 3.5
        * dotnetfx35lp.iss - .NET Framework 3.5 Language Pack
        * dotnetfx35sp1.iss - .NET Framework 3.5 + Service Pack 1
        * dotnetfx35sp1lp.iss - .NET Framework 3.5 Service Pack 1 Language Pack
        * dotnetfx40client.iss - .NET Framework 4.0 Client Profile
        * dotnetfx40full.iss - .NET Framework 4.0 Full
        * dotnetfx46.iss - .NET Framework 4.6
        * ie6.iss - Internet Explorer 6
        * iis.iss - Internet Information Services (just a check if it is installed)
        * jet4sp8.iss - Jet 4 + Service Pack 8
        * kb835732.iss - Security Update (KB835732) which is required by .NET Framework 2.0 Service Pack 1 on Windows 2000 Service Pack 4
        * mdac28.iss - Microsoft Data Access Components (MDAC) 2.8
        * msi20.iss - Windows Installer 2.0
        * msi31.iss - Windows Installer 3.1
        * msi45.iss - Windows Installer 4.5
        * sql2005express.iss - SQL Server 2005 Express + Service Pack 3
        * sql2008express.iss - SQL Server 2008 Express R2
        * sqlcompact35sp2.iss - SQL Server Compact 3.5 + Service Pack 2
        * vcredist2005.iss - Visual C++ 2005 Redistributable
        * vcredist2008.iss - Visual C++ 2008 Redistributable
        * vcredist2010.iss - Visual C++ 2010 Redistributable
        * vcredist2012.iss - Visual C++ 2012 Redistributable
        * vcredist2013.iss - Visual C++ 2013 Redistributable
        * vcredist2015.iss - Visual C++ 2015 Redistributable
        * wic.iss - Windows Imaging Component
        * winversion.iss - helper functions to determine the installed Windows version
        * fileversion.iss - helper functions to determine the version of a file
        * stringversion.iss - helper functions to correctly parse a version string
        * dotnetfxversion.iss - helper functions to determine the installed .NET Framework version including service packs
        * msiproduct.iss - helper functions to check for installed msi products

你很可能需要调整setup.iss，以适应不同Windows版本所需的依赖项。

如果依赖项没有安装，安装过程会检查相关依赖项的安装文件是否存在于.\MyProgramDependencies.文件夹下。如果不存在那么程序将会自动下载。
![](http://p4au3q1y8.bkt.clouddn.com/20180218232333/20180218112911329.png)
![](http://p4au3q1y8.bkt.clouddn.com/20180218232333/20180218112917754.png)
用于脚本的应用程序包括：

* [Inno Setup](http://www.jrsoftware.org/isinfo.php) - (版本5.5.5)
* [ISTool](http://www.istool.org/default.aspx) -  Inno Setup的扩展组件。但是我只需要 isxdl.dll downloader (版本5.3.0)