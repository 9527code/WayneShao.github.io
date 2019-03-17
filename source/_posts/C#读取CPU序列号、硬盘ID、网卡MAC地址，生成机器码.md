---
title: 'C#读取CPU序列号、硬盘ID、网卡MAC地址，生成机器码'
tags:
  - 'C#'
  - 硬件信息
abbrlink: 13625
date: 2017-08-29 17:08:12
categories:
  - 踩坑笔记
---
话不多说，直接上代码，类库中的机器码使用序列号、硬盘ID、网卡MAC地址组合取MD5生成。
<!-- more -->
```csharp
using System;
using System.Linq;
using System.Management;
using System.Security.Cryptography;
using System.Text;

namespace WayneShao.Common
{
    internal static class MachineCode
    {
        private static string _machineCodeString;
        public static string Value
        {
            get
            {
                if (string.IsNullOrWhiteSpace(_machineCodeString))
                    _machineCodeString = GetMD5($"{CPUCode}_{HDId}_{MacAddress}");
                return _machineCodeString;
            }
        }

        private static string _cpuCode;
        public static string CPUCode => _cpuCode ?? (_cpuCode = GetCPUInfo());

        private static string _hdId;
        public static string HDId => _hdId ?? (_hdId = GetHDid());

        private static string _macAddress;
        public static string MacAddress => _macAddress ?? (_macAddress = GetMACAddress());

        ///   <summary> 
        ///   获取cpu序列号     
        ///   </summary> 
        ///   <returns> string </returns> 
        private static string GetCPUInfo()
        {
            using (var cimobject = new ManagementClass("Win32_Processor"))
            {
                var hdids = cimobject.GetInstances().Cast<ManagementObject>().Select(o => o.Properties["ProcessorId"].Value).Cast<string>().ToArray();
                return hdids.Any() ? hdids.First() : string.Empty;
            }
        }

        ///   <summary> 
        ///   获取硬盘ID     
        ///   </summary> 
        ///   <returns> string </returns> 
        private static string GetHDid()
        {
            using (var cimobject1 = new ManagementClass("Win32_DiskDrive"))
            {
                var hdids = cimobject1.GetInstances().Cast<ManagementObject>().Select(o => o.Properties["Model"].Value).Cast<string>().ToArray();
                return hdids.Any() ? hdids.First() : string.Empty;
            }
        }

        ///   <summary> 
        ///   获取网卡硬件地址 
        ///   </summary> 
        ///   <returns> string </returns> 
        private static string GetMACAddress()
        {
            using (var mc = new ManagementClass("Win32_NetworkAdapterConfiguration"))
            {
                var macs = mc.GetInstances().Cast<ManagementObject>().Where(o => (bool)o["IPEnabled"]).Select(o => o["MacAddress"].ToString()).ToArray();
                return macs.Any() ? macs.First() : string.Empty;
            }
        }

        ///   <summary> 
        ///   获取字符串的MD5值
        ///   </summary> 
        ///   <returns> string </returns> 
        public static string GetMD5(string source)
        {
            var result = Encoding.Default.GetBytes(source);
            var md5 = new MD5CryptoServiceProvider();
            var output = md5.ComputeHash(result);
            return BitConverter.ToString(output).Replace("-", "").ToLower();
        }

    }
}
```
另附MSDN中关于 WMI Class 的文档供大家参考
https://msdn.microsoft.com/zh-cn/library/aa394173(VS.85).aspx