---
title: 【爬虫学习笔记】基于Bloom Filter的url去重模块UrlSeen
tags:
  - 爬虫
  - Bloom Filter
abbrlink: 41470
date: 2016-09-26 21:50:43
---
Url Seen用来做url去重。对于一个大的爬虫系统，它可能已经有百亿或者千亿的url，新来一个url如何能快速的判断url是否已经出现过非常关键。因为大的爬虫系统可能一秒钟就会下载几千个网页，一个网页一般能够抽取出几十个url，而每个url都需要执行去重操作，可想每秒需要执行大量的去重操作。因此Url Seen是整个爬虫系统中非常有技术含量的一个部分。
<!-- more -->

为了提高过滤的效率,我们使用有极低误判率但是效率非常高的算法——Bloom Filter，已经有高手写好了Bloom Filter的算法实现，我们这里就直接站在巨人的肩膀上直接使用他写好的类库啦。

Nuget:
> Install-Package BloomFilter

代码实现:
```csharp
using System;
using BloomFilterDotNet;

namespace Crawler.Processing
{
    /// <summary>
    /// Url Seen用来做url去重。对于一个大的爬虫系统，它可能已经有百亿或者千亿的url，新来一个url如何能快速的判断url是否已经出现过非常关键。因为大的爬虫系统可能一秒钟就会下载几千个网页，一个网页一般能够抽取出几十个url，而每个url都需要执行去重操作，可想每秒需要执行大量的去重操作。因此Url Seen是整个爬虫系统中非常有技术含量的一个部分。
    /// </summary>
    public class UrlSeen
    {
        private BloomFilter<string> Seen { set; get; }
        public UrlSeen()
        {
            Seen = new BloomFilter<string>(1000000, 0.0001, null);
        }
        public UrlSeen(int targetCapacity, double falsePositiveRate)
        {
            Seen = new BloomFilter<string>(targetCapacity, falsePositiveRate, null);
        }
        public bool MatchUrl(Uri url)
        {
            return Seen.Contains(url.ToString());
        }
        public int Count
        {
            get { return Seen.Count; }
        }
        public void Add(Uri url)
        {
            Seen.Add(url.ToString());
        }
    }
}
```