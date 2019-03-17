---
title: 【MVC学习笔记】5.使用Controller来代替Filter完成登录验证（Session校验）
tags:
  - MVC
  - 登录验证
abbrlink: 21523
date: 2016-09-17 10:16:24
categories:
  - 学习笔记
---
之前的学习中，在对Session校验完成登录验证时，通常使用Filter来处理，方法类似与前文的错误日志过滤，即新建Filter类继承ActionFilterAttribute类，重写OnActionExecuting方法，之后直接在需要验证的Action前加上Filter标记即可。
<!-- more -->
## Filter式实现
### 新建登陆校验类
新建登陆校验类CheckLoginAttribute
```csharp
using System.Web.Mvc;

namespace PMS.WebApp.Models
{
    public class CheckLoginAttribute:ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            base.OnActionExecuting(filterContext);
            if (filterContext.HttpContext.Session == null || filterContext.HttpContext.Session["user"] == null)
            {
                filterContext.HttpContext.Response.Redirect("/User/Login");
            }
        }
    }
}
```
### 增加特性
在需要校验的Action增加标记以完成校验
```csharp
using System.Web.Mvc;
using PMS.IBLL;
using PMS.WebApp.Models;

namespace PMS.WebApp.Controllers
{
    public class UserController : Controller
    {
        //
        // GET: /User/
        //private IUserService _userService;
        //private IUserService UserService
        //{
        //    get { return _userService ?? (_userService = new UserService()); }
        //    set { _userService = value; }
        //}
        private IUserService UserService { get; set; }
        [CheckLogin]
        public ActionResult Index()
        {
            return Content("OK");
        }

    }
}
```
注意：不要在RegisterGlobalFilters方法中注册校验类，否则则会相当于给所有Action都添加了校验

这种方法使用起来需要在每个Action方法前添加过滤标签，且效率并不十分高，我们的项目中使用的是一种更为简单高效的方法：使用Controller进行登录验证
## Controller式实现
### 新建验证父类
新建一个用于验证的Controller父类，并在其内重写OnActionExecuting方法完成登陆校验：
```csharp
using System.Web.Mvc;

namespace PMS.WebApp.Controllers
{
    public class FilterController : Controller
    {
        protected override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            base.OnActionExecuting(filterContext);
            if (Session["user"] == null)
            {
                //filterContext.HttpContext.Response.Redirect("/User/Login");
                filterContext.Result = Redirect("/User/Login");
            }
        }
    }
}
```
在Controller校验类的OnActionExecuting方法中，有如下代码
```csharp
//filterContext.HttpContext.Response.Redirect("/User/Login");
filterContext.Result = Redirect("/User/Login");
```
我们使用后者而放弃前者的原因是，ASP.NET MVC中规定，Action必须返回ActionResult，如果使用前者，在完成跳转前会先进入到请求的页面，这样不符合我们使用过滤器的初衷。
### 继承校验父类
然后使需要校验的Controller继承于我们定义的校验Controller即可完成全局登录校验操作：
```csharp
using System.Web.Mvc;
using PMS.IBLL;

namespace PMS.WebApp.Controllers
{
    public class UserController : FilterController//Controller
    {
        //
        // GET: /User/
        //private IUserService _userService;
        //private IUserService UserService
        //{
        //    get { return _userService ?? (_userService = new UserService()); }
        //    set { _userService = value; }
        //}
        private IUserService UserService { get; set; }
        //[CheckLogin]
        public ActionResult Index()
        {
            return Content("OK");
        }

    }
}
```
下面我们对比两种方法的优缺点:

  Filter定义过程比较复杂，效率也稍低些，但是却可以对每一个Action进行单独的过滤，同一Action也可以有多条过滤信息，使用比较灵活。

  Controller定义更为简便，效率高，但是却只能对整个Controller中所有方法进行过滤，同一Controller也不太容易有多个Controller过滤父类。


 **综上所述，实际项目中大多需求都是同一Controller下所有方法都需要完成登陆验证，所以其实使用Controller过滤更为高效，应对复杂需求时，灵活混用两种方法也不失为一种好的策略。**