---
title: 【爬虫学习笔记】.Net 基于ARSoft.Tools.Net的DNS解析模块（半成品）
tags:
  - 爬虫
  - ARSoft.Tools.Net
  - 'C#'
  - DNS
abbrlink: 61444
date: 2016-09-11 01:01:24
categories:
  - 学习笔记
---
最近在做爬虫的作业，今天学习的内容是关于DNS解析模块的制作的。使用的库为ARSoft.Tools.Net，它是一个非常强大的开源DNS控件库，包含.Net SPF validation, SenderID validation以及DNS Client、DNS Server接口。使用该接口可轻松实现DNS客户请求端及服务器解析端。
<!-- more -->
项目地址：http://arsofttoolsnet.codeplex.com/，

Nuget包地址：https://www.nuget.org/packages/ARSoft.Tools.Net/。
## 引入
首先引入nuget包:
> **Install-Package ARSoft.Tools.NetInstall-Package ARSoft.Tools.Net**

## 具体实现
### 实现代码:
```csharp
/// <summary>
/// DNS解析
/// </summary>
/// <param name="dnsServer">DNS服务器IP</param>
/// <param name="timeOut">解析超时时间</param>
/// <param name="url">解析网址</param>
/// <param name="isSuccess">是否解析成功</param>
/// <returns>解析到的IP信息</returns>
public static IPAddress DnsResolver(string dnsServer, int timeOut, string url, out bool isSuccess)
{
    //初始化DnsClient，第一个参数为DNS服务器的IP，第二个参数为超时时间
    var dnsClient = new DnsClient(IPAddress.Parse(dnsServer), timeOut);
    //解析域名。将域名请求发送至DNS服务器解析，第一个参数为需要解析的域名，第二个参数为
    //解析类型， RecordType.A为IPV4类型
    //DnsMessage dnsMessage = dnsClient.Resolve("www.sina.com", RecordType.A);
    var s = new Stopwatch();
    s.Start();
    var dnsMessage = dnsClient.Resolve(DomainName.Parse(url));
    s.Stop();
    Console.WriteLine(s.Elapsed.Milliseconds);
    //若返回结果为空，或者存在错误，则该请求失败。
    if (dnsMessage == null || (dnsMessage.ReturnCode != ReturnCode.NoError && dnsMessage.ReturnCode != ReturnCode.NxDomain))
    {
        isSuccess= false;
    }
    //循环遍历返回结果，将返回的IPV4记录添加到结果集List中。
    if (dnsMessage != null)
        foreach (var dnsRecord in dnsMessage.AnswerRecords)
        {
            var aRecord = dnsRecord as ARecord;
            if (aRecord == null) continue;
            isSuccess = true;
            return aRecord.Address;
        }
    isSuccess= false;
    return null;
}
```
### 调用代码
```csharp
bool isSuccess;
IPAddress ip = DnsResolver("223.5.5.5", 200, "shaoweicloud.cn", out isSuccess);
if (isSuccess)
    Console.WriteLine(ip);
```
## 进一步封装
懂的使用方法后我们可以对它做进一步封装,得到**DnsResolver**类:
### 实现代码
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Net;
using ARSoft.Tools.Net;
using ARSoft.Tools.Net.Dns;

namespace Crawler.Protocol
{
    public class DnsResolver
    {
        public TimeSpan TimeSpan { get; set; }
        public string Url { get; set; }
        public List Record { get; set; }
        public string DnsServer { get; set; }
        public int TimeOut { get; set; }
        public ReturnCode ReturnCode { get; set; }
        public bool IsSuccess { get; private set; }
        public DnsResolver(string url, string dnsServer = "223.5.5.5", int timeOut = 200)
        {
            Url = url;
            DnsServer = dnsServer;
            TimeOut = timeOut;
            Record=new List();
            Dig();
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
                foreach (var dnsRecord in dnsMessage.AnswerRecords)
                {
                    var aRecord = dnsRecord as ARecord;
                    if (aRecord == null) continue;
                    IsSuccess = true;
                    Record.Add(aRecord);
                }
            if (dnsMessage != null) ReturnCode = dnsMessage.ReturnCode;
        }
    }
}
```
### 调用方法
```csharp
DnsResolver dns = new DnsResolver("shaoweicloud.cn");
if (dns.IsSuccess)
    Console.WriteLine(dns.Record[0]);
```

## 结束
 至此，DNS解析模块就基本结束了，至于为什么标题中标注了半成品，是因为我想在基本的DNS解析功能的基础上根据解析到DNS信息中的TTL做一套信息缓存机制，减少不必要的重复查询，目前还在考虑使用何种方法，后续实现会更新。