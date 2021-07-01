# Http
## 定义
超文本传输协议(**H**yper**T**ext **T**ransfer **P**rotocol)是一种用于分布式、协作式和超媒体信息系统的*应用层*协议

## 特点
1. 通常使用**TCP**协议传输
2. 默认端口为 80
3. 无状态: 在同一个连接中，两个执行成功的请求是没有关系的。 (可以通过cookie来保存状态)
4. 无连接: 限制每次连接只处理一个请求。（理解成HTTP每个请求都有各自的响应体，每个请求之间彼此相互独立，即使在同一个连接中有多个请求也互不干涉）

## HTTP报文
### 请求报文
```
<method> <path> <version> // 请求行
<headers> // 请求头
// 空行
<request body> // 请求体
```
eg:
```
GET / HTTP/1.1
Host: www.example.com
Accept-Language: fr
```
### 响应报文
```
<version> <status code> <status message> // 状态行
<headers> // 响应头
// 空行
<response body> // 响应体
```
eg:
```
HTTP/1.1 200 OK
Date: Sat, 09 Oct 2010 14:28:02 GMT
Server: Apache
Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
ETag: "51142bc1-7449-479b075b2891b"
Accept-Ranges: bytes
Content-Length: 29769
Content-Type: text/html

<!DOCTYPE html... (here comes the 29769 bytes of the requested web page)
```

## HTTP请求方法
1.1版本中定义了8种方法：
1. GET - 向指定的资源发出请求
2. HEAD - 和GET方法一样，但是没有响应体
3. POST - 向指定资源提交数据，通常导致在服务器上的状态变化或副作用
4. PUT - 向指定资源位置上传其最新内容
5. DELETE - 请求删除request-uri所标识的资源
6. TRACE - 沿着到目标资源的路径执行一个消息环回测试
7. OPTIONS - 通常用来测试服务器功能是否正常
8. CONNECT - HTTP/1.1协议中预留给能够将连接改为隧道方式的代理服务器
新增方法:
PATCH - 用于对资源应用部分修改

## 版本
### HTTP/0.9
已过时。只接受GET一种请求方法，没有在通讯中指定版本号，且不支持请求头。

### HTTP/1.0
和前一个版本相比，在通讯中指定了HTTP协议版本。并且支持POST请求。

### HTTP/1.1
默认采用持续连接(Connection: Keep-Alive)，指的是不断开底层的tcp连接，如果对当前服务器还有请求发送时，复用该链接。
注意：
1. Keep-Alive: timeout=5, max=100，表示这个tcp通道可以保持5秒，最多接收100次请求就断开。
2. Keep-Alive 不能保证客户端和服务端之间的连接一定是活跃的，所以不能依赖于Keep-Alive的保持连接特性
3. 当连接关闭的时候，客户端会收到一个通知

#### pipelining
指的是将多个HTTP请求整批提交的技术，在发送过程中不需要等待服务器的回应。
1. pipeline会使得html网页加载时间动态提升。
2. 由于服务端遵循 HTTP/1.1协议，必须按照客户端发送的请求顺序来回复请求，这样整个连接还是先进先出的，则有可能出现*队头阻塞*(HOL blocking)
3. 只有GET和HEAD等方法可以进行pipelining。

### HTTP/2
借鉴于*SPDY*协议，保留了HTTP/1.1大部分语义，而采用了新的方式来编码、传输数据

#### HTTP/1.1 和 HTTP/2 的区别
1. 实现无需先入先出的多路复用
2. 简化客户端和服务端的消息 - 帧机制
3. 强制性压缩(包括HTTP头部)
4. 优先级排序
5. 双向通讯

#### 二进制分帧 - Binary Format

帧(frame)包含: 类型(Type), 长度(Length), 标记(Flags), 流标识(Stream), 有效载荷(frame payload)

消息(messsage)指的是一个完成的请求或者响应，由一个或者多个 frame 组成

流标识用于描述 frame 的格式，使得每个 frame 能够基于 HTTP/2 发送。
一个独立的 HTTP/2 连接上可包含多个并发打开的流，这个并发数由客户端控制。

请求头会封装至 headers frame 内， request body 会封装至 data frame 中进行传输。

注意:
1. 客户端发送的流 id 为奇数，而服务端发起的则为偶数
2. header 帧必须在 data 帧前，因为依赖于 header 帧解析 data 帧数据

#### 多路复用 - Multiplexing

HTTP/1.1 中，同一时间针对同一域名的请求有一定数量限制，超过限制数目的请求会被阻塞（主要是 TCP 连接）

HTTP/2 中，由于存在分帧机制，不需要依赖多个 TCP 连接实现多流并行。
每个数据流(HTTP请求)都拆分成互不依赖的 frame，并且可以交错发送和区分优先级，然后在另一端将其重新组合。

#### 头部压缩

HTTP/1.x 中，请求头带有大量信息，而且每次需要重复发送。
HTTP/2 使用 *HPACK* 算法来对 HTTP 头部进行压缩。其原理在于:
1. 客户端和服务端根据[RFC 7451](https://tools.ietf.org/html/rfc7541)附录A，维护一份共同的静态字典(Static Table)，其中包含了常见头部及常见头部名称与值的组合
2. 客户端和服务端根据先入先出的原则，维护一份可动态添加内容的共同动态字典(Dynamic Table)
3. 客户端和服务端根据[RFC 7451](https://tools.ietf.org/html/rfc7541)附录B，支持基于该静态 Huffman Table 的 Huffman Coding

#### 请求优先级

HTTP/2 中，每个 stream 可以带有一个 31 比特的优先值: 0 表示最高优先级, 2^31-1 表示最低优先级。

服务器可以根据流的优先级，在响应数据准备好之后，优先将最高优先级的帧发送给客户端。

### HTTP/3
HTTP/3 中，主要的变化是改为使用基于 UDP 协议的 QUIC 协议实现。

主要原因是因为，由于 HTTP/2 中在单个 TCP 连接上使用了多路复用，受 TCP 拥塞控制的影响，少量的丢包就可能导致 TCP 连接上所有的流被阻塞。

# HTTPS
超文本传输安全协议(**H**yper**T**ext **T**ransfer **P**rotocol **S**ecure)是一种通过计算机网络进行安全通信的传输协议。

HTTPS 经由 HTTP 进行通信，但是利用 SSL/TLS 来加密数据包，主要的目的是提供对网站服务器的身份认证，保护交换资料的隐私性与完整性。

## 主要作用
对于 HTTP 的窃听和中间人攻击提供合理的防护。

HTTPS 的信任基于预置在操作系统中的证书颁发机构(CA)。


## HTTPS 建立连接的五个阶段
1. 客户端向服务端发送支持的 SSL/TLS 协议版本号，以及客户端支持的加密方法，和一个 **随机数** (Client random)
2. 服务端确认协议版本和加密方法，向客户端发送一个**服务器生成的随机数** (Server random)，以及数字证书
3. 客户端验证证书是否有效，有效则从证书中取出公钥，生成一个**随机数** (Premaster secret) ，然后用该公钥加密，发送给服务器
4. 服务器用私钥解密，获取发来的随机数
5. 客户端和服务器根据约定好的加密方法，使用前面生成的三个随机数，生成对话密钥，用来加密后续整个会话传输的数据

# 参考资料
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

[Http2.0 - 掘金](https://juejin.cn/post/6844903984524705800)

[HTTP/2 - Google](https://developers.google.com/web/fundamentals/performance/http2?hl=zh-cn)

[HTTP/3 - Wiki](https://zh.wikipedia.org/wiki/HTTP/3)

[HTTPS - 阮一峰](https://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)

[HTTPS - segmentfault](https://segmentfault.com/a/1190000021494676)
