---
title: 【MVC学习笔记】3.使用Spring.Net应用IOC（依赖倒置）
tags:
  - MVC
  - Spring.Net
  - IOC
abbrlink: 63049
date: 2016-09-16 20:39:35
---
本篇我们将使用Spring.Net进行依赖导致。
<!-- more -->
到现在，我们已经基本搭建起了项目的框架，但是项目中还存在一个问题，就是尽管层与层之间使用了接口进行隔离，但实例化接口的时候，还是引入了接口实现类的依赖，如下面的代码：
```csharp
private IUserService _userService;
private IUserService UserService
{
    get { return _userService ?? (_userService = new UserService()); }
    set { _userService = value; }
}
```
[面向接口编程](http://baike.baidu.cn/view/2493204.htm)，Controller应该只依赖于站点业务层的接口，而不能依赖于具体的实现，否则，就违背了在层之间设置接口的初衷了。

另外，如果上层只依赖于下层的接口，在做单元测试的时候，就可以用Moq，Fakes等Mock工具来按实际需求来模拟接口的实现，就可以灵活的控制接口的返回值来对各种情况进行测试，如果依赖于具体的实现，项目的可测试性将大大减小，不利于进行自动化的单元测试。

要不依赖于具体的实现，就不能使用通常的 T t = new T() 的方式来获得一个类的实例了，需要通过IOC容器来对对象生命周期，依赖关系等进行统一的管理，这里，我们将使用Spring.Net应用IOC。
### Spring.Net在控制台程序中的使用
我们将通过一个简单的控制台示例来展示Spring.Net的使用方法

创建测试用的类：
```csharp
namespace SpringNetDemo
{
    public interface IClass
    {
        string Name { get; set; }
        Student Monitor { get; set; }
        string GetMsg();
    }
    public class Class : IClass
    {
        public string Name { get; set; }
        public Student Monitor { get; set; }

        public string GetMsg()
        {
            return "班级名称：" + Name + "，班长：" + Monitor.Name;
        }
    }

    public class Student
    {
        public string Name { get; set; }
    }
}
```
两个类，一个接口，Student类中有一个string类型的属性，为Name，Class类中除了string类型的Name属性外还有一个Student类型的Monitor属性，方法GetMsg可以返回当前Class对象的简介，包括班级名和班长名两个内容。Class类实现IClass接口。

先做简单的测试：
```csharp
IClass c6=new Class()
{
    Monitor = new Student()
    {
        Name = "李芙蓉"
    },
    Name = "六班"
};
Console.WriteLine(c6.GetMsg());
Console.ReadKey();
```
输出为：
![](http://p4au3q1y8.bkt.clouddn.com/20180218215235/20180218095803520.png)
接下来，我们**换用Spring.Net容器来声明对象**
1. 首先引用dll文件
![](http://p4au3q1y8.bkt.clouddn.com/20180218215235/20180218095839887.png)
需要核心库Spring.Core.dll和Spring.Net使用的日志记录组件Common.Logging.dll
2. 然后我们需要了解当前的程序集名称和命名空间
![](http://p4au3q1y8.bkt.clouddn.com/20180218215235/20180218095922038.png)
3. 在项目中新建一个xml文件，命名为services.xml：
```xml
<?xml version="1.0" encoding="utf-8" ?>
<objects xmlns="http://www.springframework.net">
  <description>An  example that demonstrates simple IoC features.</description>
  <object name="Class" type="SpringNetDemo.Class,SpringNetDemo">
    <property name="Name" value="尖子班"/>
    <property name="Monitor" ref="Student"/>
  </object>
  <object name="Student" type="SpringNetDemo.Student, SpringNetDemo">
    <property name="Name" value="陈二蛋"/>
  </object>
</objects>
```
    在xml中新建objects根节点，其中加入需要容器生成的object子节点，object子节点的type属性中需要指明类的完整名称（带有程序集）和当前命名空间，如果需要为当前类的属性赋默认值，则可以在object节点中增加property节点，配置其value属性来为类的属性赋初值，若类的属性仍然为其他类对象时，可以新建该类型的object节点，并给予其name属性，再在当前属性的property节点中将ref属性，指向新增object节点的name属性。

      注意：要把xml文件设置为“如果较新则复制”或者“始终复制”，否则生成时将不会自动复制到程序目录

4. 然后在应用程序配置文件中配置Spring.Net的信息：
```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <sectionGroup name="spring">
      <section name="context" type="Spring.Context.Support.ContextHandler, Spring.Core"/>
      <section name="objects" type="Spring.Context.Support.DefaultSectionHandler, Spring.Core" />
    </sectionGroup>
  </configSections>
  <spring>
    <context>
      <resource uri="file://services.xml"/>
    </context>
  </spring>
</configuration>
```
运行程序，得到输出结果：
![](http://p4au3q1y8.bkt.clouddn.com/20180218215235/20180218100404037.png)
**成功实现IOC**
### Spring.Net在ASP.NET MVC中的使用
方法和在控制台程序中大同小异
1. 同样，首先要导入dll文件
![](http://p4au3q1y8.bkt.clouddn.com/20180218215235/20180218100505181.png)
    MVC项目中需要引用的dll文件稍多些，需要五个，除了值钱的两个外，还需要三个Web相关的dll。
2. 为了便于管理，我们在MVC项目更目录新建Config文件夹来保存配置文件，并在其中新建两个xml文件
controllers.xml：
```xml
<?xml version="1.0" encoding="utf-8" ?>
<objects xmlns="http://www.springframework.net">、
  <object type="PMS.WebApp.Controllers.UserController , PMS.WebApp" singleton="false" >
    <property name="UserService" ref="UserService" />
  </object>
</objects>
```
    services.xml：
```xml
<?xml version="1.0" encoding="utf-8" ?>
<objects xmlns="http://www.springframework.net">
  <object name="UserService" type="PMS.BLL.UserService, PMS.BLL" singleton="false" >
  </object>
</objects>
```
      同样是出于方便管理考虑，我们将控制器和业务类分两个文件来保存，文件中节点的规则与控制台示例中完全相同。
3. 修改Web.config配置文件
![](http://p4au3q1y8.bkt.clouddn.com/20180218215235/20180218100703550.png)
    在配置文件的configSections节点中增加如图的sectionGrup节点，configuration节点中增加Spring节点，并在spring节点中的context节点中使用resource节点设置配置文件的路径。
4. 修改Global文件
修改根目录的Global.asax文件，将MvcApplication类的父类由HttpApplication更改为SpringMvcApplication。
```csharp
public class MvcApplication : SpringMvcApplication//HttpApplication
```
5. 最后，将原来的控制器中代码修改，就成功地在MVC项目中使用Spring.Net实现了IOC
```csharp
//private IUserService _userService;
//private IUserService UserService
//{
//    get { return _userService ?? (_userService = new UserService()); }
//    set { _userService = value; }
//}
private IUserService UserService { get; set; }
```