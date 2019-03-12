---
title: 【51NOD刷题】1305 Pairwise Sum and Divide
tags:
  - 51NOD
  - 刷题
  - 'C#'
mathjax: true
abbrlink: 34469
date: 2018-02-24 17:25:50
categories:
  - 51NOD刷题
---
[**1305 Pairwise Sum and Divide  **](http://www.51nod.com/onlineJudge/questionCode.html#!problemId=1305)

题目来源： HackerRank
基准时间限制：1 秒&#8195;空间限制：131072 KB&#8195;分值: 5&#8195;难度：1级算法题

有这样一段程序，fun会对整数数组A进行求值，其中Floor表示向下取整：
```python
fun(A)
    sum = 0
    for i = 1 to A.length
        for j = i+1 to A.length
            sum = sum + Floor((A[i]+A[j])/(A[i]*A[j])) 
    return sum
```
给出数组A，由你来计算fun(A)的结果。例如：A = {1, 4, 1}，fun(A) = [5/4] + [2/1] + [5/4] = 1 + 2 + 1 = 4。
<!--more-->
## 输入输出
**Input**
第1行：1个数N，表示数组A的长度(1 <= N <= 100000)。
第2 - N + 1行：每行1个数A[i]（1 <= A[i] <= 10^9)。
**Output**
输出fun(A)的计算结果。
**Input示例**
> 3
1 
4 
1

**Output示例**
> 4

## 题目分析
因为题目给出的已经是近乎伪代码了，所以初始很容易直接按照程序中给出的逻辑来提交，然而可能是C#的效率问题，差不多一般的测试都超时了，下面是第一版的代码：
```csharp
using System;
using System.IO;

public class Sum
{
    public static void Main()
    {
        var sr = new StreamReader(Console.OpenStandardInput());
        var sw = new StreamWriter(Console.OpenStandardOutput());
        var count = Convert.ToInt32(sr.ReadLine());
        var sum = 0L;
        var list = new long[count];

        for (var i = 0; i < count; i++)
            list[i] = Convert.ToInt64(sr.ReadLine());

        for (var i = 0; i < count; i++)
            for (var j = i + 1; j < count; j++)
                sum += (list[i] + list[j]) / (list[i] * list[j]);
        sw.WriteLine(sum);
        sw.Flush();
        sr.Close();
        sw.Close();
    }
}
```
这个时候返回来分析题目，100,000次输入操作，之后双层循环差不多10,000,000,000次循环内操作，C#在1.5s内确实不大可能完的成，只能从题目入手重新分析了。

题目总的来分析就是每个数和集合中的所有其他数做一次和除以积取整的操作后的和。

仔细考虑后不难发现，其实所有的数里只有1和2是可以做有效贡献的，其他数做上述运算一定是0，所以其实只需要统计1和2的数量即可。
其中，1和1的运算结果为2，1和其他数的计算结果是1，2和2的计算结果是1。设一个长度为n的集合里，1的个数为a，2的个数为b，则这个集合的运算结果公式为：

$C_a^2+a(n-a)+C_b^2$

化简可得：

$a(a-1)+a(n-a)+\frac{1}{2}(b*(b-1))$

PS.因为每个数的范围最大可达$10^9$，所以我们使用64位的long类型来接收和运算。
## Accepted
```csharp
using System;
using System.IO;

public class Sum
{
    public static void Main()
    {
        var sr = new StreamReader(Console.OpenStandardInput());
        var sw = new StreamWriter(Console.OpenStandardOutput());
        var count = Convert.ToInt32(sr.ReadLine());
        var sum = 0L;
        var a = 0L;
        var b = 0L;
        for (var i = 0; i < count; i++)
        {
            var s = Convert.ToInt64(sr.ReadLine());
            switch (s)
            {
                case 1:
                    a++;
                    break;
                case 2:
                    b++;
                    break;
            }
        }

        sum = a * (a - 1) + a * (count - a) + (b * (b - 1) / 2);
        sw.WriteLine(sum);
        Console.ReadLine();
        sw.Flush();
        sr.Close();
        sw.Close();
    }
}
```