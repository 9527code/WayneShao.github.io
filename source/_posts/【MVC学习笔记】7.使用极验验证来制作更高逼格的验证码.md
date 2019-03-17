---
title: 【MVC学习笔记】7.使用极验验证来制作更高逼格的验证码
tags:
  - MVC
  - 极验
abbrlink: 47398
date: 2016-09-20 20:15:16
categories:
  - 学习笔记
---
在之前的项目中，如果有需要使用验证码，基本都是自己用GDI+画图出来，简单好用，但是却也存在了一些小问题，首先若较少干扰线，则安全性不是很高，验证码容易被机器识别，若多画太多干扰线条，机器人识别率下降的同时，人眼的识别率也同步下降（震惊哭）。更为重要的是，GDI+绘制的验证码一般来说也不会很美观，如果做一个炫酷的登陆界面却配了这样一个验证码，画风诡异，丑到极致。

再后来浏览网页的过程中，发现很多很多网站项目中都使用了一种叫极验验证的验证码，采用移动滑块的方式进行验证，方便美观。而一番搜索之后了解到，官方提供的免费版也足以应付我手头的大多数项目了，不禁想把在MVC学习过程中试着使用极验验证来作为登录的验证码。
<!-- more -->
极验官方提供了[C#的SDK和Demo](https://github.com/GeeTeam/gt-csharp-sdk)供开发者参考，不过是Webform版本的，可读性不是很高，而现在使用Webform进行网站开发的也基本消失了，我将在官方Webform代码的基础上，将其用在ASP.NET MVC程序中。
## 注册极验
到极验官网注册账号之后进入后台管理界面，点击添加验证
![](http://qiniucdn.wayneshao.com/20180218223716/20180218103931147.png)
添加后我们可以得到ID和KEY
![](http://qiniucdn.wayneshao.com/20180218223716/20180218103949448.png)
## 完成验证逻辑
### 首先我们需要引入官方的[Geetestlib类](https://github.com/GeeTeam/gt-csharp-sdk/blob/master/src/GeetestSDK/GeetestSDK/GeetestLib.cs)
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Security.Cryptography;
using System.Net;
using System.IO;

namespace PMS.WebApp.Models
{
    /// <summary>
    /// GeetestLib 极验验证C# SDK基本库
    /// </summary>
    public class GeetestLib
    {
        /// <summary>
        /// SDK版本号
        /// </summary>
        public const String version = "3.2.0";
        /// <summary>
        /// SDK开发语言
        /// </summary>
        public const String sdkLang = "csharp";
        /// <summary>
        /// 极验验证API URL
        /// </summary>
        protected const String apiUrl = "http://api.geetest.com";
        /// <summary>
        /// register url
        /// </summary>
        protected const String registerUrl = "/register.php";
        /// <summary>
        /// validate url
        /// </summary>
        protected const String validateUrl = "/validate.php";
        /// <summary>
        /// 极验验证API服务状态Session Key
        /// </summary>
        public const String gtServerStatusSessionKey = "gt_server_status";
        /// <summary>
        /// 极验验证二次验证表单数据 Chllenge
        /// </summary>
        public const String fnGeetestChallenge = "geetest_challenge";
        /// <summary>
        /// 极验验证二次验证表单数据 Validate
        /// </summary>
        public const String fnGeetestValidate = "geetest_validate";
        /// <summary>
        /// 极验验证二次验证表单数据 Seccode
        /// </summary>
        public const String fnGeetestSeccode = "geetest_seccode";
        private String userID = "";
        private String responseStr = "";
        private String captchaID = "";
        private String privateKey = "";

        /// <summary>
        /// 验证成功结果字符串
        /// </summary>
        public const int successResult = 1;
        /// <summary>
        /// 证结失败验果字符串
        /// </summary>
        public const int failResult = 0;
        /// <summary>
        /// 判定为机器人结果字符串
        /// </summary>
        public const String forbiddenResult = "forbidden";

        /// <summary>
        /// GeetestLib构造函数
        /// </summary>
        /// <param name="publicKey">极验验证公钥</param>
        /// <param name="privateKey">极验验证私钥</param>
        public GeetestLib(String publicKey, String privateKey)
        {
            this.privateKey = privateKey;
            this.captchaID = publicKey;
        }
        private int getRandomNum()
        {
            Random rand =new Random();
            int randRes = rand.Next(100);
            return randRes;
        }

        /// <summary>
        /// 验证初始化预处理
        /// </summary>
        /// <returns>初始化结果</returns>
        public Byte preProcess()
        {
            if (this.captchaID == null)
            {
                Console.WriteLine("publicKey is null!");
            }
            else
            {
                String challenge = this.registerChallenge();
                if (challenge.Length == 32)
                {
                    this.getSuccessPreProcessRes(challenge);
                    return 1;
                }
                else
                {
                    this.getFailPreProcessRes();
                    Console.WriteLine("Server regist challenge failed!");
                }
            }

            return 0;

        }
        public Byte preProcess(String userID)
        {
            if (this.captchaID == null)
            {
                Console.WriteLine("publicKey is null!");
            }
            else
            {
                this.userID = userID;
                String challenge = this.registerChallenge();
                if (challenge.Length == 32)
                {
                    this.getSuccessPreProcessRes(challenge);
                    return 1;
                }
                else
                {
                    this.getFailPreProcessRes();
                    Console.WriteLine("Server regist challenge failed!");
                }
            }

            return 0;

        }
        public String getResponseStr()
        {
            return this.responseStr;
        }
        /// <summary>
        /// 预处理失败后的返回格式串
        /// </summary>
        private void getFailPreProcessRes()
        {
            int rand1 = this.getRandomNum();
            int rand2 = this.getRandomNum();
            String md5Str1 = this.md5Encode(rand1 + "");
            String md5Str2 = this.md5Encode(rand2 + "");
            String challenge = md5Str1 + md5Str2.Substring(0, 2);
            this.responseStr = "{" + string.Format(
                 "\"success\":{0},\"gt\":\"{1}\",\"challenge\":\"{2}\"", 0,
                this.captchaID, challenge) + "}";
        }
        /// <summary>
        /// 预处理成功后的标准串
        /// </summary>
        private void getSuccessPreProcessRes(String challenge)
        {
            challenge = this.md5Encode(challenge + this.privateKey);
            this.responseStr ="{" + string.Format(
                "\"success\":{0},\"gt\":\"{1}\",\"challenge\":\"{2}\"", 1, 
                this.captchaID, challenge) + "}";
        }
        /// <summary>
        /// failback模式的验证方式
        /// </summary>
        /// <param name="challenge">failback模式下用于与validate一起解码答案， 判断验证是否正确</param>
        /// <param name="validate">failback模式下用于与challenge一起解码答案， 判断验证是否正确</param>
        /// <param name="seccode">failback模式下，其实是个没用的参数</param>
        /// <returns>验证结果</returns>
        public int failbackValidateRequest(String challenge, String validate, String seccode)
        {
            if (!this.requestIsLegal(challenge, validate, seccode)) return GeetestLib.failResult;
            String[] validateStr = validate.Split('_');
            String encodeAns = validateStr[0];
            String encodeFullBgImgIndex = validateStr[1];
            String encodeImgGrpIndex = validateStr[2];
            int decodeAns = this.decodeResponse(challenge, encodeAns);
            int decodeFullBgImgIndex = this.decodeResponse(challenge, encodeFullBgImgIndex);
            int decodeImgGrpIndex = this.decodeResponse(challenge, encodeImgGrpIndex);
            int validateResult = this.validateFailImage(decodeAns, decodeFullBgImgIndex, decodeImgGrpIndex);
            return validateResult;
        }
        private int validateFailImage(int ans, int full_bg_index, int img_grp_index)
        {
            const int thread = 3;
            String full_bg_name = this.md5Encode(full_bg_index + "").Substring(0, 10);
            String bg_name = md5Encode(img_grp_index + "").Substring(10, 10);
            String answer_decode = "";
            for (int i = 0;i < 9; i++)
            {
                if (i % 2 == 0) answer_decode += full_bg_name.ElementAt(i);
                else if (i % 2 == 1) answer_decode += bg_name.ElementAt(i);
            }
            String x_decode = answer_decode.Substring(4);
            int x_int = Convert.ToInt32(x_decode, 16);
            int result = x_int % 200;
            if (result < 40) result = 40;
            if (Math.Abs(ans - result) < thread) return GeetestLib.successResult;
            else return GeetestLib.failResult;
        }
        private Boolean requestIsLegal(String challenge, String validate, String seccode)
        {
            if (challenge.Equals(string.Empty) || validate.Equals(string.Empty) || seccode.Equals(string.Empty)) return false;
            return true;
        }

        /// <summary>
        /// 向gt-server进行二次验证
        /// </summary>
        /// <param name="challenge">本次验证会话的唯一标识</param>
        /// <param name="validate">拖动完成后server端返回的验证结果标识字符串</param>
        /// <param name="seccode">验证结果的校验码，如果gt-server返回的不与这个值相等则表明验证失败</param>
        /// <returns>二次验证结果</returns>
        public int enhencedValidateRequest(String challenge, String validate, String seccode)
        {
            if (!this.requestIsLegal(challenge, validate, seccode)) return GeetestLib.failResult;
            if (validate.Length > 0 && checkResultByPrivate(challenge, validate))
            {
                String query = "seccode=" + seccode + "&sdk=csharp_" + GeetestLib.version;
                String response = "";
                try
                {
                    response = postValidate(query);
                }
                catch (Exception e)
                {
                    Console.WriteLine(e);
                }
                if (response.Equals(md5Encode(seccode)))
                {
                    return GeetestLib.successResult;
                }
            }
            return GeetestLib.failResult;
        }
        public int enhencedValidateRequest(String challenge, String validate, String seccode, String userID)
        {
            if (!this.requestIsLegal(challenge, validate, seccode)) return GeetestLib.failResult;
            if (validate.Length > 0 && checkResultByPrivate(challenge, validate))
            {
                String query = "seccode=" + seccode + "&user_id=" + userID + "&sdk=csharp_" + GeetestLib.version;
                String response = "";
                try
                {
                    response = postValidate(query);
                }
                catch (Exception e)
                {
                    Console.WriteLine(e);
                }
                if (response.Equals(md5Encode(seccode)))
                {
                    return GeetestLib.successResult;
                }
            }
            return GeetestLib.failResult;
        }
        private String readContentFromGet(String url)
        {
            try
            {
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                request.Timeout = 20000;
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();
                Stream myResponseStream = response.GetResponseStream();
                StreamReader myStreamReader = new StreamReader(myResponseStream, Encoding.GetEncoding("utf-8"));
                String retString = myStreamReader.ReadToEnd();
                myStreamReader.Close();
                myResponseStream.Close();
                return retString;
            }
           catch
           {
               return "";     
           }

        }
        private String registerChallenge()
        {
            String url = "";
            if (string.Empty.Equals(this.userID))
            {
                url = string.Format("{0}{1}?gt={2}", GeetestLib.apiUrl, GeetestLib.registerUrl, this.captchaID);
            }
            else
            {
                url = string.Format("{0}{1}?gt={2}&user_id={3}", GeetestLib.apiUrl, GeetestLib.registerUrl, this.captchaID, this.userID);
            }
            string retString = this.readContentFromGet(url);
            return retString;
        }
        private Boolean checkResultByPrivate(String origin, String validate)
        {
            String encodeStr = md5Encode(privateKey + "geetest" + origin);
            return validate.Equals(encodeStr);
        }
        private String postValidate(String data)
        {
            String url = string.Format("{0}{1}", GeetestLib.apiUrl, GeetestLib.validateUrl);
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
            request.Method = "POST";
            request.ContentType = "application/x-www-form-urlencoded";
            request.ContentLength = Encoding.UTF8.GetByteCount(data);
            // 发送数据
            Stream myRequestStream = request.GetRequestStream();
            byte[] requestBytes = System.Text.Encoding.ASCII.GetBytes(data);
            myRequestStream.Write(requestBytes, 0, requestBytes.Length);
            myRequestStream.Close();

            HttpWebResponse response = (HttpWebResponse)request.GetResponse();
            // 读取返回信息
            Stream myResponseStream = response.GetResponseStream();
            StreamReader myStreamReader = new StreamReader(myResponseStream, Encoding.GetEncoding("utf-8"));
            string retString = myStreamReader.ReadToEnd();
            myStreamReader.Close();
            myResponseStream.Close();

            return retString;

        }
        private int decodeRandBase(String challenge)
        {
            String baseStr = challenge.Substring(32, 2);
            List<int> tempList = new List<int>();
            for(int i = 0; i < baseStr.Length; i++)
            {
                int tempAscii = (int)baseStr[i];
                tempList.Add((tempAscii > 57) ? (tempAscii - 87)
                    : (tempAscii - 48));
            }
            int result = tempList.ElementAt(0) * 36 + tempList.ElementAt(1);
            return result;
        }
        private int decodeResponse(String challenge, String str)
        {
            if (str.Length>100) return 0;
            int[] shuzi = new int[] { 1, 2, 5, 10, 50};
            String chongfu = "";
            Hashtable key = new Hashtable();
            int count = 0;
            for (int i=0;i<challenge.Length;i++)
            {
                String item = challenge.ElementAt(i) + "";
                if (chongfu.Contains(item)) continue;
                else
                {
                    int value = shuzi[count % 5];
                    chongfu += item;
                    count++;
                    key.Add(item, value);
                }
            }
            int res = 0;
            for (int i = 0; i < str.Length; i++) res += (int)key[str[i]+""];
            res = res - this.decodeRandBase(challenge);
            return res;
        }
        private String md5Encode(String plainText)
        {
            MD5CryptoServiceProvider md5 = new MD5CryptoServiceProvider();
            string t2 = BitConverter.ToString(md5.ComputeHash(UTF8Encoding.Default.GetBytes(plainText)));
            t2 = t2.Replace("-", "");
            t2 = t2.ToLower();
            return t2;
        }

    }
}
```
### 获取验证码
引入Jquery库
```html
<script src="~/Content/plugins/jquery/jquery-1.8.2.min.js"></script>
```
添加用于放置验证码的div（需要放到form表单中）
```html
<div id="geetest-container">

</div>
```
添加JS代码用于获取验证码
```html
<script>
    window.addEventListener('load', processGeeTest);

    function processGeeTest() {
        $.ajax({
            // 获取id，challenge，success（是否启用failback）
            url: "/Login/GeekTest",
            type: "get",
            dataType: "json", // 使用jsonp格式
            success: function (data) {
                // 使用initGeetest接口
                // 参数1：配置参数，与创建Geetest实例时接受的参数一致
                // 参数2：回调，回调的第一个参数验证码对象，之后可以使用它做appendTo之类的事件
                initGeetest({
                    gt: data.gt,
                    challenge: data.challenge,
                    product: "float", // 产品形式
                    offline: !data.success
                },
                    handler);
            }
        });
    }

    var handler = function (captchaObj) {
        // 将验证码加到id为captcha的元素里
        captchaObj.appendTo("#geetest-container");

        captchaObj.onSuccess = function (e) {
            console.log(e);
        }

    };
</script>
```
processGeeTest方法中我们异步请求的地址“/Login/GeekTest”就是获取验证码是后台需要执行的方法
```csharp
public ActionResult GeekTest()
{
    return Content(GetCaptcha(),"application/json");
}

private string GetCaptcha()
{
    var geetest = new GeetestLib("3594e0d834df77cedc7351a02b5b06a4", "b961c8081ce88af7e32a3f45d00dff84");
    var gtServerStatus = geetest.preProcess();
    Session[GeetestLib.gtServerStatusSessionKey] = gtServerStatus;
    return geetest.getResponseStr();
}
```
### 校验验证码
注意，当提交form表单时，会将三个和极验有关的参数传到后台方法（geetest_challenge、geetest_validate、geetest_seccode），若验证码未验证成功，则参数为空值。

后台验证方法为：
```csharp
private bool CheckGeeTestResult()
{
    var geetest = new GeetestLib("3594e0d834df77cedc7351a02b5b06a4", "b961c8081ce88af7e32a3f45d00dff84 ");
    var gtServerStatusCode = (byte)Session[GeetestLib.gtServerStatusSessionKey];
    var userId = (string)Session["userID"];

    var challenge = Request.Form.Get(GeetestLib.fnGeetestChallenge);
    var validate = Request.Form.Get(GeetestLib.fnGeetestValidate);
    var seccode = Request.Form.Get(GeetestLib.fnGeetestSeccode);
    var result = gtServerStatusCode == 1 ? geetest.enhencedValidateRequest(challenge, validate, seccode, userId) : geetest.failbackValidateRequest(challenge, validate, seccode);
    return result == 1;
}
```
我们可以在表单中判断验证码是否成功校验：
```csharp
public ActionResult Login()
{
    if (!CheckGeeTestResult())
        return Content("no:请先完成验证操作。");
    ....
}
```