# Okhttp3学习  
>OkHttp官方API：square.github.io/okhttp/3.x/okhttp/
## 一 添加依赖    
需要在项目Moudle的build.gradle中添加如下依赖：  
	
	compile 'com.squareup.okhttp3:okhttp:3.6.0'
    compile 'com.squareup.okio:okio:1.11.0'  

*不光是okhttp的URL，这里还有一个okio的URL，相关的内容本文最后说明。*  
## 二 基本使用  
![](https://images2017.cnblogs.com/blog/833973/201708/833973-20170801173428318-280682338.png)
### 2.1 几个常见的类的认识  
#### 2.1.1 OkHttpClient相关  
OkHttpClient顾名思义，是一个客户相关的类，它的作用就是**作为发起请求（Request）的客户端**。**在这个类的层次上可以进行的设置有：**  

* 设置链接，读取，写入时间  
* 设置缓存，cookie缓存
* 设置拦截器（Interceptor）
* 设置proxy代理等  

最简单的创建一个OkHttpClient对象的方式就是直接构造器OkHttpClient client =new OkHttpClient();一个出来，如果这样的话，上面所说的配置的话就全部都是默认的了，没有办法进行设置了，如果想要根据实际情况进行一些设定就可以使用**OkHttpClient.Builder类**  
有了这个类，我们可以这样来创建一个OkHttpClient客户端：  

	OkHttpClient client1 =new OkHttpClient.Builder()
                .connectTimeout(1000, TimeUnit.MILLISECONDS)
                .readTimeout(1200,TimeUnit.MILLISECONDS)
                .proxy(null)
                .build();
详细的还是查看API文档进行了解。  
#### 2.1.2 Request相关  
不管是GET还是POST，都是一个“请求”，在OkHttp中用**Request类封装请求**，作为一个请求，我们可以在Request层面上做的事情有：  

* 设置URL  
* 设置请求方法（GET还是POST）
* 设置请求头（Header）
* 设置请求体（POST方法中）
* 添加缓存  

同样，为了根据实际情况进行一些设置，我们通常也是通过**Request.Builder类**来进行创建的：  

	Request request=new Request.Builder()
                .url(url)
                .get()
                .build();
**在默认情况下是GET方法,所以上面的get（）去掉也行，如果想要使用POST方法：**  

	request=new Request.Builder()
                .url(url)
                .post(body)
                .addHeader("Content-Type","application/x-www-form-urlencoded")
                .build();
**或者**  

	request=new Request.Builder()
                .url(url)
                .method("post",body)
                .addHeader("Content-Type","application/x-www-form-urlencoded")
                .build();

**HTTP 头的数据结构是 Map<String, List<String>>类型。也就是说，对于每个 HTTP 头，可能有多个值**。

* 使用header(name,value)来设置HTTP头的唯一值
* 使用addHeader(name,value)来补充新值
* 使用header(name)读取唯一值或多个值的最后一个值
* 使用headers(name)获取所有值  

#### 2.1.3 RequestBody  
如果需要在POST方法中添加请求体的话，则需要使用到**RequestBody类，这是一个抽象类**  
该类是对请求体的封装，构造器是一个无参构造器：  
**流式上传**  
当请求内容较大时，应该使用流来提交。示例中创建了 RequestBody 的一个匿名子类。该子类的 contentType 方法需要返回内容的媒体类型，而 writeTo 方法的参数是一个 BufferedSink 对象。我们需要做的就是把请求的内容写入到 BufferedSink 对象即可。

	OkHttpClient client = new OkHttpClient();
	    final MediaType MEDIA_TYPE_TEXT = MediaType.parse("text/plain");
	    final String postBody = "Hello World";
	 
	    RequestBody requestBody = new RequestBody() {
	        @Override
	        public MediaType contentType() {
	            return MEDIA_TYPE_TEXT;
	        }
	 
	        @Override
	        public void writeTo(BufferedSink sink) throws IOException {
	            sink.writeUtf8(postBody);
	        }
	 
	 	@Override
	        public long contentLength() throws IOException {
	            return postBody.length();
	        }
	    };
	 
	    Request request = new Request.Builder()
	            .url("http://www.baidu.com")
	            .post(requestBody)
	            .build();
	 
	    Response response = client.newCall(request).execute();
	    if (!response.isSuccessful()) {
	        throw new IOException("服务器端错误: " + response);
	    }
	 
	    System.out.println(response.body().string());

我们也通常使用其实现类：  

1. **FormBody类**  
该类是RequestBody的一个实现类，可以实现表单提交，同样有一个**FormBody.Builder类**来实现这一功能：

**表单提交**  

	RequestBody body=new FormBody.Builder()
                .add("showapi_appid","55635")
                .add("showapi_sign","ae82a9ef73e24d2bb2f08822a70d7d28")
                .add("qq",qq)
                .build();  
2. **MultipartBody类**  
该类也是RequestBody的一个实现类，与FormBody不同的是，同时**还可以进行二进制数据（如文件）的上传，也有一个MultipartBody.Builder类**   

**文件上传**   
使用MultipartBuilder指定MultipartBuilder.FORM类型并通过addPart方法添加不同的Part（每个Part由Header和RequestBody两部分组成），最后调用builde()方法构建一个RequestBody。  


	MediaType MEDIA_TYPE_TEXT = MediaType.parse("text/plain");
	RequestBody requestBody = new MultipartBuilder()
    	.type(MultipartBuilder.FORM)
    	.addPart(
           		Headers.of("Content-Disposition", "form-data; name=\"title\""),
            	RequestBody.create(null, "测试文档"))
    	.addPart(
            	Headers.of("Content-Disposition", "form-data; name=\"file\""),
            	RequestBody.create(MEDIA_TYPE_TEXT, new File("input.txt")))
    	.build();

#### 2.1.4 Call类相关  
Call代表了一个实际的HTTP请求，它是连接Request和Response的桥梁，通过OkHttpClient对象的newCall(Request)方法可以得到一个Call对象。Call对象既支持同步获取数据，也可以异步获取数据。
  
* 执行Call对象的execute()方法，会阻塞当前线程去获取数据，该方法返回一个Response对象。
* 执行Call对象的enqueue(CallBack)方法，不会阻塞当前线程，该方法接收一个Callback对象，当异步获取到数据之后，会回调执行Callback对象的相应方法。如果请求成功，则执行Callback对象的onResponse方法，并将Response对象传入该方法中；如果请求失败，则执行Callback对象的onFailure方法。  

#### 2.1.5 Response类  
Response类封装了响应报文信息：状态吗（200、404等）、响应头（Content-Type、Server等）以及可选的响应体。可以通过Call对象的execute()方法获得Response对象，异步回调执行Callback对象的onResponse方法时也可以获取Response对象。对于这个类，我们**通常是通过response.body()来获取ResponseBody对象**  
在API文档中有这么一段话：  

>Each response body is backed by a limited resource like a socket (live network responses) or an open file (for cached responses). Failing to close the response body will leak resources and may ultimately cause the application to slow down or crash. 
Both this class and Response implement Closeable. Closing a response simply closes its response body. If you invoke Call.execute() or implement Callback.onResponse(okhttp3.Call, okhttp3.Response) you must close this body by calling any of the following methods:   

* Response.close()
* Response.body().close()
* Response.body().source().close()
* Response.body().charStream().close()
* Response.body().byteString().close()
* Response.body().bytes()
* Response.body().string()

>There is no benefit to invoking multiple close() methods for the same response body.   

### 2.2 缓存(只有GET方法才有用)  
#### 2.2.1 响应缓存
OkHttp 可以对 HTTP 响应的内容在磁盘上进行缓存。在进行 HTTP 请求时，如果该请求的响应已经被缓存而且没有过期，OkHttp 会直接使用缓存中的响应内容，而不需要真正的发出 HTTP 请求到远程服务器。在创建缓存时需要指定一个磁盘目录和缓存的大小。创建出 Cache 对象之后，通过 OkHttpClient 的 setCache 进行设置。**通过 Response 对象的 cacheResponse 和 networkResponse 方法可以得到缓存的响应和从实际的 HTTP 请求得到的响应。如果该请求的响应来自实际的网络请求，则 cacheResponse 方法的返回值为 null；如果响应来自缓存，则 networkResponse 的返回值为 null。**OkHttp 在进行缓存时会遵循 HTTP 协议的要求，因此可以通过标准的 HTTP 头 Cache-Control 来控制响应的缓存时间。

	int cacheSize = 100 * 1024 * 1024;
    File cacheDirectory = Files.createTempDirectory("cache").toFile();
    Cache cache = new Cache(cacheDirectory, cacheSize);
    OkHttpClient client = new OkHttpClient();
    client.setCache(cache);
 
    Request request = new Request.Builder()
            .url("http://www.baidu.com")
            .build();
 
    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) {
        throw new IOException("服务器端错误: " + response);
    }
 
    System.out.println(response.cacheResponse());
    System.out.println(response.networkResponse());
#### 2.2.2 缓存控制  
**不同于缓存的设置，一次请求也可以跳过缓存的文件或者即使缓存过期也强制使用缓存，而这些都是通过CacheControl类来进行控制的，是在Request.Builder的构造Request中设置的。**
如果想让某次请求禁用缓存，可以调用request.cacheControl(CacheControl.FORCE_NETWORK)方法，这样即便缓存目录有对应的缓存，也会忽略缓存，强制发送网络请求，这对于获取最新的响应结果很有用。如果想强制某次请求使用缓存的结果，可以调用request.cacheControl(CacheControl.FORCE_CACHE)，这样不会发送实际的网络请求，而是读取缓存，即便缓存数据过期了，也会强制使用该缓存作为响应数据，如果缓存不存在，那么就返回504 Unsatisfiable Request错误。也可以向请求中中加入类似于Cache-Control: max-stale=3600之类的请求头，Okhttp也会使用该配置对缓存进行处理。
在这个类中可以设置缓存的有效时间，强制进行请求，强制使用缓存等，通过new Request.Builder().cacheControl(CacheControl)来设置；这些CacheControl设置也是通过CacheControl.Builder来进行设置的。
>可以参考API文档：square.github.io/okhttp/3.x/okhttp/  
### 2.3 拦截器  
拦截器是OkHttp中一个强大的功能，先看看官方的说明：  

>Interceptors are a powerful mechanism that can monitor, rewrite, and retry calls. Here's a simple interceptor that logs the outgoing request and the incoming response.   

拦截器是 OKHttp 设计的精髓所在，每个拦截器负责不同的功能，使用责任链模式，通过链式调用执行所有的拦截器对象中的 Response intercept(Chain chain) 方法。拦截器在某种程度上也借鉴了网络协议中的分层思想，请求时从最上层到最下层，响应时从最下层到最上层。
#### 2.3.1 两种拦截器的区别示例
通常的使用方式是**继承Interceptor类**，而添加拦截器也是在OkHttpClient.Builder.addXXX()方法中添加的，拦截器的种类有两种**应用拦截器（通过addInterceptor（Interceptor）添加）和网络拦截器（通过addNetworkInterceptor（Interceptor）添加）**。下面就通过官网的例子说明这两种拦截器的区别：  

**首先，继承Interceptor**  

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

**添加为应用拦截器**  

	OkHttpClient client = new OkHttpClient.Builder()
    	.addInterceptor(new LoggingInterceptor())
    	.build();

	Request request = new Request.Builder()
    	.url("http://www.publicobject.com/helloworld.txt")
    	.header("User-Agent", "OkHttp Example")
    	.build();

	Response response = client.newCall(request).execute();
	response.body().close();  

>The URL http://www.publicobject.com/helloworld.txt redirects to https://publicobject.com/helloworld.txt, and OkHttp follows this redirect automatically. 这是官方文档中的一句话，也就是说原来的URL最终是会被重定向到新的URL的  

**打印结果**  

	INFO: Sending request http://www.publicobject.com/helloworld.txt on null
	User-Agent: OkHttp Example

	INFO: Received response for https://publicobject.com/helloworld.txt in 1179.7ms
	Server: nginx/1.4.6 (Ubuntu)
	Content-Type: text/plain
	Content-Length: 1759
	Connection: keep-alive  
可见，这个拦截器被调用了一次，且Response处获取的Request的URL已经被重定向了。  

**添加为网络拦截器**  

	OkHttpClient client = new OkHttpClient.Builder()
    	.addNetworkInterceptor(new LoggingInterceptor())
    	.build();

	Request request = new Request.Builder()
    	.url("http://www.publicobject.com/helloworld.txt")
    	.header("User-Agent", "OkHttp Example")
    	.build();

	Response response = client.newCall(request).execute();
	response.body().close();  

**打印结果**  

	INFO: Sending request http://www.publicobject.com/helloworld.txt on Connection{www.publicobject.com:80, proxy=DIRECT hostAddress=54.187.32.157 cipherSuite=none protocol=http/1.1}
	User-Agent: OkHttp Example
	Host: www.publicobject.com
	Connection: Keep-Alive
	Accept-Encoding: gzip

	INFO: Received response for http://www.publicobject.com/helloworld.txt in 115.6ms
	Server: nginx/1.4.6 (Ubuntu)
	Content-Type: text/html
	Content-Length: 193
	Connection: keep-alive
	Location: https://publicobject.com/helloworld.txt

	INFO: Sending request https://publicobject.com/helloworld.txt on Connection{publicobject.com:443, proxy=DIRECT hostAddress=54.187.32.157 cipherSuite=TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA protocol=http/1.1}
	User-Agent: OkHttp Example
	Host: publicobject.com
	Connection: Keep-Alive
	Accept-Encoding: gzip

	INFO: Received response for https://publicobject.com/helloworld.txt in 80.9ms
	Server: nginx/1.4.6 (Ubuntu)
	Content-Type: text/plain
	Content-Length: 1759
	Connection: keep-alive  
可见，网络拦截器被调用了两次，第一次是初始化请求到 http://www.publicobject.com/helloworld.txt的时候调用,另一个用于重定向到 https://publicobject.com/helloworld.txt的时候.  
#### 2.3.2 区别分析  
![](https://upload-images.jianshu.io/upload_images/2173870-82a5eace3bd9f88c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

>从上面的调用关系可以看出除了红色圈出的拦截器之外都是系统提供的拦截器，这整个过程是递归的执行过程，在 CallServerInterceptor 中得到最终的 Response 之后，将 response 按递归逐级进行返回，期间会经过 NetworkInterceptor 最后到达自定义的 Interceptor 。  

在官网给出的 NetworkInterceptor 和 Application Interceptor 的区别在上面的图中就可以看出：

* 1.自定义 Interceptor 是第一个 Interceptor 因此它会被第一个执行，因此这里的 request 还是最原始的，并没有经过 BridgeInterceptor 进行加工过，因此他只有一个自定义的 User-Agent 请求头。而对于 response 而言呢，因为整个过程是递归的调用过程，因此它会在 CallServerInterceptor 执行完毕之后才会将 Response 进行返回，因此这里得到的 response 就是最终的响应，虽然中间有重定向，但是这里只关心最终的 response 。

* 2.NetwrokInterceptor 处于第 6 个拦截器中，它会经过 RetryAndFollowIntercptor 进行重定向并且也会通过 BridgeInterceptor 进行 request 请求头和 响应 resposne 的处理，因此这里可以得到的是更多的信息。在打印结果可以看到它内部是发生了一次重定向操作，在上图可以看出，为什么 NetworkInterceptor 可以比 Application Interceptor 得到更多的信息了。

#### 2.3.3 两种拦截器的区别
以下是官网的话：  
**应用拦截器**  

* Don't need to worry about intermediate responses like redirects and retries.不需要去关心中发生的重定向和重试操作。
* Are always invoked once, even if the HTTP response is served from the cache.只会被调用一次，即使这个响应是从缓存中获取的。
* Observe the application's original intent. Unconcerned with -OkHttp-injected headers like If-None-Match.只关注最原始的请求，不去关系请求的资源是否发生了改变，我只关注最后的 response 结果而已。
* Permitted to short-circuit and not call Chain.proceed().因为是第一个被执行的拦截器，因此它有权决定了是否要调用其他拦截，也就是 Chain.proceed() 方法是否要被执行。
* Permitted to retry and make multiple calls to Chain.proceed()
.因为是第一个被执行的拦截器，因此它有可以多次调用 Chain.proceed() 方法，其实也就是相当与重新请求的作用了。

**网络拦截器**  

* Able to operate on intermediate responses like redirects and retries.因为 NetworkInterceptor 是排在第 6 个拦截器中，因此会经过 RetryAndFollowup 进行失败重试或者重定向，因此可以操作中间的 resposne。
* Not invoked for cached responses that short-circuit the network.这个我不知道是什么意思，望大神指点。TODO
* Observe the data just as it will be transmitted over the network.观察数据在网络中的传输，具体我也不太清除是干嘛的。TODO
* Access to the Connection that carries the request.因为它排在 ConnectInterceptor 后执行，因此返回执行这个 request 请求的 Connection 连接。

### 2.4 URL的拼接  
之前我们看到的在创建一个Request对象的时候，Request.Builder.url(String)方法为Request设置url的时候传入的url是最终的Url，但是如果我们的url是需要动态输入的呢？比如url中有一个查询字符串是根据不同的客户会发生变化，这种情况下就需要我们进行Url的拼接了，而OkHttp中使用**HttpUTL与HttpURL.Builder两个类**来完成，先看一个实例：  

	HttpUrl httpUrl = new HttpUrl.Builder()  
               .scheme("http")  
               .host("www.baidu.com")  
               .addPathSegment("user")  
               .addPathSegment("login")  
               .addQueryParameter("username", "zhangsan")  
               .addQueryParameter("password","123456")  
               .build();  
	最后拼接后的URL是：http://www.baidu.com/user/login?username=zhangsan&password=123456

我们先通过一段java代码来了解一下URL各部分是什么意思：  

	URL url=new URL("https://juejin.im/search?query=okhttp3");
			System.out.println("协议名："+url.getProtocol());
			System.out.println("主机（域名）:"+url.getHost());
			System.out.println("端口号："+url.getPort());
			System.out.println("文件路径："+url.getPath());
			System.out.println("相对路径："+url.getRef());
			System.out.println("文件名："+url.getFile());
			System.out.println("查询字符串"+url.getQuery());
这段代码的输出是：  

	协议名：https
	主机（域名）:juejin.im
	端口号：-1
	文件路径：/search
	相对路径：null
	文件名：/search?query=okhttp3
	查询字符串query=okhttp3

