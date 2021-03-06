---
layout: post
title: "断点续传"
subtitle: "这篇博客主要介绍单线程和多线程分别如何实现HTTP协议下的资源分段下载，底部附代码链接。"
date: 2016-05-16
author: LuJiangBo
category: HTTP
tags: HTTP
finished: true
---

## 意图   
模拟迅雷等下载工具，将一个大文件拆分成多次请求。

## 条件  
服务端资源地址必须支持范围请求。

判断是否支持的方法只需要看Responses Header中是否带有Accept-Ranges响应头。

`Accept-Ranges  表明服务器是否支持指定范围请求及哪种类型的分段请求   Accept-Ranges: bytes`

## 原理

涉及到的主要Request Header  
 &emsp;&emsp;`Range:  只请求实体的一部分，指定范围  Range: bytes=500-999`  
 &emsp;&emsp;`If-None-Match: 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 `  

 涉及到的主要Response Header  
 &emsp;&emsp;`Accept-Ranges:  表明服务器是否支持指定范围请求及哪种类型的分段请求   Accept-Ranges: bytes`  
 &emsp;&emsp;`ETag:   请求变量的实体标签的当前值   ETag: “737060cd8c284d8af7ad3082f209582d”`  
 &emsp;&emsp;`Content-Length: 响应体的长度  Content-Length: 348`  

单线程主要流程：  
&emsp;&emsp;第一步：发送一个HTTP HEAD请求，向服务器请求资源的响应头，判断是否支持范围请求。［若不支持，就不需要走下去了］  
&emsp;&emsp;第二步：拿到资源的ETag值，以及Content-Length值，保存下来。  
&emsp;&emsp;第三步：根据你一次想下载多少字节的数据，通过ContentLength计算出需要请求的总次数。  
&emsp;&emsp;第四步：将需要请求的数据范围写到Range请求头中，发送请求。  
&emsp;&emsp;第五步：保存数据到同一个文件。  
&emsp;&emsp;第六步：若请求的次数还没完成，回到第四步，若完成了，结束进程。

多线程主要流程：  
&emsp;&emsp;基本思路与单线程一致，区别在于多线程因为一次有多个请求同时发出，所以拿到的数据不一定与资源顺序一致，所以不能像单线程那样下载一部分，保存一部分。  
&emsp;&emsp;基于上面的问题，我想到的解决方案是创建一个Dictionary对象，每次请求到数据将数据连同它对于的请求编号保存到队列中。当所以线程都结束后，调用写方法，将队列中的数据排序以后依次写入到文件当中。

## 代码实现

main:
{% highlight c# %}
public static System.Diagnostics.Stopwatch watch2 = new System.Diagnostics.Stopwatch();
        static void Main(string[] args)
        {
            //测试，资源保存的路径
            string folderPath = ("D:/VSTest/Test0509/download");
            //测试，资源保存的文件名
            string fileName1 = "/image1.jpg";
            //测试 Url
            string url = "http://manyou.mobi/data/image/2e/a9/2ea9378eab9fb16a354154f5a7f4cad5.jpg";
            
            //单线程实现类
            SingleDemo singleDemo = new SingleDemo();

            //创建一个计时器
            System.Diagnostics.Stopwatch watch1 = new System.Diagnostics.Stopwatch();

            watch1.Start();

            singleDemo.DownloadImgSingleThread(url, folderPath, fileName1);

            watch1.Stop();

            Console.WriteLine("Single thread method using time is " + watch1.ElapsedMilliseconds + " millisecond");
            //-----------------------------------------------------------

            string fileName2 = "/image2.jpg";
            //多线程实现类
            MultiDemo multiDemo = new MultiDemo();

            Program.watch2.Start();

            multiDemo.DownloadImgMultiThread(url, folderPath, fileName2);
          
         
            Console.ReadLine();

        }  
{% endhighlight %}

SingleDemo.cs:  
{% highlight c# %}

using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
namespace 断点下载
{
    public class SingleDemo
    {
        public void DownloadImgSingleThread(string url, string folderPath, string fileName)
        {

            HttpResponseMessage responseMessage = GetContentHead(url);
            ShowResponseHeaders(responseMessage);

            //获取当前资源的ETag值
            string eTag = responseMessage.Headers.ETag.Tag;

            //获取当前资源ContentLength
            long? contentLength = responseMessage.Content.Headers.ContentLength;

            //设置单次请求点字节数
            int requestRangeContent = 1024 * 512;

            //根据每次请求0.5M大小的资源将总length划分需要请求的总次数，向上取整
            int requestCount = (int)Math.Ceiling((long)contentLength * 1.0 / requestRangeContent);



            //循环获取资源，追加到文件中
            for (int i = 0; i < requestCount; i++)
            {

                HttpResponseMessage responseHeader = GetContentHead(url);
                string responseETag = responseHeader.Headers.ETag.Tag;
                //get前先发送一个HEAD请求，查看当前资源的ETag是否发生改变(也不需要每次都操作)
                //相同则继续下载数据，
                //不同表示资源发生改变，当前本地已下载部分已经不可用，删除已下载部分
                if (string.Equals(eTag, responseETag))
                {
                    byte[] data = GetScopeContent(url, eTag, i * requestRangeContent, (i + 1) * requestRangeContent > contentLength ? contentLength : (i + 1) * requestRangeContent);
                    SaveFile(fileName, folderPath, data);
                }
                else
                {
                    File.Delete(folderPath + fileName);
                    break;
                }
            }

        }

        //获取responseHeader
        private HttpResponseMessage GetContentHead(string url)
        {
            Uri uri = new Uri(url);
            HttpClient client = new HttpClient();
            HttpRequestMessage requstMessage = new HttpRequestMessage(HttpMethod.Head, uri);
            HttpResponseMessage responseMessage = client.SendAsync(requstMessage).Result;
            client.Dispose();
            return responseMessage;
        }
        //获取指定路径下资源的特定范围字节数据
        private byte[] GetScopeContent(string url, string eTag, long? from, long? to)
        {
            Uri uri = new Uri(url);
            HttpClient client = new HttpClient();

            //If-None-Match:只有在eTag值发生改变时才返回数据status 200，否则返回status 304
            // client.DefaultRequestHeaders.IfNoneMatch.Add(new EntityTagHeaderValue(eTag));

            //If-Range :只有在eTag值发生改变时才rang范围的部分数据 status 206，否则返回整个文档status 200
            client.DefaultRequestHeaders.IfRange = new RangeConditionHeaderValue(eTag);
            client.DefaultRequestHeaders.Range = new RangeHeaderValue(from, to);
            byte[] dataBytes = client.GetByteArrayAsync(uri).Result;

            return dataBytes;
        }
        //将资源保存到文件中
        private void SaveFile(string fileName, string folderPath, byte[] fileBytes)
        {


            if (!Directory.Exists(folderPath))
            {
                Directory.CreateDirectory(folderPath);
            }
            if (!System.IO.File.Exists(folderPath + fileName))
            {
                File.Create(folderPath + fileName).Dispose();

            }
            FileStream fileStream = new FileStream(folderPath + fileName, FileMode.Append);
            fileStream.Write(fileBytes, 0, fileBytes.Length);
            fileStream.Dispose();

        }
        //将header数据显示到控制台
        private void ShowResponseHeaders(HttpResponseMessage responseMessage)
        {
            Console.WriteLine("Content-Length:" + responseMessage.Content.Headers.ContentLength);
            Console.WriteLine("ContentRange:" + (responseMessage.Content.Headers.ContentRange != null ? (responseMessage.Content.Headers.ContentRange.From + "  " + responseMessage.Content.Headers.ContentRange.HasLength + "  " + responseMessage.Content.Headers.ContentRange.Length) : ""));
            Console.WriteLine("ETage:" + responseMessage.Headers.ETag.Tag);
            Console.WriteLine("Last-Modified:" + responseMessage.Content.Headers.LastModified.ToString());
            Console.WriteLine();
        }
    }
}

{% endhighlight %}
MultiDemo.cs:    
{% highlight c# %}
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading;
using System.Linq;

namespace 断点下载
{
    public class MultiDemo
    {
        //线程计数器
        public static int ThreadCount { get; set; }

        //请求总次数
        public static int RequestCount { get; set; }

        //当前线程运行次序【请求数据的范围】
        public static int RequestCountIndex { get; set; }

        //数据临时存放队列
        public static Dictionary<int, byte[]> ContentQueue = new Dictionary<int, byte[]>();

        public static object ThreadRunLookObject = new object();
        public static object ThreadCountLookObject = new object();


        public  void DownloadImgMultiThread(string url, string folderPath, string fileName)
        {
            HttpResponseMessage responseMessage = GetContentHead(url);
            MultiDemo.ShowResponseHeaders(responseMessage);

            //获取当前资源的ETag值
            string eTag = responseMessage.Headers.ETag.Tag;

            //获取当前资源ContentLength
            long? contentLength = responseMessage.Content.Headers.ContentLength;

            //设置单次请求点字节数
            int requestRangeContent = 1024 * 512;

            //根据每次请求0.5M大小的资源将总length划分需要请求的总次数，向上取整
            MultiDemo.RequestCount = (int)Math.Ceiling((long)contentLength * 1.0 / requestRangeContent);

            //创建4个后台线程
            for (int i = 0; i < 4; i++)
            {
                MultiThreadWork work = new MultiThreadWork(url, eTag, contentLength, folderPath, fileName);
                Thread thread = new Thread(new ThreadStart(work.ThreadMain));
                thread.Start();
            }

        }
        //保存数据队列中的数据
        public static void SaveMultiFile(string fileName, string folderPath)
        {
            //队列排序遍历
            foreach (var item in MultiDemo.ContentQueue.OrderBy(t => t.Key))
            {
                SaveFile(fileName, folderPath, item.Value);
            }
            Program.watch2.Stop();
            Console.WriteLine("Multi thread method using time is " + Program.watch2.ElapsedMilliseconds + " millisecond");
            Console.WriteLine("Save Success!");
        }

        //获取responseHeader
        public static HttpResponseMessage GetContentHead(string url)
        {
            Uri uri = new Uri(url);
            HttpClient client = new HttpClient();
            HttpRequestMessage requstMessage = new HttpRequestMessage(HttpMethod.Head, uri);
            HttpResponseMessage responseMessage = client.SendAsync(requstMessage).Result;
            client.Dispose();
            return responseMessage;
        }
        //获取指定路径下资源的特定范围字节数据
        public static byte[] GetScopeContent(string url, string eTag, long? from, long? to)
        {
            Uri uri = new Uri(url);
            HttpClient client = new HttpClient();

            //If-None-Match:只有在eTag值发生改变时才返回数据status 200，否则返回status 304
            // client.DefaultRequestHeaders.IfNoneMatch.Add(new EntityTagHeaderValue(eTag));

            //If-Range :只有在eTag值发生改变时才rang范围的部分数据 status 206，否则返回整个文档status 200
            client.DefaultRequestHeaders.IfRange = new RangeConditionHeaderValue(eTag);
            client.DefaultRequestHeaders.Range = new RangeHeaderValue(from, to);
            byte[] dataBytes = client.GetByteArrayAsync(uri).Result;

            return dataBytes;
        }
        //将资源保存到文件中
        public static void SaveFile(string fileName, string folderPath, byte[] fileBytes)
        {


            if (!Directory.Exists(folderPath))
            {
                Directory.CreateDirectory(folderPath);
            }
            if (!System.IO.File.Exists(folderPath + fileName))
            {
                File.Create(folderPath + fileName).Dispose();

            }
            FileStream fileStream = new FileStream(folderPath + fileName, FileMode.Append);
            fileStream.Write(fileBytes, 0, fileBytes.Length);
            fileStream.Dispose();

        }
        //将header数据显示到控制台
        public static void ShowResponseHeaders(HttpResponseMessage responseMessage)
        {
            Console.WriteLine("Content-Length:" + responseMessage.Content.Headers.ContentLength);
            Console.WriteLine("ContentRange:" + (responseMessage.Content.Headers.ContentRange != null ? (responseMessage.Content.Headers.ContentRange.From + "  " + responseMessage.Content.Headers.ContentRange.HasLength + "  " + responseMessage.Content.Headers.ContentRange.Length) : ""));
            Console.WriteLine("ETage:" + responseMessage.Headers.ETag.Tag);
            Console.WriteLine("Last-Modified:" + responseMessage.Content.Headers.LastModified.ToString());
            Console.WriteLine();
        }
    }
    public class MultiThreadWork
    {
        public MultiThreadWork(string _url, string _eTag, long? _contentLength, string _folderPath, string _fileName)
        {
            url = _url;
            eTag = _eTag;
            folderPath = _folderPath;
            fileName = _fileName;
            contentLength = _contentLength;
            lock (MultiDemo.ThreadCountLookObject)
            {
                MultiDemo.ThreadCount += 1;
            }
        }
        public string folderPath;
        public string fileName;
        public string url;
        public string eTag;

        //当前资源ContentLength
        public long? contentLength;

        //单次请求点字节数
        public int requestRangeContent = 1024 * 512;

        //线程执行函数
        public void ThreadMain()
        {
            bool run = true;
            int thisIndex = 0;
            while (run)
            {
                //针对 MultiDemo.RequestCountIndex 的变动加锁
                lock (MultiDemo.ThreadRunLookObject)
                {
                    if (MultiDemo.RequestCountIndex >= MultiDemo.RequestCount)
                    {
                        break;
                    }
                    //拿到本次需要请求的数据块初始范围
                    thisIndex = MultiDemo.RequestCountIndex;
                    MultiDemo.RequestCountIndex += 1;
                    run = MultiDemo.RequestCountIndex < MultiDemo.RequestCount;
                }

                //获取数据
                byte[] data =MultiDemo.GetScopeContent(url, eTag, thisIndex * requestRangeContent, (thisIndex + 1) * requestRangeContent > contentLength ? contentLength : (thisIndex + 1) * requestRangeContent);
                //将数据添加到队列中
                MultiDemo.ContentQueue.Add(thisIndex, data);

            }
            lock (MultiDemo.ThreadCountLookObject)
            {
                //线程结束，线程计数器-1
                MultiDemo.ThreadCount -= 1;
            }
            //当线程计数器为0时，表示数据已全部放到队列中，执行数据写入到本地方法。
            if (MultiDemo.ThreadCount == 0)
            {
                MultiDemo.SaveMultiFile(fileName, folderPath);
            }

        }
    }
}

{% endhighlight %}

## 源代码  
[源代码链接]({{ post.url| prepend: site.url  }}/content/zip/201605/Breakpoint-ContinuinglyDemo.rar) 











