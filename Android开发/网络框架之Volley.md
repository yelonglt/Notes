#Volley
***
###Volley的使用
1. 创建一个RequestQueue请求队列对象Volley.newRequestQueue(context)，内部会检查手机系统版本，决定使用HttpURLConnection（版本大于9使用）还是HttpClient，然后调用start()方法。start内部会创建一个缓存调度线程CacheDispatcher和四个网络调度线程NetworkDispatcher并运行。
2. 添加请求到请求队列。在默认情况下，每条请求都是有缓存的，当然我们可以通过setShouldCache(false)禁止缓存。
3. 请求优先加入缓存队列CacheQueue，缓存调度线程CacheDispatcher从缓存中取出响应结果。如果为空，则把请求加入到网络队列NetworkQueue，如果不为空，判断是否过期，过期同样把请求加入到网络队列。
4. CacheQueue和NetworkQueue使用的是优先级阻塞队列。默认每个Request是优先级都是Normal，也可以单独重写Request的getPriority方法设置其优先级。优先级高的先被取出，假如优先级一样，则按照FIFO（先进先出）的方式取出。
5. 网络调度线程NetworkDispatcher处理请求，调用Network的performRequest()方法去发送请求，返回一个NetworkResponse对象。然后request执行parseNetworkResponse(networkResponse)，解析请求结果以及将数据写入缓存。这个方法的实现交给Request的子类来完成，因为不同种类的Request解析的方式不同，因此自定义request必须重写parseNetworkResponse()方法。解析完数据之后又会调用deliverResponse(response.result)，返回结果。因此自定义request必须重写deliverResponse()方法

###Volley的源码分析
1. request是如何执行的？缓存调度线程(CacheDispatcher)和网络调度线程(NetworkDispatcher)死循环执行;当有一个request被add时，根据缓存策略把请求添加到调度线程，假设这个request没有被添加过，请求最终会被添加到网络调度线程，执行run方法。Network接口的实现类BasicNetwork, BasicNetwork的构造方法传参HttpStack。执行网络请求，NetworkResponse networkResponse = mNetwork.performRequest(request)，最终调用HttpStack类的方法performRequest()。request根据不同的请求方式调用不同请求设置参数方法。
2. response是如何取回的？Response<?> response = request.parseNetworkResponse(networkResponse)解析返回数据；ResponseDelivery接口的实现类ExecutorDelivery，然后调用mDelivery.postResponse(request, response)返回结果。执行器执行ResponseDeliveryRunnable对象的run方法，返回调用request的deliverResponse()方法。

###Volley和OkHttp进行搭配使用
Volley默认使用的是HttpURLConnection，使用HurlStack类实现HttpStack，重写performRequest方法。假如我们使用OkHttp，可以重写performRequest方法。示例代码如下

```
public class OkHttpStack implements HttpStack {

    private static final int TIME_OUT_SECONDS = 10;
    private OkHttpClient mOkHttpClient;

    public OkHttpStack() {
        this(VolleyHelper.getSSLSocketFactory(), OkHostnameVerifier.INSTANCE);
    }

    public OkHttpStack(SSLSocketFactory sslSocketFactory, HostnameVerifier hostnameVerifier) {
        mOkHttpClient = new OkHttpClient.Builder()
                .sslSocketFactory(sslSocketFactory)
                .hostnameVerifier(hostnameVerifier)
                .connectTimeout(TIME_OUT_SECONDS, TimeUnit.SECONDS)
                .writeTimeout(TIME_OUT_SECONDS, TimeUnit.SECONDS)
                .readTimeout(TIME_OUT_SECONDS, TimeUnit.SECONDS)
                .build();
    }

    @Override
    public HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders) throws IOException, AuthFailureError {
        okhttp3.Request.Builder builder = new okhttp3.Request.Builder();

        builder.url(request.getUrl());

        Map<String, String> headers = request.getHeaders();
        for (final String name : headers.keySet()) {
            builder.addHeader(name, headers.get(name));
        }
        for (final String name : additionalHeaders.keySet()) {
            builder.addHeader(name, additionalHeaders.get(name));
        }

        setConnectionParametersForRequest(builder, request);
        okhttp3.Request okHttpRequest = builder.build();
        okhttp3.Response okHttpResponse = mOkHttpClient.newCall(okHttpRequest).execute();

        BasicStatusLine responseStatus = new BasicStatusLine(parseProtocol(okHttpResponse.protocol()), okHttpResponse.code(), okHttpResponse.message());
        BasicHttpResponse response = new BasicHttpResponse(responseStatus);
        if (hasResponseBody(okHttpRequest.method(), responseStatus.getStatusCode())) {
            response.setEntity(entityFromOkHttpResponse(okHttpResponse));
        }

        Headers responseHeaders = okHttpResponse.headers();
        for (int i = 0, len = responseHeaders.size(); i < len; i++) {
            final String name = responseHeaders.name(i), value = responseHeaders.value(i);
            if (name != null) {
                response.addHeader(new BasicHeader(name, value));
            }
        }

        return response;
    }

    private HttpEntity entityFromOkHttpResponse(okhttp3.Response response) {
        BasicHttpEntity entity = new BasicHttpEntity();
        ResponseBody body = response.body();

        entity.setContent(body.byteStream());
        entity.setContentLength(body.contentLength());
        entity.setContentEncoding(response.header("Content-Encoding"));

        if (body.contentType() != null) {
            entity.setContentType(body.contentType().type());
        }
        return entity;
    }

    private boolean hasResponseBody(String requestMethod, int responseCode) {
        return !"HEAD".equals(requestMethod)
                && !(HttpStatus.SC_CONTINUE <= responseCode && responseCode < HttpStatus.SC_OK)
                && responseCode != HttpStatus.SC_NO_CONTENT
                && responseCode != HttpStatus.SC_NOT_MODIFIED;
    }

    private void setConnectionParametersForRequest(okhttp3.Request.Builder builder, Request<?> request) throws AuthFailureError {
        switch (request.getMethod()) {
            case Request.Method.DEPRECATED_GET_OR_POST:
                byte[] body = request.getBody();
                if (body != null) {
                    builder.post(RequestBody.create(MediaType.parse(request.getBodyContentType()), body));
                }
                break;
            case Request.Method.GET:
                builder.get();
                break;
            case Request.Method.DELETE:
                builder.delete();
                break;
            case Request.Method.POST:
                builder.post(createRequestBody(request));
                break;
            case Request.Method.PUT:
                builder.put(createRequestBody(request));
                break;
            case Request.Method.HEAD:
                builder.head();
                break;
            case Request.Method.OPTIONS:
                builder.method("OPTIONS", null);
                break;
            case Request.Method.TRACE:
                builder.method("TRACE", null);
                break;
            case Request.Method.PATCH:
                builder.patch(createRequestBody(request));
                break;
            default:
                throw new IllegalStateException("Unknown method type.");
        }
    }

    private RequestBody createRequestBody(Request<?> request) throws AuthFailureError {
        final byte[] body = request.getBody();
        if (body == null) return null;

        return RequestBody.create(MediaType.parse(request.getBodyContentType()), body);
    }

    private ProtocolVersion parseProtocol(final Protocol protocol) {
        switch (protocol) {
            case HTTP_1_0:
                return new ProtocolVersion("HTTP", 1, 0);
            case HTTP_1_1:
                return new ProtocolVersion("HTTP", 1, 1);
            case SPDY_3:
                return new ProtocolVersion("SPDY", 3, 1);
            case HTTP_2:
                return new ProtocolVersion("HTTP", 2, 0);
        }

        throw new IllegalAccessError("Unkwown protocol");
    }
}

```
###相关问题
1. 为什么说Volley适合数据量小，通信频繁的网络操作？（优缺点）
   
   1. ByteArrayPool根据类名，知道这是一个字节数组缓存池。这是用来缓存网络请求获得的数据。当网络请求得到返回数据以后，我们需要在内存中开辟出一块区域来存放我们得到的网络数据，不论是json还是图片，都会存在于内存的某一块区域，然后显示在UI上。
   2. 假如频繁的请求数据，那么拿到数据就需要在堆内存中开辟存储空间，然后GC回收这块区域，如此反复造成GC频繁影响性能。ByteArrayPool的实现原理是通过实现缓存从而减少GC。
   3. ByteArrayPool维护两个list集合。LinkedList mBuffersByLastUse按照最近使用对byte[]排序；ArrayList mBuffersBySize按照byte[]大小对byte[]数组排序。
   4. 所以Volley适合数据量小的，因为拿到数据都是内存操作，数据量大譬如大图片容易造成内存溢出；通信频繁的操作是因为使用了字节数组缓存池，可以减少GC操作。