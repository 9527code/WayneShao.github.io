---
title: 【MVC学习笔记】1.项目结构搭建及单个类在各个层次中的实现
tags:
  - MVC
abbrlink: 21959
date: 2016-09-16 18:14:43
---
 新人刚开始学习ASP.NET MVC，若有不足之处希望能得到您的指点，不胜感激！
<!-- more -->
## 层级结构
先来一张项目的层级结构图:
![](http://qiniucdn.wayneshao.com/20180218213243/20180218093427987.png)
 Model：模型层，主要是各种类型、枚举以及ORM框架，框架完成数据库和实体类的映射。项目中选用了微软的开源ORM框架 EntityFramework 6.0 （以下简称EF），数据库则选择了微软的轻量级数据库SQL Server Compact 4.0本地数据库（简称Compact），Compact对EF支持比较完美，又属于文档型数据库，部署起来比较简洁。

* ***DAL***：**数据访问层**，主要是对数据库的操作层，为业务逻辑层或表示层提供数据服务。

* ***IDAL***：**数据访问接口层**，是数据访问层的接口，降低耦合。

* ***DALFactory***：**数据会话层**，封装了所有数据操作类实例的创建，将数据访问层与业务逻辑层解耦。

* ***BLL***：**业务逻辑层**，主要负责对数据层的操作，把一些数据层的操作进行组合以完成业务的需要。

* ***IBLL***：**业务逻辑接口层**，业务逻辑层的接口，降低耦合。

* ***WebApp***：**表现层**，是一个ASP.NET MVC项目，完成具体网站的实现。

* ***Common***：**通用层**，用来存放一些工具类。

下面是各个层级之间具体的实现，首先创建以 项目名.层级名 命名的各个层次，除WebApp层为ASP.NET MVC项目外，其余均创建为类库项目。
![](http://qiniucdn.wayneshao.com/20180218213243/20180218093803742.png)
## 各层级搭建
### 模型层的构建
先建立模型层，新建ASP.NET 实体数据模型，关联到已经设计好的数据库，EF自动完成模型类的创建。
![](http://qiniucdn.wayneshao.com/20180218213243/20180218093918538.png)
### 数据访问层的构建
DAL层中，我们首先需要一个方法来获取单例的EF数据操纵上下文对象，以保证每个用户访问时只有使用一个上下文对象对数据库进行操作。DbContextFactory.cs
```csharp
using System.Data.Entity;
using System.Runtime.Remoting.Messaging;
using PMS.Model;

namespace PMS.DAL
{
    public class DbContextFactory
    {
        /// <summary>
        /// 负责创建EF数据操作上下文实例,必须保证线程内唯一
        /// </summary>
        public static DbContext CreateContext()
        {
            DbContext dbContext = (DbContext)CallContext.GetData("dbContext");
            if (dbContext != null) return dbContext;
            dbContext = new PMSEntities();
            CallContext.SetData("dbContext", dbContext);
            return dbContext;
        }
    }
}
```
为User类创建DAL层，实现查询、分页查询、增加、删除和修改这五个基本的方法：UserDAL.cs
```csharp
using System;
using System.Data.Entity;
using System.Linq;
using PMS.IDAL;

namespace PMS.DAL
{
    public partial class UserDal 

        {
        public DbContext DbEntities = DbContextFactory.CreateContext();

        /// <summary>
        /// 查询过滤
        /// </summary>
        /// <param name="whereLamada">过滤条件Lambda表达式</param>
        /// <returns>实体集合</returns>
        public IQueryable<UserDal> LoadEntities(System.Linq.Expressions.Expression<Func<UserDal, bool>> whereLamada)
        {
            return DbEntities.Set<UserDal>().Where(whereLamada);
        }

        /// <summary>
        /// 分页查询
        /// </summary>
        /// <typeparam name="TS">排序类型</typeparam>
        /// <param name="pageIndex">查询的页码</param>
        /// <param name="pageSize">每页显示的数目</param>
        /// <param name="totalCount">符合条件的总行数</param>
        /// <param name="whereLambda">过滤条件Lambda表达式</param>
        /// <param name="orderbyLambda">排序Lambda表达式</param>
        /// <param name="isAsc">排序方向</param>
        /// <returns>实体集合</returns>
        public IQueryable<UserDal> LoadPageEntities<TS>(int pageIndex, int pageSize, out int totalCount, System.Linq.Expressions.Expression<Func<UserDal, bool>> whereLambda, System.Linq.Expressions.Expression<Func<UserDal, TS>> orderbyLambda, bool isAsc)
        {
            var temp = DbEntities.Set<UserDal>().Where(whereLambda);
            totalCount = temp.Count();
            temp = isAsc ? temp.OrderBy(orderbyLambda).Skip((pageIndex - 1) * pageSize).Take(pageSize) : temp.OrderByDescending(orderbyLambda).Skip((pageIndex - 1) * pageSize).Take(pageSize);
            return temp;
        }

        /// <summary>
        /// 删除数据
        /// </summary>
        /// <param name="entity">待删数据</param>
        /// <returns>删除结果</returns>
        public bool DeleteEntity(UserDal entity)
        {
            DbEntities.Entry(entity).State = EntityState.Deleted;
            return true;
        }

        /// <summary>
        /// 编辑数据
        /// </summary>
        /// <param name="entity">待编辑数据</param>
        /// <returns>编辑结果</returns>
        public bool EditEntity(UserDal entity)
        {
            DbEntities.Entry(entity).State = EntityState.Modified;
            return true;
        }

        /// <summary>
        /// 添加数据
        /// </summary>
        /// <param name="entity">待添加数据</param>
        /// <returns>已添加数据</returns>
        public UserDal AddEntity(UserDal entity)
        {
            entity = DbEntities.Set<UserDal>().Add(entity);
            return entity;
        }       
    }
}
```
**注**：这里的增删改操作并不即时进行，而是在封装在数据会话层中，以实现工作单元模式，提高数据库的操作效率。

考虑到每个类都需要实现相同的数据操作，我们可以将以上方法封装到一个泛型基类中，各类型只需要继承泛型基类就可以实现以上方法：BaseDal.cs
```csharp
using System;
using System.Data.Entity;
using System.Linq;

namespace PMS.DAL
{
    public class BaseDal<T> where T:class ,new()
    {
        public DbContext DbEntities = DbContextFactory.CreateContext();

        /// <summary>
        /// 查询过滤
        /// </summary>
        /// <param name="whereLamada">过滤条件Lambda表达式</param>
        /// <returns>实体集合</returns>
        public IQueryable<T> LoadEntities(System.Linq.Expressions.Expression<Func<T, bool>> whereLamada)
        {
            return DbEntities.Set<T>().Where(whereLamada);
        }

        /// <summary>
        /// 分页查询
        /// </summary>
        /// <typeparam name="TS">排序类型</typeparam>
        /// <param name="pageIndex">查询的页码</param>
        /// <param name="pageSize">每页显示的数目</param>
        /// <param name="totalCount">符合条件的总行数</param>
        /// <param name="whereLambda">过滤条件Lambda表达式</param>
        /// <param name="orderbyLambda">排序Lambda表达式</param>
        /// <param name="isAsc">排序方向</param>
        /// <returns>实体集合</returns>
        public IQueryable<T> LoadPageEntities<TS>(int pageIndex, int pageSize, out int totalCount, System.Linq.Expressions.Expression<Func<T, bool>> whereLambda, System.Linq.Expressions.Expression<Func<T, TS>> orderbyLambda, bool isAsc)
        {
            var temp = DbEntities.Set<T>().Where(whereLambda);
            totalCount = temp.Count();
            temp = isAsc ? temp.OrderBy(orderbyLambda).Skip((pageIndex - 1) * pageSize).Take(pageSize) : temp.OrderByDescending(orderbyLambda).Skip((pageIndex - 1) * pageSize).Take(pageSize);
            return temp;
        }

        /// <summary>
        /// 删除数据
        /// </summary>
        /// <param name="entity">待删数据</param>
        /// <returns>删除结果</returns>
        public bool DeleteEntity(T entity)
        {
            DbEntities.Entry(entity).State = EntityState.Deleted;
            return true;
        }

        /// <summary>
        /// 编辑数据
        /// </summary>
        /// <param name="entity">待编辑数据</param>
        /// <returns>编辑结果</returns>
        public bool EditEntity(T entity)
        {
            DbEntities.Entry(entity).State = EntityState.Modified;
            return true;
        }

        /// <summary>
        /// 添加数据
        /// </summary>
        /// <param name="entity">待添加数据</param>
        /// <returns>已添加数据</returns>
        public T AddEntity(T entity)
        {
            entity = DbEntities.Set<T>().Add(entity);
            //DbEntities.SaveChanges();
            return entity;
        }
    }
}
```
UserDal继承BaseDal
```csharp
using PMS.IDAL;
using PMS.Model;

namespace PMS.DAL
{
    public partial class UserDal : BaseDal<User>
    {
        
    }
}
```
### 数据访问接口层的构建
然后我们建立相应的IbaseDal接口和IUserDal接口，并且使UserDal类实现IUserDal接口
IBaseDal：
```csharp
using System;
using System.Linq;

namespace PMS.IDAL
{
    public interface IBaseDal<T> where T:class,new()
    {
        IQueryable<T> LoadEntities(System.Linq.Expressions.Expression<Func<T, bool>> whereLamada);

        IQueryable<T> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount,
            System.Linq.Expressions.Expression<Func<T, bool>> whereLambda,
            System.Linq.Expressions.Expression<Func<T, s>> orderbyLambda, bool isAsc);

        bool DeleteEntity(T entity);
        
        bool EditEntity(T entity);

        T AddEntity(T entity);
    }
}
```
IUserDal：
```csharp
using PMS.Model;

namespace PMS.IDAL
{
    public partial interface IUserDal:IBaseDal<User>
    {

    }
}
```
UserDal实现IUserDal接口：
```csharp
public partial class UserDal : BaseDal<User>,IUserDal
```
### 数据会话层的构建
抽象工厂类AbstractFactory：
```csharp
using System.Configuration;
using System.Reflection;
using PMS.IDAL;

namespace PMS.DALFactory
{
    public partial class AbstractFactory
    {
        //读取保存在配置文件中的程序集名称与命名空间名
        private static readonly string AssemblyPath = ConfigurationManager.AppSettings["AssemblyPath"];
        private static readonly string NameSpace = ConfigurationManager.AppSettings["NameSpace"];
        /// <summary>
        /// 获取UserDal的实例
        /// </summary>
        /// <returns></returns>
        public static IUserDal CreateUserInfoDal()
        {
            var fullClassName = NameSpace + ".UserInfoDal";
            return CreateInstance(fullClassName) as IUserDal;
        }
        /// <summary>
        /// 通过反射获得程序集中某类型的实例
        /// </summary>
        /// <param name="className"></param>
        /// <returns></returns>
        private static object CreateInstance(string className)
        {
            var assembly = Assembly.Load(AssemblyPath);
            return assembly.CreateInstance(className);
        }
    }
}
```
数据会话类DbSession：
```csharp
using System.Data.Entity;
using PMS.IDAL;
using PMS.DAL;

namespace PMS.DALFactory
{
    public partial class DbSession:IDbSession
    {
        public DbContext Db
        {
            get { return DbContextFactory.CreateContext(); }
        }

        private IUserDal _userDal;
        public IUserDal UserDal
        {
            get { return _userDal ?? (_userDal = AbstractFactory.CreateUserInfoDal()); }
            set { _userDal = value; }
        }

        /// <summary>
        /// 工作单元模式，统一保存数据
        /// </summary>
        /// <returns></returns>
        public bool SaveChanges()
        {
            return Db.SaveChanges() > 0;
        }
    }
}
```
### 业务逻辑层的构建
业务类基类BaseService
```csharp
using System;
using System.Linq;
using System.Linq.Expressions;
using PMS.DALFactory;
using PMS.IDAL;

namespace PMS.BLL
{
    public abstract class BaseService<T> where T:class,new()
    {
        public IDbSession CurrentDbSession
       {
           get
           {
               return new DbSession();
           }
       }
       public IBaseDal<T> CurrentDal { get; set; }
       public abstract void SetCurrentDal();
       public BaseService()
       {
           SetCurrentDal();//子类一定要实现抽象方法，以指明当前类的子类类型。
       }

       /// <summary>
       /// 查询过滤
       /// </summary>
       /// <param name="whereLambda"></param>
       /// <returns></returns>
       public IQueryable<T> LoadEntities(Expression<Func<T, bool>> whereLambda)
       {
           return CurrentDal.LoadEntities(whereLambda);
       }

       /// <summary>
       /// 分页
       /// </summary>
       /// <typeparam name="s"></typeparam>
       /// <param name="pageIndex"></param>
       /// <param name="pageSize"></param>
       /// <param name="totalCount"></param>
       /// <param name="whereLambda"></param>
       /// <param name="orderbyLambda"></param>
       /// <param name="isAsc"></param>
       /// <returns></returns>
       public IQueryable<T> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount, Expression<Func<T, bool>> whereLambda,
           Expression<Func<T, s>> orderbyLambda, bool isAsc)
       {
           return CurrentDal.LoadPageEntities<s>(pageIndex, pageSize, out totalCount, whereLambda, orderbyLambda, isAsc);
       }

       /// <summary>
       /// 删除
       /// </summary>
       /// <param name="entity"></param>
       /// <returns></returns>
       public bool DeleteEntity(T entity)
       {
           CurrentDal.DeleteEntity(entity);
           return CurrentDbSession.SaveChanges();
       }

       /// <summary>
       /// 编辑
       /// </summary>
       /// <param name="entity"></param>
       /// <returns></returns>
       public bool EditEntity(T entity)
       {
           CurrentDal.EditEntity(entity);
           return CurrentDbSession.SaveChanges();
       }

       /// <summary>
       /// 添加数据
       /// </summary>
       /// <param name="entity"></param>
       /// <returns></returns>
       public T AddEntity(T entity)
       {
           CurrentDal.AddEntity(entity);
           CurrentDbSession.SaveChanges();
           return entity;
       }
    }
}
```
UserService类：
```csharp
using PMS.IBLL;
using PMS.Model;

namespace PMS.BLL
{
    public partial class UserService : BaseService<User>
    {
        public override void SetCurrentDal()
        {
            CurrentDal = CurrentDbSession.UserDal;
        }
    }
}
```
### 业务逻辑接口层的构建
直接建立对应的接口并使用UserService类实现IUserService接口

IBaseService接口：
```csharp
using System;
using System.Linq;
using System.Linq.Expressions;
using PMS.IDAL;

namespace PMS.IBLL
{
    public interface IBaseService<T> where T : class,new()
    {
        IDbSession CurrentDbSession { get; }
        IBaseDal<T> CurrentDal { get; set; }
        void SetCurrentDal();
        IQueryable<T> LoadEntities(Expression<Func<T, bool>> whereLambda);

        IQueryable<T> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount,
            Expression<Func<T, bool>> whereLambda,
            Expression<Func<T, s>> orderbyLambda, bool isAsc);

        bool DeleteEntity(T entity);
        bool EditEntity(T entity);
        T AddEntity(T entity);
    }
}
```
IUserService接口:
```csharp
using PMS.Model;

namespace PMS.IBLL
{
    public partial interface IUserService:IBaseService<User>
    {

    }
}
```
使用UserService类实现IUserService接口:
```csharp
public partial class UserService : BaseService<User>, IUserService
```
以上我们就完成了整个框架中关于User类的各层次的实现。