# Android网络

![img](https://upload-images.jianshu.io/upload_images/3832193-282a9cc20d3b8a0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

在Android中发送Http网络请求一般有三种方式，分别为HttpURLConnection、HttpClient和AndroidHttpClient。

### HttpUrlConnection

~~~java
HttpURLConnection connection = null;
try {
      URL url = new URL("http://www.baidu.com");
      connection = (HttpURLConnection) url.openConnection();
      connection.setRequestMethod("GET");
      connection.setRequestProperty("Charset", "UTF-8");
      connection.setRequestProperty("Content-Type", "text/html;charset=UTF-8");
      connection.connect();
      if (connection.getResponseCode() == 200) {
             InputStream is = connection.getInputStream();
             //do something
      }
} catch (IOException e) {
      e.printStackTrace();
} finally {
      if (null != connection) {
             connection.disconnect();
      }
}
~~~

**注意:**不过在Android 2.2版本之前，HttpURLConnection一直存在着一些令人厌烦的bug。比如说对一个可读的InputStream调用close()方法时，就有可能会导致连接池失效了。

* HttpURLConnection对象不能直接构造，需要通过URL类中的openConnection()方法来获得。

* HttpURLConnection的connect()函数，实际上只是建立了一个与服务器的TCP连接，并没有实际发送HTTP请求。HTTP请求实际上直到我们获取服务器响应数据（如调用getInputStream()、getResponseCode()等方法）时才正式发送出去。

* 对HttpURLConnection对象的配置都需要在connect()方法执行之前完成。

* HttpURLConnection是基于HTTP协议的，其底层通过socket通信实现。如果不设置超时（timeout），在网络异常的情况下，可能会导致程序僵死而不继续往下执行。

* HTTP正文的内容是通过OutputStream流写入的， 向流中写入的数据不会立即发送到网络，而是存在于内存缓冲区中，待流关闭时，根据写入的内容生成HTTP正文。

* 调用getInputStream()方法时，返回一个输入流，用于从中读取服务器对于HTTP请求的返回信息。

* 我们可以使用HttpURLConnection.connect()方法手动的发送一个HTTP请求，但是如果要获取HTTP响应的时候，请求就会自动的发起，比如我们使用HttpURLConnection.getInputStream()方法的时候，所以完全没有必要调用connect()方法。

### HttpClient

DefaultHttpClient和它的兄弟AndroidHttpClient都是HttpClient具体的实现类，它们都拥有众多的API，而且实现比较稳定，bug数量也很少。

使用HttpClient发送请求、接收响应很简单，只要如下几步即可。

- 创建HttpClient对象。
- 如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
- 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HttpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
- 调用HttpClient对象的execute(HttpUriRequest request)发送请求，执行该方法返回一个HttpResponse。
- 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。

~~~java

~~~

在Android在6.0后删除了该类库，如果仍然想要使用，需要在build.gradle文件中进行配置：

~~~json
android {
    useLibrary 'org.apache.http.legacy'
}
~~~

### 总结

在Android 2.2版本之前，HttpClient拥有较少的bug，因此使用它是最好的选择。
而在Android 2.3版本及以后，HttpURLConnection则是最佳的选择。