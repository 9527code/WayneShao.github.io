---
title: 【爬虫学习笔记】基于 SimHash 的去重复处理模块ContentSeen的构建
tags:
  - 爬虫
abbrlink: 38230
date: 2016-09-13 20:00:39
---
Internet上的一些站点常常存在着镜像网站（mirror），即两个网站的内容一样但网页对应的域名不同。这样会导致对同一份网页爬虫重复抓取多次。为了避免这种情况，对于每一份抓取到的网页，它首先需要进入ContentSeen模块。该模块会判断网页的内容是否和已下载过的某个网页的内容一致，如果一致，则该网页不会再被送去进行下一步的处理。这样的做法能够显著的降低爬虫需要下载的网页数。至于如果判断两个网页的内容是否一致，一般的思路是这样的：并不会去直接比较两个网页的内容，而是将网页的内容经过计算生成FingerPrint（指纹），通常FingerPrint是一个固定长度的字符串，要比网页的正文短很多。如果两个网页的FingerPrint一样，则认为它们内容完全相同。

为了完成这一模块，首先我们需要一个强大的指纹算法，将我们的网页内容计算成指纹存入数据库，下次直接判断指纹在保存前通过指纹的对比即可成功完成去重复操作。
<!-- more -->
## SmiHash算法
首先来看一下大名鼎鼎的Google公司使用的网页去重复算法SimHash吧：

GoogleMoses Charikar发表的一篇论文“detecting near-duplicates for web crawling”中提出了simhash算法，专门用来解决亿万级别的网页的去重任务。

SimHash作为locality sensitive hash（局部敏感哈希）的一种：

其主要思想是降维，将高维的特征向量映射成低维的特征向量，通过两个向量的Hamming Distance来确定文章是否重复或者高度近似。

其中，Hamming Distance，又称汉明距离，在信息论中，两个等长字符串之间的汉明距离是两个字符串对应位置的不同字符的个数。也就是说，它就是将一个字符串变换成 另外一个字符串所需要替换的字符个数。例如：1011101 与 1001001 之间的汉明距离是 2。至于我们常说的字符串编辑距离则是一般形式的汉明距离。

如此，通过比较多个文档的SimHash值的海明距离，可以获取它们的相似度。

详情可以看这里[SimHash算法](http://www.cnblogs.com/chenying99/p/3830728.html)
## 算法实现
下面我们进行代码实现:
```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace Crawler.Common
{
    public class SimHashAnalyser
    {

        private const int HashSize = 32;

        public static float GetLikenessValue(string needle, string haystack, TokeniserType type = TokeniserType.Overlapping)
        {
            var needleSimHash = GetSimHash(needle, type);
            var hayStackSimHash = GetSimHash(haystack, type);
            return GetLikenessValue(needleSimHash, hayStackSimHash);
        }

        public static float GetLikenessValue(int needleSimHash, int hayStackSimHash)
        {
            return (HashSize - GetHammingDistance(needleSimHash, hayStackSimHash)) / (float)HashSize;
        }

        private static IEnumerable<int> DoHashTokens(IEnumerable<string> tokens)
        {
            return tokens.Select(token => token.GetHashCode()).ToList();
        }

        private static int GetHammingDistance(int firstValue, int secondValue)
        {
            var hammingBits = firstValue ^ secondValue;
            var hammingValue = 0;
            for (var i = 0; i < 32; i++)
                if (IsBitSet(hammingBits, i))
                    hammingValue += 1;
            return hammingValue;
        }

        private static bool IsBitSet(int b, int pos)
        {
            return (b & (1 << pos)) != 0;
        }


        public static int GetSimHash(string input)
        {
            return GetSimHash(input, TokeniserType.Overlapping);
        }

        public static int GetSimHash(string input, TokeniserType tokeniserType)
        {
            ITokeniser tokeniser;
            if (tokeniserType == TokeniserType.Overlapping)
                tokeniser = new OverlappingStringTokeniser();
            else
                tokeniser = new FixedSizeStringTokeniser();

            var hashedtokens = DoHashTokens(tokeniser.Tokenise(input));
            var vector = new int[HashSize];
            for (var i = 0; i < HashSize; i++)
                vector[i] = 0;

            foreach (var value in hashedtokens)
                for (var j = 0; j < HashSize; j++)
                    if (IsBitSet(value, j))
                        vector[j] += 1;
                    else
                        vector[j] -= 1;
            var fingerprint = 0;
            for (var i = 0; i < HashSize; i++)
                if (vector[i] > 0)
                    fingerprint += 1 << i;
            return fingerprint;
        }

    }

    public interface ITokeniser
    {
        IEnumerable<string> Tokenise(string input);
    }

    public class FixedSizeStringTokeniser : ITokeniser
    {
        private readonly ushort _tokensize;
        public FixedSizeStringTokeniser(ushort tokenSize = 5)
        {
            if (tokenSize < 2)
                throw new ArgumentException("Token 不能超出范围");
            if (tokenSize > 127)
                throw new ArgumentException("Token 不能超出范围");
            _tokensize = tokenSize;
        }

        public IEnumerable<string> Tokenise(string input)
        {
            var chunks = new List<string>();
            var offset = 0;
            while (offset < input.Length)
            {
                chunks.Add(new string(input.Skip(offset).Take(_tokensize).ToArray()));
                offset += _tokensize;
            }
            return chunks;
        }

    }

    public class OverlappingStringTokeniser : ITokeniser
    {

        private readonly ushort _chunkSize;
        private readonly ushort _overlapSize;

        public OverlappingStringTokeniser(ushort chunkSize = 4, ushort overlapSize = 3)
        {
            if (chunkSize <= overlapSize)
                throw new ArgumentException("Chunck 必须大于 overlap");
            _overlapSize = overlapSize;
            _chunkSize = chunkSize;
        }

        public IEnumerable<string> Tokenise(string input)
        {
            var result = new List<string>();
            var position = 0;
            while (position < input.Length - _chunkSize)
            {
                result.Add(input.Substring(position, _chunkSize));
                position += _chunkSize - _overlapSize;
            }
            return result;
        }


    }

    public enum TokeniserType
    {
        Overlapping,
        FixedSize
    }
}
```
## 调用
调用方法如下:
```csharp
var s1 = "the cat sat on the mat.";
var s2 = "the cat sat on a mat.";

var similarity = SimHashAnalyser.GetLikenessValue(s1, s2);

Console.Clear();
Console.WriteLine("相似度: {0}%", similarity * 100);
Console.ReadKey();
```
输出为:
**相似度: 78.125%**
## 封装
接下来就是对ContentSeen模块的简单封装:
```csharp
using Crawler.Common;

namespace Crawler.Processing
{
    /// <summary>
    /// 对于每一份抓取到的网页，它首先需要进入Content Seen模块。该模块会判断网页的内容是否和已下载过的某个网页的内容一致，如果一致，则该网页不会再被送去进行下一步的处理。
    /// </summary>
    public class ContentSeen
    {
        public static int GetFingerPrint(string html)
        {
            return SimHashAnalyser.GetSimHash(html);
        }

        public static float Similarity(int print1, int print2)
        {
            return SimHashAnalyser.GetLikenessValue(print1, print2);
        }

    }
}
```