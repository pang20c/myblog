---
title: http 知识点总结
date: 2017-07-24 18:20:57
tags: 
- http
- 网络
category:
- 网络
---

# 格式
HTTP报文 由 `报文头部` 和 `报文主体` 组成，两者用CRLF（\r\n）分割
报文头部 包括 请求行（请求） 或 状态行（响应）、首部字段、和其他（例如cookie）
首部字段也是用CRLF分割


# 常用错误码
* 301 永久跳转
* 302 临时跳转
* 304 文件未修改
* 400 请求语法错误
* 402 请求未授权
* 403 禁止访问
* 503 服务不可用

# 长连接（keep-alive）
TCP链接建立需要三次握手，传输完数据又需要四次握手，如果每传输一次数据就建立一个tcp链接那是十分浪费资源的，为了能更高效的利用tcp链接在HTTP/1.0 中提出了`Connection: keep-alive`（在HTTP/1.1中这是个默认选项，不在需要客户端显示发送，能否启用该功能取决于服务端设定）,作用是当一块数据发送完之后不会关闭链接，而是等待后续的tcp请求，注意这里是等待tcp而不是等待后续的http，因为即使用了keep-alive 一个tcp链接也是只能处理一个http的请求响应的。而keep-alive带出的一个问题是接收数据的一端（一般是浏览器）如何判断这个链接的数据已经发送完了。有两种办法 一种是服务端在这次请求中事先就告诉客户端内容的长度即设置 `Content-Length:`头，比如`Content-Length:100` 那么客户端在收到第一百个字节的时候（指报文主体长度）就关闭链接，另一种是下面介绍的`分块传输`

# 分块传输(Transfer-Encoding: chunked)
分块编码主要应用于如下场景，即要传输大量的数据，但是在请求在没有被处理完之前响应的长度是无法获得的，服务器无法预先知道`Content-Length`，这时候就可以在http头部设定`Transfer-Encoding: chunked` 告诉客户端我后面的数据是分块传输的，而每一块数据包括16进制的长度和真正的数据,而最后一个块必须是一个长度为0的块表示结束，客户端收到这个这个长度为0的块就可以发起`FIN`请求关闭链接了，更详细参见[这里](https://imququ.com/post/transfer-encoding-header-in-http.html)

```
HTTP/1.1 200 OK\r\n
Transfer-Encoding: chunked\r\n
\r\n

b\r\n
01234567890\r\n

5\r\n
12345\r\n

0\r\n
\r\n
```

# Cache-Control 缓存处理策略
| 值|含义 |
| --- | --- |
| public | 代理服务器和客户端都可缓存 |
| private | 仅客户端可以缓存 |
| no-cache | 请求头时表示不要缓存服务器的内容直接请求源服务器，响应头时表示浏览器每次必须先与服务器确认返回的响应是否被更改，然后才能使用该响应来满足后续对同一个网址的请求。 |
| no-store | 所有内容都不会被缓存到缓存或 Internet 临时文件中 |
| must-revalidation/proxy-revalidation | 如果缓存的内容失效，请求必须发送到服务器/代理以进行重新验证 |
| max-age=xxx | 请求头时表示只接受age小于这个值的缓存， 响应头时表示缓存的内容将在 xxx 秒后失效, 这个选项只在HTTP 1.1可用, 并如果和Last-Modified一起使用时, 优先级较高 |






# 常用头部
## 请求头
* Connection ： close告诉服务端请求完不要保持这个链接直接关闭，keep-alive 告诉服务端请求完别关闭链接 还有后续请求。
* If-Modified-Since：如果请求的对象在该头部指定的时间之后修改了，才执行请求的动作（比如返回对象），否则返回代码304，告诉浏览器该对象没有修改。例如：If-Modified-Since：Thu, 10 Apr 2008 09:14:42 GMT
* If-None-Match：如果对象的 ETag 改变了，其实也就意味著对象也改变了，才执行请求的动作。

## 响应头
* Cache-Control 缓存处理策略
* Content-Encoding 压缩方式gzip
* Content-Length 响应长度
* Content-Type 内容的类型 application/xml
* Transfer-Encoding 传输编码方式 目前只有chunked
* ETag 响应内容的签名

# 安全

## csrf 跨站请求伪造  防范
* 判断referer
* 表单中加入随机token，并且每次表单提交后变换token

## xss 跨站脚本攻击 防范
* 过滤html java用spring的 HtmlUtils.htmlEscape php 用 htmlspecialchars
* 必须输入html的地方 设置标签白名单，解析html重新整理。





