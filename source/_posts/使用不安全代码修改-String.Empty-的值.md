---
title: 使用不安全代码 + 反射修改 String.Empty 的值
abbrlink: 25922
date: 2019-03-11 20:42:10
author: 玮仔Wayne
tags:
  - unsafe
  - String.Empty
  - 反射
  - 不安全代码
categories:
  - 学习笔记
---
前几天的时候看到了 [吕毅](https://mvp.microsoft.com/en-us/PublicProfile/5003225) 大佬写的博客 
[为什么 C# 的 string.Empty 是一个静态只读字段，而不是一个常量呢？](https://blog.walterlv.com/post/why-string-empty-is-a-readonly-field-but-not-a-constant.html)，
非常感谢吕毅大佬的分享，在文章的末尾大佬提到了通过反射修改 String.Empty 的可能，于是我打算自己实践一下。
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311084845629.png)

<!--more-->

## 反射修改
首先上一个我自己封装的 ReflectionExtension 类，方便直接进行反射操作（这里只展示部分代码，完整代码会附上下载链接）

```csharp
namespace System.Reflection
{
    public static class ReflectionExtension
    {
        public const BindingFlags Flags = BindingFlags.Public | BindingFlags.NonPublic |
                         BindingFlags.Static | BindingFlags.Instance |
                         BindingFlags.DeclaredOnly;

        /// <summary>
        /// 通过反射设置字段的值
        /// </summary>
        /// <param name="instance"></param>
        /// <param name="fieldName">字段名称</param>
        /// <param name="value">要修改为的值</param>
        public static void SetField(this object instance, string fieldName, object value)
        {
            var type = instance.GetType();
            var field = type.GetField(fieldName) ?? type.GetField(fieldName, Flags);
            field?.SetValue(instance, value);
        }
    }
}
```
新建项目的框架为 .NET Framework 3.5，使用以下代码完成反射修改 String.Empty ：
```csharp
static void Main(string[] args)
{

    Console.WriteLine($"Empty : {string.Empty}");


    string.Empty.SetField("m_stringLength", 3);
    "".SetField("Empty","CBA");

    Console.WriteLine($"Empty Reflection : {string.Empty}");

    Console.ReadLine();
}
```

运行结果如图：
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311090724793.png)
有效!
查看一下 String.cs 的源码
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311112213462.png)
.NET Framework 3.5 以下版本中的 String.Empty 就是一个普通的共有只读字段。

下面将框架版本升级到 4.0 以上再次尝试：
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311112442989.png)
同样的方法在 4.0 以上的版本就行不通了，转到源码看一下：
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311112600692.png)
String.Empty 上增加了 **`__DynamicallyInvokable `** Attribute，查找资料:
```csharp
// Each blessed API will be annotated with a "__DynamicallyInvokableAttribute".
// This "__DynamicallyInvokableAttribute" is a type defined in its own assembly.
// So the ctor is always a MethodDef and the type a TypeDef.
// We cache this ctor MethodDef token for faster custom attribute lookup.
// If this attribute type doesn't exist in the assembly, it means the assembly
// doesn't contain any blessed APIs.
Type invocableAttribute = GetType("__DynamicallyInvokableAttribute", false);
if (invocableAttribute != null)
{
 Contract.Assert(((MetadataToken)invocableAttribute.MetadataToken).IsTypeDef);

 ConstructorInfo ctor = invocableAttribute.GetConstructor(Type.EmptyTypes);
 Contract.Assert(ctor != null);

 int token = ctor.MetadataToken;
 Contract.Assert(((MetadataToken)token).IsMethodDef);

 flags |= (ASSEMBLY_FLAGS)token & ASSEMBLY_FLAGS.ASSEMBLY_FLAGS_TOKEN_MASK;
}
```
根据注释中的介绍，这个特性只是某种用于提升速度的标记，看来大约是跟 CLR 有关，与这个 Attribute 并没有什么关系。

## 不安全代码

下面加入不安全代码，使用指针来实现我们的设置 String.Empty 。
首先打开项目属性，设置生成中的允许**不安全代码**选项
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311114311684.png)
代码：
```csharp
using System.Reflection;

namespace System
{
    public static class StringHelper
    {
        /// <summary>
        /// 使用反射 + 不安全代码直接修改 string 的值
        /// </summary>
        /// <param name="str">原始字符串</param>
        /// <param name="newStr">新字符串</param>
        public static unsafe void SetValue(this string str, string newStr)
        {
            //使用反射设置字符串的长度属性
            str.SetField("m_stringLength", newStr.Length);
            //指针大法好
            fixed (char* pe = str)
            {
                for (var i = 0; i < newStr.Length; i++)
                    pe[i] = newStr[i];
                pe[newStr.Length] = '\0';
            }
        }
    }
}
```
在 .NET Framework 4.7.2 下重新运行下面的测试代码

```csharp
static void Main(string[] args)
{

    Console.WriteLine($"Empty : {string.Empty}");

    string.Empty.SetField("m_stringLength", 3);
    "".SetField("Empty","CBA");

    Console.WriteLine($"Empty Reflection : {string.Empty}");

    string.Empty.SetValue("ABC");

    Console.WriteLine($"Empty Unsafe : {string.Empty}");

    Console.ReadLine();
}
```
![](http://qiniucdn.wayneshao.com/使用不安全代码修改-String.Empty-的值/20190311114612307.png)
不安全代码可以在最新版 Framework 上实现修改 String.Empty。

## 总结

在各个平台下运行测试代码，结论如下：

|平台| 反射 | 不安全代码 |
| ------------- | ------------- | ------------- |
|.NET Framework 1.0 - 3.5 | **√** | **√** |
|.NET Framework 4.0 - 4.7.2 | **×** | **√** |
|.NET Core 1.0 - 2.0 | **×** | **√** |
|.NET Core 2.1 - 2.2 | **×** | **×** |
|.NET Core 3.0 preview | **FieldAccessException**<br/>Cannot set initonly static field 'Empty' after type 'System.String' is initialized. | **×** |


