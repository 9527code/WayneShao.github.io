---
title: 【MVC学习笔记】4.使用Log4Net来进行错误日志的记录
tags:
  - MVC
  - Log4Net
abbrlink: 43794
date: 2016-09-17 09:39:20
---
在Web应用运行过程中，我们难免会遇到程序运行异常，这个时候我们就应该将异常信息记录下来，以便开发人员和维护人员对异常原因进行还原，对异常原因进行修复。在ASP.NET平台中进行日志记录的组件也有很多，如Log4Net、CommonLogging等，我们这里选用Log4Net进行异常日志的记录。
<!-- more -->
## 捕获异常
在ASP.NET MVC中提供了一个全局的异常处理过滤器：HandleErrorAttribute，可以通过该过滤器捕获异常信息。

我们在Models文件夹下新建类型Log4ExceptionAttribute，继承HandleErrorAttribute类，同时重写OnException方法来捕获异常数据：      
```csharp
using System.Web.Mvc;

namespace PMS.WebApp.Models
{
    public class Log4ExceptionAttribute:HandleErrorAttribute
    {
        /// <summary>
        /// 重写OnException方法来捕获异常数据
        /// </summary>
        /// <param name="filterContext"></param>
        public override void OnException(ExceptionContext filterContext)
        {
            base.OnException(filterContext);
            //捕获当前异常数据
            var ex = filterContext.Exception;
        }
    }
}
```
新建过滤器后我们还需要在Global文件中调用的RegisterGlobalFilters方法中完成自己定义异常处理过滤的注册。
```csharp
using System.Web.Mvc;
using PMS.WebApp.Models;

namespace PMS.WebApp
{
    public class FilterConfig
    {
        public static void RegisterGlobalFilters(GlobalFilterCollection filters)
        {
            //filters.Add(new HandleErrorAttribute());
            filters.Add(new Log4ExceptionAttribute());
        }
    }
}
```
## 队列处理
考虑到多用户并发操作时可能产生的问题，我们需要新建一个队列来进行异常信息的暂存，同时开辟一个线程专门对队列中的异常信息进行处理。

在Log4ExceptionAttribute类中新建一个静态的异常类型的队列，在发生异常后，程序自动触发OnException方法，方法中将当前的异常信息入队后，跳转到错误页面。
```csharp
using System;
using System.Collections.Generic;
using System.Web.Mvc;

namespace PMS.WebApp.Models
{
    public class Log4ExceptionAttribute:HandleErrorAttribute
    {
        public static Queue<Exception> Exceptions=new Queue<Exception>();
        /// <summary>
        /// 重写OnException方法来捕获异常数据
        /// </summary>
        /// <param name="filterContext"></param>
        public override void OnException(ExceptionContext filterContext)
        {
            base.OnException(filterContext);
            //捕获当前异常数据
            var ex = filterContext.Exception;
            //将异常数据入队
            Exceptions.Enqueue(ex);
            //跳转到错误页面
            filterContext.HttpContext.Response.Redirect("/Error.html");
        }
    }
}
```
Log4Net的配置是在应用程序配置文件中进行的，我们先在配置文件中进行Log4Net的配置。Log4Net需要配置的节点位置和SpringNet完全相同，首先需要在configSessions中新增子节点,然后在configuration节点中增加log4net节点完成具体配置。
```xml
<configuration>
  <configSections>
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    
    <!--↓Log4Net配置↓-->
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net"/>
    <!--↑Log4Net配置↑-->
    
    <!--↓Spring.Net配置↓-->
    <sectionGroup name="spring">
      <section name="context" type="Spring.Context.Support.MvcContextHandler, Spring.Web.Mvc4"/>
    </sectionGroup>
    <!--↑Spring.Net配置↑-->
    
  </configSections>
  
  <!--↓Spring.Net配置↓-->
  <spring>
    <context>
      <resource uri="file://~/Config/controllers.xml"/>
      <resource uri="file://~/Config/services.xml"/>
    </context>
  </spring>
  <!--↑Spring.Net配置↑-->
  
  <!--↓Log4Net配置↓-->
  <log4net>
    <!-- OFF, FATAL, ERROR, WARN, INFO, DEBUG, ALL -->
    <!-- Set root logger level to ERROR and its appenders -->
    <root>
      <level value="ALL"/>
      <appender-ref ref="SysAppender"/>
    </root>

    <!-- Print only messages of level DEBUG or above in the packages -->
    <logger name="WebLogger">
      <level value="DEBUG"/>
    </logger>

    <appender name="SysAppender" type="log4net.Appender.RollingFileAppender,log4net" >
      <param name="File" value="App_Data/" />
      <param name="AppendToFile" value="true" />
      <param name="RollingStyle" value="Date" />
      <param name="DatePattern" value="&quot;Logs_&quot;yyyyMMdd&quot;.txt&quot;" />
      <param name="StaticLogFileName" value="false" />
      <layout type="log4net.Layout.PatternLayout,log4net">
        <param name="ConversionPattern" value="%d [%t] %-5p %c - %m%n" />
        <param name="Header" value="&#13;&#10;----------------------header--------------------------&#13;&#10;" />
        <param name="Footer" value="&#13;&#10;----------------------footer--------------------------&#13;&#10;" />
      </layout>
    </appender>
    <appender name="consoleApp" type="log4net.Appender.ConsoleAppender,log4net">
      <layout type="log4net.Layout.PatternLayout,log4net">
        <param name="ConversionPattern" value="%d [%t] %-5p %c - %m%n" />
      </layout>
    </appender>
  </log4net>
  <!--↑Log4Net配置↑-->
  ...
</configuration>
```
## 进阶配置
在配置文件中可以对日志记录的信息、格式、文件名等作出具体的配置，下面是配置信息的详解
```xml
<?xml version="1.0"?>
<configuration>
  <configSections>
    <section name="log4net" 
             type="log4net.Config.Log4NetConfigurationSectionHandler,log4net"/>
  </configSections>
  <!--站点日志配置部分-->
  <log4net>
    <root>
      <!--控制级别，由低到高: ALL|DEBUG|INFO|WARN|ERROR|FATAL|OFF-->
      <!--比如定义级别为INFO，则INFO级别向下的级别，比如DEBUG日志将不会被记录-->
      <!--如果没有定义LEVEL的值，则缺省为DEBUG-->
      <level value="ERROR"/>
      <appender-ref ref="RollingFileAppender"/>
    </root>
    <appender name="RollingFileAppender" type="log4net.Appender.RollingFileAppender">
      <!--日志文件名开头-->
      <file value="c:\Log\TestLog4net.TXT"/>
      <!--多线程时采用最小锁定-->
      <lockingModel type="log4net.Appender.FileAppender+MinimalLock"/>
      <!--日期的格式，每天换一个文件记录，如不设置则永远只记录一天的日志，需设置-->
      <datePattern value="(yyyyMMdd)"/>
      <!--是否追加到文件,默认为true，通常无需设置-->
      <appendToFile value="true"/>
      <!--变换的形式为日期，这种情况下每天只有一个日志-->
      <!--此时MaxSizeRollBackups和maximumFileSize的节点设置没有意义-->
      <!--<rollingStyle value="Date"/>-->
      <!--变换的形式为日志大小-->
      <!--这种情况下MaxSizeRollBackups和maximumFileSize的节点设置才有意义-->
      <RollingStyle value="Size"/>
      <!--每天记录的日志文件个数，与maximumFileSize配合使用-->
      <MaxSizeRollBackups value="10"/>
      <!--每个日志文件的最大大小-->
      <!--可用的单位:KB|MB|GB-->
      <!--不要使用小数,否则会一直写入当前日志-->
      <maximumFileSize value="2MB"/>
      <!--日志格式-->
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%date [%t]%-5p %c - %m%n"/>
      </layout>
    </appender>
  </log4net>
</configuration>
```
在Global文件中的Application_Start方法中开启一个线程，用于将队列中的错误信息写入日志文件。
```csharp
using System.Linq;
using System.Threading;
using System.Web.Http;
using System.Web.Mvc;
using System.Web.Optimization;
using System.Web.Routing;
using log4net;
using PMS.WebApp.Models;
using Spring.Web.Mvc;

namespace PMS.WebApp
{
    // 注意: 有关启用 IIS6 或 IIS7 经典模式的说明，
    // 请访问 http://go.microsoft.com/?LinkId=9394801

    public class MvcApplication : SpringMvcApplication//HttpApplication
    {
        protected void Application_Start()
        {
            log4net.Config.XmlConfigurator.Configure();//读取Log4Net配置信息
            AreaRegistration.RegisterAllAreas();

            WebApiConfig.Register(GlobalConfiguration.Configuration);
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);

            //开启一个线程,扫描异常信息队列.
            var filePath = Server.MapPath("/Log/");
            ThreadPool.QueueUserWorkItem((a) =>
            {
                while (true)
                {
                    //判断队列中是否有数据
                    if (Log4ExceptionAttribute.Exceptions.Any())
                    {
                        //出队一条异常信息
                        var ex = Log4ExceptionAttribute.Exceptions.Dequeue();
                        //若异常信息不为空
                        if (ex == null) continue;
                        //将异常信息写入到日志文件中
                        var logger = LogManager.GetLogger("errorMsg");
                        logger.Error(ex.ToString());
                    }
                    else
                    {
                        //若异常信息队列为空，则线程休息三秒
                        Thread.Sleep(3000);
                    }
                }
            }, filePath);
        }
    }
}
```
成功完成错误日志的配置。 