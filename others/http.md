### HTTP协议

HTTP 协议，即超文本传输协议，Hyper Text Transfer Protocol，是应用层协议，用于从 WWW 服务器传输超文本到本地浏览器的传送协议。HTTP 协议是以 ASCII 码传输的，构建于 TCP/IP 协议之上，同时承载于 TLS 或 SSL 协议之上时成为了 HTTPS （即，HTTP over SSL/TLS）协议。HTTP 协议默认的端口号是80，HTTPS 协议默认的端口号是443。

![HTTP协议在TCP/IP协议栈中的位置](/assets/20110826201312465.jpg)

#### 1、HTTP 通信过程

HTTP 协议永远都是客户端发起请求，服务器响应，是**无状态**（同一个客户端的请求与上一次请求是没有对应关系的，每次都是一个新的连接）的协议。

HTTP 协议的**连接**：

- 最初HTTP/0.9时代，是**短连接**，每次 HTTP 请求都要经历一次DNS解析、三次握手、传输和四次挥手。需要反复创建和断开 TCP 连接，开销巨大；
- HTTP/1.0 时代，开始支持持久连接，即一个 TCP 连接服务多次请求，客户端在请求头中携带`Connection:Keep-Alive`，向服务器请求持久连接，如果服务器接受持久连接，会在响应头中携带`Connection:Keep-Alive`，这样客户端可以使用同一个 TCP 连接发送多个请求；
- HTTP/1.1 时代，**持久连接**是默认的连接方式，即使请求头中没有携带`Connection:Keep-Alive`，传输也会默认以持久连接的方式进行，可以通过`Connection:Close`关闭持久连接。但是，持久连接存在 HOLB （Head of Line Blocking）的弊端，即连接中请求是串行的，如果某个请求出现网络阻塞等问题，会导致同一个连接上的后续请求被阻塞。HTTP/1.1提出了 pipelining 的概念，即同一个连接中，客户端可以在没有收到上一个请求的响应的情况下发起第二个请求，服务端根据收到请求的顺序依次返回响应，这样极大地降低了延迟。但是 pipelining 没有被广泛接受。
- HTTP/2 中，是Chrome SPDY/3 draft 的优化版本，实现了 multiplexing ，即**多路复用**，一个连接中多个请求和响应的传输可以完全混杂在一起进行，通过 streamId 进行区别，从而彻底解决了 HOLB 问题，同时运行为请求设置优先级，服务端会优先响应优先级高的请求。

HTTP 协议的**流程**：

- 浏览器建立与服务器之间的连接
- 浏览器将请求数据打包发送到 web 服务器
- web服务器处理请求，将结果打包发送给浏览器
- web服务器关闭连接

#### 2、HTTP 数据包

**HTTP 的请求数据包**由三部分（状态行、请求头、请求体，请求头与请求体之间由**空行**隔开）组成。

- **状态行**（请求行），包含了请求方法（GET、POST、PUT等）、请求URL、协议的版本和类型，比如：

  ```
  GET /search/getSolrAllPageByTitle?title=%E5%9%A9%B0 HTTP/1.1
  ```

- **请求头**（request headers），一些键值对，一般由w3c定义，浏览器与web服务器之间都可以发送，表示特定的某种含义，比如：

  ```
  Host: cmsapi.tom.com
  Connection: keep-alive
  Accept: application/json, text/javascript, */*; q=0.01
  User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.123 Safari/537.36
  DNT: 1
  Origin: http://ent.tom.com
  Referer: http://ent.tom.com/202103/1919561415.html
  Accept-Encoding: gzip, deflate
  Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
  ```

- **请求体**，发送的数据

**HTTP 的响应数据包**同样由三部分组成，状态行、响应头、响应正文，响应头与响应正文之间由**空行**隔开。

- **状态行**：协议版本、数字形式的状态代码和状态描述，各元素之间以空格分隔，比如：

  ```
  HTTP/1.1 200
  ```

- **响应头**（Response Headers）：服务器类型、日期、长度、内容类型等，比如：

  ```
  Server: nginx
  Date: Tue, 02 Mar 2021 12:09:33 GMT
  Content-Type: application/json;charset=UTF-8
  Transfer-Encoding: chunked
  Connection: keep-alive
  Vary: Origin
  Vary: Access-Control-Request-Method
  Vary: Access-Control-Request-Headers
  Access-Control-Allow-Origin: *
  ```

- 响应正文

#### 3、HTTPS

HTTPS 协议在 HTTP 协议之上增加了 TLS 或 SSL 协议而构建的加密传输、身份认证的网络安全协议，针对的是 HTTP 传输的是明文文本，不安全的问题。

##### 3.1、加密

HTTPS使用的是混合加密的方式。

- **对称加密**：加密和解密使用相同的密钥（比如AES加密），密钥的传输时可以被截获。

- **非对称加密**：加密和解密使用不同的密钥，即公钥加密的内容使用私钥解密，私钥加密的内容使用公钥解密。

  但是，非对称加密相对于对称加密来说，加解密速度慢。HTTPS 的加解密实际是结合了对称加密和非对称加密的。

  客户端请求https连接，服务端将公钥发送给客户端，客户端产生随机对称密钥，使用公钥对客户端的对称密钥加密后，将加密后的对称密钥发送给服务端，客户端和服务端使用对称密钥加密的密文通信。

  公钥也有被截获的可能，中间人可以通过伪造公钥，使用虚假的密钥进行欺骗。

- **数字签名**（digital signature）：验证传输的内容是对方发送的数据，并且发送的数据没有被篡改。

  客户端使用私钥对通信内容的摘要（digest，可以通过Hash函数获取）进行加密，生成数字签名，发送请求时，附带数字签名；客户端收到请求，使用公钥解密数字签名，得到摘要；然后，提取通信内容的摘要，通过摘要对比来确认通信内容是否被篡改。

  同样，如果无法验证公钥的真实性，公钥被伪造，数字签名也无法保证安全。

- **数字证书**（Digital Certificate）和**证书颁发机构**（Certificate Authortity，简称CA）：证书颁发机构用自己的私钥根据公钥和一些相关信息生成数字证书。

  服务端在收到客户端的加密请求后，用自己的私钥加密通信内容，将数字签名和数字证书一起发送给客户端。客户端有“受信任的根证书颁发机构”列表，客户端根据这个列表对证书进行验证，如果数字证书是信任的根证书颁发机构颁发的，客户端就使用证书中的公钥，对通信内容加密与服务端进行加密通信。

#### 4、HTTP/2

HTTP/2（超文本传输协议第2版，最初命名为 HTTP 2.0），简称为h2（基于TLS/1.2或以上版本的加密连接）或h2c（非加密连接）。

- 二进制分帧

- 多路复用：多个内容在一个Stream中传输，使用StreamId和不同优先级（priority）以保证顺序和优先级

- 使用HPACK对头部进行压缩：HTTP/1.1每次请求都要带上请求头，HTTP/2 在服务器端缓存请求头，在连接有效期间，客户端重用请求头。

  HPACK压缩分两步，首先传输的值会经过Huffman编码，为了服务端和客户端的同步，两侧都保留一份header list，每次发送请求都会检查更新。

  ![1614664862914](/assets/1614664862914.png)

  其实，header list还分为static list和dynamic list；前者存储公用头（common header），比如，method path等，后者存储自定义的头。

  HPACK压缩流程如下：

  ![1614665173953](/assets/1614665173953.png)

  请求头中有几个特殊的头:method, :scheme, :authority, 和:path，基本头前加`:`作为伪头（pseudo-header）

- 服务器推送：服务器会主动向客户端推送数据

实际上HTTP2并没有改变HTTP1.x的语义，只是把原来HTTP1.x的header和body部分用二进制帧（frame）重新封装了一层而已。

Python有`hyper`模块作为HTTP/2客户端（同时也是支持HTTP/1.1的）。

HTTP/2 的请求头、响应头，相比HTTP 1会有不同。

#### 5、HTTP 常见状态码

**1xx**：提示信息——表示请求已收到，继续处理

**2xx**：发送成功（200）

**3xx**：重定向（302）

**4xx**：客户端错误

　　400.发送请求有语法错误

　　401.访问页面没有授权

　　403.没有权限访问该页面

　　404.没有该页面

**5xx**：服务端错误

　　500.服务器内部异常

　　504.服务器请求超时，没有返回结果