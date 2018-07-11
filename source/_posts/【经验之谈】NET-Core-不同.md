---
title: .NET Core 不同程序集中存在相同命名空间时的解决思路
abbrlink: 19579
date: 2018-07-10 19:36:46
tags:
  - .NET Core
  - 别名
  - 命名空间冲突
categories:
  - 经验之谈
---
　　前几天同事遇到了这个问题，没查到资料找到了我这位老司机，隐约记得.NET Framework应该用别名解决的我给出了两个字的解决方案 “别名”，被告知别名在.NET Core 中是不能像.NET Framework中那样设置的，连忙打开VS亲自尝试了一下，以下是他遇到问题的两个包：
```
PM> Install-Package StackExchange.Redis
PM> Install-Package StackExchange.Redis.StrongName
```
<!--more-->
　　随便按照文档的调用简单写了一句代码
```csharp
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect("localhost");
```
　　！！
　　果然出了问题！
![](http://qiniucdn.wayneshao.com/.NET-Core-不同/20180710080350865.png)
　　错误提示的大意为，StackExchange.Redis.dll 和 StackExchange.Redis.StrongName.dll中都存在 StackExchange.Redis 命名空间，且命名空间下都有一个 ConnectionMultiplexer 类型，先不追究同事为何需要同时使用这样两个程序集，假设这是必要的，我们来尝试解决这个问题。
　　先试试使用.NET Framework的老方法
　　![](http://qiniucdn.wayneshao.com/.NET-Core-不同/20180710081024412.png)
　　果然行不通，菜单里根本没有别名可以设置。
　　祭谷歌：
![](http://qiniucdn.wayneshao.com/.NET-Core-不同/20180710080824886.png)
　　然而翻了几页之后，大量的解决方法都是.Net Framework 的。
　　根据对微软的了解，我判断.NET Core里属性菜单里虽然没有别名可以设置，但是解决方法一定还是落在“别名”上，很有可能是需要手动在项目文件中进行指定，中文描述不太准确，这方面的文章也比较滞后，再次尝试使用英文关键词搜索：
　　![](http://qiniucdn.wayneshao.com/.NET-Core-不同/20180710081620557.png)
　　只翻了两条，就在Nuget的一个[Issue](https://github.com/NuGet/Home/issues/4989)中找到了解决方法，果然是使用别名：
  ![](http://qiniucdn.wayneshao.com/.NET-Core-不同/20180710082052253.png)
　　```xml
<Target Name="ChangeAliasesOfStrongNameAssemblies" BeforeTargets="FindReferenceAssembliesForReferences;ResolveReferences">
    <ItemGroup>
      <ReferencePath Condition="'%(FileName)' == 'StackExchange.Redis.StrongName'">
        <Aliases>signed</Aliases>
      </ReferencePath>
    </ItemGroup>
  </Target>
　　```
　　成功搞定。