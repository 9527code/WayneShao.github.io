title: EFCore MySQL System.TypeLoadException occurred 问题解决
author: 玮仔Wayne
abbrlink: 11533
tags:
  - EFCore
  - MySQL
categories:
  - 经验之谈
date: 2018-04-09 19:02:00
---
　　今天在使用 EFCore + MySQL 搭一个小Demo的时候,在 Migration 环节遇到了这样一个问题。
  > System.TypeLoadException occurred
HResult=0x80131522
Message=Method 'Clone' in type 'MySQL.Data.EntityFrameworkCore.Infraestructure.Internal.MySQLOptionsExtension' from assembly 'MySql.Data.EntityFrameworkCore, Version=8.0.8.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d' does not have an implementation.

  几经排查无果之后求助了Google，发现了一条状况相同的 [Issue](https://github.com/jasonsturges/mysql-dotnet-core/issues/1)
  项目作者的回复如下
> 　　@lixiandai Hi, yes, this is frustrating - Oracle's MySQL does not yet fully support .NET Core 2.

> 　　In the interim, I suggest using Pomelo, which can be installed by executing the following command:

> 　　$ dotnet add package Pomelo.EntityFrameworkCore.MySql --version 2.0.0-rtm-10062
Or, add the following line to your .csproj ItemGroup

> 　　<PackageReference Include="Pomelo.EntityFrameworkCore.MySql" Version="2.0.0-rtm-10062" />

　　所以其实是MySQL的官方驱动包的锅喽?
  
　　按照作者的推荐选择了一位国内大佬的项目[Pomelo.EntityFrameworkCore.MySql]( https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql)，成功解决问题。