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

## 参考资料
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
