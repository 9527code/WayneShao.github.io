---
title: 【迷宫中的算法实践】迷宫生成算法——Prim算法
tags:
  - 迷宫
  - 算法
  - Prim
abbrlink: 47068
date: 2016-09-20 21:04:44
categories:
  - 算法笔记
---
  普里姆算法（Prim算法），图论中的一种算法，可在加权连通图里搜索最小生成树。意即由此算法搜索到的边子集所构成的树中，不但包括了连通图里的所有顶点（英语：Vertex (graph theory)），且其所有边的权值之和亦为最小。该算法于1930年由捷克数学家沃伊捷赫·亚尔尼克（英语：Vojtěch Jarník）发现；并在1957年由美国计算机科学家罗伯特·普里姆（英语：Robert C. Prim）独立发现；1959年，艾兹格·迪科斯彻再次发现了该算法。因此，在某些场合，普里姆算法又被称为DJP算法、亚尔尼克算法或普里姆－亚尔尼克算法。

——来自百度百科
<!-- more -->
当我们将Prim算法用于迷宫生成时，情况有些不同，维基百科中给出了[随机Prim迷宫生成算法(Randomized Prim's algorithm)](https://en.wikipedia.org/wiki/Prim%27s_algorithm)的解释及实现过程：
![](http://qiniucdn.wayneshao.com/20180218224344/20180218104613142.png)
我们将算法实现部分翻译成中文 
1. 让迷宫全都是墙.。
2. 选一个格，作为迷宫的通路，然后把它的邻墙放入列表.。
3. 当列表里还有墙时:
    1). 从列表里随机选一个墙，如果它对面的格子不是迷宫的通路:
    2). 把墙打通，让对面的格子成为迷宫的通路.。
把那个格子的邻墙加入列表。
4. 如果对面的格子已经是通路了，那就从列表里移除这面墙。


简单研究算法实现过程我们可以发现，Prim算法就是不断地从所有可以是通路的位置中随意选一个挖洞，直到没有可能为通路的位置。

 整个实现过程还是相当于随意为路线附权值的Prim算法。

下面我们来做C#下的代码实现：
```csharp
/// <summary>
/// 普利姆迷宫生成法
/// </summary>
/// <param name="startX">起始点X坐标</param>
/// <param name="startY">起始点Y坐标</param>
/// <param name="widthLimit">迷宫宽度</param>
/// <param name="heightLimit">迷宫高度</param>
/// <param name="haveBorder">迷宫是否含有墙</param>
private int[,] Prim(int startX, int startY, int widthLimit, int heightLimit,bool haveBorder)
{
    //block:不可通行    unBlock:可通行
    const int block = 0,unBlock = 1;
    var r=new Random();
    //迷宫尺寸合法化
    if (widthLimit < 1)
        widthLimit = 1;
    if (heightLimit < 1)
        heightLimit = 1;
    //迷宫起点合法化
    if (startX < 0 || startX >= widthLimit)
        startX = r.Next(0, widthLimit);
    if (startY < 0 || startY >= heightLimit)
        startY = r.Next(0, heightLimit);
    //减去边框所占的格子
    if (!haveBorder)
    {
        widthLimit--;
        heightLimit--;
    }
    //迷宫尺寸换算成带墙尺寸
    widthLimit *= 2;
    heightLimit *= 2;
    //迷宫起点换算成带墙起点
    startX *= 2;
    startY *= 2;
    if (haveBorder)
    {
        startX++;
        startY++;
    }
    //产生空白迷宫
    var mazeMap = new int[widthLimit + 1, heightLimit + 1];
    for (int x = 0; x <= widthLimit; x++)
    {
        //mazeMap.Add(new BitArray(heightLimit + 1));
        for (int y = 0; y <= heightLimit; y++)
        {
            mazeMap[x, y] = block;
        }
    }

    //邻墙列表
    var blockPos = new List<int>();
    //将起点作为目标格
    int targetX = startX, targetY = startY;
    //将起点标记为通路
    mazeMap[targetX, targetY] = unBlock;

    //记录邻墙
    if (targetY > 1)
    {
        blockPos.AddRange(new int[] { targetX, targetY - 1, 0 });
    }
    if (targetX < widthLimit)
    {
        blockPos.AddRange(new int[] { targetX + 1, targetY, 1 });
    }
    if (targetY < heightLimit)
    {
        blockPos.AddRange(new int[] { targetX, targetY + 1, 2 });
    }
    if (targetX > 1)
    {
        blockPos.AddRange(new int[] { targetX - 1, targetY, 3 });
    }
    while (blockPos.Count > 0)
    {
        //随机选一堵墙
        var blockIndex = r.Next(0, blockPos.Count / 3) * 3;
        //找到墙对面的墙
        if (blockPos[blockIndex + 2] == 0)
        {
            targetX = blockPos[blockIndex];
            targetY = blockPos[blockIndex + 1] - 1;
        }
        else if (blockPos[blockIndex + 2] == 1)
        {
            targetX = blockPos[blockIndex] + 1;
            targetY = blockPos[blockIndex + 1];
        }
        else if (blockPos[blockIndex + 2] == 2)
        {
            targetX = blockPos[blockIndex];
            targetY = blockPos[blockIndex + 1] + 1;
        }
        else if (blockPos[blockIndex + 2] == 3)
        {
            targetX = blockPos[blockIndex] - 1;
            targetY = blockPos[blockIndex + 1];
        }
        //如果目标格未连通
        if (mazeMap[targetX, targetY] == block)
        {
            //联通目标格
            mazeMap[blockPos[blockIndex], blockPos[blockIndex + 1]] = unBlock;
            mazeMap[targetX, targetY] = unBlock;
            //添加目标格相邻格
            if (targetY > 1 && mazeMap[targetX, targetY - 1] == block && mazeMap[targetX, targetY - 2] == block)
            {
                blockPos.AddRange(new int[] { targetX, targetY - 1, 0 });
            }
            if (targetX < widthLimit && mazeMap[targetX + 1, targetY] == block && mazeMap[targetX + 2, targetY] == block)
            {
                blockPos.AddRange(new int[] { targetX + 1, targetY, 1 });
            }
            if (targetY < heightLimit && mazeMap[targetX, targetY + 1] == block && mazeMap[targetX, targetY + 2] == block)
            {
                blockPos.AddRange(new int[] { targetX, targetY + 1, 2 });
            }
            if (targetX > 1 && mazeMap[targetX - 1, targetY] == block && mazeMap[targetX - 1, targetY] == block)
            {
                blockPos.AddRange(new int[] { targetX - 1, targetY, 3 });
            }
        }
        blockPos.RemoveRange(blockIndex, 3);
    }
    return mazeMap;
}
```