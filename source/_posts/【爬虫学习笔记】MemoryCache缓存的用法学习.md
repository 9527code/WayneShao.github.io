---
title: 【爬虫学习笔记】MemoryCache缓存的用法学习
tags:
  - 爬虫
  - 缓存
abbrlink: 30836
date: 2016-09-11 13:43:17
categories:
  - 学习笔记
---
在完成了DNS解析模块之后，我意识到了DNS缓存机制也很有必要。在Redis，Memcache，和.Net自带的Cache之间,考虑到部署问题，最终选择了后者，之前在学习Web及开发的过程中用过System.Web.Caching.Cache这个类库，但是这次的爬虫程序我打算部署为桌面软件，所以选用了System.Runtime.Caching.MemoryCache（后期如有必要也会加入System.Web.Caching.Cache来适配Web端程序）。

MemoryCache的使用网上介绍的不多，不过这个是.NET4.0新引入的缓存对象，估计主要是替换原来企业库的缓存模块，使得.NET的缓存可以无处不在，而不用基于特定的Windows版本上使用。

出于方便考虑，我们将不再实例化新的MemoryCache对象，只对MemoryCache的默认示例Memory.Default进行增删查操作。
## 基础操作
### 增加
![](http://qiniucdn.wayneshao.com/20180218211417/20180218091804918.png)
增加缓存需要提供两个参数，CacheItem类表示缓存中的单个缓存项,

构造函数: 
CacheItem(String, Object, String)            用缓存项的指定键、值和区域初始化新的 CacheItem 实例。

三个参数分别为：键、值和区域。

CacheItemPolicy类则表示缓存项的过期信息，只含有默认的构造函数。

增加一条缓存：
```csharp
var item = new CacheItem("习大大", "两学一做");
var policy = new CacheItemPolicy();
policy.SlidingExpiration = new TimeSpan(500);
//插入一条key为"习大大",value为"两学一做",500毫秒后自动销毁的缓存
MemoryCache.Default.Add(item, policy);
//重新设置policy的过期时间为当前时间+十分钟
policy.AbsoluteExpiration = DateTimeOffset.Now + TimeSpan.FromMinutes(10);
//注意,如果要使用Sliding时间,则Absolute必须为DateTimeOffset.MaxValue,反之,则Sliding必须为TimeSpan.Zero
policy.SlidingExpiration = TimeSpan.Zero;
//重新插入,覆盖前一条数据
MemoryCache.Default.Add(item, policy);
```
<font style="background-color: #ffff00">注意,如果要使用Sliding时间,则Absolute<strong>必须为DateTimeOffset.MaxValue</strong>,反之,则Sliding<strong>必须为TimeSpan.Zero</strong></font><strong> </strong>
### 查询
缓存对象类似于字典集,查询可以直接采用memoryCache[key]来进行,例如我们查询一下前面插入的那条数据:
```csharp
var idea = MemoryCache.Default["习大大"];
```
### 移除
![](http://qiniucdn.wayneshao.com/20180218211417/20180218092011241.png)
**参数**
***key***:要移除的缓存项的唯一标识符。
***regionName***:缓存中的一个添加了缓存项的命名区域。不要为该参数传递值。默认情况下，此参数为null，因为 MemoryCache 类未实现区域。
**返回值**
***Type***: *System.Object*  如果在缓存中找到该项，则为已移除的缓存项；否则为 null。

删除前面加入的那一项:
```csharp
MemoryCache.Default.Remove("习大大");
```
## 进一步封装
明白了基本的用法之后，我们就可以对它做进一步的封装，使之使用起来更为便捷：
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Caching;

namespace Crawler.Common
{
    /// <summary>
    /// 基于MemoryCache的缓存辅助类
    /// </summary>
    public static class MemoryCacheHelper
    {
        private static readonly object _locker = new object();

        public static bool Contains(string key)
        {
            return MemoryCache.Default.Contains(key);
        }


        /// <summary>
        /// 获取Catch元素
        /// </summary>
        /// <typeparam name="T">所获取的元素的类型</typeparam>
        /// <param name="key">元素的键</param>
        /// <returns>特定的元素值</returns>
        public static T Get<T>(string key)
        {
            if (string.IsNullOrWhiteSpace(key)) throw new ArgumentException("不合法的key!");
            if (!MemoryCache.Default.Contains(key))
                throw new ArgumentException("获取失败,不存在该key!");
            if (!(MemoryCache.Default[key] is T))
                throw new ArgumentException("未找到所需类型数据!");
            return (T)MemoryCache.Default[key];
        }

        /// <summary>
        /// 添加Catch元素
        /// </summary>
        /// <param name="key">元素的键</param>
        /// <param name="value">元素的值</param>
        /// <param name="slidingExpiration">元素过期时间(时间间隔)</param>
        /// <param name="absoluteExpiration">元素过期时间(绝对时间)</param>
        /// <returns></returns>
        public static bool Add(string key, object value, TimeSpan? slidingExpiration = null, DateTime? absoluteExpiration = null)
        {
            var item = new CacheItem(key, value);
            var policy = CreatePolicy(slidingExpiration, absoluteExpiration);
            lock (_locker)
                return MemoryCache.Default.Add(item, policy);
        }

        /// <summary>
        /// 移出Cache元素
        /// </summary>
        /// <typeparam name="T">待移出元素的类型</typeparam>
        /// <param name="key">待移除元素的键</param>
        /// <returns>已经移出的元素</returns>
        public static T Remove<T>(string key)
        {
            if (string.IsNullOrWhiteSpace(key)) throw new ArgumentException("不合法的key!");
            if (!MemoryCache.Default.Contains(key))
                throw new ArgumentException("获取失败,不存在该key!");
            var value = MemoryCache.Default.Get(key);
            if (!(value is T))
                throw new ArgumentException("未找到所需类型数据!");
            return (T)MemoryCache.Default.Remove(key);
        }

        /// <summary>
        /// 移出多条缓存数据,默认为所有缓存
        /// </summary>
        /// <typeparam name="T">待移出的缓存类型</typeparam>
        /// <param name="keyList"></param>
        /// <returns></returns>
        public static List<T> RemoveAll<T>(IEnumerable<string> keyList = null)
        {
            if (keyList != null)
                return (from key in keyList
                        where MemoryCache.Default.Contains(key)
                        where MemoryCache.Default.Get(key) is T
                        select (T)MemoryCache.Default.Remove(key)).ToList();
            while (MemoryCache.Default.GetCount() > 0)
                MemoryCache.Default.Remove(MemoryCache.Default.ElementAt(0).Key);
            return new List<T>();
        }

        /// <summary>
        /// 设置过期信息
        /// </summary>
        /// <param name="slidingExpiration"></param>
        /// <param name="absoluteExpiration"></param>
        /// <returns></returns>
        private static CacheItemPolicy CreatePolicy(TimeSpan? slidingExpiration, DateTime? absoluteExpiration)
        {
            var policy = new CacheItemPolicy();

            if (absoluteExpiration.HasValue)
            {
                policy.AbsoluteExpiration = absoluteExpiration.Value;
            }
            else if (slidingExpiration.HasValue)
            {
                policy.SlidingExpiration = slidingExpiration.Value;
            }

            policy.Priority = CacheItemPriority.Default;

            return policy;
        }
    }
}
```