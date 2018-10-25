---
title: Selenium PhantomJS 巧妙过渡到 Firefox/Chrome
author: 玮仔Wayne
tags:
  - 'C#'
  - selenium
  - full page screenshot
  - 全页面截图
categories:
  - 经验之谈
abbrlink: 40690
date: 2018-07-11 20:19:29
---

# PhantomJS Obsolete
## Origin
前段时间因为一些个人爱好，想要对某网站的数据进行整站采集，其中需要对某些页面的一些区块进行截图采集，整个采集任务中还涉及一些验证码识别之类的工作。学艺不精，我当前掌握的 Scrapy 知识很难完成这样一个爬虫，就使用了 Selenium + PhantomJS 制作了一个模拟浏览器访问来爬取数据的小爬虫，完成了整套抓取任务。
![](http://qiniucdn.wayneshao.com/20180711045204672.gif)
<!--more-->
然而上个月手误格掉了整块数据硬盘，之前的代码也没留下备份，我还仍然有同样的数据采集需要，只能准备按照原有思路重新做一个爬虫，这本来应该只是个体力活，只要重新抓样本，做好验证码识别，之后就应该一马平川，一泻千里了。
![](http://qiniucdn.wayneshao.com/Selenium-+-Firefox-Chrome-%E5%85%A8%E9%A1%B5%E9%9D%A2%E6%88%AA%E5%9B%BE/20180711050254778.png)
然而就在我开始动手的时候，PhantomJSDriver 类型下的蓝色下划线成功吸引了有强迫症的程序员本尊的注意
![](http://qiniucdn.wayneshao.com/20180711213420950/20180711101818131.png)
![](http://qiniucdn.wayneshao.com/20180711213420950/20180711101850349.png)
![](http://qiniucdn.wayneshao.com/Selenium%E5%85%A8%E9%A1%B5%E9%9D%A2%E6%88%AA%E5%9B%BE/20180711080149417.png)

运用我考了三遍都没过的四级英语定睛一看，这个意思是说， PhantomJSDriver 类型已经被**弃用**，PhantomJS 的开发工作已经停止，PhantomJS 的驱动**将会在未来的某个 release 版本上被移除**。
**天哪~！**
![](http://qiniucdn.wayneshao.com/Selenium%E5%85%A8%E9%A1%B5%E9%9D%A2%E6%88%AA%E5%9B%BE/20180711080102494.png)
告诉我不是真的！Selenium 居然放弃了他的好基友 PhantomJS！
***（这个声明颇有 “不是我他跟不上我的进步被我抛弃，而是他渣，他抛弃了我” 的戏剧性，让我不由得想要查证一下）***

## Investigate
不敢相信的我祭出了谷歌神器

> Chrome 59 将支持 Headless 模式。而在 Chrome 未提供原生 Headless 模式前，Web 开发者可以使用 PhantomJS 等第三方 Headless 浏览器。现在官方准备提供 Headless 了，PhantomJS 主要的贡献者 Vitaly Slobodin 随即在邮件列表上宣布辞职。
> ![](http://qiniucdn.wayneshao.com/Selenium%E5%85%A8%E9%A1%B5%E9%9D%A2%E6%88%AA%E5%9B%BE/20180711080802730.png)
因为这半年都没有过写新爬虫的需求，而最近一直在跑着的爬虫用的是老版本 Selenium 开发，所以还 PhantomJS 玩得很嗨，殊不知已经 Out 了。查了一下，**去年四月份的 Chrome 59 版本和六月份的 Firefox 56 版本都引入了 Headless 模式**，PhantomJS 的独领风骚地位瞬间丧失，开发者流失，仅剩的一位开发者 Vitaly Slobodin 看不到 PhantomJS 的未来，选择了**停止开发**，然后 “不思进取” 的 PhantomJS 逐渐消失在历史的尘埃中…… 小厂出的创新产品，大厂做出类似产品之后，小厂 GG，大概也就是这么一回事吧……
***（虽然果真是 PhantomJS 做了负心汉，但还是莫名悲壮，有种丈夫不思进取，觉得配不上努力上进妻子然后自我了断给妻子自由的既视感）***

# Headless Chrome/Firefox
想要使用 Selenium 控制 Firefox 进行页面浏览，需要先做以下的准备工作：
## Headless Firefox
### Prepare
1. 安装最新版本的 Firefox 浏览器。
2. 下载最新版本的适应当前系统的 GeckoDriver。
3. 将步骤 2 下载的 GeckoDriver 的程序文件移动到 Firefox 的程序目录中，使两个程序的执行文件处于同一目录中，并将程序所在的目录加入到环境变量中。

然后引入官方的四个 Nuget 包：
![](http://qiniucdn.wayneshao.com/Selenium%E5%85%A8%E9%A1%B5%E9%9D%A2%E6%88%AA%E5%9B%BE/20180711082732423.png)

> Install-Package Selenium.WebDriver
Install-Package Selenium.WebDriverBackedSelenium
Install-Package Selenium.Support
Install-Package Selenium.RC

### Coding
随意拉一个窗体用于测试，然后敲入以下代码：
```csharp
var firefoxOption = new FirefoxOptions();
firefoxOption.AddArguments("-headless");
var firefoxDriver = new FirefoxDriver(firefoxOption);

firefoxDriver.Navigate().GoToUrl("http://www.baidu.com/");

textBox1.Text = firefoxDriver.PageSource;
```
运行结果：
![](http://qiniucdn.wayneshao.com/20180711213420950/20180711105729884.png)
## Headless Chrome
### Prepare
想要使用 Selenium 控制 Chrome 进行页面浏览，需要做的准备工作和上面的 Firefox 大同小异：

1. 安装最新版本的 Chrome 浏览器（也可以考虑像我一样使用国内大牛写的 Chrome 绿色化工具 MyChrome 安装绿色版 Chrome ，在版本控制、用户文件本地化方面更具优势）。
2. 下载最新版本的适应当前系统的 ChromeDriver。
3. 将步骤 2 下载的 ChromeDriver 的程序文件移动到 Chrome 的程序目录中，使两个程序的执行文件处于同一目录中，并将程序所在的目录加入到环境变量中。

依然是官方的四个 Nuget 包（如果已经安装过，则直接跳过）：
![](http://qiniucdn.wayneshao.com/Selenium%E5%85%A8%E9%A1%B5%E9%9D%A2%E6%88%AA%E5%9B%BE/20180711082732423.png)

> Install-Package Selenium.WebDriver
Install-Package Selenium.WebDriverBackedSelenium
Install-Package Selenium.Support
Install-Package Selenium.RC

### Coding
随意拉一个窗体用于测试，然后敲入以下代码：
```csharp
var chromeOption = new ChromeOptions();
chromeOption.AddArguments("--headless", "--disable-gpu");
var chromeDriver = new ChromeDriver(chromeOption);

chromeDriver.Navigate().GoToUrl("http://www.baidu.com/");

textBox1.Text = chromeDriver.PageSource;
```
运行结果：
![](http://qiniucdn.wayneshao.com/20180711213420950/20180711111201947.png)

## Full Page ScreenShot
无头模式是已经实现了，在打开时间上效率略差于 PhantomJS，但是执行页面抓取是却要更优于 PhantomJS ，无愧于老牌浏览器的称号。可是接下来就遇到了新的问题，上面提到的，我的爬虫求有截取页面某一区域图片的需求，而 Selenium 的驱动 API 标准获取图片的只有 GetScreenShot ，在之前使用 PhantomJS 时，由于 PhantomJS 从从诞生时起就是一个为爬虫服务的没有界面的浏览器，所以截图 API 截到的就是整个页面的图片，在获取某一区域的渲染图片时，只需要从截到的全页面图中将区域所在的矩形取出来，就可以完成要求。但是对于 Chrome 和 Firefox 这样的浏览器，虽然有 Headless 模式，但是窗口的概念是一只存在的， GetScreenShot 截到的只会是浏览器窗口显示的部分页面的截图，所以我们需要找到一种可以截全图的方法。
### Thinking
想要在每次只能截到浏览器显示区域截图的情况下得到整个页面的截图，有如下两个思路：

1. 控制浏览器滚动条移动，将所有区域的截图全都获取到，再根据每次截图时滚动条所处的位置信息，将所有截图合并到一起，最终得到全页面的截图。
2. 把浏览器的窗口大小设置到页面一样大，甚至比页面稍大些，再进行截图，就可以得到全页面的截图。
比较而言无疑是思路 2 更为简单高效，而且在 Headless 模式下，浏览器窗口的变化也完全不会有什么影响，故我们选用第二种思路来实现全页面截图。

### Coding
这里我们使用. NET 知名开源图片处理组件 ImageProcessor 来进行图片裁剪。

> Install-Package ImageProcessor

并非专业前端的我开始觉得 html 标签的尺寸应该就是整个页面的尺寸了，所以有了如下的代码：

```csharp
public static Image GetElementImage(this RemoteWebDriver driver, IWebElement element)
{
    driver.Manage().Window.Size = driver.FindElementByTagName("html").Size;
    var photoBytes = driver.GetScreenshot().AsByteArray;
    using (var inStream = new MemoryStream(photoBytes))
    {
        using (var outStream = new MemoryStream())
        {
            using (var imageFactory = new ImageFactory(true))
            {
                imageFactory.Load(inStream)
                    .Crop(new Rectangle(element.Location, element.Size))
                    .Save(outStream);
            }
            return Image.FromStream(outStream);
        }
    }
}
```

但是在测试过程中发现并非如此，具体测试页面为 “Selenium 的维基百科关键词主页”https://en.wikipedia.org/wiki/Selenium_(software)
调用代码：

```csharp
pictureBox1.Image =
                chromeDriver.GetElementImage(
                    chromeDriver.FindElementByXPath(@"//*[@id=""footer-copyrightico""]/a/img"));
```

![](http://qiniucdn.wayneshao.com/20180711213420950/20180711115157926.png)

调用代码中的 XPath 命中的标签为页面底部的维基百科 logo 图片，调试信息可知，该标签的 Y 坐标远大于 Html 标签的 Height ，故 Html 的尺寸应该和页面实际尺寸并不完全吻合。

居然不对？！
![](http://qiniucdn.wayneshao.com/20180711213420950/20180712120904055.png)

[查询资料大法：](https://www.zhihu.com/question/20816879)

```csharp
//页面尺寸
var pageWidth = Math.max(
     document.body.scrollWidth,
     document.documentElement.scrollWidth,
     document.body.offsetWidth, 
     document.documentElement.offsetWidth,
     document.documentElement.clientWidth
);


var pageHeight = Math.max(
     document.body.scrollHeight,
     document.documentElement.scrollHeight,
     document.body.offsetHeight, 
     document.documentElement.offsetHeight,
     document.documentElement.clientHeight
);
```
于是我决定改用执行 JS 代码来获取页面实际尺寸：
封装 JS 执行方法：
```csharp
public static T Execute<T>(this IWebDriver driver, string script) 
            => (T)((IJavaScriptExecutor)driver).ExecuteScript(script);
```
获取实际尺寸
```csharp
var height = driver.Execute<long>("return Math.max(document.body.scrollHeight, document.body.offsetHeight, document.documentElement.clientHeight, document.documentElement.scrollHeight, document.documentElement.offsetHeight);");
var width = driver.Execute<long>("return Math.max(document.body.scrollWidth, document.body.offsetWidth, document.documentElement.clientWidth, document.documentElement.scrollWidth, document.documentElement.offsetWidth);");
```

使用新思路重新封装方法
```csharp
public static Image GetElementImage(this RemoteWebDriver driver, IWebElement element)
{
    //driver.Manage().Window.Size = driver.FindElementByTagName("html").Size;

    var height = driver.Execute<long>("return Math.max(document.body.scrollHeight, document.body.offsetHeight, document.documentElement.clientHeight, document.documentElement.scrollHeight, document.documentElement.offsetHeight);");
    var width = driver.Execute<long>("return Math.max(document.body.scrollWidth, document.body.offsetWidth, document.documentElement.clientWidth, document.documentElement.scrollWidth, document.documentElement.offsetWidth);");

    driver.Manage().Window.Size = new Size((int)width + 100, (int)height + 100);

    var photoBytes = driver.GetScreenshot().AsByteArray;

    using (var inStream = new MemoryStream(photoBytes))
    {
        using (var outStream = new MemoryStream())
        {
            using (var imageFactory = new ImageFactory(true))
            {
                imageFactory.Load(inStream)
                    .Crop(new Rectangle(element.Location, element.Size))
                    .Save(outStream);
            }
            return Image.FromStream(outStream);
        }
    }
}
```
代码执行结果如下：
![](http://qiniucdn.wayneshao.com/20180711213420950/20180712120513778.png)

成功！

![](http://qiniucdn.wayneshao.com/20180711213420950/20180712120712325.png)