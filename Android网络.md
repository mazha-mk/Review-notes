# Android网络

![img](https://upload-images.jianshu.io/upload_images/3832193-282a9cc20d3b8a0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

在Android中发送Http网络请求一般有三种方式，分别为HttpURLConnection、HttpClient和AndroidHttpClient。

### HttpUrlConnection

```java
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
```

**注意:**不过在Android 2.2版本之前，HttpURLConnection一直存在着一些令人厌烦的bug。比如说对一个可读的InputStream调用close()方法时，就有可能会导致连接池失效了。

- HttpURLConnection对象不能直接构造，需要通过URL类中的openConnection()方法来获得。
- HttpURLConnection的connect()函数，实际上只是建立了一个与服务器的TCP连接，并没有实际发送HTTP请求。HTTP请求实际上直到我们获取服务器响应数据（如调用getInputStream()、getResponseCode()等方法）时才正式发送出去。
- 对HttpURLConnection对象的配置都需要在connect()方法执行之前完成。
- HttpURLConnection是基于HTTP协议的，其底层通过socket通信实现。如果不设置超时（timeout），在网络异常的情况下，可能会导致程序僵死而不继续往下执行。
- HTTP正文的内容是通过OutputStream流写入的， 向流中写入的数据不会立即发送到网络，而是存在于内存缓冲区中，待流关闭时，根据写入的内容生成HTTP正文。
- 调用getInputStream()方法时，返回一个输入流，用于从中读取服务器对于HTTP请求的返回信息。
- 我们可以使用HttpURLConnection.connect()方法手动的发送一个HTTP请求，但是如果要获取HTTP响应的时候，请求就会自动的发起，比如我们使用HttpURLConnection.getInputStream()方法的时候，所以完全没有必要调用connect()方法。

### HttpClient

DefaultHttpClient和它的兄弟AndroidHttpClient都是HttpClient具体的实现类，它们都拥有众多的API，而且实现比较稳定，bug数量也很少。

使用HttpClient发送请求、接收响应很简单，只要如下几步即可。

- 创建HttpClient对象。
- 如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
- 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HttpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
- 调用HttpClient对象的execute(HttpUriRequest request)发送请求，执行该方法返回一个HttpResponse。
- 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。

```java

```

在Android在6.0后删除了该类库，如果仍然想要使用，需要在build.gradle文件中进行配置：

```json
android {
    useLibrary 'org.apache.http.legacy'
}
```

### 总结

在Android 2.2版本之前，HttpClient拥有较少的bug，因此使用它是最好的选择。
而在Android 2.3版本及以后，HttpURLConnection则是最佳的选择。

# TCP与IP

### TCP/IP模型

基于TCP/IIP的参考模型将协议分为四个层次，链路层，网络层，传输层和应用层。

OSI模式与TCP/IP模型的对照：

![](http://upload-images.jianshu.io/upload_images/3985563-0cc2edef95a71669.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "OSI与TCP/IP")

数据入栈与出栈的过程:

![img](http://upload-images.jianshu.io/upload_images/3985563-8a876f4d031aeacf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 数据链路层

数据链路层负责将0、1序列划分为数据帧从一个节点传输到临近的另一个节点，这些节点是通过MAC来唯一标识的。MAC地址即网卡的唯一标识地址，用来识别不同的节点。

![img](http://upload-images.jianshu.io/upload_images/3985563-626f9f89f361c492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 封装成帧：把网络层数据报加头和尾，封装成帧，帧头中包括源MAC地址和目的MAC地址。
- 透明传输：零比特填充、转义字符。
- 可靠传输：在出错率很低的链路上很少用，但是无线链路WLAN会保证可靠传输。
- 差错检测(CRC)：接收者检测错误,如果发现差错，丢弃该帧。

### 网络层

#### IP地址

32位IP地址分为网络位和地址位。

- A类IP地址：1.0.0.0~127.0.0.0
- B类IP地址：128.0.0.0~191.255.255.255
- C类IP地址：192.0.0.0~223.255.255.255

##### IP协议首部：

![img](http://upload-images.jianshu.io/upload_images/3985563-f17dbaa542787fdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ARP及RARP协议

ARP 是根据IP地址获取MAC地址的一种协议。

> 检查ARP高速缓存是否有对应的IP-MAC值,有则返回
>
> 没有，进行一次广播，收到广播的所有主机会检查自己的IP地址，若发现自己符合条件，就会发送一个包含自己信息的ARP包传回。即可得到对应IP-MAC值。

#### ICMP协议

ICMP(网络控制报文)协议，用来保证数据送达的工作。

比如主机不可达，路由不可达等等，ICMP协议将会把错误信息封包，然后传送回给主机

### TCP/UDP

![img](http://upload-images.jianshu.io/upload_images/3985563-4bcddf13416e9f0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

TCP和UDP协议的一些应用:

![img](http://upload-images.jianshu.io/upload_images/3985563-d68974689d48cee9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### DNS:（Domain Name System，域名系统）

因特网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。

**主机 ------> 本地域名服务器：一般都是采用递归查询**

​        如果主机所询问的本地域名服务器不知道被查询的域名的IP地址，那么**本地域名服务器**就以DNS客户端的身份（递归思想），向**根域名服务器**继续发出查询报文(替主机查询)，不让主机自己进行查询。递归查询返回的结果或者是IP，或者报错。这是从上到下的递归查询过程。

**本地域名服务器 ------> 根域名服务器：一般采用迭代查询**

​        当**根域名服务器**收到**本地域名服务器**的查询请求，要么给出ip，要么通知本地域名服务器下一步应该去请求哪一个**顶级域名服务器**查询（并告知本地域名服务器自己知道的顶级域名的IP），让本地域名服务器继续查询，而不是替他查询。同理，顶级域名服务器无法返回IP的时候，也会通知本地域名服务器下一步向谁查询（查询哪一个权限域名服务器）……这是一个迭代过程。

#### TCP连接的建立与终止

##### 三次握手

三次握手的目的是同步连接双方的序列号和确认号并交换 TCP窗口大小信息。

![img](http://upload-images.jianshu.io/upload_images/3985563-cd5a153e44696643.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

|            | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 第一次握手 | 将同步为SYN置为1，表明这是一个连接请求报文段，序号Seq为X，客户端进入SYN_SEND状态 |
| 第二次握手 | 服务端接收到SYN报文段，发送一个同步SYN置1，序号为Y，确认ACK=X+1的报文段，进入SYN_RECV状态 |
| 第三次握手 | 客户端收到服务器的SYN+ACK报文段，将确认ACK为y + 1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。 |

##### 为什么要三次握手？

若没有三次握手，client发送一个连接请求报文，该报文由于网络滞留，延误到连接超时才到达sever，sever接收到已失效的报文段后，发送确认报文，就会建立新的连接，一直等待并没有连接的client，造成资源的浪费。

也就是防止服务器端因接收了**早已失效的连接请求报文**，从而一直等待客户端请求，最终导致**形成死锁、浪费资源**

#### 四次挥手

![img](https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike116%2C5%2C5%2C116%2C38/sign=5c988baea3cc7cd9ee203c8b58684a5a/b58f8c5494eef01fca1e8886e0fe9925bc317d6f.jpg)

FIN：1表示已经没有数据要发送了

Wait 2MSL （最长报文段寿命）再closed：

1. 保证最后的连接释放确认报文能到达服务器，让服务器正常释放

![img](https://upload-images.jianshu.io/upload_images/944365-1cfff0282bdac472.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/880/format/webp)

2. 防止错过服务器因连接释放确认报文丢失而发送的超时重发的连接可以释放报文

##### 为什么要四次挥手？

主机1发出FIN报文段时，只是表示主机1已经没有数据要发送了，但主机1还是可以接受来自主机2的数据；当主机2返回ACK报文段时，表示它已经知道主机1没有数据发送了，但是主机2还是可以发送数据到主机1的；当主机2也发送了FIN报文段时，这个时候就表示主机2也没有数据要发送了，主机2收到主机1的ACK报文段以后，就可以关闭连接。

### 无差错传输

#### 滑动窗口

![img](https://upload-images.jianshu.io/upload_images/944365-9675d7fa2007d374.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/860/format/webp)

只有接收窗口向前滑动、接收方发送了确认帧时，发送窗口才有可能（只有发送方收到确认帧才是一定）向前滑动。

#### 无差错传输的解决方案

1. 出现差错时，让发送方重传差错数据：即 出错重传
2. 当接收方来不及接收收到的数据时，可通知发送方降低发送数据的效率：即 速度匹配

针对上述2个问题，分别采用的解决方案是：自动重传协议 和 流量控制 & 拥塞控制协议。

### 自动重传请求协议ARQ

![img](https://upload-images.jianshu.io/upload_images/944365-b35ec57c26668491.png?imageMogr2/auto-orient/)

![img](https://upload-images.jianshu.io/upload_images/944365-30fd78ac1589939f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740/format/webp)

#### 连续ARQ协议

即后退N帧协议。

发送方：采用多帧滑动窗口的原理，可连续发送多个数据帧 而不需等待对方确认
接收方：采用 **累计确认 & 后退N帧**的原理，只允许按顺序接收帧。具体原理如下：

![img](https://upload-images.jianshu.io/upload_images/944365-2784660582f0ff18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

### TCP流量控制

所谓**流量控制**就是让发送方的发送速率不要太快，要让接收方来得及接收。

利用**滑动窗口机制**可以很方便地在TCP连接上实现对发送方的流量控制。

![img](http://upload-images.jianshu.io/upload_images/3985563-846230b2b91e5696.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**大写ACK表示首部中的确认位ACK，小写ack表示确认字段的值ack。**

**rwnd表示窗口的大小**

### TCP拥塞控制

发送方维持一个拥塞窗口 cwnd ( congestion window )的状态变量，发送方会让自己的发送窗口等于拥塞窗口。

发送方控制拥塞窗口的原则是：只要网络没有出现拥塞，拥塞窗口就再增大一些，以便把更多的分组发送出去。但只要网络出现拥塞，拥塞窗口就减小一些，以减少注入到网络中的分组数。

#### 慢开始和拥塞避免

需要设置一个慢开始门限ssthresh状态变量。慢开始门限ssthresh的用法如下：

- 当 cwnd < ssthresh 时，使用上述的慢开始算法。
- 当 cwnd > ssthresh 时，停止使用慢开始算法而改用拥塞避免算法。
- 当 cwnd = ssthresh 时，既可使用慢开始算法，也可使用拥塞控制避免算法。

##### 慢开始：先把cwnd设置为一个最大报文段MSS，每经过一个传输轮次，拥塞窗口 cwnd 就加倍。

![img](http://upload-images.jianshu.io/upload_images/3985563-470235b1e99d8111.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 拥塞避免：每经过一个传输轮次，拥塞窗口 cwnd 加一，而不是加倍。

![img](http://upload-images.jianshu.io/upload_images/3985563-8bbeee141535e125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**当出现拥塞时（其根据就是没有收到确认），就把慢开始门限ssthresh=当前cwnd/2，然后cwnd置为一**

#### 快重传和快恢复

##### 快重传：发送方只要一连收到三个重复确认就应当立即重传对方尚未收到的报文段M3，而不必 继续等待M3设置的重传计时器到期。

![img](http://upload-images.jianshu.io/upload_images/3985563-3e4f9c585200ed18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##### 快恢复：当发送端连续收到三个重复确认，将cwnd=ssthresh/2，同时将慢开始门限ssthresh=ssthresh/2，并执行拥塞避免算法。

![img](http://upload-images.jianshu.io/upload_images/3985563-8e0bf3c2c9554f5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# HTTP

### 状态码

##### 1xx 请求正在处理

##### 2xx请求成功

* 200：ok
* 204：No Content （返回的响应报文不含实体）

##### 3xx重定向

* 301：永久性重定向
* 302：临时性重定向
* 303：请求对应资源存在于另一个URI，应使用GET方法定向获取
* 304：请求未满足条件（与重定向无关）
* 307：同302，临时性重定向

##### 4xx客户端错误

* 400：请求报文存在语法错误
* 401：表示需要认证信息或认证失败
* 403：资源被服务端拒绝请求
* 404：请求的资源不存在

##### 5xx服务端错误

* 500：服务端执行请求发生错误，出现了bug
* 503：服务端正在停机维护或者超负载状态

# HTTPS

## 加密类型

### 非对称加密

分为公钥与私钥。公钥是公开的，别人可以获取的到；私钥是私有的，只能自己拥有。公钥和私钥都可以将信息进行加密，并且能用对应的密钥将信息解码，也就是说，如果用公钥将信息加密，那么只可以私钥将信息解码，反之，如果用私钥将信息加密，只可以用公钥将信息解码！

### 对称加密

只有私钥，两方用同一套的密钥来进行加密解密。

## 数字签名

用来，保证信息传输的**完整性**、发送者的身份认证、防止交易中的抵赖发生。

>  A将原始明文通过 hash 算法得到摘要`a`，这个摘要`a`是不可逆的；将明文加密，连同摘要`a`一起就是数字签名。A将数字前面的信息报文发送给B；B接收到后解密，得到这个摘要 `a` 和加密的明文，再将加密明文解密得到原始明文，然后通过同一 hash 算法得到新的摘要 `b`，比较 `a` 与 `b` 就可得知在传输过程中是否被更改过。

## 数字证书

用来认证这个公钥是属于谁的，验证公钥的真伪。

"证书中心"（certificate authority，简称CA），为公钥做认证。证书中心用自己的私钥，对需要认证的公钥和一些相关信息一起加密，生成"数字证书"（Digital Certificate）。

认证数字证书的真实性，即用CA的公钥解密数字证书即可查看该认证的公钥的相关信息是否符合正确。例：

> A想要和B进行通信，首先就要告知B自己的公钥，A先向B发送自己的数字证书，B收到数字证书后，会向权威的数字证书认证中心进行认证，确认是否是A的数字证书。（这个认证的过程，实际上也是通过公钥和私钥的机制，B会根据数字证书的类别，查找发布这个数字证书的中心的公钥，然后用相应的公钥对证书进行家解码，如果能正确解码则说明这个数字证书确实是此中心颁布的，然后根据解码后的信息验证是否是A的数字证书，最后从解码后的信息中，获取A的公钥）。

## SSL握手

![img](https://upload-images.jianshu.io/upload_images/128529-abd6f1e1e6e126b5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/638/format/webp)

### 一.Client Hello

客户端向服务端发送client hello消息，其中包括一个客户端生成的随机数random1，以及客户端支持的加密套件和ssl版本等信息。

### 二.Server Hello

服务端向客户端发送server hello消息，其中包括一个服务端生成的随机数random2，以及从客户端支持的加密套件中确认的一份加密方式的信息，包括后续加密和生成摘要的算法等。此时客户端和服务端都拥有了两个随机数（Random1+ Random2）。

### 三.Server Hello Done

服务端会向客户端发送一系列的证书，并通知客户端 server_hello 信息发送结束

### 三.certificate

客户端验证服务端的身份，客户端验证通过后取出证书中的公钥。然后再生成一个随机数random3，用取出的公钥对random3进行非对称加密，生成 **PreMaster Key**。

### 四.Pre-master secret

将这个**PreMaster Key**传给服务端，服务端再用自己的私钥解出这个 **PreMaster Key** 得到客户端生成的 Random3。至此，客户端和服务端都拥有 Random1 + Random2 + Random3，两边再根据同样的算法就可以生成一份秘钥。

### 五.Client Finished

服务端通知客户端后面再发送的消息都会使用加密

### 六.Server Finished

服务端也会将握手过程的消息生成摘要再用秘钥加密，这是服务端发出的第一条加密消息。客户端接收后会用秘钥解密，能解出来说明协商的秘钥是一致的。

### 七.握手结束

开始使用协商密钥与算法进行加密通信。

