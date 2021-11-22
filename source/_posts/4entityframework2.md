---
title: entityframework基础(二)
date: 2021-11-04 11:04:52
layout: 2021-11-04 11:04:52
tags:
 - C#
 - EntityFramework
---

记录一些EntityFramework中的使用技巧，不定期更新。
* 日志记录及拦截
* 
<!--more-->

### 日志记录及拦截
EntityFramework作用是ORM，那么最终它是需要和数据库打交道的，就一定少不了SQL语句。如何知道EntityFramework生成的sql语句是怎样的呢？
通过设置`DbContext`属性设置为采用字符串的任何方法的委托(`DbContext.Database.Log=Console.Write`)

``` CSharp
Console.WriteLine("from db, get blogs.....");
using (BlogDbContext db = new BlogDbContext())
{
    db.Database.Log = Console.Write;
    Console.WriteLine("connection status:" + db.Database.Connection.State);

    var blogs = (from b in db.BlogContents
                    select b).ToList();
    foreach (var blog in blogs)
    {
        Console.WriteLine(String.Format("ID:{0}, Title: {1}", blog.ID, blog.Title));
    }

    Console.WriteLine("after select command, \n");
    Console.WriteLine("connection status:" + db.Database.Connection.State);
}
```
比如上面的代码，会将生成的查询语句记录到Console控制台中，如下图：
![entity输出日志](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/logs1.png)

如果需要将日志记录至其他位置，则可以使用其他日志框架，并且定义一个日志记录方法：
```CSharp
//定义日志访问类及方法
public class MyLogger
{
    public void Log(string component, string message)
    {
        Console.WriteLine("Component: {0} Message: {1} ", component, message);
    }
}

//将日志输出指向定义的方法上
var logger = new MyLogger();
context.Database.Log = s => logger.Log("EFApp", s);
```

设置自己的记录日志格式：
```CSharp
public class OneLineFormatter : DatabaseLogFormatter
{
    public OneLineFormatter(DbContext context, Action<string> writeAction)
        : base(context, writeAction)
    {
    }

    public override void LogCommand<TResult>(
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext)
    {
        Write(string.Format(
            "Context '{0}' is executing command '{1}'{2}",
            Context.GetType().Name,
            command.CommandText.Replace(Environment.NewLine, ""),
            Environment.NewLine));
    }

    public override void LogResult<TResult>(
        DbCommand command, DbCommandInterceptionContext<TResult> interceptionContext)
    {
    }
}

//将该类放在BlogDbContext类中定义
public class MyDbConfiguration : DbConfiguration
{
    public MyDbConfiguration()
    {
        SetDatabaseLogFormatter(
            (context, writeAction) => new OneLineFormatter(context, writeAction));
    }
}
```
![按自定义格式输出日志](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/logs2.png)