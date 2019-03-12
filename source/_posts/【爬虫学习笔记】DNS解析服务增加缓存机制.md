---
title: 【爬虫学习笔记】DNS解析服务增加缓存机制
tags:
  - 爬虫
  - 缓存
abbrlink: 2858
date: 2016-09-11 14:08:43
categories:
  - 学习笔记
---
 之前我们已经基于ARSoft.Tools.Net简单实现了DNS解析模块的功能，但是当性能要求升高时，每一次爬取都要进行DNS请求，甚至很有可能一段时间内每次请求的都是相同的地址，频繁的DNS请求就会成为性能瓶颈，所以我们要通过缓存机制将DNS解析结果缓存下来，降低DNS解析操作，提升系统性能。
 <!-- more -->
 如此，我们基于之前封装的MemoryCacheHelper类对DnsResolver类进行改造：
 ```csharp
 using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Net;
using ARSoft.Tools.Net;
using ARSoft.Tools.Net.Dns;
using Mem = Crawler.Common.MemoryCacheHelper;

namespace Crawler.Protocol
{
    public class DnsResolver
    {
        public TimeSpan TimeSpan { get; set; }
        public string Url { get; set; }
        public ARecord Record { get; set; }
        public string DnsServer { get; set; }
        public int TimeOut { get; set; }
        public ReturnCode ReturnCode { get; set; }
        public bool IsSuccess { get; private set; }
        public TimeSpan TimeToLive { get; set; }
        public DnsResolver(string url, string dnsServer = "223.5.5.5", int timeOut = 10000)
        {
            Url = url;
            DnsServer = dnsServer;
            TimeOut = timeOut;
            IsSuccess = false;
            if (Mem.Contains(url))
                Fill(Mem.Get<DnsResolver>(url));
            else
                Dig();
        }

        private void Fill(DnsResolver resolver)
        {
            TimeSpan = resolver.TimeSpan;
            Url = resolver.Url;
            Record = resolver.Record;
            DnsServer = resolver.DnsServer;
            TimeOut = resolver.TimeOut;
            ReturnCode = resolver.ReturnCode;
            IsSuccess = resolver.IsSuccess;
        }

        public void Dig()
        {
            //初始化DnsClient，第一个参数为DNS服务器的IP，第二个参数为超时时间
            var dnsClient = new DnsClient(IPAddress.Parse(DnsServer), TimeOut);
            var s = new Stopwatch();
            s.Start();
            //解析域名。将域名请求发送至DNS服务器解析，参数为需要解析的域名
            var dnsMessage = dnsClient.Resolve(DomainName.Parse(Url));
            s.Stop();
            TimeSpan = s.Elapsed;
            //若返回结果为空，或者存在错误，则该请求失败。
            if (dnsMessage == null || (dnsMessage.ReturnCode != ReturnCode.NoError && dnsMessage.ReturnCode != ReturnCode.NxDomain))
                IsSuccess = false;
            //循环遍历返回结果，将返回的IPV4记录添加到结果集List中。
            if (dnsMessage != null)
            {
                if (dnsMessage.AnswerRecords.Count > 0)
                {
                    Record = dnsMessage.AnswerRecords[0] as ARecord;
                    if (Record != null)
                    {
                        IsSuccess = true;
                        TimeToLive=new TimeSpan(0,0,Record.TimeToLive);
                        Mem.Add(Url, this, TimeToLive);
                    }
                }
            }
            if (dnsMessage != null) ReturnCode = dnsMessage.ReturnCode;
        }
    }
}
 ```
 这样,每次做完DNS解析后，会根据域名的TTL将解析结果缓存下来，下次查询时可以直接调用缓存，提高系统性能。