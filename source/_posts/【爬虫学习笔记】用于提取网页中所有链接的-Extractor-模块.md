---
title: 【爬虫学习笔记】用于提取网页中所有链接的 Extractor 模块
tags:
  - 爬虫
  - Extractor
abbrlink: 33241
date: 2016-09-25 00:55:25
---
   Extractor的工作是从下载的网页中将它包含的所有URL提取出来。这是个细致的工作，你需要考虑到所有可能的url的样式，比如网页中常常会包含相对路径的url，提取的时候需要将它转换成绝对路径。这里我们选择使用正则表达式来完成链接的提取。

   html标签中的链接地址通常会出现在href属性或者src属性中，所以我们采用两个正则表达式来匹配网页中的所有链接地址。
<!-- more -->
网页链接提取器Extractor类：
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Crawler.Common;

namespace Crawler.Processing
{
    /// <summary>
    /// Extractor的工作是从下载的网页中将它包含的所有URL提取出来。这是个细致的工作，你需要考虑到所有可能的url的样式，比如网页中常常会包含相对路径的url，提取的时候需要将它转换成绝对路径。
    /// </summary>
    public class Extractor
    {
        public List<Uri> GetAllUrl(string html, string host)
        {
            var list = new List<string>();
            //匹配href属性
            var href = RegexHelper.ExtractStringArray(html, "href *= *['\"]*(\\S+)[\"']");
            //去掉匹配到字符串的空格、双引号和前面的href=，得到链接
            var temp = from h in href
                       select h.Replace(" ", "").Replace("\"", "").Substring(5);
            //加入数组
            list.AddRange(temp);

            //匹配src属性
            var src = RegexHelper.ExtractStringArray(html, "src *= *['\"]*(\\S+)[\"']");
            temp = from s in src
                   select s.Replace(" ", "").Replace("\"", "").Substring(4);
            list.AddRange(temp);

            //去重
            list = list.Distinct().ToList();

            //将链接地址中的相对路径转换为绝对路径
            var uriList = list.Select(s => s.IndexOf("http://", StringComparison.Ordinal) != 0 ? new Uri(new Uri(host), s) : new Uri(s)).ToList();
            return uriList.ToList();
        }
    }
}
```