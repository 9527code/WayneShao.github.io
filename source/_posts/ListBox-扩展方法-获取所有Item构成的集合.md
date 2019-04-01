---
title: ListBox 扩展方法-获取所有Item构成的集合
author: 玮仔Wayne
tags:
  - 'C#'
  - WinForm
categories:
  - 踩坑笔记
abbrlink: 5516
date: 2019-03-29 23:12:07
---
今天在实现一个小功能 “插入更新 ListBox 中的 Item ”时，发现 `ListBox.Items` 的类型 `ListBox.ObjectCollection` 实现了 `IList`, `ICollection`, `IEnumerable`三个接口，整体的方法已经非常接近一个数组，但是方便程度跟数据确实差距较大，Linq 也完全不能使用，故封装一个扩展方法实现直接获取 `ListBox.Items`中所储存的数据的集合。

<!--more-->
```cs
using System;
using System.Collections.Generic;
using System.Linq;

namespace System.Windows.Forms
{
    public static class ListBoxExtension
    {
        /// <summary>
        /// 获取 ListBox.Items 中的数据构成的集合
        /// </summary>
        /// <typeparam name="T">ListBox.Items中储存的数据的类型</typeparam>
        /// <param name="listbox">ListBox对象</param>
        /// <returns></returns>
        public static List<T> GetItems<T>(this ListBox listbox) where T : class
        {
            var items = new object[listbox.Items.Count];
            listbox.Items.CopyTo(items, 0);

            var current = items.Select(i => i as T).Where(i => i != null).ToList();
            if (listbox.Items.Count != current.Count)
                throw new Exception($"并非所有元素都为{typeof(T).Name}类型");

            return current.ToList();
        }
    }
}
```
