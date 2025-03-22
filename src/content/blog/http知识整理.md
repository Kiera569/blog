---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: HTTP知识整理
slug: HTTP知识整理
featured: false
draft: false
tags:
  - HTTP
description: HTTP相关知识整理
---

## 1. 启发式缓存

在不设置**cache-control/expires**的情况下，浏览器不会默认进入协商缓存。而是根据Date/LastModified去自动计算出合适的缓存时间。
计算方式为：(Date - LastModified) \* n
n：LM-Factor，处于[0,1]之间

## 2. 强制缓存 ---- Cache-control: max-age=60 （60s内直接使用缓存内容）

特定时间内直接使用本地缓存的资源，无需再次发送请求到服务器。

- **Cache-control:** 具有强大的缓存控制能力；

  常用值：

  - **no-cache:** 每次请求需要校验服务器资源的新鲜度（协商缓存）。
  - **max-age=31536000:** 浏览器在一年内都不需要向服务器请求资源。
  - **Expires:** 过期时间，使用绝对时间。

## 3. 协商缓存 ---- Cache-control： no-cache;

通过服务器来判断资源是否被修改，如果被修改则返回最新资源；如果未被修改则返回304，从缓存（浏览器）中获取；

- **Last-Modified/If-Modified-Since:** 匹配响应头的Last-Modified与请求头的If-Modified-Since是否一致。
- **Etag/If-None-Match:** 匹配响应头中的Etag与请求头中的If-None-Match是否一致。

## 4. Last-Modified 局限性

a. 时间精度只有秒级别，但某文件在一秒内可以被更改n次。

b. 某一文件经过修改后但内容并未发生改变。（比如添加一行再删除一行）

## 5. Etag --- Last-Modified的‘补丁’，精度更高。

针对文件内容进行hash计算，得到的hash值作为Etag的值。

## 6. 图片防盗链

a. **Referrer**: 标识网页来源地址。

原理：通过判断referrer来验证访问是否合法，不合法则返回403，禁止访问图片。

b. 给图片添加水印。

## 7. 防止图片防盗链

不发referer请求头，两种方式：

a. 直接浏览器输入url，自动不会携带referrer。

b.Referrer-Policy设置不发送Referrer请求头： **Referrer-Policy: no-referrer**

## 8. User-Agent

标识客户端，简称UA.
通过该头可获取到所用浏览器、浏览器版本、操作系统等信息。

## 9. 浏览器如何判断PC/Mobile

推荐库：*https://github.com/kaimallea/isMobile*

## 10. 项目缓存策略

对带有hash值的文件资源设置一年强缓存；（当源文件内容发生变更时，hash值发生改变，生成新的可永久缓存的资源地址，此时打包会遍历文件，更新缓存地址并清除掉旧缓存）。
对不带hash值得文件资源**一定要显式的配置\*\***Cache-Control: no-cache**。
如果不配置，就会触发启发式缓存，从而造成应用升级但是刷新不生效的问题。
（\***tips:\*\*\* 不设置强缓存的时候，默认进入的是启发式缓存，而不是协商缓存）

## 11. No-cache/No-store 区别

**no-cache:** 可以在客户端存储资源，每次都必须去服务端做新鲜度校验，以此来决定从服务端获取新资源还是从客户端获取缓存。（协商缓存）
**no-store**：永远不在客户端存储资源，每次都去原始服务器获取资源。（彻底不缓存）

## 12. Max-age = 0,代表什么？

**max-age=0** 暗示内容立即被认为是陈旧的（并且必须重新获取），这实际上与 Cache-Control: no-cache 相同。
**no-cache**跟**max-age=0**作为请求头，二者均会重新向服务器发起请求，哪怕该请求已被缓存。

## 13. 预检请求？

当一个请求跨域且不是简单请求时，就会发起一个OPTIONS请求，被称为预检请求。

预检请求用于检查服务器是否支持CORS, 以便确认最终是否发送正常请求，由于它通过响应头进行判断，无需 Body，因此关于 OPTIONS 请求的状态码可选择 **204 No Content**。

如果预检请求失败，不允许跨域，则该复杂请求则会在浏览器控制台看到 **CORS Error**，并将该请求标记为红色。

在 Chrome 浏览器控制台中，跨域复杂请求旁，有 **pre-flight** 链接指向该请求的预检请求

预检请求一般会携带以下**三个请求头**，用以向服务器咨询是否支持该跨域请求：

**是否支持 POST 方法**
Access-Control-Request-Method: POST

**是否支持携带 content-type 请求头**
Access-Control-Request-Headers: content-type

**来自域名 https://shanyue.tech 的跨域请求**
Origin: https://shanyue.tech
同时回复响应头，**用以确认是否支持CORS**.

**允许跨域的头部**
Access-Control-Allow-Headers: Content-Type,Content-Length,Accept-Encoding,X-CSRF-Token,accept,origin,Cache-Control,X-Requested-With,X-USE-PPE,X-TT-ENV

**允许跨域的方法**
Access-Control-Allow-Methods: POST, OPTIONS, GET

**允许跨域的域名**
Access-Control-Allow-Origin: \*

**OPTIONS 预检请求的缓存时间**，即在 600s 内不会再次发送 OPTIONS 请求
Access-Control-Max-Age: 600

**是否允许携带权限信息，比如 cookie 一类**，
Access-Control-Allow-Credentials: true

## 14. 简单请求

满足以下条件的即为简单请求：

- **1>. Method**: 请求方法是GET，POST以及HEAD
- **2>. Header:** 请求头是conten-Type且其值为:Accept-Language、Content-Language.
- **3>. Content-Type**: 请求类型是 **application/x-www-form-urlencoded**、multipart/form-data 或 text/plain

## 15. 非简单请求

一般需要开发者主动构造，常见的简单请求有：content-type: application/json及Authorization:<token>。

## 16. 设置默认携带cookie。（不设置则不携带）

fetch 发送请求时配置 **credentials: include**
server 响应时，配置响应头 **access-control-allow-credentials: true**
另外，无法向配置 **access-control-allow-origin: \*** 的域名发送 Cookie，否则报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/4ec636dd8f314d65b924f546012f7700.png)

## 17. Fetch credentials

- **credentials**指在使用fetch发送请求时是否应当发送cookie.
- **Omit**: 从不发送
- **Same-origin:** 同源时发送，跨域则不发送。
- **Include:** 同源与跨域都会发送cookie。

## 18. Access-Control-Allow-Origin

取值为：

a. \*：这个值表示允许来自所有域的跨源请求。

    Access-Control-Allow-Origin: \*

b. 源的URI：如果你只希望允许特定的源进行跨源请求，那么就可以设置为那个源的URI。

    Access-Control-Allow-Origin: http://example.com

## 19. 如何设置多个域名？

根据Origin请求头来设置响应头：Access-Control-Allow-Origin;

a. 总是设置 Vary: Origin，避免 CDN 缓存破坏 CORS 配置

b. 如果请求头不带有 Origin，证明未跨域，则不作任何处理

c. 如果请求头带有 Origin，证明浏览器访问跨域，根据 Origin 设置相应的 Access-Control-Allow-Origin: \<Origin\>

## 20. Vary: Origin，代表为不同的origin缓存不同的资源。

使用语法为：Vary:\<header-name>,\<header-name>, ...
可理解为用以缓存的 key 值，常用于内容协商，因为内容协商往往会根据 Accept/Accept-Language 返回不同的资源。

## 21. 同域名浏览器会自动携带上cookie。

## 22. Cookie的属性 --- 可跨域但不能跨站。

- **name**: 名称
- **value**: 值
- **Domain**: 为Cookie指定的域名。
- **Path**: 为Cookie指定的路径
- \***\*Expire**/Max-age\*\*: Cookie的缓存时间。
- **HttpOnly**：无法通过JS修改Cookie,但可以再浏览器看到其值。（可避免XSS攻击--跨站脚本攻击：通过注入脚本的方式获取到cookie,避免XSS攻击最有效的办法是CSP（内容安全策略），也是通过http配置.）
- **Secure**: 仅能通过HTTPS协议传递
- **SameSite**: 跨站点Cookie发送策略 (可用于防止CSRF攻击)

## 23. CSRF（cross-site request forgery）

跨站请求伪造，通过恶意引导用户一次点击劫持cookie及逆行攻击。

## 24. SameSite的值？

- **None**: 任何情况下都会向第三方网站请求发送Cookie.
- **Lax**: 只有导航到第三方网站的Get链接会发送cookie,而跨域的图片iframe,fetch请求,form Post表单等都不会发送cookie.
- **Strict**: 任何情况下都不会向第三方网站请求发送cookie。

## 25. 状态码

- **100**: 服务器检查请求头是否有问题。没问题返回100，有问题返回417.
- **101**： 表示正在切换协议，如从http切换到websocket.
- **103**: 提前声明对某些资源的优化提示
- **preload**： 预加载。
- **preconnect**: 提前建立链接。
- **201**: Create; 资源创建成功。（github 创建 issue）。
- **204**：no-content，请求成功不返回内容；
- **206**： 当客户端指定 Range 范围请求头时，服务器端将会返回部分资源。当请求音视频资源体积过大时，一般使用 206 较多
- **301**：moved permanently,永久重定向。（如果设置了 301，后续想取消但浏览器中已有缓存，还是会重定向）
- **302**：found,临时重定向。但是会在重定向的时候改变 method:把 post 改成 get，于是就有了 307。
- **304**：not Modified
- **307**：temporary redirect，临时重定向，会改变 method
- **308**：permanent redirect，永久重定向，在重定向时不会改变 method。
- **400**：bad request;
- **401**：unAuthrized; 没有权限;(当没有权限的用户请求需要带有权限的资源时，会返回 401，此时携带正确的权限凭证再试一次可以解决问题)
- **403**：forbidden 禁止访问
- **405**：method not allowed.请求方法错误
- **406**：not acceptable, 内容协商失败
- **413**： Request Body过大，服务器处理不过来
- **429**: too many request，超过某一个 API 的 Rate Limit 规则，会被限流.(不同的接口，有不同的限流规则)
- **500**：Internal Server Error,服务器内部错误
- **502**：Bad Gateway，网关失败
- **503**：Service Unavailable，服务不可访问
- **504**：Gateway Timeout，网关超时

## 26. 502跟504区别

- **502 Bad Gateway**: 一般表现为你自己写的应用层服务(Java/Go/PHP)挂了，或者网关指定的上游服务直接指错了地址，网关层无法接收到响应

- **504 Gateway Timeout**: 一般表现为应用层服务 (Upstream) 超时，超过了 Gatway 配置的 Timeout，如查库操作耗时三分钟，超过了 Nginx 配置的超时时间

## 27. 什么是内容协商

用于决定客户端和服务器之间最佳匹配的资源版本；根据请求头响应最符合的资源内容；

相关请求头：

a. Accept: 指定资源媒体类型，如：text/html；

b. Accept-Language: 指定语言类型，如：zh=CN;

c. Accept-Charset: 指定字符集，如： utf-8

d. Accept-EnCoding: 指定编码格式，如：gzip/br

相关响应头：

a. Content-Language: 响应体的自然语言

b. Content-Type: 响应体的MIME类型

c. Content-Encoding: 响应体的编码格式

## 扩展

**1. etag的生成策略**

(详见参考：https://blog.csdn.net/weixin_42989576/article/details/123695991）

**静态文件（如css,js,图片等**）：文件大小的16进制+修改时间

**字符串或Buffer**：字符串/Buffer长度的16进制+对应hash值。
