---
title: C#异步编程之async/await
date: 2021-11-08 20:57:14
layout: 2021-11-08 20:57:14
id: 0026
tags:
 - C#
 - 异步编程
categories:
 - C#
---

* 异步编程场景
* async与await
* `asp.net mvc`中使用async/await

<!--more-->

## 异步编程场景
解决耗时任务占用主线程执行的问题。因此耗时高的任务适用于异步编程，C#中有以下场景：
1. 对于 I/O 绑定代码，等待一个在 async 方法中返回 Task 或 Task<T> 的操作。
2. 对于 CPU 绑定代码，等待一个使用 Task.Run 方法在后台线程启动的操作。

如何判断是CPU耗时操作还是IO耗时操作呢？简单来说，如果需要等待某些内容（如数据库中的数据、大文本文件的读取等），则为IO耗时操作；如果需要执行大量的计算，则属于CPU耗时操作。
## async await
async/await成对出现，用于简化异步编程。async出现在方法的声明前，表示这是一个异步方法。await则出现在这个异步方法的内部，一般在Task前。
async/await关键字本身不会创建新线程，但被await的方法内部一般是通过创建线程来实现。

微软官方给定的异步方法执行顺序：

![异步执行步骤](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/navigation-trace-async-program.png)

### 到底异步还是同步?

这个标题算是个问题？问题又不太对。async/await成对出现的方法是异步方法，那哪来的同步呢？实际上最好先从多线程说起，关于异步编程这块，我准备整理一个系列出来。顺序上可能是这篇博客先出来，那就用下面的例子先展示一下。

我看到有的文章说，async/await既可以是异步，又可以是同步。所以就会有疑惑，到底是同步还是异步呢？看下面两个例子。


#### 异步方式执行
两个Task输出并没有按顺序，且并没有按主线程中代码顺序来执行。可以看到，两个异步方法尚未输出完成，主方法中`main end.....`已经输出。且Method1和Method2中的输出也是交替来的。
输出结果:
![异步执行结果](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202112/%E5%BC%82%E6%AD%A5%E6%89%A7%E8%A1%8C%E7%BB%93%E6%9E%9C.png)
```csharp
static void Main()
{
    Console.WriteLine("main start....");
    AsyncMethod1();
    AsyncMethod2();
    Thread.Sleep(1000);
    Console.WriteLine("main end.....");

    Console.ReadLine();
}

static async void AsyncMethod1()
{
    Console.WriteLine("start AsyncMethod1 async");
    var result = await MyMethod1();
    Console.WriteLine("end AsyncMethod1 async");
}

static async Task<int> MyMethod1()
{
    for(int i = 0; i < 5; i++)
    {
        Console.WriteLine("Async MyMethod1 start:" + i.ToString());
        await Task.Delay(1000);
    }
    return 0;
}

static async void AsyncMethod2()
{
    Console.WriteLine("start AsyncMethod2 async");
    var result = await MyMethod2();
    Console.WriteLine("end AsyncMethod2 async");
}

static async Task<int> MyMethod2()
{
    for(int i = 0; i < 5; i++)
    {
        Console.WriteLine("Async MyMethod2 start:" + i.ToString());
        await Task.Delay(1000);
    }
    return 0;
}
```

#### 异步方法同步执行
结合上面那个例子，再看这个例子的输出，就容易理解所谓的异步方法同步执行了。意思是说，Method1和Method2两个方法是异步执行的（理解成创建一个新线程去执行的吧）。但是，通过一些手段，让这两个任务按顺序来执行。即Method1执行完再来执行Method2。

两个Task相对主线程来说，是异步去执行的，但这个异步也是按顺序来的。Method1输出完了，再执行Method2的输出。
![异步方法的同步执行](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202112/%E5%90%8C%E6%AD%A5%E6%89%A7%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

```csharp
static void Main()
{
    Console.WriteLine("Async Test job");
    Console.WriteLine("main start.....");
    Console.WriteLine("MyMethod() 异步方法同步执行:");

    int i = MyMethod().GetAwaiter().GetResult();
    int j = MyMethod2().GetAwaiter().GetResult();
    
    Console.WriteLine($" i : {i}");
    Console.WriteLine($" j : {j}");
    Console.WriteLine("main end .....");

    Console.ReadLine();
}


static async Task<int> MyMethod()
{
    for(int i = 0; i < 5; i++)
    {
        Console.WriteLine("Async Method start: " + i.ToString());
        await Task.Delay(1000);
    }
    return 0;
}

static async Task<int> MyMethod2()
{
    for(int i = 0; i < 5; i++)
    {
        Console.WriteLine("Async Method2 start: " + i.ToString());
        await Task.Delay(1000);
    }
    return 0;
}
```


## 何时使用async/await

`asp.net mvc`中使用async/await并不能提升访问速度。但使用异步编程，可以提高响应能力（吞吐量），即使用异步方式在同一时间可以处理更多的请求。使用同步方式，线程会被耗时操作一直占用，直到耗时操作结束；使用异步方式，程序走到await关键字会立即return，释放线程，剩下的代码将放到一个回调（Task.GetAwaiter()的UnsafeOnCompleted(Action)），耗时操作完成时才会回调执行。


>在web服务器上, `.NET Framework`维护用于处理`ASP.NET`请求的线程池。当请求到达时，将调度线程池中的线程以处理该请求。如果以同步方式处理请求，则处理请求的线程将在处理请求时处于繁忙状态，并且该线程无法处理其他请求。

>在启动时看到大量并发请求的web应用中，或具有突发负载（其中并发增长突然增加）时，使web服务器调用异步会提高应用程序的相应能力。异步请求与同步请求所需的处理时间相同。如果请求发出需要两秒时间才能完成web服务调用，则该请求将需要两秒钟，无论是同步执行还是异步执行。但是，在异步调用期间，线程在等待第一个请求完成时不会被阻止响应其他请求。因此，当有多个并发请求调用长时间运行的操作时，异步请求会组织请求队列和线程池的增长。

#### 同步方式执行耗时任务
```csharp
 public ActionResult Index()
        {
            DateTime startTime = DateTime.Now;//进入DoSomething方法前的时间
            var startThreadId = Thread.CurrentThread.ManagedThreadId;//进入DoSomething方法前的线程ID

            DoSomething();//耗时操作

            DateTime endTime = DateTime.Now;//完成DoSomething方法的时间
            var endThreadId = Thread.CurrentThread.ManagedThreadId;//完成DoSomething方法后的线程ID
            return Content($"startTime:{ startTime.ToString("yyyy-MM-dd HH:mm:ss:fff") } startThreadId:{ startThreadId }\nendTime:{ endTime.ToString("yyyy-MM-dd HH:mm:ss:fff") } endThreadId:{ endThreadId }");
        }

        /// <summary>
        /// 耗时操作
        /// </summary>
        /// <returns></returns>
        private void DoSomething()
        {
            Thread.Sleep(10000);
        }
```


#### 异步方式执行耗时任务
```csharp
 public async Task<ActionResult> Index()
        {
            DateTime startTime = DateTime.Now;//进入DoSomething方法前的时间
            var startThreadId = Thread.CurrentThread.ManagedThreadId;//进入DoSomething方法前的线程ID

            await DoSomething();//耗时操作

            DateTime endTime = DateTime.Now;//完成DoSomething方法的时间
            var endThreadId = Thread.CurrentThread.ManagedThreadId;//完成DoSomething方法后的线程ID
            return Content($"startTime:{ startTime.ToString("yyyy-MM-dd HH:mm:ss:fff") } startThreadId:{ startThreadId }\nendTime:{ endTime.ToString("yyyy-MM-dd HH:mm:ss:fff") } endThreadId:{ endThreadId }");
        }

        /// <summary>
        /// 耗时操作
        /// </summary>
        /// <returns></returns>
        private async Task DoSomething()
        {
            await Task.Run(() => Thread.Sleep(10000));
        }
```

上面说了那么多，那到底该不该用async/await，什么时候用他俩？
>总结：
>* 对于计算密集型工作，使用多线程
>* 对于IO密集型工作，采用异步机制


---

参考链接：
《.NET Web应用中为什么要使用async/await异步编程》 https://www.cnblogs.com/yanglang/p/13071091.html

《C# 多线程(18)：一篇文章就理解async和await》 https://www.cnblogs.com/whuanle/p/12822705.html

《异步编程》 https://docs.microsoft.com/zh-cn/dotnet/csharp/async

