---
title: 【爬虫学习笔记】.Net 使用 ScrapySharp 并行下载天涯图片
tags:
  - 爬虫
  - 'C#'
  - ScrapySharp
abbrlink: 31517
date: 2016-09-10 01:16:42
---
最近因为一个作业需要完成CNKI爬虫，研究爬虫架构的时候发现了这个疑似移植于Python的著名开源爬虫框架Scrapy的ScrapySharp，然而在网上寻找之后只发现了这个[F#的Demo](http://blog.csdn.net/hadstj/article/details/18891227)，就使用原文中示例的网站写了这个C#版本的代码。
<!-- more -->
## 实现
下面是代码:
```csharp
using System;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using HtmlAgilityPack;
using ScrapySharp.Extensions;
using ScrapySharp.Network;

namespace ScrapySharpDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            //示例网站地址
            var url = "http://bbs.tianya.cn/post-12-563201-1.shtml";
            var web = new ScrapingBrowser();
            var html = web.DownloadString(new Uri(url));
            var doc = new HtmlDocument();
            doc.LoadHtml(html);
            //获取网站中的图片地址
            var urls= doc.DocumentNode.CssSelect("div.bbs-content > img").Select(node => node.GetAttributeValue("original")).ToList();
            //并行下载图片
            Parallel.ForEach(urls, SavePic);
        }

        public static void SavePic(string url)
        {
            var web = new ScrapingBrowser();
            //因天涯网站限制,所有站外来源都无法访问图片,故先设置请求头Refer属性为当前页地址
            web.Headers.Add("Referer", "http://bbs.tianya.cn/post-12-563201-1.shtml");
            var pic = web.NavigateToPage(new Uri(url)).RawResponse.Body;
            var file = url.Substring(url.LastIndexOf("/", StringComparison.Ordinal));
            if (!Directory.Exists("imgs"))
                Directory.CreateDirectory("imgs");
            File.WriteAllBytes("imgs" + file, pic);
        }
    }
}
```
## 结论:
研究之后发现，ScrapySharp和Scrapy差距还是挺大的，没有Scrapy那样完善的八大组件，只含有Http请求的Downloader和基于HtmlAgilityPack扩展的网页解析功能，莫名有些小失望。