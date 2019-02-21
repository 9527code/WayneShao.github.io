---
title: Consul 服务治理发现时实践
abbrlink: 15089
date: 2019-02-21 23:11:56
author: 玮仔Wayne
tags:
  - Consul
  - HashiCorp
  - 服务治理发现
  - .NET Core
categories:
  - 学习笔记
---
## Consul 服务治理发现
### 简介

　　Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其他分布式服务注册与发现的方案，Consul的方案更“**一站式**”，内置了服务注册与发现框架、分布一致性协议实现、健康检查、Key/Value存储、多数据中心方案，不再需要依赖其他工具（比如ZooKeeper等）。使用起来也较为简单。Consul使用Go语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与Docker等轻量级容器可无缝配合 。
* **service discovery**：consul通过DNS或者HTTP接口使服务注册和服务发现变的很容易，一些外部服务，例如saas提供的也可以一样注册。
* **health checking**：健康检测使consul可以快速的告警在集群中的操作。和服务发现的集成，可以防止服务转发到故障的服务上面。
* **key/value storage**：一个用来存储动态配置的系统。提供简单的HTTP接口，可以在任何地方操作。
* **multi-datacenter*：无需复杂的配置，即可支持任意数量的区域。

　　Consul 是注册中心，服务提供者、服务消费者等都要注册到 Consul 中，这样就可以实现服务提供者、服务消费者的隔离。
　　除了 Consul 之外，还有 Eureka、Zookeeper、Etcd 等类似服务发现框架。
### Consul 相关概念
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190220082541153.png)
* **CLIENT**
CLIENT表示consul的client模式，就是客户端模式。是consul节点的一种模式，这种模式下，所有注册到当前节点的服务会被转发到SERVER，本身是不持久化这些信息。

* **SERVER**
SERVER表示consul的server模式，表明这个consul是个server，这种模式下，功能和CLIENT都一样，唯一不同的是，它会把所有的信息持久化的本地，这样遇到故障，信息是可以被保留的。

* **SERVER-LEADER**
中间那个SERVER下面有LEADER的字眼，表明这个SERVER是它们的老大，它和其它SERVER不一样的一点是，它需要负责同步注册的信息给其它的SERVER，同时也要负责各个节点的健康监测。

* **其它信息**
其它信息包括它们之间的通信方式，还有一些协议信息，算法。它们是用于保证节点之间的数据同步，实时性要求等等一系列集群问题的解决。这些有兴趣的自己看看官方文档。

### 我的理解
　　Consul 的整体功能其实就类似于互联网中的 DNS 服务器，Consul 服务端根据客户端传入的服务名，返回所有提供该服务的地址，整个流程跟 DNS 服务器将域名转化为 IP 地址有异曲同工之妙。

## Consul 在 .NET Core 下的实践
　　我是用的开发平台是 Windows 64bit，鉴于 Windows 使用 Docker 的繁琐和诸多问题，以下流程直接下载运行 Consul 而不使用 Docker。
### 下载安装运行

1. 从[Consul 下载页面](https://www.consul.io/downloads.html)下载对应平台的最新版本的 Consul 程序并解压。
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190220085150466.png)
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190220085227813.png)
2. 运行 **consul.exe agent -dev** （使用开发模式进行测试，如需生产环境集群使用，只要需要一台 Server，多台 Agent）
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190220085306129.png)
3. 访问自带的 Web 后台查看即时信息
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190220085443153.png)

### .NET Core 下的 Consul 实践
   > Install-Package Consul

　　**程序与 Consul 的交互主要有三种：**
* 服务注册
* 服务查询
* 服务健康检查

#### 服务注册与反注册
##### 随机获取一个可用端口
```csharp
using System;
using System.Linq;
using System.Net.NetworkInformation;

namespace ConsulDemo.Extensions
{
    public static class PortHelper
    {
        /// <summary>
        /// 产生一个随机可用端口
        /// </summary> /// <param name="minPort"></param>
        /// <param name="maxPort"></param>
        /// <returns></returns>
        public static int GetRandAvailablePort(int minPort = 1024, int maxPort = 65535)
        {
            var rand = new Random();
            while (true)
            {
                var port = rand.Next(minPort, maxPort);
                if (!IsPortInUsed(port))
                    return port;
            }
        }


        /// <summary>
        /// 判断端口是否在使用中
        /// </summary>
        /// <param name="port"></param>
        /// <returns></returns>
        public static bool IsPortInUsed(int port) =>
            IPGlobalProperties.GetIPGlobalProperties().GetActiveTcpListeners().Any(p => p.Port == port) ||
            IPGlobalProperties.GetIPGlobalProperties().GetActiveUdpListeners().Any(p => p.Port == port) ||
            IPGlobalProperties.GetIPGlobalProperties().GetActiveTcpConnections().Any(conn => conn.LocalEndPoint.Port == port);
    }
}
```
##### Program 增加属性 **CurrentPort**
```csharp
public static IWebHostBuilder using ConsulDemo.Extensions;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;

namespace ConsulDemo
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }

        public static int CurrentPort { get; set; }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args) => 
            WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .UseUrls($"http://*:{CurrentPort = PortHelper.GetRandAvailablePort()}");
    }
}

```
##### 将注册与反注册的方法绑定到生命周期的开始和结束
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, IApplicationLifetime lifetime)
{
    if (env.IsDevelopment())
        app.UseDeveloperExceptionPage();


    app.UseMvc();


    const string ip = "127.0.0.1";
    var port = Program.CurrentPort;
    var serviceID = $"ConsulDemo_{Environment.TickCount}";

    //Consul 客户端
    var client = new ConsulClient((obj) =>
    {
        obj.Address = new Uri("http://127.0.0.1:8500");
        obj.Datacenter = "dc1";
    });

    // 在生命周期开始时注册服务
    lifetime.ApplicationStarted.Register(() =>
    {
        var result = client.Agent.ServiceRegister(new AgentServiceRegistration
        {
            ID = serviceID,
            Name = "ConsulDemo",
            Address = ip,
            Port = port,
            Check = new AgentServiceCheck
            {
                DeregisterCriticalServiceAfter = TimeSpan.FromSeconds(5),
                Interval = TimeSpan.FromSeconds(5),
                HTTP = $"http://{ip}:{port}/api/values/0",
                Timeout = TimeSpan.FromSeconds(5)
            }
        });

        Console.WriteLine($"Consul-ServiceRegister:{result.Result.StatusCode} - {result.Result.RequestTime}");
    });

    // 在生命周期结束时反注册服务
    lifetime.ApplicationStopping.Register(() =>
    {
        var result = client.Agent.ServiceDeregister(serviceID);
        Console.WriteLine($"Consul-ServiceDeregister:{result.Result.StatusCode} - {result.Result.RequestTime}");
    });
}
```
##### 结果视图
启动五个服务提供程序，图中可以看得到连接成功后的打印以及 Consul 健康检查得请求日志。
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221072754356.png)
从 Consul 的后台可以清楚的看到已经成功注册了五个 ConsulDemo 服务提供程序。
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221073245220.png)
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221073230199.png)

#### 服务消费程序
##### 首先我们新建一个 RestTemplate
RestTemplate（模仿 Spring Cloud 中的）
```csharp
using Consul;
using Newtonsoft.Json;
using System;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;

namespace ConsulTemplate
{
    public class RestTemplate
    {
        private readonly string _consulServerUrl;
        public RestTemplate(string consulServerUrl = "http://127.0.0.1:8500")
        {
            _consulServerUrl = consulServerUrl;
        }
        /// <summary>
        /// 获取服务的第一个实现地址
        /// </summary>
        /// <param name="serviceName"></param>
        /// <returns></returns>
        private async Task<string> ResolveRootUrlAsync(string serviceName)
        {
            using (var consulClient = new ConsulClient(c => c.Address = new Uri(_consulServerUrl)))
            {
                var services = (await consulClient.Agent.Services()).Response;

                Console.WriteLine("当前所有服务");
                foreach (var service in services)
                    Console.WriteLine($"Service:{service.Value.Service}Address:{service.Value.Address}Port:{service.Value.Port}");

                var agentServices = services.Where(s => s.Value.Service.Equals(serviceName, StringComparison.CurrentCultureIgnoreCase)).Select(s => s.Value).ToArray();

                

                //根据当前TickCount对服务器个数取模，“随机”取一个机器出来，避免“轮询”的负载均衡策略需要计数加锁问题
                var agentService = agentServices.ElementAt(Environment.TickCount % agentServices.Length);

                 Console.WriteLine($"随机选取 Service:{agentService.Service}Address:{agentService.Address}Port:{agentService.Port}");
                return agentService.Address + ":" + agentService.Port;
            }
        }

        /// <summary>
        /// 转化到实际接口地址
        /// </summary>
        /// <param name="url"></param>
        /// <returns></returns>
        private async Task<string> ResolveUrlAsync(string url)
        {
            var uri = new Uri(url);
            var serviceName = uri.Host;
            var realRootUrl = await ResolveRootUrlAsync(serviceName);
            return uri.Scheme + "://" + realRootUrl + uri.PathAndQuery;
        }
        public async Task<ResponseEntity<T>> GetForEntityAsync<T>(string url, HttpRequestHeaders requestHeaders = null)
        {
            using (var httpClient = new HttpClient())
            {
                var requestMsg = new HttpRequestMessage();
                if (requestHeaders != null)
                    foreach (var header in requestHeaders)
                        requestHeaders.Add(header.Key, header.Value);

                requestMsg.Method = HttpMethod.Get;
                requestMsg.RequestUri = new Uri(await ResolveUrlAsync(url));
                var result = await httpClient.SendAsync(requestMsg);
                var respEntity = new ResponseEntity<T>
                {
                    StatusCode = result.StatusCode
                };
                var bodyStr = await result.Content.ReadAsStringAsync();
                respEntity.Body = JsonConvert.DeserializeObject<T>(bodyStr);
                respEntity.Headers = respEntity.Headers;
                return respEntity;
            }
        }
    }
}

public class ResponseEntity<T>
{
    public HttpStatusCode StatusCode { get; set; }
    public T Body { get; set; }//返回的json反序列化出来的对象
    public HttpResponseHeaders Headers { get; set; }//响应的报文头
}

```

##### 调用代码
```csharp
static void Main(string[] args)
{
    while (true)
    {
        Console.WriteLine("请求开始");
        var rest = new RestTemplate();
        var data = rest.GetForEntityAsync<DateTime>("http://ConsulDemo/api/Values").Result;
        Console.WriteLine(data.StatusCode);
        Console.WriteLine(string.Join(",", data.Body));
        Console.WriteLine("请求结束\r\n\r\n");
        Console.ReadKey();
    }
}
```
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221075743368.png)

手动关掉一个服务端

![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221080109388.png)
关掉的程序成功消失(反注册)
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221080152025.png)
资源管理器杀掉一个程序
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221080431270.png)
![](http://qiniucdn.wayneshao.com/Consul-服务治理发现/20190221080513331.png)

源码 [ConsulDemo.7z](http://qiniucdn.wayneshao.com/ConsulDemo.7z)