---
title: 'C#激爽特性——扩展方法'
tags:
  - 特性
  - 扩展方法
abbrlink: 35525
date: 2015-05-27 21:16:49
---
在最近的学习中，发现了一种用起来特别爽的C#特性——扩展方法，之前拜读《大话设计模式》一书的时候，书中提到这样一句话：“反射，反射，程序员的快乐”，本人菜鸟一只，到现在还未曾使用过反射，对于其是否真的快乐自然无从体会，不过扩展方法用起来称得上是相当快乐！
<!-- more -->
## 简介
下面是MSDN中对于扩展方法的解释
>  扩展方法使你能够向现有类型”添加“方法,而无需创建新的派生类型、重新编译或以其他方法修改原始类型。扩展方法是一种特殊的静态方法，但可以像扩展类型上的实力方法一样进行调用。对于C#和Visual Basic 编写的客户端代码，调用扩展方法与调用在类型中实际定义的方法之间没有明显的差异。

刚开始学习C#的时候,经常碰到各种需要状态转换的场合,那个时候代码里常常是大坨大坨的Convert.ToXX();当时非常羡慕String类可以直接调用ToString方法,省时省力,代码看起来还特别美观,之后学会了把Convert.ToXX()打包到一个方法里,代码不那么冗余了,但还是不甚美观,且费时费力,直到今天,我发现了C#中这个让人激爽无比的特性,这件事情终于有了完美的解决方案!
## 语法
下面来看一下基本的语法:
```csharp
public static 返回类型 方法名(this 需要添加扩展方法的类名 变量名,,,)
{
    return;
}
```
## 实现
首先我们写这样一个类: 
```csharp
namespace ExtensionMethods
{
    public static class ExtensionMethods
    {
        public static int ToInt(this string s)
        {
            return Convert.ToInt32(s);
        }
    }
}
```
然后只需要在另一个类中引用这个类的命名空间，就可以方便地使用我们写好的扩展方法进行类型转换了:
```csharp
using ExtensionMethods;
namespace xxx
{
    public static class xxxx
    {
        public static int Test()
        {
            int i = "123".ToInt();
        }
    }
}
```
怎么样？是不是感觉一股凉气当头灌下——**真爽啊！**
别急，光这样怎么能够满足我们的需求呢，接下来我们继续优化。
## 优化
以上已经方便了不少，但还是有以下两个问题：
>** 每次还需要引用命名空间，太浪费时间精力。**

解决方案：将扩展方法直接放到类型所在的命名空间下
依旧以上面的代码为例，就是：
```csharp
namespace System
{
    public static class ExtensionMethods
    {
        public static int ToInt(this string s)
        {
            return Convert.ToInt32(s);
        }
    }
}
```

> **如果输入的String并不能转化为数字,会报错。**

解决方案：加入另一个参数为默认值，若不能转化成功，则返回默认值。
代码如下:
```csharp
public static int ToInt32(this string s, int def = default(int))
{
    int result;
    return Int32.TryParse(s, out result) ? result : def;
}
```
按照上面的思路,我们可以打包一个包含各种常用类型转换的类,写程序的时候只要直接加入这个类,就可以方便地进行各种进制转换了.

下面提供一个自用的Convert类[Convert.zip](http://p4au3q1y8.bkt.clouddn.com/Convert.zip)
