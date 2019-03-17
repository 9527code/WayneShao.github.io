---
title: 【爬虫学习笔记】ScrapySharp简单封装为Requester
tags:
  - 爬虫
  - ScrapySharp
abbrlink: 41785
date: 2016-09-13 00:26:46
categories:
  - 学习笔记
---
为了便于使用及日后的扩展，将Scrapy简单封装为了Requester。
<!-- more -->
具体代码如下：
```csharp
using System;
using System.Collections.Generic;
using Crawler.Common;

namespace Crawler.Protocol
{
    public class Requester
    {
        private Uri Url { get; set; }
        private Browser Browser { get; set; }

        public Requester(string url, Dictionary<string, string> headers = null, Browser browser = null)
        {
            var u = new Uri(url);
            //检测地址是域名还是IP地址,如果是域名,则使用DnsResolver解析为IP地址
            var leftPart = u.GetLeftPart(UriPartial.Authority).Replace(u.GetLeftPart(UriPartial.Scheme), "");
            //正则匹配是否为IP地址
            if (!RegexHelper.IsMatch(leftPart, @"\d+\.\d+\.\d+\.\d+\w"))
            {
                var dns = new DnsResolver(leftPart);
                if (dns.IsSuccess)
                    u = new Uri(url.Replace(leftPart, dns.Record.Address.ToString()));
            }
            Url = u;
            Browser = browser ?? new Browser();
            if (headers == null) return;
            foreach (var header in headers)
                Browser.Headers[header.Key] = header.Value;
        }



        public string GetHtml()
        {
            return Browser.DownloadString(Url);
        }

        public byte[] GetFile()
        {
            return Browser.NavigateToPage(Url).RawResponse.Body;
        }
    }
}
```
考虑到可能对ScrapyBrowser做一些扩展（例如增加对FTP等其他协议的支持），故新建了Browser类继承自ScrapyBrowser类：
```csharp
using ScrapySharp.Network;

namespace Crawler.Protocol
{
    public class Browser : ScrapingBrowser
    {

    }
}
```