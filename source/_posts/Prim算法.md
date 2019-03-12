---
title: 【算法复习】贪心算法之最小生成树Prim算法
tags:
  - 51NOD
  - 刷题
  - 'C#'
  - Prim
  - 贪心
abbrlink: 44726
date: 2018-02-28 21:16:20
categories:
  - 算法笔记
---
最小生成树的Prim算法也是贪心算法的一大经典应用。Prim算法的特点是时刻维护一棵树，算法不断加边，加的过程始终是一棵树。
<!--more-->
## 简述
Prim算法过程：

一条边一条边地加， 维护一棵树。

初始 E ＝ ｛｝空集合， V = ｛任意节点｝

循环（n – 1）次，每次选择一条边（v1,v2）， 满足：v1属于V , v2不属于V。且（v1,v2）权值最小。

E = E + （v1,v2）
V = V + v2

最终E中的边是一棵最小生成树， V包含了全部节点。
## 执行过程
以下图为例介绍Prim算法的执行过程。
![](http://qiniucdn.wayneshao.com/20180228211614459/20180228092129488.png)
Prim算法的过程从A开始 V = {A}, E = {}
![](http://qiniucdn.wayneshao.com/20180228211614459/20180228092216396.png)
选中边AF , V = {A, F}, E = {(A,F)} 
![](http://qiniucdn.wayneshao.com/20180228211614459/20180228092234421.png)
选中边FB, V = {A, F, B}, E = {(A,F), (F,B)}
![](http://qiniucdn.wayneshao.com/20180228211614459/20180228092252771.png)
选中边BD, V = {A, B, F, D},   E = {(A,F), (F,B), (B,D)}
![](http://qiniucdn.wayneshao.com/20180228211614459/20180228092305430.png)
选中边DE, V = {A, B, F, D, E},   E = {(A,F), (F,B), (B,D), (D,E)}
![](http://qiniucdn.wayneshao.com/20180228211614459/20180228092428701.png)
选中边BC, V = {A, B, F, D, E, c},   E = {(A,F), (F,B), (B,D), (D,E), (B,C)}, 算法结束。
## 算法证明
Prim算法的证明：假设Prim算法得到一棵树P，有一棵最小生成树T。假设P和T不同，我们假设Prim算法进行到第(K – 1)步时选择的边都在T中，这时Prim算法的树是P’, 第K步时,Prim算法选择了一条边e = (u, v)不在T中。假设u在P’中，而v不在。

因为T是树，所以T中必然有一条u到v的路径，我们考虑这条路径上第一个点u在P’中，最后一个点v不在P’中，则路径上一定有一条边f = (x,y)，x在P’中，而且y不在P’中。
我们考虑f和e的边权w(f)与w(e)的关系：

若w(f) > w(e)，在T中用e换掉f （T中加上e去掉f)，得到一个权值和更小的生成树，与T是最小生成树矛盾。
若w(f) < w(e), Prim算法在第K步时应该考虑加边f，而不是e,矛盾。

因此只有w(f) = w(e),我们在T中用e换掉f，这样Prim算法在前K步选择的边在T中了，有限步之后把T变成P,而树权值和不变， 从而Prim算法是正确的。
请仔细理解Prim算法——时刻维护一棵生成树。我们的证明构造性地证明了所有地最小生成树地边权（多重）集合都相同！
## 题目测试
最后，我们来提供输入输出数据，由你来写一段程序，实现这个算法，只有写出了正确的程序，才能继续后面的课程。

**输入**

第1行：2个数N,M中间用空格分隔，N为点的数量，M为边的数量。（2 <= N <= 1000, 1 <= M <= 50000)
第2 - M + 1行：每行3个数S E W，分别表示M条边的2个顶点及权值。(1 <= S, E <= N，1 <= W <= 10000)

**输出**

输出最小生成树的所有边的权值之和。

**输入示例**

> 9 14
1 2 4
2 3 8
3 4 7
4 5 9
5 6 10
6 7 2
7 8 1
8 9 7
2 8 11
3 9 2
7 9 6
3 6 4
4 6 14
1 8 8

输出示例

> 37

请选取你熟悉的语言，并在下面的代码框中完成你的程序，注意数据范围，最终结果会造成Int32溢出，这样会输出错误的答案。
不同语言如何处理输入输出，请查看下面的语言说明。

## 题目分析
声明一个结构类型储存“边”的信息。
```csharp
struct Side
{
    public int[] Endpoints;

    public int Weight;

    public Side(int endpoint1, int endpoint2, int weight)
    {
        Endpoints = new int[2] { endpoint1, endpoint2 };
        Weight = weight;
    }
}
```
首先读入总边数和总点数
```csharp
var line1 = Console.ReadLine().Split(' ');
var n = Convert.ToInt32(line1[0]);
var m = Convert.ToInt32(line1[1]);
```
读入所有“边”，储存在数组中
```csharp
var sides = new Side[m];
var points = new List<int>();
var totalWeight = 0L;
for (var i = 0; i < m; i++)
{
    var line = Console.ReadLine().Split(' ');
    sides[i] = new Side(Convert.ToInt32(line[0]), Convert.ToInt32(line[1]), Convert.ToInt32(line[2]));
}
```
将所有边按照权值排序
```csharp
var orderSides = sides.OrderBy(s => s.Weight
```
加入起点
```csharp
points.AddRange(orderSides[0].Endpoints);
totalWeight += orderSides[0].Weight;
orderSides.Rem
```
Prim：
按权值从小到大循环遍历数组里的边，如果发现某一个边的一个端点在端点数组里有，另一个端点在端点数组里没有，就把另一端点也加入，然后累加权值。直到端点数组内端点个数等于总端点数。
```csharp
while (points.Count != n)
{
    for (var i = 0; i < orderSides.Count; i++)
    {
        if ((!points.Contains(orderSides[i].Endpoints[0]) && !points.Contains(orderSides[i].Endpoints[1]))||(points.Contains(orderSides[i].Endpoints[0]) && points.Contains(orderSides[i].Endpoints[1]))) continue;
        points.Add(points.Contains(orderSides[i].Endpoints[0]) ? orderSides[i].Endpoints[1] : orderSides[i].Endpoints[0]);
        totalWeight += orderSides[i].Weight;
        orderSides.RemoveAt(i);
        break;
    }
}
```
## Accepted
```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class Sum
{
    struct Side
    {
        public int[] Endpoints;

        public int Weight;

        public Side(int endpoint1, int endpoint2, int weight)
        {
            Endpoints = new int[2] { endpoint1, endpoint2 };
            Weight = weight;
        }
    }

    public static void Main()
    {
        var line1 = Console.ReadLine().Split(' ');

        var n = Convert.ToInt32(line1[0]);
        var m = Convert.ToInt32(line1[1]);
        var sides = new Side[m];
        var points = new List<int>();
        var totalWeight = 0L;
        for (var i = 0; i < m; i++)
        {
            var line = Console.ReadLine().Split(' ');
            sides[i] = new Side(Convert.ToInt32(line[0]), Convert.ToInt32(line[1]), Convert.ToInt32(line[2]));
        }

        var orderSides = sides.OrderBy(s => s.Weight).ToList();

        points.AddRange(orderSides[0].Endpoints);
        totalWeight += orderSides[0].Weight;
        orderSides.RemoveAt(0);
        
        while (points.Count != n)
        {
            for (var i = 0; i < orderSides.Count; i++)
            {
                if ((!points.Contains(orderSides[i].Endpoints[0]) && !points.Contains(orderSides[i].Endpoints[1]))||(points.Contains(orderSides[i].Endpoints[0]) && points.Contains(orderSides[i].Endpoints[1]))) continue;
                points.Add(points.Contains(orderSides[i].Endpoints[0]) ? orderSides[i].Endpoints[1] : orderSides[i].Endpoints[0]);
                totalWeight += orderSides[i].Weight;
                orderSides.RemoveAt(i);
                break;
            }
        }

        Console.WriteLine(totalWeight);
    }
}
```