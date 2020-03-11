# Http

## Http请求

* 请求行 —— 请求方法如 post/get、请求路径 url、协议版本等
* 请求头 —— 请求头(request header) 、普通头(general header)、实体头(entity header)，里面包含了很多字段
* 请求体 —— 发送的数据

## HTTP响应

* 响应行 —— HTTP版本、响应状态码（1 ：继续、2 ：成功、3 ：重定向、4 ：请求错误、5： 服务器内部错误）、状态码的描述
* 响应头 —— 响应头(response header)、普通头(general header) 、实体头(entity header)
* 响应体 —— 响应的正文数据

## Http缓存

### 强制缓存

#### 概念

当客户端第一次请求数据时，服务端返回了缓存的过期时间（Expires与Cache-Control），没有过期就可以继续使用缓存，否则不使用，无需再向服务端询问。

#### 缓存标识

##### Expires

Http1.0版本，服务器绝对时间，弃用

##### Cache-Control

* private：客户端可以缓存
* public：客户端和代理服务器都可缓存
* max-age=xxx：缓存的内容将在 xxx 秒后失效
* s-maxage=xxx：同max-age，但只针对代理服务器缓存
* no-cache：需要使用对比缓存来验证缓存数据
* no-store：所有内容都不会缓存，强制缓存，对比缓存都不会触发

### 对比缓存

#### 概念

当客户端第一次请求数据时，服务端会将缓存标识（Last-Modified/If-Modified-Since与Etag/If-None-Match）与数据一起返回给客户端，客户端将两者都备份到缓存中 ，再次请求数据时，客户端将上次备份的缓存标识发送给服务端，服务端根据缓存标识进行判断，如果返回304，则表示通知客户端可以继续使用缓存。

#### 缓存标识

##### Etag/If-None-Match

* Etag：服务端返回的当前资源标识码
* If-None-Match：将上次服务端返回的资源标识码上传给服务端

##### Last-Modified/If-Modified-Since

* Last-Modified：服务端返回的资源上次修改的时间
* If-Modified-Since：将上次服务端返回的资源上次修改时间上传给服务端

## Https

1. 客户端给出协议版本号、一个客户端生成的随机数（Client random），以及客户端支持的加密方法。
2. 服务器确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数（Server random）。
3. 客户端确认数字证书有效，然后生成一个新的随机数（Premaster secret），并使用数字证书中的公钥，加密这个随机数，发给服务器。
4. 服务器使用自己的私钥，获取客户端丝发来的随机数（即Premaster secret）。
5. 客户端和服务器根据约定的加密方法，使用前面的三个随机数，生成"对话密钥"（session key），用来加密接下来的整个对话过程。

## Cookie/Session

Cookie 是客户端保存用户信息的一种机制；Session 是服务端保存用户信息的一种机制。

## 版本区别

1.0 -> 1.1：

* 支持长连接（1.0需要在Header里加Connection： keep-alive）
* 增加了host域
* 增加了range头域，支持断点续传

1.1 -> 2.0：

* 支持多路复用
* 采用二进制分帧
* 首部压缩
* 服务器推送
