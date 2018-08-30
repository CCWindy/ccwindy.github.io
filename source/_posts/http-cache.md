---
title: HTTP 缓存
date: 2018-08-27
tags: HTTP cache
---
当用户重新访问一个网站时，很多时候请求的数据内容（包含文件和 API 响应数据）都是相同并未改变的。
浏览器及代理服务器等已实现了本地缓存功能，若服务器对这些不变的数据内容进行再次返回，将对网络资源造成不必要的浪费。
HTTP 协议提供了多种用于缓存控制的头部字段，本文将对这些字段进行总结与对比。
<!-- more -->

### HTTP 1.0  的缓存技术
#### 1. Pragma
HTTP 1.0 使用 [Pragma](https://www.w3.org/Protocols/HTTP/1.0/spec.html#Pragma) 请求头字段禁止直接使用缓存，将 Pragma 的值设为 <code>no-cache</code> 将强制客户端不使用缓存直接向服务器进行请求。
在 HTTP 1.1 协议中，设置 Pragma 字段值( HTTP 1.1 中 Pragma 字段值也只能设为 <code>no-cache</code> )和将 Cache-Control 字段值设为 <code>no-cache</code> 的作用是一样的。
但当发送 <code>no-cache</code> 请求时，客户端也应同时为请求头部加上 Pragma 和 Cache-Control 字段以[兼容服务器不能理解 Cache-Control 的情况](https://tools.ietf.org/html/rfc7234#section-5.4)。
#### 2. Expires
Pragma 字段用于在请求时禁止直接使用缓存，可以说 Expires 字段用于启用缓存。
服务器通过给响应头部的 Expires 字段设置值可以通过规定了缓存过期时间启用缓存，如 Mon, 27 Aug 2018 08:28:13 GMT。
需要注意的是该时间是服务器上设置的时间，若服务器上的时间和客户端时间并不一致，该缓存过期时间便可能没有意义了。
这个问题 HTTP 1.1 通过给 Cache-Control 字段设定 max-age 的相对时间值进行解决。
#### 3. Last-Modified 和 If-Modified-Since
Pragma 字段使得每次请求都不使用缓存而向服务器请求数据以保证数据的准确性，但如果每次服务器收到请求都不管数据有没有改变就直接返回数据就浪费了网络资源了。
使用 Last-Modified 和 If-Modified-Since 可以帮助服务器判断客户端的内容是否已为最新内容。
服务器响应头使用 Last-Modified 字段记录该数据最后更改时间，客户端再次向服务器请求这个数据时，在请求头加上 If-Modified-Since 字段并将字段值设为上次请求服务器响应的 Last-Modified 字段值。
服务器收到请求后便可以通过判断 If-Modified-Since 字段值和数据最后修改时间是否一致，一致的话服务器仅需要返回含 304 状态码响应体为空的响应信息给客户端，客户端便知道这个请求文件没有被修改，使用浏览器缓存即可。
然而 Last-Modified 使用了 GMT 时间最多只能精确到1秒，1秒内发生的变化并不能准备进行数据响应。
此外只要是服务器的数据更新了， Last-Modified 字段值便会变化，有可能会出现文件更新了，但实际上里面的数据内容并没有修改，这种情况服务器也会将文件数据放进响应体进行响应，造成不必要的网络资源浪费及延时。
这些问题 HTTP 1.1 新增了 ETag 和 If-None-Match 字段进行解决。
### HTTP 1.1  的缓存技术
#### 1. Cache-Control
可以说 HTTP 1.1 使用了 Cache-Control 对 HTTP1.0 中的 Pragma 及 Expires 字段功能进行了整合和优化。
如 <code>Cache-Control: no-cache</code> 可以禁止缓存， <code>Cache-Control: max-age=3600</code> 启用缓存并设置了缓存过期时间为 3600 秒之后。
此外，Cache-Control 还可以同时配置多个值实现提供对缓存条件的设置。
如 <code>Cache-Control: max-age=3600, public </code> 不仅规定了缓存将在1小时后过期，还通过 <code>public</code> 规定了可以在代理服务器上进行缓存。

作为[请求头部字段](https://tools.ietf.org/html/rfc7234#section-5.2.1)，Cache-Control的可选值有：

| 字段名称 | 说明 |
| :---- | :---- |
| no-cache | 告知（代理）服务器不直接使用缓存而需向服务器发送请求 |
| no-store | 所有内容都不会被保存在缓存中 |
| max-age = delta-seconds | 告知（代理）服务器客户端希望收到存在时间 （Age) 不大于 delta-seconds 秒的资源 |
| max-stale [ = delta-seconds ] | 告知（代理）服务器客户端愿意收到超过缓存时间的资源，若有定义 delta-seconds 则为 delta-seconds 秒，没有则为超出任何时间 |
| min-fresh = delta-seconds | 告知（代理）服务器客户端希望接收一个在小于 delta-seconds 秒内被更新过的资源 |
| no-transform | 告知（代理）服务器客户端希望收到实体数据没有被转换（比如压缩）过的资源 |
| only-if-cached | 告知（代理）服务器客户端希望获取缓存内容（若有）而不去向服务器发送请求 |
| cache-extension | 自定义扩展值，若服务器不识别该值将被忽略掉 |

作为[响应头部字段](https://tools.ietf.org/html/rfc7234#section-5.2.2)，Cache-Control的可选值有：

| 字段名称 | 说明 |
| :---- | :---- |
| public | 表明任何情况下都得缓存该资源（即使需要 HTTP 认证的资源） |
| private [ = "file-name"] | 表明返回报文中全部或部分（若指定了 file-name 则为 file-name 的字段数据）仅开放给某些用户做缓存使用，其他用户则不能缓存这些数据 |
| no-transform | 告知客户端希望收到的实体数据不能做任何改变 |
| no-cache | 强制任何情况下需先发送请求确认缓存（新鲜度验证）为最新再使用缓存 |
| no-store | 所有内容都不会被保存在缓存中 |
| must-revalidate | 当前资源一定是向原服务器发去验证请求的，若请求失败会返回504（而非代理服务器上的缓存） |
| poxy-revalidate | 同 must-revalidate ，但仅限共享缓存（如代理） |
| max-age = delta-seconds | 告知客户端希望收到的资源在 delta-seconds 秒内是新鲜的，不需向服务器发送请求 |
| s-maxage = delta-seconds | 同 max-age ，但仅限共享缓存（如代理） |
| no-transform | 告知客户端希望收到的实体数据不能做任何改变 |
| cache-extension | 自定义扩展值，若客户端不识别该值将被忽略掉。 |

##### 2.  ETag 和 If-None-Match
ETag 及 If-None-Match 功能和 Last-Modified 及 If-Modified-Since 功能相同，都是服务器判断客户端内容是否已为最新内容的方法。
服务器将 ETag 字段值设置为对请求的数据内容进行 Hash 之后的值，客户端再次向服务器请求这个数据时，在请求头加上 If-None-Match 字段并将其设置为之前收到的 ETag 字段值。
同样的，服务器收到请求后通过判断 If-None-Match 字段值的内容和对最新的数据内容进行 Hash 之后的值是否一样即可知道是否需要返回数据内容还是只是返回 304 状态码响应体为空的响应信息。
需要注意的是，使用 ETag 是根据对数据内容进行 Hash 运算判断是否数据变化，无论数据内容多小，这也会给服务器带来一定的运算负担。
### 总结
HTTP 提供了实现缓存的多种方式，HTTP 1.1 协议中也保留了 HTTP 1.0 中的缓存字段，不仅可以解决兼容性问题，一般服务器也会优先考虑 Last-Modified 及 If-Modified-Since 字段以减少使用 ETag Hash运算对服务器性能的消耗。    
