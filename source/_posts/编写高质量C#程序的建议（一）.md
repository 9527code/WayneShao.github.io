---
title: '编写高质量C#程序的建议（1-5）'
abbrlink: 61676
date: 2019-03-28 21:40:55
author: 玮仔Wayne
tags:
  - C#
  - 高质量代码
categories:
  - 读书笔记
---
　　最近开始阅读[陆敏技先生](https://www.cnblogs.com/luminji/)在[机械工业出版社](http://www.cmpedu.com/)出版的[《编写高质量代码：改善C#程序的157个建议》](https://item.jd.com/10860284.html)一书，打算把其中涉及的所有的观点做一下总结和分析，用于总结和事后翻阅，如果有侵权请联系我删除。

## 建议一：正确操作字符串
### 确保尽量小的装箱
　　在自己编写的代码中，应当尽可能地避免编写不必要的装箱代码。
```csharp
var str1 = "str1" + 9;
var str2 = "str2" + 9.ToString();
```
　　第一句代码中，+ 连接时是将 值类型 int 转换为 引用类型 string 之后在进行 Concat 操作，故而性能更差。

　　装箱之所以带来性能损耗的原因是，装箱需要以下三个步骤：
1. 首先，会为值类型在托管堆中分配内存。除了值类型本身所分配的内存外，内存总量还要加上类型对象指针和同步块索引所占用的内存。
2. 将值类型的值复制到新分配的堆内存中。
3. 返回已经成为引用类型的对象的地址。

### 避免分配额外的内存空间

　　频繁的进行字符串的拼接操作时，最好使用 StringBuilder，字符串的任何方法或者进行任何运算都会在内存中创建一个新的字符串对象。
  
## 建议二：使用默认转型方法
1. 使用类型的转换运算符。
2. 使用类型内置的 Parse、TryParse，或者如 ToString、ToDouble 和 ToDateTime 等方法。
3. 使用帮助类提供的方法。
4. 使用 CLR 支持的转型

## 建议三：区别对待强制转型与 as 和 is
1. 如果类型之间都上溯到了某个共同的基类，那么根据此基类进行的转型（即基类转型为子类本身）应该使用 as。子类与子类之间的转型，则应该提供转换操作符，以便进行强制转型。
2. as 操作符永远不会抛出异常，如果类型不匹配则会返回 null。
3. as 不能操作基元类型，如果涉及基元类型的算法，就需要通过 is 转型前的类型进行判断，以避免转型失败。
4. C# 7.0 中提供了 is 的新语法，以下两个方法其实是完全等价的：
```cs
public void A2(object obj)
{
    var str = obj as string;
    if (str != null)
        Console.WriteLine(str);
}
public void A3(object obj)
{
    if (obj is string str)
        Console.WriteLine(str);

}
```

## 建议四： TryParse 比 Parse 好
Parse 和 TryParse 如果执行成功，他们的效率在一个数量级上，但如果执行失败，Parse 方法在转化失败的时候会引发异常，极大地消耗效率，而 TryParse 并不会。

## 建议五： 使用 int? 来确保值类型也可以为 null
　　业务需求中，int 类型的字段在无意义时在业务上为 null 比它的默认值 0 更为合适。