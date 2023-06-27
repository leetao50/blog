---
title: Asp.Net Core 读取Request.Body的正取方式
date: 2023-06-15 11:10:31
tags:
---

# 常用读取方式

~~~C#
public override void OnActionExecuting(ActionExecutingContext context)
{
    //在ASP.NET Core中Request Body是Stream的形式
    StreamReader stream = new StreamReader(context.HttpContext.Request.Body);
    string body = stream.ReadToEnd();
    _logger.LogDebug("body content:" + body);
    base.OnActionExecuting(context);
}

~~~
直接报一个这个错System.InvalidOperationException: Synchronous operations are disallowed. Call ReadAsync or set AllowSynchronousIO to true instead.大致的意思就是同步操作不被允许，请使用ReadAsync的方式或设置AllowSynchronousIO为true。

## 同步读取
如何设置AllowSynchronousIO的值。第一种方式是在ConfigureServices中配置，操作如下
~~~C#
services.Configure<KestrelServerOptions>(options =>
{
    options.AllowSynchronousIO = true;
});
~~~
还有一种方式，可以不用在ConfigureServices中设置，通过IHttpBodyControlFeature的方式设置，具体如下
~~~C#
public override void OnActionExecuting(ActionExecutingContext context)
{
    var syncIOFeature = context.HttpContext.Features.Get<IHttpBodyControlFeature>();
    if (syncIOFeature != null)
    {
        syncIOFeature.AllowSynchronousIO = true;
    }
    StreamReader stream = new StreamReader(context.HttpContext.Request.Body);
    string body = stream.ReadToEnd();
    _logger.LogDebug("body content:" + body);
    base.OnActionExecuting(context);
}
~~~

这种方式同样有效，通过这种方式操作，不需要每次读取Body的时候都去设置，只要在准备读取Body之前设置一次即可。这两种方式都是去设置AllowSynchronousIO为true，但是我们需要思考一点，微软为何设置AllowSynchronousIO默认为false，说明微软并不希望我们去同步读取Body。通过查找资料得出了这么一个结论
> Kestrel：默认情况下禁用 AllowSynchronousIO（同步IO），线程不足会导致应用崩溃，而同步I/O API（例如HttpRequest.Body.Read）是导致线程不足的常见原因。

## 异步读取
通过上面我们了解到微软并不希望我们通过设置AllowSynchronousIO的方式去操作，因为会影响性能。那我们可以使用异步的方式去读取，这里所说的异步方式其实就是使用Stream自带的异步方法去读取，如下所示

~~~C#
public override void OnActionExecuting(ActionExecutingContext context)
{
    StreamReader stream = new StreamReader(context.HttpContext.Request.Body);
    string body = stream.ReadToEndAsync().GetAwaiter().GetResult();
    _logger.LogDebug("body content:" + body);
    base.OnActionExecuting(context);
}
~~~

ASP.NET Core中许多操作都是异步操作，甚至是过滤器或中间件都可以直接返回Task类型的方法，因此我们可以直接使用异步操作
~~~C#
public override async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
{
    StreamReader stream = new StreamReader(context.HttpContext.Request.Body);
    string body = await stream.ReadToEndAsync();
    _logger.LogDebug("body content:" + body);
    await next();
}
~~~

这两种方式的操作优点是不需要额外设置别的，只是通过异步方法读取即可，也是我们比较推荐的做法。

# 重复读取
上面我们演示了使用同步方式和异步方式读取RequestBody，但是这样真的就可以了吗？其实并不行，这种方式每次请求只能读取一次正确的Body结果，如果继续对RequestBody这个Stream进行读取，将读取不到任何内容，首先来举个例子

~~~C#
public override async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
{
    StreamReader stream = new StreamReader(context.HttpContext.Request.Body);
    string body = await stream.ReadToEndAsync();
    _logger.LogDebug("body content:" + body);

    StreamReader stream2 = new StreamReader(context.HttpContext.Request.Body);
    string body2 = await stream2.ReadToEndAsync();
    _logger.LogDebug("body2 content:" + body2);

    await next();
}
~~~
上面的例子中body里有正确的RequestBody的结果，但是body2中是空字符串。

那到底该如何解决呢？也很简单，微软知道自己刨下了坑，自然给我们提供了解决办法，用起来也很简单就是加EnableBuffering
~~~C#
public override async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
{
    //操作Request.Body之前加上EnableBuffering即可
    context.HttpContext.Request.EnableBuffering();

    StreamReader stream = new StreamReader(context.HttpContext.Request.Body);
    string body = await stream.ReadToEndAsync();
    _logger.LogDebug("body content:" + body);

    context.HttpContext.Request.Body.Seek(0, SeekOrigin.Begin);
    StreamReader stream2 = new StreamReader(context.HttpContext.Request.Body);
    //注意这里！！！我已经使用了同步读取的方式
    string body2 = stream2.ReadToEnd();
    context.HttpContext.Request.Body.Seek(0, SeekOrigin.Begin);
    _logger.LogDebug("body2 content:" + body2);

    await next();
}
~~~

通过添加Request.EnableBuffering()我们就可以重复的读取RequestBody了，看名字我们可以大概的猜出来，他是和缓存RequestBody有关，需要注意的是Request.EnableBuffering()要加在准备读取RequestBody之前才有效果，否则将无效，而且每次请求只需要添加一次即可。而且大家看到了我第二次读取Body的时候使用了同步的方式去读取的RequestBody，是不是很神奇，待会的时候我们会从源码的角度分析这个问题。


# 源码探究

上面我们看到了通过StreamReader的ReadToEnd同步读取Request.Body需要设置AllowSynchronousIO为true才能操作，但是使用StreamReader的ReadToEndAsync方法却可以直接操作。

## StreamReader和Stream的关系
我们看到了都是通过操作StreamReader的方法即可，那关我Request.Body啥事，别急咱们先看一看这里的操作，首先来大致看下ReadToEnd的实现了解一下StreamReader到底和Stream有啥关联，找到ReadToEnd方法

~~~C#
public override string ReadToEnd()
{
    ThrowIfDisposed();
    CheckAsyncTaskInProgress();
    // 调用ReadBuffer，然后从charBuffer中提取数据。 
    StringBuilder sb = new StringBuilder(_charLen - _charPos);
    do
    {
        //循环拼接读取内容
        sb.Append(_charBuffer, _charPos, _charLen - _charPos);
        _charPos = _charLen; 
        //读取buffer，这是核心操作
        ReadBuffer();
    } while (_charLen > 0);
    //返回读取内容
    return sb.ToString();
}
~~~
通过这段源码我们了解到了这么个信息，一个是StreamReader的ReadToEnd其实本质是通过循环读取ReadBuffer然后通过StringBuilder去拼接读取的内容，核心是读取ReadBuffer方法，由于代码比较多，我们找到大致呈现一下核心操作
~~~C#
if (_checkPreamble)
{
    //通过这里我们可以知道本质就是使用要读取的Stream里的Read方法
    int len = _stream.Read(_byteBuffer, _bytePos, _byteBuffer.Length - _bytePos);
    if (len == 0)
    {
        if (_byteLen > 0)
        {
            _charLen += _decoder.GetChars(_byteBuffer, 0, _byteLen, _charBuffer, _charLen);
            _bytePos = _byteLen = 0;
        }
        return _charLen;
    }
    _byteLen += len;
}
else
{
    //通过这里我们可以知道本质就是使用要读取的Stream里的Read方法
    _byteLen = _stream.Read(_byteBuffer, 0, _byteBuffer.Length);
    if (_byteLen == 0) 
    {
        return _charLen;
    }
}
~~~
通过上面的代码我们可以了解到StreamReader其实是工具类，只是封装了对Stream的原始操作，简化我们的代码ReadToEnd方法本质是读取Stream的Read方法。接下来我们看一下ReadToEndAsync方法的具体实现
~~~C#
public override Task<string> ReadToEndAsync()
{
    if (GetType() != typeof(StreamReader))
    {
        return base.ReadToEndAsync();
    }
    ThrowIfDisposed();
    CheckAsyncTaskInProgress();
    //本质是ReadToEndAsyncInternal方法
    Task<string> task = ReadToEndAsyncInternal();
    _asyncReadTask = task;

    return task;
}

private async Task<string> ReadToEndAsyncInternal()
{
    //也是循环拼接读取的内容
    StringBuilder sb = new StringBuilder(_charLen - _charPos);
    do
    {
        int tmpCharPos = _charPos;
        sb.Append(_charBuffer, tmpCharPos, _charLen - tmpCharPos);
        _charPos = _charLen; 
        //核心操作是ReadBufferAsync方法
        await ReadBufferAsync(CancellationToken.None).ConfigureAwait(false);
    } while (_charLen > 0);
    return sb.ToString();
}
~~~
通过这个我们可以看到核心操作是ReadBufferAsync方法，代码比较多我们同样看一下核心实现

~~~C#
byte[] tmpByteBuffer = _byteBuffer;
//Stream赋值给tmpStream 
Stream tmpStream = _stream;
if (_checkPreamble)
{
    int tmpBytePos = _bytePos;
    //本质是调用Stream的ReadAsync方法
    int len = await tmpStream.ReadAsync(new Memory<byte>(tmpByteBuffer, tmpBytePos, tmpByteBuffer.Length - tmpBytePos), cancellationToken).ConfigureAwait(false);
    if (len == 0)
    {
        if (_byteLen > 0)
        {
            _charLen += _decoder.GetChars(tmpByteBuffer, 0, _byteLen, _charBuffer, _charLen);
            _bytePos = 0; _byteLen = 0;
        }
        return _charLen;
    }
    _byteLen += len;
}
else
{
    //本质是调用Stream的ReadAsync方法
    _byteLen = await tmpStream.ReadAsync(new Memory<byte>(tmpByteBuffer), cancellationToken).ConfigureAwait(false);
    if (_byteLen == 0) 
    {
        return _charLen;
    }
}
~~~
通过上面代码我可以了解到StreamReader的本质就是读取Stream的包装，核心方法还是来自Stream本身。我们之所以大致介绍了StreamReader类，就是为了给大家呈现出StreamReader和Stream的关系，否则怕大家误解这波操作是StreamReader的里的实现，而不是Request.Body的问题，其实并不是这样的所有的一切都是指向Stream的Request的Body就是Stream这个大家可以自己查看一下，了解到这一步我们就可以继续了。

# HttpRequest的Body
上面我们说到了Request的Body本质就是Stream，Stream本身是抽象类，所以Request.Body是Stream的实现类。默认情况下Request.Body的是HttpRequestStream的实例，我们这里说了是默认，因为它是可以改变的，我们一会再说。我们从上面StreamReader的结论中得到ReadToEnd本质还是调用的Stream的Read方法，即这里的HttpRequestStream的Read方法，我们来看一下具体实现

~~~C#
public override int Read(byte[] buffer, int offset, int count)
{
    //知道同步读取Body为啥报错了吧
    if (!_bodyControl.AllowSynchronousIO)
    {
        throw new InvalidOperationException(CoreStrings.SynchronousReadsDisallowed);
    }
    //本质是调用ReadAsync
    return ReadAsync(buffer, offset, count).GetAwaiter().GetResult();
}
~~~
通过这段代码我们就可以知道了为啥在不设置AllowSynchronousIO为true的情下读取Body会抛出异常了吧，这个是程序级别的控制，而且我们还了解到Read的本质还是在调用ReadAsync异步方法
~~~C#
public override ValueTask<int> ReadAsync(Memory<byte> destination, CancellationToken cancellationToken = default)
{
    return ReadAsyncWrapper(destination, cancellationToken);
}
~~~
ReadAsync本身并无特殊限制，所以直接操作ReadAsync不会存在类似Read的异常。

> 通过这个我们得出了结论Request.Body即HttpRequestStream的同步读取Read会抛出异常，而异步读取ReadAsync并不会抛出异常只和HttpRequestStream的Read方法本身存在判断AllowSynchronousIO的值有关系。


# EnableBuffering神奇的背后
我们在上面的示例中看到了，如果不添加EnableBuffering的话直接设置RequestBody的Position会报NotSupportedException这么一个错误，而且加了它之后我居然可以直接使用同步的方式去读取RequestBody，首先我们来看一下为啥会报错，我们从上面的错误了解到错误来自于HttpRequestStream这个类，上面我们也说了这个类继承了Stream抽象类,通过源码我们可以看到如下相关代码

~~~C#
//不能使用Seek操作
public override bool CanSeek => false;
//允许读
public override bool CanRead => true;
//不允许写
public override bool CanWrite => false;
//不能获取长度
public override long Length => throw new NotSupportedException();
//不能读写Position
public override long Position
{
    get => throw new NotSupportedException();
    set => throw new NotSupportedException();
}
//不能使用Seek方法
public override long Seek(long offset, SeekOrigin origin)
{
    throw new NotSupportedException();
}
~~~

相信通过这些我们可以清楚的看到针对HttpRequestStream的设置或者写相关的操作是不被允许的，这也是为啥我们上面直接通过Seek设置Position的时候为啥会报错，还有一些其他操作的限制，总之默认是不希望我们对HttpRequestStream做过多的操作，特别是设置或者写相关的操作。但是我们使用EnableBuffering的时候却没有这些问题，究竟是为什么?接下来我们要揭开它的什么面纱了。首先我们从Request.EnableBuffering()这个方法入手

~~~C#

/// <summary>
/// 确保Request.Body可以被多次读取
/// </summary>
/// <param name="request"></param>
public static void EnableBuffering(this HttpRequest request)
{
    BufferingHelper.EnableRewind(request);
}


//默认内存中可缓存的大小为30K,超过这个大小将会被存储到磁盘
internal const int DefaultBufferThreshold = 1024 * 30;

/// <summary>
/// 这个方法也是HttpRequest扩展方法
/// </summary>
/// <returns></returns>
public static HttpRequest EnableRewind(this HttpRequest request, int bufferThreshold = DefaultBufferThreshold, long? bufferLimit = null)
{
    if (request == null)
    {
        throw new ArgumentNullException(nameof(request));
    }
    //先获取Request Body
    var body = request.Body;
    //默认情况Body是HttpRequestStream这个类CanSeek是false所以肯定会执行到if逻辑里面
    if (!body.CanSeek)
    {
        //实例化了FileBufferingReadStream这个类，看来这是关键所在
        var fileStream = new FileBufferingReadStream(body, bufferThreshold,bufferLimit,AspNetCoreTempDirectory.TempDirectoryFactory);
        //赋值给Body，也就是说开启了EnableBuffering之后Request.Body类型将会是FileBufferingReadStream
        request.Body = fileStream;
        //这里要把fileStream注册给Response便于释放
        request.HttpContext.Response.RegisterForDispose(fileStream);
    }
    return request;
}
~~~
从上面这段源码实现中我们可以大致得到两个结论

+ BufferingHelper的EnableRewind方法也是HttpRequest的扩展方法，可以直接通过Request.EnableRewind的形式调用，效果等同于调用Request.EnableBuffering因为EnableBuffering也是调用的EnableRewind
+ 启用了EnableBuffering这个操作之后实际上会使用FileBufferingReadStream替换掉默认的HttpRequestStream，所以后续处理RequestBody的操作将会是FileBufferingReadStream实例

通过上面的分析我们也清楚的看到了，核心操作在于FileBufferingReadStream这个类，而且从名字也能看出来它肯定是也继承了Stream抽象类

# 总结

本篇文章篇幅比较多，如果你想深入的研究相关逻辑，希望本文能给你带来一些阅读源码的指导。为了防止大家深入文章当中而忘记了具体的流程逻辑，在这里我们就大致的总结一下关于正确读取RequestBody的全部结论

* 首先关于同步读取Request.Body由于默认的RequestBody的实现是HttpRequestStream，但是HttpRequestStream在重写Read方法的时候会判断是否开启AllowSynchronousIO，如果未开启则直接抛出异常。但是HttpRequestStream的ReadAsync方法并无这种限制，所以使用异步方式的读取RequestBody并无异常。
* 虽然通过设置AllowSynchronousIO或使用ReadAsync的方式我们可以读取RequestBody，但是RequestBody无法重复读取，这是因为HttpRequestStream的Position和Seek都是不允许进行修改操作的，设置了会直接抛出异常。为了可以重复读取，我们引入了Request的扩展方法EnableBuffering通过这个方法我们可以重置读取位置来实现RequestBody的重复读取。
* 关于开启EnableBuffering方法每次请求设置一次即可，即在准备读取RequestBody之前设置。其本质其实是使用FileBufferingReadStream代替默认RequestBody的默认类型HttpRequestStream，这样我们在一次Http请求中操作Body的时候其实是操作FileBufferingReadStream，这个类重写Stream的时候Position和Seek都是可以设置的，这样我们就实现了重复读取。
* FileBufferingReadStream带给我们的不仅仅是可重复读取，还增加了对RequestBody的缓存功能，使得我们在一次请求中重复读取RequestBody的时候可以在Buffer里直接获取缓存内容而Buffer本身是一个MemoryStream。当然我们也可以自己实现一套逻辑来替换Body，只要我们重写的时候让这个Stream支持重置读取位置即可。



摘抄 yi念之间 [原文地址](https://www.cnblogs.com/wucy/p/14699717.html)

