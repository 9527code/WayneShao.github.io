---
title: 【爬虫学习笔记】Url过滤模块UrlFilter
tags:
  - 爬虫
  - UrlFilter
abbrlink: 27677
date: 2016-09-26 20:50:45
categories:
  - 学习笔记
---
Url Filter则是对提取出来的URL再进行一次筛选。不同的应用筛选的标准是不一样的，比如对于baidu/google的搜索，一般不进行筛选，但是对于垂直搜索或者定向抓取的应用，那么它可能只需要满足某个条件的url，比如不需要图片的url，比如只需要某个特定网站的url等等。Url Filter是一个和应用密切相关的模块。
```csharp
using System;
using System.Collections.Generic;
using Crawler.Common;

namespace Crawler.Processing
{
    public class UrlFilter
    {
        public static List<Uri> RemoveByRegex(List<Uri> uris, params string[] regexs)
        {
            var uriList=new List<Uri>(uris);
            for (var i = 0; i < uriList.Count; i++)
            {
                foreach (var r in regexs)
                {
                    if (!RegexHelper.IsMatch(uriList[i].ToString(), r)) continue;
                    uris.RemoveAt(i);
                    i--;
                }
            }
            return uriList;
        }

        public static List<Uri> SelectByRegex(List<Uri> uris, params string[] regexs)
        {
            var uriList = new List<Uri>();
            foreach (var t in uris)
                foreach (var r in regexs)
                    if (RegexHelper.IsMatch(t.ToString(), r))
                        if(!uriList.Contains(t))
                            uriList.Add(t);
            return uriList;
        }

    }
}
```
