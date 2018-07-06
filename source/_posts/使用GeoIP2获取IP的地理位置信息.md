---
title: .Net 使用GeoIP2获取IP的地理位置信息
tags:
  - GeoIP2
  - .Net
  - IP
abbrlink: 20420
date: 2018-02-21 21:25:39
---
[GeoIP® 数据库&服务：业界领先的IP智能](https://www.maxmind.com/zh/geoip2-services-and-databases)，MaxMind GeoIP2 服务能识别互联网用户的地点位置与其他特征，应用广泛，包括个性化定制内容、诈欺检测、广告定向、网站流量分析、执行规定、地理目标定位、地理围栏定位 (geo-fencing)以及数字版权管理。
<!--more-->
![](http://p4au3q1y8.bkt.clouddn.com/20180221212532042/20180221093334576.png)
* [MaxMind是IP地理定位准确性的行业领导者.](https://blog.maxmind.com/2014/01/31/who-has-the-most-accurate-ip-geolocation-data/)
* [按照不同国家，比较MaxMind GeoIP2数据服务与数据库的准确性](https://www.maxmind.com/zh/geoip2-city-database-accuracy)。

[GeoIP2精准版服务](https://www.maxmind.com/zh/geoip2-precision-services)向您提供本公司最准确的数据，为您省去在您服务器上托管数据或部署更新项目的麻烦。 我们的精准版服务产品可通过API或文件手动上传方式使用，为您提供最新的数据。

[MaxMind的GeoIP2数据库](https://www.maxmind.com/zh/geoip2-databases)为大容量环境提供IP智能数据。 通过在本地托管我们的数据库，您既可避免网络延迟问题，又可避免按每次查询计价的费用。

GeoIP 分为商业版和免费版，免费版比商业版精度差了许多，经测试对于城市定位确实有差距，能否接受看你的精度要求！

## 免费版介绍
免费版仅有数据库服务，目前有两个版本
1、GeoLite 版本，网上流传较广，数据库类型为 dat 格式文件，库文件较小、未进行精准度测试且不再更新。
2、GeoLite2版本，目前最新版本，数据库文件为 mmdb 格式或csv格式。
[**GeoLite2 特性**](https://dev.maxmind.com/geoip/geoip2/geoip2-city-country-csv-databases/)
## 下载数据库
### GeoIP数据库
[**GeoIP2数据库**](https://www.maxmind.com/zh/geoip2-databases)
本地维护的数据库适用于容量大、延迟性低的环境，购买机构可以获取站点许可证，即可在公司内进行无限次使用。

* 对于选定地点，含有简体中文、法文、德文、日文、西班牙文、巴西葡萄牙文及俄文版的本地化名称
* 为多数常用语言提供开放源代码的API
* 可提供自动更新

### GeoLite2 开源数据库
#### 数据库
[GeoLite2数据库](https://dev.maxmind.com/zh-hans/geoip/geoip2/geolite2-开源数据库/)是GeoIP2的免费版，其准确率稍低于付费版。
#### 技术支持
MaxMind 不为免费数据库提供技术支持。如果您有问题请前往[stackoverflow’s GeoIP问题以及解答。](http://stackoverflow.com/questions/tagged/geoip)
#### 许可证
GeoLite2使用的是开源许可证：[Creative Commons Attribution-ShareAlike 3.0 Unported License](https://creativecommons.org/licenses/by-sa/3.0/deed.zh_TW). 您只需要在页面中添加如下代码即可：
```html
该产品使用MaxMind公司的GeoLite2数据，可以在此获取：
<a href="http://www.maxmind.com">http://www.maxmind.com</a>.
```
官方提供 [二次销售许可证](http://www.maxmind.com/zh/geolite2-developer-package).

#### 下载

| 数据库 | 	[MaxMind DB](http://maxmind.github.io/MaxMind-DB/) 二进制格式, 压缩  | [CSV 格式](https://dev.maxmind.com/geoip/geoip2/geoip2-city-country-csv-databases/), 压缩 |
|:------------:|:---------------:|:-----:|
| GeoLite2 城市 | 	[Download](http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz) ([md5 校验](http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.md5)) | 	[Download](http://geolite.maxmind.com/download/geoip/database/GeoLite2-City-CSV.zip) ([md5 校验](http://geolite.maxmind.com/download/geoip/database/GeoLite2-City-CSV.zip.md5))|
| GeoLite2 国家 | 	[Download](http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.mmdb.gz) ([md5 校验](http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.md5)) | 	[Download](http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country-CSV.zip) ([md5 校验](http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country-CSV.zip.md5)) |

#### 更新数据库
你可以使用 [GeoIP 更新](https://dev.maxmind.com/zh-hans/geoip/geoipupdate/)来自动更新您的数据库。
#### MaxMind API 接口
参阅 [GeoIP2 可下载数据库](https://dev.maxmind.com/zh-hans/geoip/geoip2/downloadable/#MaxMind_APIs) 以下载API。付费版和免费版API互通。
## .Net调用MaxMind API
.Net调用MaxMind API可以使用官方发布的[nuget包](https://www.nuget.org/packages/MaxMind.GeoIP2/)，官方提供了[文档](http://maxmind.github.io/GeoIP2-dotnet/)和[源码地址](https://github.com/maxmind/GeoIP2-dotnet)。
### 安装Nuget包
```
Install-Package MaxMind.GeoIP2
```
### 代码调用
因GeoLite2只提供了City和Country两个数据库版本。
故只能进行这两种调用方式，调用方式非常简单
#### City
```csharp
using (var reader = new DatabaseReader("GeoLite2-City.mmdb"))
{
    var city = reader.City("65.49.134.29");
}
```
city即为查询结果，结构如下：
```javascript
{
    "city": {
        "geoname_id": 5125591,
        "names": {
            "en": "Macedon"
        }
    },
    "continent": {
        "code": "NA",
        "geoname_id": 6255149,
        "names": {
            "de": "Nordamerika",
            "en": "North America",
            "es": "Norteamérica",
            "fr": "Amérique du Nord",
            "ja": "北アメリカ",
            "pt-BR": "América do Norte",
            "ru": "Северная Америка",
            "zh-CN": "北美洲"
        }
    },
    "country": {
        "geoname_id": 6252001,
        "is_in_european_union": false,
        "iso_code": "US",
        "names": {
            "de": "USA",
            "en": "United States",
            "es": "Estados Unidos",
            "fr": "États-Unis",
            "ja": "アメリカ合衆国",
            "pt-BR": "Estados Unidos",
            "ru": "США",
            "zh-CN": "美国"
        }
    },
    "location": {
        "accuracy_radius": 20,
        "latitude": 43.1089,
        "longitude": -77.3226,
        "metro_code": 538,
        "time_zone": "America/New_York"
    },
    "maxmind": {},
    "postal": {
        "code": "14502"
    },
    "registered_country": {
        "geoname_id": 6252001,
        "is_in_european_union": false,
        "iso_code": "US",
        "names": {
            "de": "USA",
            "en": "United States",
            "es": "Estados Unidos",
            "fr": "États-Unis",
            "ja": "アメリカ合衆国",
            "pt-BR": "Estados Unidos",
            "ru": "США",
            "zh-CN": "美国"
        }
    },
    "represented_country": {
        "is_in_european_union": false,
        "names": {}
    },
    "subdivisions": [
        {
            "geoname_id": 5128638,
            "iso_code": "NY",
            "names": {
                "de": "New York",
                "en": "New York",
                "es": "Nueva York",
                "fr": "New York",
                "ja": "ニューヨーク州",
                "pt-BR": "Nova Iorque",
                "ru": "Нью-Йорк",
                "zh-CN": "纽约州"
            }
        }
    ],
    "traits": {
        "ip_address": "66.66.66.66",
        "is_anonymous": false,
        "is_anonymous_proxy": false,
        "is_anonymous_vpn": false,
        "is_hosting_provider": false,
        "is_legitimate_proxy": false,
        "is_public_proxy": false,
        "is_satellite_provider": false,
        "is_tor_exit_node": false
    }
}
```
其中包含了比较详细的信息，有具体的经纬度。
#### Country
```csharp
using (var reader = new DatabaseReader("GeoLite2-Country.mmdb"))
{
    var country = reader.Country("66.66.66.66");
}
```
country即为查询结果，结构如下：
```javascript
{
    "continent": {
        "code": "NA",
        "geoname_id": 6255149,
        "names": {
            "de": "Nordamerika",
            "en": "North America",
            "es": "Norteamérica",
            "fr": "Amérique du Nord",
            "ja": "北アメリカ",
            "pt-BR": "América do Norte",
            "ru": "Северная Америка",
            "zh-CN": "北美洲"
        }
    },
    "country": {
        "geoname_id": 6252001,
        "is_in_european_union": false,
        "iso_code": "US",
        "names": {
            "de": "USA",
            "en": "United States",
            "es": "Estados Unidos",
            "fr": "États-Unis",
            "ja": "アメリカ合衆国",
            "pt-BR": "Estados Unidos",
            "ru": "США",
            "zh-CN": "美国"
        }
    },
    "maxmind": {},
    "registered_country": {
        "geoname_id": 6252001,
        "is_in_european_union": false,
        "iso_code": "US",
        "names": {
            "de": "USA",
            "en": "United States",
            "es": "Estados Unidos",
            "fr": "États-Unis",
            "ja": "アメリカ合衆国",
            "pt-BR": "Estados Unidos",
            "ru": "США",
            "zh-CN": "美国"
        }
    },
    "represented_country": {
        "is_in_european_union": false,
        "names": {}
    },
    "traits": {
        "ip_address": "66.66.66.66",
        "is_anonymous": false,
        "is_anonymous_proxy": false,
        "is_anonymous_vpn": false,
        "is_hosting_provider": false,
        "is_legitimate_proxy": false,
        "is_public_proxy": false,
        "is_satellite_provider": false,
        "is_tor_exit_node": false
    }
}
```
country数据库中的信息的详细程度较之city就差了很多，但数据库大小仅为city的 1/20，视使用场景来决定使用对应的数据库。
