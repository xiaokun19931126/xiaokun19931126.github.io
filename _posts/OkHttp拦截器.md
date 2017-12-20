拦截器是一个拥有监控、复写、复用能力的强大机制。

下面是一个能打印请求输出和返回结果的拦截器案例：

```
class LoggingInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Request request = chain.request();

    long t1 = System.nanoTime();
    logger.info(String.format("Sending request %s on %s%n%s",
        request.url(), chain.connection(), request.headers()));

    Response response = chain.proceed(request);

    long t2 = System.nanoTime();
    logger.info(String.format("Received response for %s in %.1fms%n%s",
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));

    return response;
  }
}
```

chain.proceed(request)方法的调用是每一个拦截器实现中的重要部分。这个看起来简单的方法在所有HTTP工作发生和满足请求产生结果的地方。

拦截器可以链接。假设你既有压缩的拦截器还有检验的拦截器，你需要决定是否数据是先压缩后检验还是先检验后压缩。OkHttp使用列表跟踪拦截器，拦截器按顺序调用。

![interceptors@2x.png_temp](C:\Users\xiaokun\Desktop\MarkDown文件\interceptors@2x.png_temp.bmp)

应用拦截器

拦截器都是作为应用拦截器或者网络拦截器注册的。我们将使用LoggingInterceptor来定义显示不同。

注册一个应用拦截器通过调用addInterceptor()方法在OkHttpClient.Builder：

```
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new LoggingInterceptor())
    .build();

Request request = new Request.Builder()
    .url("http://www.publicobject.com/helloworld.txt")
    .header("User-Agent", "OkHttp Example")
    .build();

Response response = client.newCall(request).execute();
response.body().close();
```

http://www.publicobject.com/helloworld.txt会被重定向到https://publicobject.com/helloworld.txt，而且OkHttp自动遵循此重定向。我们的应用拦截器被调用一次，而且从chain.proceed()返回的响应已经重定向响应：

```
INFO: Sending request http://www.publicobject.com/helloworld.txt on null
User-Agent: OkHttp Example

INFO: Received response for https://publicobject.com/helloworld.txt in 1179.7ms
Server: nginx/1.4.6 (Ubuntu)
Content-Type: text/plain
Content-Length: 1759
Connection: keep-alive
```

我们能看到我们被重定向是因为response.request().url() 和request.url()不同。两个日志语句记录了两种不同的URLs。

