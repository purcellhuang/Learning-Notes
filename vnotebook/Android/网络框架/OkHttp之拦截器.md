# OkHttp

[官方文档](https://square.github.io/okhttp/interceptors/)
## 简介
Interceptors are a powerful mechanism that can monitor, rewrite, and retry calls.
意思大概是，拦截器是一个强有力的机制，能够监控，重写以及重试（请求的）调用。

从上篇可以知道当call调用execute或者enqueue时，都是RealCall在调用,最终会执行Response result = getResponseWithInterceptorChain();这段代码.
**execute**
```
 @Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } finally {
      client.dispatcher().finished(this);
    }
  }
```

**getResponseWithInterceptorChain**
```
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }
```
由此我们可以知道在getResponseWithInterceptorChain中添加了以下的拦截器：

* client.interceptors()：我们通过okHttpClient自定义的拦截器。
* retryAndFollowUpInterceptor：重试及重定向拦截器
* BridgeInterceptor：桥接拦截器
* CacheInterceptor：缓存拦截器
* ConnectInterceptor：连接拦截器
* client.networkInterceptors()：网络请求的拦截器
* CallServerInterceptor(forWebSocket)：读写拦截器

由此添加的顺序也可以得知拦截器执行的顺序。

## 拦截器的分类
![](_v_images/20200321142753084_28407.png =720x)
从图中我们可以看出有两种拦截器，一种是Application Interceptors，另一种是Network Interceptors。中间这快OkHttp Core是内部拦截器。
不管是哪种拦截器，其都是基于Interceptor接口。我们来看看Interceptor这个接口！
```
/**
 * Observes, modifies, and potentially short-circuits requests going out and the corresponding
 * responses coming back in. Typically interceptors add, remove, or transform headers on the request
 * or response.
 */
public interface Interceptor {
  Response intercept(Chain chain) throws IOException;

  interface Chain {
    Request request();

    Response proceed(Request request) throws IOException;

    Connection connection();
  }
}
```


分为两类：

**ApplicationInterceptor(应用拦截器)**

* Don’t need to worry about intermediate responses like redirects and retries.
不需要关心是否重定向或者失败重连
* Are always invoked once, even if the HTTP response is served from the cache.
应用拦截器只会调用一次，即使数据来源于缓存
* Observe the application’s original intent. Unconcerned with OkHttp-injected headers like If-None-Match.
只考虑应用的初始意图，不去考虑Okhhtp注入的Header比如：if-None-Match,意思就是不管其他外在因素只考虑最终的返回结果
* Permitted to short-circuit and not call Chain.proceed().
自定义的应用拦截器是第一个开始执行的拦截器，所以这句话的意思就是，应用拦截器可以决定是否执行其他的拦截器，通过Chain.proceed().
* Permitted to retry and make multiple calls to Chain.proceed().
和上一句的意思差不多，可以执行多次调用其他拦截器，通过Chain.proceed().

**NetworkInterceptor（网络拦截器）**

* Able to operate on intermediate responses like redirects and retries.
网络拦截器可以操作重定向和失败重连的返回值
* Not invoked for cached responses that short-circuit the network.
取缓存中的数据就不会去执行Chain.proceed().所以就不能执行网络拦截器
* Observe the data just as it will be transmitted over the network.
通过网络拦截器可以观察到所有通过网络传输的数据
* Access to the Connection that carries the request.
请求服务连接的拦截器先于网络拦截器执行，所以在进行网络拦截器执行时，就可以看到Request中服务器请求连接信息，因为应用拦截器是获取不到对应的连接信息的。

**异同**

相同点：
1. 都能对response和request进行拦截。
2. 这两种拦截器本质上都是基于Interceptor接口，由开发者实现这个接口，然后将自定义的Interceptor类的对象设置到okhttpClient对象中。
3. 两者都会被add到OkHttpClient内的一个ArrayList中。当请求执行的时候，多个拦截器会依次执行。

不同点：
1. 添加应用拦截器的接口是addInterceptor()，而添加网络拦截器的接口是addNetworkInterceptor()。
2. 作用区域不同，从最上方图中可以看出，应用拦截器作用于OkHttpCore到Application之间，网络拦截器作用于 Network和OkHttpCore之间。

## 案例
**自定义应用拦截器**
自定义CookieInterceptor来添加Request的Header信息，并且打印Response的头部信息
可以用来添加cookie信息，保存会话信息
```
public class CookieInterceptor implements Interceptor {

    private static String TAG = "CookieInterceptor";

    @Override
    public Response intercept(Chain chain) throws IOException {

        Request request = chain.request().newBuilder()
                .addHeader("username","PurcellHuang")
                .addHeader("Cookie","sessionID=asdasfdsdas;token=asd15sadc4zxc15s")
                .build();

        Response response = chain.proceed(request);

        Headers headers = response.headers();
        Set<String> names = headers.names();
        for (String s:names){
            Log.i(TAG,"---------"+s+":"+headers.get(s));
        }


        return response;
    }
}

```

**添加自定义拦截器**
```
        //设置一下okHttp的参数
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .connectTimeout(CONNECT_TIME_OUT, TimeUnit.MILLISECONDS)
                .addInterceptor(new CookieInterceptor())
                .build();
        mRetrofit = new Retrofit.Builder()
                .baseUrl(BASE_URL)//设置BaseUrl
                .client(okHttpClient)//设置请求的client
                .addConverterFactory(GsonConverterFactory.create())//设置转换器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
```