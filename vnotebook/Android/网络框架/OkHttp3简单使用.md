# OkHttp3简单使用
## 简介
官方地址：[https://github.com/square/okhttp](https://github.com/square/okhttp)

HTTP是现代应用常用的一种交换数据和媒体的网络方式，高效地使用HTTP能让资源加载更快，节省带宽。OkHttp是一个高效的HTTP客户端，它有以下默认特性：

* 支持HTTP/2，允许所有同一个主机地址的请求共享同一个socket连接
* 连接池减少请求延时
* 透明的GZIP压缩减少响应数据的大小
* 缓存响应内容，避免一些完全重复的请求

当网络出现问题的时候OkHttp依然坚守自己的职责，它会自动恢复一般的连接问题，如果你的服务有多个IP地址，当第一个IP请求失败时，OkHttp会交替尝试你配置的其他IP。

推荐让 OkHttpClient 保持单例，用同一个 OkHttpClient 实例来执行你的所有请求，因为每一个 OkHttpClient 实例都拥有自己的连接池和线程池，重用这些资源可以减少延时和节省资源，如果为每个请求创建一个 OkHttpClient 实例，显然就是一种资源的浪费。当然，也可以使用如下的方式来创建一个新的 OkHttpClient 实例，它们共享连接池、线程池和配置信息。

```
    OkHttpClient eagerClient = client.newBuilder()
        .readTimeout(5000, TimeUnit.MILLISECONDS)
        .build();
    Response response = eagerClient.newCall(request).execute();
```
每一个Call（其实现是RealCall）只能执行一次，否则会报异常

## 基本使用

```
    implementation 'com.squareup.okhttp3:okhttp:3.10.0'
    //添加网络权限
    <uses-permission android:name="android.permission.INTERNET"/>
```
### 异步Get请求
```
    private static String TAG = "MainActivity";
    //1. 创建client端（浏览器）
    private OkHttpClient okHttpClient = new OkHttpClient.Builder().connectTimeout(10000, TimeUnit.MILLISECONDS).build();

    /**
     * @param url
     * @param callback
     */
    private void asyncGet(String url, Callback callback) {
        //2. 创建request请求
        Request request = new Request.Builder()
                .url(url)
                .get()//默认就是GET请求，可以不写
                .build();
        //3. 操作任务
        Call call = okHttpClient.newCall(request);
        //4. 执行操作
        call.enqueue(callback);
    }

        String url = "http://wwww.baidu.com";

        //异步请求
        asyncGet(url, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Log.d(TAG, "onFailure: ");
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Log.d(TAG, "onResponse: " + response.body().string());
            }
        });

```
异步发起的请求会被加入到 Dispatcher 中的 runningAsyncCalls双端队列中通过线程池来执行。


### 同步Get请求
注意这种方式会阻塞调用线程，所以在Android中应放在子线程中执行，否则有可能引起ANR异常，Android3.0 以后已经不允许在主线程访问网络。
```
        Request request = new Request.Builder()
                .url(url)
                .build();
        final Call call = okHttpClient.newCall(request);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Response response = call.execute();
                    Log.d(TAG, "run: " + response.body().string());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
```
### Post提交String/File
这种方式与前面的区别就是在构造Request对象时，需要多构造一个RequestBody对象，用它来携带我们要提交的数据。在构造 RequestBody 需要指定MediaType，用于描述请求/响应 body 的内容类型。
![](_v_images/20200318150831587_10302.png =606x)

```

    /**
     * 
     * @param url
     * @param content
     * @param callback
     */
    private void post(String url, String content, Callback callback) {
        Request request = new Request.Builder()
                .url(url)
                .post(RequestBody.create(MediaType.parse("text/x-markdown; charset=utf-8"),content))
                .build();
        Call call = okHttpClient.newCall(request);
        call.enqueue(callback);
    }

        String content = "hello OkHttp!";
        //异步请求
        post(url, content, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Log.d(TAG, "onFailure: ");
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Log.d(TAG, "onResponse: " + response.body().string());
            }
        });
```

### Post提交流
```
RequestBody requestBody = new RequestBody() {
    @Nullable
    @Override
    public MediaType contentType() {
        return MediaType.parse("text/x-markdown; charset=utf-8");
    }

    @Override
    public void writeTo(BufferedSink sink) throws IOException {
        sink.writeUtf8("I am Jdqm.");
    }
};
```

### Post提交表单
提交表单时，使用 RequestBody 的实现类FormBody来描述请求体。
```
RequestBody requestBody = new FormBody.Builder()
        .add("search", "Jurassic Park")
        .build();
```

### Post提交分块请求
MultipartBody 可以构建复杂的请求体，与HTML文件上传形式兼容。多块请求体中每块请求都是一个请求体，可以定义自己的请求头。这些请求头可以用来描述这块请求，例如它的 Content-Disposition 。如果 Content-Length 和 Content-Type 可用的话，他们会被自动添加到请求头中。


```
    MultipartBody body = new MultipartBody.Builder("AaB03x")
            .setType(MultipartBody.FORM)
            .addPart(
                    Headers.of("Content-Disposition", "form-data; name=\"title\""),
                    RequestBody.create(null, "Square Logo"))
            .addPart(
                    Headers.of("Content-Disposition", "form-data; name=\"image\""),
                    RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
            .build();
```