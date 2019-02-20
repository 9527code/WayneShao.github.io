---
title: .Net 讯飞语音识别Demo
tags:
  - ASR
  - TTS
  - NAudio
  - 讯飞语音识别
categories: []
abbrlink: 43271
date: 2018-03-02 11:35:00
---
[讯飞语音识别](http://www.xfyun.cn/services/voicedictation)官方号称具有以下六个优势：
1. 超过95%的准确率
2. 支持多种语种和方言
3. 方便快捷的信息沟通
4. 个性的语音识别
5. 中文标点智能预测
6. 支持垂直领域和应用级听写

![](http://qiniucdn.wayneshao.com/20180302113535116/20180302114420073.png)
<!--more-->
## 获得APPID和调用Dll
在讯飞开放平台的[控制台](http://console.xfyun.cn/app/myapp)新建一个应用，平台选择Windows，新建完成后为应用添加语音听写服务。
![](http://qiniucdn.wayneshao.com/20180302113535116/20180302114712451.png)
下载SDK
![](http://qiniucdn.wayneshao.com/20180302113535116/20180302114758665.png)

解压出你下载的压缩包bin目录中的msc.dll等待使用
![](http://qiniucdn.wayneshao.com/20180302113535116/20180302114914214.png)
**注意：下面步骤里的Dll必须使用自行下载的版本，此Dll并不通用**
## Coding
### 识别文件实现
```csharp
/// <summary>
/// 执行语音识别的异步方法
/// </summary>
/// <param name="inFile">音频文件，pcm无文件头，采样率16k，数据16位，单声道</param>
/// <param name="outFile">输出识别结果到文件</param>
public void Audio2TxtAsync(string inFile, string outFile = null)
{
    var dlt = new DltSpeek(Audio2Txt);
    dlt.BeginInvoke(inFile, outFile, null, null);
}

/// <summary>
/// 进行声音识别
/// </summary>
/// <param name="inFile">音频文件，pcm无文件头，采样率16k，数据16位，单声道</param>
/// <param name="outFile">输出识别结果到文件</param>
public void Audio2Txt(string inFile, string outFile = null)
{
    var ret = 0;
    var result = "";
    try
    {
        //模拟录音，输入音频
        if (!File.Exists(inFile)) throw new Exception("文件" + inFile + "不存在！");
        if (inFile.Substring(inFile.Length - 3, 3).ToUpper() != "WAV" && inFile.Substring(inFile.Length - 3, 3).ToUpper() != "PCM")
            throw new Exception("音频文件格式不对！");
        var fp = new FileStream(inFile, FileMode.Open);
        if (inFile.Substring(inFile.Length - 3, 3).ToUpper() == "WAV") fp.Position = 44;
        var buff = new byte[BufferNum];
        var bp = Marshal.AllocHGlobal(BufferNum);
        int len;
        var status = AudioStatus.IsrAudioSampleContinue;
        var epStatus = EpStatus.IsrEpNull;
        var recStatus = RecogStatus.IsrRecNull;
        var rsltStatus = RecogStatus.IsrRecNull;
        //ep_status        端点检测（End-point detected）器所处的状态
        //rec_status       识别器所处的状态
        //rslt_status      识别器所处的状态
        while (fp.Position != fp.Length)
        {
            len = fp.Read(buff, 0, BufferNum);
            Marshal.Copy(buff, 0, bp, buff.Length);
            //开始向服务器发送音频数据
            ret = AsrDll.QISRAudioWrite(_sessID, bp, (uint)len, status, ref epStatus, ref recStatus);
            if (ret != 0)
            {
                fp.Close();
                throw new Exception("QISRAudioWrite err,errCode=" + ((ErrorCode)ret).ToString("G"));
            }
            //服务器返回部分结果
            if (recStatus == RecogStatus.IsrRecStatusSuccess)
            {
                var p = AsrDll.QISRGetResult(_sessID, ref rsltStatus, WaitTime, ref ret);
                if (p != IntPtr.Zero)
                {
                    var tmp = FlyTts.Ptr2Str(p);
                    DataArrived?.Invoke(this, new DataArrivedEventArgs(tmp));
                    result += tmp;
                    Console.WriteLine(@"返回部分结果！:" + tmp);
                }
            }
            Thread.Sleep(500);
        }
        fp.Close();

        //最后一块数据
        status = AudioStatus.IsrAudioSampleLast;

        ret = AsrDll.QISRAudioWrite(_sessID, bp, 1, status, ref epStatus, ref recStatus);
        if (ret != 0) throw new Exception("QISRAudioWrite write last audio err,errCode=" + ((ErrorCode)ret).ToString("G"));
        Marshal.FreeHGlobal(bp);
        var loopCount = 0;
        //最后一块数据发完之后，循环从服务器端获取结果
        //考虑到网络环境不好的情况下，需要对循环次数作限定
        do
        {
            var p = AsrDll.QISRGetResult(_sessID, ref rsltStatus, WaitTime, ref ret);
            if (p != IntPtr.Zero)
            {
                var tmp = FlyTts.Ptr2Str(p);
                DataArrived?.Invoke(this, new DataArrivedEventArgs(tmp)); //激发识别数据到达事件
                result += tmp;
                Console.WriteLine(@"传完音频后返回结果！:" + tmp);
            }
            if (ret != 0) throw new Exception("QISRGetResult err,errCode=" + ((ErrorCode)ret).ToString("G"));
            Thread.Sleep(200);
        } while (rsltStatus != RecogStatus.IsrRecStatusSpeechComplete && loopCount++ < 30);
        if (outFile != null)
        {
            var fout = new FileStream(outFile, FileMode.OpenOrCreate);
            fout.Write(Encoding.Default.GetBytes(result), 0, Encoding.Default.GetByteCount(result));
            fout.Close();
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
    finally
    {
        ret = AsrDll.QISRSessionEnd(_sessID, string.Empty);
        ret = AsrDll.MSPLogout();
        IsrEnd?.Invoke(this, new EventArgs()); //通知识别结束
    }
}
```
### 录音实现
```csharp
public void StartRecoding()
{
    WaveMonitor = new WaveInEvent { WaveFormat = new WaveFormat(16000, 16, 1) };

    if (!Directory.Exists("temp"))
        Directory.CreateDirectory("temp");
    _fileName = Path.Combine("temp", Guid.NewGuid() + ".wav");

    Writer = new WaveFileWriter(_fileName, WaveMonitor.WaveFormat);

    WaveMonitor.DataAvailable += (s, a) => Writer.Write(a.Buffer, 0, a.BytesRecorded);
    WaveMonitor.RecordingStopped += (s, a) => { Writer?.Dispose(); WaveMonitor?.Dispose(); };

    WaveMonitor.StartRecording();
}

public void StopRecoding()
{
    WaveMonitor.StopRecording();
    Writer?.Close();

    Audio2Txt(_fileName);
}
```
### 完整源码
托管在[GitHub](https://github.com/WayneShao/iFlyDotNet)


## 结论
讯飞语音识别实测识别率其实并没有比百度好多少，准确率在服务提供商看来是越精确越好，但在实际应用中90%和95%差距并不大，故虽然讯飞看起来在数据上更好一些，但是API易用性实在比较差，还是更推荐百度一些。