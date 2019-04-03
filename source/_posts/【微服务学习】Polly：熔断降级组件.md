---
title: 【微服务学习】Polly：熔断降级组件
abbrlink: 15090
date: 2019-03-04 23:31:24
author: 玮仔Wayne
tags:
  - Polly
  - 熔断降级
  - 微服务
  - .NET
categories:
  - 学习笔记
---

## 何为熔断降级
　　“熔断器如同电力过载保护器。它可以实现快速失败，如果它在一段时间内侦测到许多类似的错误，会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费时间去等到长时间的超时产生。”
　　降级的目的是当某个服务提供者发生故障的时候，向调用方返回一个替代响应。
　　简单一句话概括，降级就是在调用的下游服务A出现问题（常见超时），提供PLAN-B，返回的效果可能没有服务A好，但是聊胜于无。而熔断器的存在就是要保障何时走到降级方法，何时恢复，以什么样的策略恢复。

<!--more-->
## .NET Core 熔断降级实践

### 简介
　　Polly是一种.NET弹性和瞬态故障处理库，允许我们以非常顺畅和线程安全的方式来执诸如行重试，断路，超时，故障恢复等策略。 
　　Polly当前版本可以工作在 .NET Standard 1.1 (包括: .NET Framework 4.5-4.6.1, .NET Core 1.0, Mono, Xamarin, UWP, WP8.1+) 和 .NET Standard 2.0+ (包括: .NET Framework 4.6.1, .NET Core 2.0+, 新版本的 Mono, Xamarin and UWP targets).上，同时也为旧版本的.NET Framework提供了一些可用的旧版本，具体版本对应如下：
  
| 目标框架 | 最低适配版本 | 最高适配版本 |
| --- | --- | --- |
| .NET&nbsp;Standard&nbsp;2.1<br/>for use with IHttpClientFactory | [6.0.1](https://www.nuget.org/packages/Polly/6.0.1) | [Current](https://www.nuget.org/packages/polly) |
| .NET&nbsp;Standard&nbsp;2.0 (dedicated&nbsp;target) | [6.0.1](https://www.nuget.org/packages/Polly/6.0.1) | [Current](https://www.nuget.org/packages/polly) |
| .NET&nbsp;Standard&nbsp;2.0 | [5.0.3](https://www.nuget.org/packages/Polly/5.0.3) via .Net&nbsp;Standard&nbsp;1.0 (upward compatible) | [Current](https://www.nuget.org/packages/polly) |
| .NET&nbsp;Standard&nbsp;1.1 | [5.0.3](https://www.nuget.org/packages/Polly/5.0.3) via  .Net&nbsp;Standard&nbsp;1.0 (upward compatible) | [Current](https://www.nuget.org/packages/polly) |
| .NET&nbsp;Standard&nbsp;1.0 | [5.0.3](https://www.nuget.org/packages/Polly/5.0.3)  | [5.1.0](https://www.nuget.org/packages/Polly/5.1.0) |
| .NET&nbsp;Framework&nbsp;4.5 | [1.0.0](https://www.nuget.org/packages/Polly/1.0.0)  | [Current](https://www.nuget.org/packages/polly)&nbsp;(via&nbsp;.Net&nbsp;Standard);<br/>[5.9.0](https://www.nuget.org/packages/Polly/5.9.0)&nbsp;(as&nbsp;dedicated&nbsp;target) |
| .NET&nbsp;Framework&nbsp;4.0<br/>with Async support | [4.2.2](https://www.nuget.org/packages/Polly.Net40Async/4.2.2)  | [5.9.0](https://www.nuget.org/packages/Polly.Net40Async/5.9.0)|
| .NET&nbsp;Framework&nbsp;4.0 | [1.0.0](https://www.nuget.org/packages/Polly/1.0.0)  | [5.9.0](https://www.nuget.org/packages/Polly.Net40Async/5.9.0)|
| .NET&nbsp;Framework&nbsp;3.5 | [1.0.0](https://www.nuget.org/packages/Polly/1.0.0)  | [4.3.0](https://www.nuget.org/packages/Polly/4.3.0) |
| Various PCL targets | [2.0.0](https://www.nuget.org/packages/Polly/2.0.0)  | [Current](https://www.nuget.org/packages/polly)&nbsp;(via&nbsp;.Net&nbsp;Standard);<br/>[4.3.0](https://www.nuget.org/packages/Polly/4.3.0)&nbsp;(as&nbsp;dedicated&nbsp;PCL&nbsp;target) |

　　该项目作者现已成为.NET基金会一员，项目一直在不停迭代和更新，项目地址[【https://github.com/App-vNext/Polly】](https://github.com/App-vNext/Polly)。

### 七种恢复策略

|策略| 前置条件 | 例 | 此策略解决什么问题?|
| ------------- | ------------- |:-------------: |------------- |
|**重试策略（Retry）** <br/>(policy family)<br/><sub>([快速开始](#retry)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/Retry))</sub>|重试策略针对的前置条件是短暂的故障延迟且在短暂的延迟之后能够自我纠正。| "也许这只是昙花一现" |  允许我们做的是能够自动配置重试机制。 |
|**断路器（Circuit-breaker）**<br/>(policy family)<br/><sub>([快速开始](#circuit-breaker)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/Circuit-Breaker))</sub>|断路器策略针对的前置条件是当系统繁忙时，快速响应失败总比让用户一直等待更好。 <br/><br/>保护系统故障免受过载，Polly可以帮其恢复。  | "痛了，自然就会放下" <br/><br/>"让它歇一下" | 当故障超过某个预先配置的阈值时, 中断电路 (块执行) 一段时间。 | 
|**超时（Timeout）**<br/><sub>([快速开始](#timeout)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/Timeout))</sub>|超时策略针对的前置条件是超过一定的等待时间，想要得到成功的结果是不可能的。| "你不必等待，她不会再来"  | 保证调用者不必等待太长时间。| 
|**隔板隔离（Bulkhead Isolation）**<br/><sub>([快速开始](#bulkhead)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/Bulkhead))</sub>|隔板隔离针对的前置条件是当进程出现故障时，多个失败一直在主机中对资源（例如线程/ CPU）一直占用。下游系统故障也可能导致上游失败。<br/><br/>这两个风险都将造成严重的后果。 | "一颗老鼠屎坏了一锅汤"  |将受管制的操作限制在固定的资源池中，避免其他资源受其影响。 |
|**缓存（Cache）**<br/><sub>([快速开始](#cache)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/Cache))</sub>|数据不会很频繁的进行更新，相同请求的响应是相似的。| "听说 <br/>你还会再来<br/>我翘首以盼"  |首次加载数据时将响应数据进行缓存，请求时若缓存中存在则直接从缓存中读取。 |
|**回退（Fallback）**<br/><sub>([快速开始](#fallback)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/Fallback))</sub>|操作将仍然失败 - 但是你可以实现准备好失败后要做的补救措施。| "你若安好，我备胎到老。"  |定义失败时要返回 (或要执行的操作) 的替代值。. | 
|**策略包装（PolicyWrap）**<br/><sub>([快速开始](#policywrap)&nbsp;;&nbsp;[深入学习](https://github.com/App-vNext/Polly/wiki/PolicyWrap))</sub>|不同的故障需要不同的策略，也就意味着弹性灵活使用组合。| "谋定而后动"  |允许灵活地组合上述任何策略。 | 


## 实践

### 故障处理（被动策略）
故障处理策略处理通过策略执行的代码所引发的特定的异常或返回结果。
#### 第一步：指定希望处理的异常（可选-指定要处理的返回结果）
##### 指定希望处理的异常：
```csharp
// 单一异常种类
Policy
  .Handle<HttpRequestException>()

// 带条件判断的单一异常
Policy
  .Handle<SqlException>(ex => ex.Number == 1205)

// 多种异常
Policy
  .Handle<HttpRequestException>()
  .Or<OperationCanceledException>()

// 带条件判断的多种异常
Policy
  .Handle<SqlException>(ex => ex.Number == 1205)
  .Or<ArgumentException>(ex => ex.ParamName == "example")

// 普通异常或聚合异常的内部异常, 可以带有条件
Policy
  .HandleInner<HttpRequestException>()
  .OrInner<OperationCanceledException>(ex => ex.CancellationToken != myToken)
```
##### [指定要处理的返回结果](https://github.com/App-vNext/Polly#handing-return-values-and-policytresult)
从Polly v4.3.0起，包含返回TResult的调用的策略也可以处理TResult返回值
```csharp
// 带条件判断的单种返回值处理
Policy
  .HandleResult<HttpResponseMessage>(r => r.StatusCode == HttpStatusCode.NotFound)

// 带条件判断的多种返回值处理
Policy
  .HandleResult<HttpResponseMessage>(r => r.StatusCode == HttpStatusCode.InternalServerError)
  .OrResult(r => r.StatusCode == HttpStatusCode.BadGateway)

// 原始返回值处理 (隐式调用 .Equals())
Policy
  .HandleResult<HttpStatusCode>(HttpStatusCode.InternalServerError)
  .OrResult(HttpStatusCode.BadGateway)
 
// 在一个策略中同时处理异常和返回值
HttpStatusCode[] httpStatusCodesWorthRetrying = {
   HttpStatusCode.RequestTimeout, // 408
   HttpStatusCode.InternalServerError, // 500
   HttpStatusCode.BadGateway, // 502
   HttpStatusCode.ServiceUnavailable, // 503
   HttpStatusCode.GatewayTimeout // 504
}; 
HttpResponseMessage result = await Policy
  .Handle<HttpRequestException>()
  .OrResult<HttpResponseMessage>(r => httpStatusCodesWorthRetrying.Contains(r.StatusCode))
  .RetryAsync(...)
  .ExecuteAsync( /* Func<Task<HttpResponseMessage>> */ )
```

#### 第二步：指定策略应如何处理这些错误

##### 重试（Retry）
```csharp
// 重试一次
Policy
  .Handle<SomeExceptionType>()
  .Retry()

// 重试多次
Policy
  .Handle<SomeExceptionType>()
  .Retry(3)

// 重试多次，每次重试触发事件（参数为此次异常和当前重试次数）
Policy
    .Handle<SomeExceptionType>()
    .Retry(3, onRetry: (exception, retryCount) =>
    {
        // do something 
    });

// 重试多次，每次重试触发事件（参数为此次异常、当前重试次数和当前执行的上下文）
Policy
    .Handle<SomeExceptionType>()
    .Retry(3, onRetry: (exception, retryCount, context) =>
    {
        // do something 
    });
```

##### 不断重试直到成功（Retry forever until succeeds）
```csharp
// 不断重试
Policy
  .Handle<SomeExceptionType>()
  .RetryForever()

// 不断重试，每次重试触发事件（参数为此次异常） 
Policy
  .Handle<SomeExceptionType>()
  .RetryForever(onRetry: exception =>
  {
        // do something       
  });

// 不断重试，每次重试触发事件（参数为此次异常和当前执行的上下文）
Policy
  .Handle<SomeExceptionType>()
  .RetryForever(onRetry: (exception, context) =>
  {
        // do something       
  });
```

##### 等待并重试（Wait and retry）
[WaitAndRetry策略处理HTTP状态代码429的重试后状态](https://github.com/App-vNext/Polly/wiki/Retry#retryafter-when-the-response-specifies-how-long-to-wait)
```csharp
// 重试多次, 每次重试之间等待指定的持续时间。(失败之后触发等待, 然后再进行下一次尝试。)
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  });

// 重试并触发事件多次, 每次重试之间等待指定的持续时间。（事件参数为当前异常和时间间隔）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  }, (exception, timeSpan) => {
    // do something    
  }); 

// 重试并触发事件多次, 每次重试之间等待指定的持续时间。（事件参数为当前异常、时间间隔和当前执行的上下文）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  }, (exception, timeSpan, context) => {
    // do something    
  });

// 重试并触发事件多次, 每次重试之间等待指定的持续时间。（事件参数为当前异常、时间间隔、当前重试次数和当前执行的上下文）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  }, (exception, timeSpan, retryCount, context) => {
    // do something    
  });

// 重试指定的次数, 根据当前重试次数计算等待时间 (允许指数回退)
// 当前这种情况下, 等待时间为:
//  2 ^ 1 = 2 s
//  2 ^ 2 = 4 s
//  2 ^ 3 = 8 s
//  2 ^ 4 = 16 s
//  2 ^ 5 = 32 s
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(5, retryAttempt => 
	TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)) 
  );

// 重试指定的次数，每次重试时触发事件，根据当前重试次数计算等待时间。（事件参数为当前异常、时间间隔和当前执行的上下文）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(
    5, 
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)), 
    (exception, timeSpan, context) => {
      // do something
    }
  );

// 重试指定的次数，每次重试时触发事件，根据当前重试次数计算等待时间。（事件参数为当前异常、时间间隔、当前重试次数和当前执行的上下文）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(
    5, 
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)), 
    (exception, timeSpan, retryCount, context) => {
      // do something
    }
  );
```
##### 不断等待并重试直到成功（Wait and retry forever until succeeds）
[如果所有重试都失败, 重试策略将重新引发最后一个异常返回到调用代码。](https://github.com/App-vNext/Polly/wiki/Retry)
```csharp
// 不断等待并重试
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetryForever(retryAttempt => 
	TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
    );

// 不断等待并重试，每次重试时触发事件。（事件参数为当前异常、时间间隔）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetryForever(
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),    
    (exception, timespan) =>
    {
        // do something       
    });


// 不断等待并重试，每次重试时触发事件。（事件参数为当前异常、时间间隔和当前执行的上下文）
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetryForever(
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),    
    (exception, timespan, context) =>
    {
        // do something       
    });
```

##### 断路器（Circuit-breaker）
断路器策略通过在程序出错时抛出**BrokenCircuitException**来屏蔽其他异常。[文档](https://github.com/App-vNext/Polly/wiki/Circuit-Breaker)
请注意, 断路器策略将[重新引发所有异常](https://github.com/App-vNext/Polly/wiki/Circuit-Breaker#exception-handling), 甚至是已处理的异常。所以使用时通常会将**重试策略**和断路器策略**组合使用**。

```csharp
// 在指定数量的连续异常后断开程序执行并在之后的一段时间内保持程序执行断开。
Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(2, TimeSpan.FromMinutes(1));

// 在指定数量的连续异常后断开程序执行并在之后的一段时间内保持程序执行断开。当程序执行断开或者重新启用时触发事件。（程序执行断开事件参数为当前异常和间隔时间，重新启用事件无参数）
Action<Exception, TimeSpan> onBreak = (exception, timespan) => { ... };
Action onReset = () => { ... };
CircuitBreakerPolicy breaker = Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(2, TimeSpan.FromMinutes(1), onBreak, onReset);

// 在指定数量的连续异常后断开程序执行并在之后的一段时间内保持程序执行断开。当程序执行断开或者重新启用时触发事件。（程序执行断开事件参数为当前异常、间隔时间和当前执行上下文，重新启用事件参数为当前执行上下文）
Action<Exception, TimeSpan, Context> onBreak = (exception, timespan, context) => { ... };
Action<Context> onReset = context => { ... };
CircuitBreakerPolicy breaker = Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(2, TimeSpan.FromMinutes(1), onBreak, onReset);

// 程序运行状态, 运行状况。
CircuitState state = breaker.CircuitState;

/*
CircuitState.Closed - 断路器未触发，允许操作执行。
CircuitState.Open - 断路器开启，阻止操作执行。
CircuitState.HalfOpen - 断路器开启指定时间后重新关闭，此状态允许操作执行，之后的开启或关闭取决于继续执行的结果。
CircuitState.Isolated - 断路器被主动开启，阻止操作执行。
*/

// 手动打开 (并保持打开) 断路器（例如需要主动隔离下游服务时）
breaker.Isolate(); 
// 重置断路器为关闭状态, 再次开始允许操作执行。
breaker.Reset(); 
````
##### 高级断路器（Advanced Circuit Breaker）
```csharp
// 在采样持续时间内, 如果已处理异常的操作的比例超过故障阈值且该时间段内通过请求操作数达到最小吞吐量，主动启动断路器。

Policy
    .Handle<SomeExceptionType>()
    .AdvancedCircuitBreaker(
        failureThreshold: 0.5, // 当>=50%的操作会导致已处理的异常时中断程序。
        samplingDuration: TimeSpan.FromSeconds(10), // 采样时间区间为10秒
        minimumThroughput: 8, // ... 在采样时间区间内进行了至少8次操作。
        durationOfBreak: TimeSpan.FromSeconds(30) // 断路30秒.
                );

// 采用状态更改委托的配置重载同样可用于高级断路器。

// 电路状态监控和手动控制同样也可用于高级断路器。
```

更多相关资料请参考: [文档](https://github.com/App-vNext/Polly/wiki/Advanced-Circuit-Breaker)

有关断路器模式的更多信息, 请参见：
* [**改造 Netflix API 增加接口弹性**](http://techblog.netflix.com/2011/12/making-netflix-api-more-resilient.html)
* [**断路器浅谈 (马丁·福勒)**](http://martinfowler.com/bliki/CircuitBreaker.html)
* [**断路器模式 (Microsoft)**](https://msdn.microsoft.com/en-us/library/dn589784.aspx)
* [**原始断路器链**](https://web.archive.org/web/20160106203951/http://thatextramile.be/blog/2008/05/the-circuit-breaker)

##### [回退策略（Fallback）](https://github.com/App-vNext/Polly/wiki/Fallback)
```csharp
// 执行错误时提供替代值。
Policy
   .Handle<Whatever>()
   .Fallback<UserAvatar>(UserAvatar.Blank)

// 执行错误时使用回调函数提供替代值。
Policy
   .Handle<Whatever>()
   .Fallback<UserAvatar>(() => UserAvatar.GetRandomAvatar()) // where: public UserAvatar GetRandomAvatar() { ... }

// 执行错误时提供替代值的同时触发事件。（事件参数为当前异常信息和当前运行上下文）
Policy
   .Handle<Whatever>()
   .Fallback<UserAvatar>(UserAvatar.Blank, onFallback: (exception, context) => 
    {
        // do something
    });
```
#### 第三步：执行策略
```csharp
// 执行操作
var policy = Policy
              .Handle<SomeExceptionType>()
              .Retry();

policy.Execute(() => DoSomething());

// 执行传递任意上下文数据的操作
var policy = Policy
    .Handle<SomeExceptionType>()
    .Retry(3, (exception, retryCount, context) =>
    {
        var methodThatRaisedException = context["methodName"];
		Log(exception, methodThatRaisedException);
    });

policy.Execute(
	() => DoSomething(),
	new Dictionary<string, object>() {{ "methodName", "some method" }}
);

// 执行返回结果的函数
var policy = Policy
              .Handle<SomeExceptionType>()
              .Retry();

var result = policy.Execute(() => DoSomething());

// 执行传递任意上下文数据且返回结果的操作
var policy = Policy
    .Handle<SomeExceptionType>()
    .Retry(3, (exception, retryCount, context) =>
    {
        object methodThatRaisedException = context["methodName"];
        Log(exception, methodThatRaisedException)
    });

var result = policy.Execute(
    () => DoSomething(),
    new Dictionary<string, object>() {{ "methodName", "some method" }}
);

// 综合使用
Policy
  .Handle<SqlException>(ex => ex.Number == 1205)
  .Or<ArgumentException>(ex => ex.ParamName == "example")
  .Retry()
  .Execute(() => DoSomething());
```
为了简单起见, 上面的示例显示了策略定义, 然后是策略执行。
但是在代码库和应用程序生命周期中, 策略定义和执行可能同样经常被**分离**。
例如, 可以选择在启动时定义策略, 然后通过**依赖注入**将其提供给使用点。
### 故障处理（主动策略）
主动策略添加了不基于当策略被引发或返回时才处理错误的弹性策略。
#### 第一步：配置
##### 超时（Timeout）
###### 乐观超时（Optimistic timeout）
乐观超时[通过 CancellationToken 运行](https://github.com/App-vNext/Polly/wiki/Timeout#optimistic-timeout), 并假定您执行支持合作取消的委托。您必须使用 `Execute/Async(...)` 重载以获取 `CancellationToken`, 并且执行的委托必须遵守该 `CancellationToken`。
```csharp
// 如果执行的委托尚未完成，在调用30秒后超时并返回 。 乐观超时: 委托应采取并遵守 CancellationToken。
Policy
  .Timeout(30)

// 使用 TimeSpan 配置超时。
Policy
  .Timeout(TimeSpan.FromMilliseconds(2500))

// 通过方法提供可变的超时。
Policy
  .Timeout(() => myTimeoutProvider)) // Func<TimeSpan> myTimeoutProvider

// 超时后触发事件。（事件参数为当前执行上下文、执行间隔、当前执行的TASK）
Policy
  .Timeout(30, onTimeout: (context, timespan, task) => 
    {
        // do something 
    });

// 示例：在超时后记录日志
Policy
  .Timeout(30, onTimeout: (context, timespan, task) => 
    {
        logger.Warn($"{context.PolicyKey} at {context.ExecutionKey}: execution timed out after {timespan.TotalSeconds} seconds.");
    });

// 示例：在超时任务完成时捕获该任务中的任何异常
Policy
  .Timeout(30, onTimeout: (context, timespan, task) => 
    {
        task.ContinueWith(t => {
            if (t.IsFaulted) logger.Error($"{context.PolicyKey} at {context.ExecutionKey}: execution timed out after {timespan.TotalSeconds} seconds, with: {t.Exception}.");
        });
    });
```

示例执行:
```csharp
Policy timeoutPolicy = Policy.TimeoutAsync(30);
HttpResponseMessage httpResponse = await timeoutPolicy
    .ExecuteAsync(
      async ct => await httpClient.GetAsync(endpoint, ct), // 执行一个有参数且响应 CancellationToken 的委托。
      CancellationToken.None // 在这种情况下, CancellationToken.None 将被传递到执行中, 这表明您没有将期望的令牌控制通过超时策略添加。 自定义 CancellationToken 也可以通过，详情请参阅 wiki 中的例子。
      );
```
###### 悲观超时（Pessimistic timeout）
悲观超时允许调用代码 "离开" 等待执行完成的委托, 即使它不支持取消。
在同步执行中, 这是以牺牲一个额外的线程为代价的。有关更多细节, 请参见[文档](https://github.com/App-vNext/Polly/wiki/Timeout#pessimistic-timeout)。
示例执行:
```csharp
Policy timeoutPolicy = Policy.TimeoutAsync(30, TimeoutStrategy.Pessimistic);
var response = await timeoutPolicy
    .ExecuteAsync(
      async () => await FooNotHonoringCancellationAsync(), // 执行不接受取消令牌且不响应取消的委托。
      );
```
超时策略在发生超时时引发 `TimeoutRejectedException`。
更多详情参见[文档](https://github.com/App-vNext/Polly/wiki/Timeout)。
##### 隔板（Bulkhead）
```csharp
// 通过该策略将执行限制为最多12个并发操作。
Policy
  .Bulkhead(12)

// 将通过策略执行的操作限制为最多12个并发操作, 如果插槽都被占满, 最多可以有两个操作被等待执行。
Policy
  .Bulkhead(12, 2)

// 限制并发执行, 如果执行被拒绝, 则调用触发事件。（事件参数为当前执行上下文）
Policy
  .Bulkhead(12, context => 
    {
        // do something 
    });

// 查看隔板可用容量, 例如健康负荷。
var bulkhead = Policy.Bulkhead(12, 2);
// ...
int freeExecutionSlots = bulkhead.BulkheadAvailableCount;
int freeQueueSlots     = bulkhead.QueueAvailableCount;
```

当隔板策略的插槽全部被正在执行的操作占满是，会引发 `BulkheadRejectedException`。
更多详情参见[文档](https://github.com/App-vNext/Polly/wiki/Bulkhead)。

##### 缓存(Cache)
```csharp
var memoryCache = new MemoryCache(new MemoryCacheOptions());
var memoryCacheProvider = new MemoryCacheProvider(memoryCache);
var cachePolicy = Policy.Cache(memoryCacheProvider, TimeSpan.FromMinutes(5));

// .NET Core CacheProviders DI 示例 请参照以下文章  https://github.com/App-vNext/Polly/wiki/Cache#working-with-cacheproviders :
// - https://github.com/App-vNext/Polly.Caching.MemoryCache
// - https://github.com/App-vNext/Polly.Caching.IDistributedCache 

// 定义每天午夜绝对过期的缓存策略。
var cachePolicy = Policy.Cache(memoryCacheProvider, new AbsoluteTtl(DateTimeOffset.Now.Date.AddDays(1));

// 定义超时过期的缓存策略: 每次使用缓存项时, 项目的有效期为5分钟。
var cachePolicy = Policy.Cache(memoryCacheProvider, new SlidingTtl(TimeSpan.FromMinutes(5));

// 定义缓存策略, 并捕获任何缓存提供程序错误以进行日志记录。
var cachePolicy = Policy.Cache(myCacheProvider, TimeSpan.FromMinutes(5), 
   (context, key, ex) => { 
       logger.Error($"Cache provider, for key {key}, threw exception: {ex}."); // (for example) 
   }
);

// 以直通缓存的身份执行缓存: 首先检查缓存;如果未找到, 请执行基础委托并将结果存储在缓存中。 
// 用于特定执行的缓存的键是通过在传递给执行的上下文实例上设置操作键 (v6 之前: 执行键) 来指定的。使用下面显示的窗体的重载 (或包含相同元素的更丰富的重载)。
// 示例: "fookey" 是将在下面的执行中使用的缓存密钥。
TResult result = cachePolicy.Execute(context => getFoo(), new Context("FooKey"));
```
有关使用其他缓存提供程序的更丰富的选项和详细信息, 请参阅:[文档](https://github.com/App-vNext/Polly/wiki/Cache)

##### 策略包装（PolicyWrap）
```csharp
// 定义由以前定义的策略构建的组合策略。
var policyWrap = Policy
  .Wrap(fallback, cache, retry, breaker, timeout, bulkhead);
// (包装策略执行任何被包装的策略: fallback outermost ... bulkhead innermost)
policyWrap.Execute(...)

// 定义标准的弹性策略
PolicyWrap commonResilience = Policy.Wrap(retry, breaker, timeout);

// ... 然后包装在额外的策略特定于一个请求类型:
Avatar avatar = Policy
   .Handle<Whatever>()
   .Fallback<Avatar>(Avatar.Blank)
   .Wrap(commonResilience)
   .Execute(() => { /* get avatar */ });

// 共享通用弹性, 但将不同的策略包装在另一个请求类型中:
Reputation reps = Policy
   .Handle<Whatever>()
   .Fallback<Reputation>(Reputation.NotAvailable)
   .Wrap(commonResilience)
   .Execute(() => { /* get reputation */ });  
```
更多详情参见[文档](https://github.com/App-vNext/Polly/wiki/PolicyWrap)

##### 无策略(NoOp)
```csharp
// 定义一个策略, 该策略将简单地导致传递给执行的委托 "按原样" 执行。
// 适用于在单元测试中或在应用程序中可能需要策略, 但您只是希望在没有策略干预的情况下通过执行的应用程序。
NoOpPolicy noOp = Policy.NoOp();
```
更多详情参见[文档](https://github.com/App-vNext/Polly/wiki/NoOp)

#### 第二步：执行策略
[同上](#)

### 执行后：捕获结果或任何最终异常

使用 ExecuteAndCapture(...)  方法可以捕获执行的结果: 这些方法返回一个执行结果实例, 该实例描述的是成功执行还是错误。
```csharp
var policyResult = await Policy
              .Handle<HttpRequestException>()
              .RetryAsync()
              .ExecuteAndCaptureAsync(() => DoSomethingAsync());
/*              
policyResult.Outcome - 调用是成功还是失败         
policyResult.FinalException - 最后一个异常。如果调用成功, 则捕获的最后一个异常将为 null
policyResult.ExceptionType - 定义为要处理的策略的最后一个异常 (如上面的 HttpRequestException) 或未处理的异常  (如 Exception). 如果调用成功, 则为 null。
policyResult.Result - 如果执行 func, 调用成功则返回执行结果, 否则为类型的默认值
*/
```
### 处理返回值和 `Policy<TResult>`
如步骤1b 所述, 从 polly v4.3.0 开始, 策略可以组合处理返回值和异常:
```csharp
// 在一个策略中处理异常和返回值
HttpStatusCode[] httpStatusCodesWorthRetrying = {
   HttpStatusCode.RequestTimeout, // 408
   HttpStatusCode.InternalServerError, // 500
   HttpStatusCode.BadGateway, // 502
   HttpStatusCode.ServiceUnavailable, // 503
   HttpStatusCode.GatewayTimeout // 504
}; 
HttpResponseMessage result = await Policy
  .Handle<HttpRequestException>()
  .OrResult<HttpResponseMessage>(r => httpStatusCodesWorthRetrying.Contains(r.StatusCode))
  .RetryAsync(...)
  .ExecuteAsync( /* some Func<Task<HttpResponseMessage>> */ )
```
要处理的异常和返回结果可以以任意顺序流畅的表达。
#### 强类型 `Policy<TResult>`
`配置策略 .HandleResult<TResult>(...) 或.OrResult<TResult>(...) 生成特定强类型策略 Policy<TResult>，例如 Retry<TResult>, AdvancedCircuitBreaker<TResult>`。
这些策略必须用于执行返回 TResult 的委托, 即:
* `Execute(Func<TResult>) (and related overloads)`
* `ExecuteAsync(Func<CancellationToken, Task<TResult>>) (and related overloads)`

#### `ExecuteAndCapture<TResult>()`

.ExecuteAndCapture(...) 在非泛型策略上返回具有属性的 PolicyResult：
```csharp
policyResult.Outcome - 调用是成功还是失败         
policyResult.FinalException - 最后一个异常。如果调用成功, 则捕获的最后一个异常将为 null
policyResult.ExceptionType - 定义为要处理的策略的最后一个异常 (如上面的 HttpRequestException) 或未处理的异常  (如 Exception). 如果调用成功, 则为 null。
policyResult.Result - 如果执行 func, 调用成功则返回执行结果, 否则为类型的默认值
```

`.ExecuteAndCapture<TResult>(Func<TResult>) `在强类型策略上添加了两个属性:
```csharp
policyResult.FaultType - 最终的故障是处理异常还是由策略处理的结果？如果委托执行成功, 则为 null。
policyResult.FinalHandledResult - 处理的最终故障结果;如果调用成功将为空或类型的默认值。
```
#### `Policy<TResult>`策略的状态更改事件
在仅处理异常的非泛型策略中, 状态更改事件 (如 onRetry 和 onBreak ) 提供 Exception 参数。
在处理 TResult 返回值的通用性策略中, 状态更改委托是相同的, 除非它们采用 DelegateResult<TResult> 参数代替异常。DelegateResult<TResult>具有两个属性:
* **Exception** // 如果策略正在处理异常则为则刚刚引发异常(否则为空),
* **Result** // 如果策略正在处理结果则为刚刚引发的 TResult (否则为 default(TResult))

#### `BrokenCircuitException<TResult>`
非通用的循环断路器策略在断路时抛出一个BrokenCircuitException。此 BrokenCircuitException 包含最后一个异常 (导致中断的异常) 作为 InnerException。
关于 `CircuitBreakerPolicy<TResult>` 策略:
* 由于异常而中断将引发一个 BrokenCircuitException, 并将 InnerException 设置为触发中断的异常 (如以前一样)。
* 由于处理结果而中断会引发 '`BrokenCircuitException<TResult>`', 其 Result  属性设置为导致电路中断的结果.

### Policy Keys 与 Context data
```csharp
// 用扩展方法 WithPolicyKey() 使用 PolicyKey 识别策略, 
// (例如, 对于日志或指标中的相关性)

var policy = Policy
    .Handle<DataAccessException>()
    .Retry(3, onRetry: (exception, retryCount, context) =>
       {
           logger.Error($"Retry {retryCount} of {context.PolicyKey} at {context.ExecutionKey}, due to: {exception}.");
       })
    .WithPolicyKey("MyDataAccessPolicy");

// 在上下文中传递 ExecutionKey , 并使用 ExecutionKey 标识呼叫站点
var customerDetails = policy.Execute(myDelegate, new Context("GetCustomerDetails"));

// "MyDataAccessPolicy" -> context.PolicyKey 
// "GetCustomerDetails  -> context.ExecutionKey


// 将其他自定义信息从调用站点传递到执行上下文中 
var policy = Policy
    .Handle<DataAccessException>()
    .Retry(3, onRetry: (exception, retryCount, context) =>
       {
           logger.Error($"Retry {retryCount} of {context.PolicyKey} at {context.ExecutionKey}, getting {context["Type"]} of id {context["Id"]}, due to: {exception}.");
       })
    .WithPolicyKey("MyDataAccessPolicy");

int id = ... // 客户id
var customerDetails = policy.Execute(context => GetCustomer(id), 
    new Context("GetCustomerDetails", new Dictionary<string, object>() {{"Type","Customer"},{"Id",id}}
```

更多资料参考[文档](https://github.com/App-vNext/Polly/wiki/Keys-And-Context-Data)

### PolicyRegistry
```csharp
// 创建策略注册表 (例如在应用程序启动时) 
PolicyRegistry registry = new PolicyRegistry();

// 使用策略填充注册表
registry.Add("StandardHttpResilience", myStandardHttpResiliencePolicy);
// 或者:
registry["StandardHttpResilience"] = myStandardHttpResiliencePolicy;

// 通过 DI 将注册表实例传递给使用站点
public class MyServiceGateway 
{
    public void MyServiceGateway(..., IReadOnlyPolicyRegistry<string> registry, ...)
    {
       ...
    } 
}
// (或者, 如果您更喜欢环境上下文模式, 请使用线程安全的单例)

// 使用注册表中的策略
registry.Get<IAsyncPolicy<HttpResponseMessage>>("StandardHttpResilience")
    .ExecuteAsync<HttpResponseMessage>(...)
```
策略注册表具有一系列进一步的类似字典的语义, 例如 .ContainsKey(...), .TryGet<TPolicy>(...), .Count, .Clear(), 和 Remove(...)，适用于 v5.2.0 以上版本


有关详细信息, 请参阅: [文档](https://github.com/App-vNext/Polly/wiki/PolicyRegistry)

### .NET Core 使用Polly重试机制

```csharp
    public class PollyController : ApiController
    {
        public readonly RetryPolicy<HttpResponseMessage> _httpRequestPolicy;
        public PollyController()
        {
            _httpRequestPolicy = Policy.HandleResult<HttpResponseMessage>(
            r => r.StatusCode == HttpStatusCode.InternalServerError)
            .WaitAndRetryAsync(3,
            retryAttempt => TimeSpan.FromSeconds(retryAttempt));
        }
        public async Task<IHttpActionResult> Get()
        {
            var httpClient = new HttpClient();
            var requestEndpoint = "http://www.baidu.com";

            HttpResponseMessage httpResponse = await _httpRequestPolicy.ExecuteAsync(() => httpClient.GetAsync(requestEndpoint));

            IEnumerable<string> numbers = await httpResponse.Content.ReadAsAsync<IEnumerable<string>>();

            return Ok(numbers);
        }
    }


```