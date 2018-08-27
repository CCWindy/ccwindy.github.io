---
title: HTTP 缓存
date: 2018-08-27
tags: HTTP cache
---
当用户重新访问一个网站时，很多时候请求的文件内容都是相同并未改变的。
浏览器及代理服务器等已实现了本地缓存功能，若服务器对这些不变的文件内容进行再次返回，将对网络资源造成不必要的浪费。
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
服务器响应头使用 Last-Modified 字段记录该文件最后更改时间，客户端再次向服务器请求这个数据时，在请求头加上 If-Modified-Since 字段并将字段值设为上次请求服务器响应的 Last-Modified 字段值。
服务器收到请求后便可以通过判断 If-Modified-Since 字段值和文件最后修改时间是否一致，一致的话服务器仅需要返回含304状态码的响应头部信息给客户端，客户端便知道这个请求文件没有被修改，使用浏览器缓存即可。
然而 Last-Modified 使用了 GMT 时间最多只能精确到1秒，1秒内发生的变化并不能准备进行数据响应。
此外只要是服务器的文件更新了， Last-Modified 字段值便会变化，有可能会出现文件更新了，但文件内容并没有修改，这种情况服务器也会将文件数据放进响应体进行响应，造成不必要的网络资源浪费及延时。
这些问题 HTTP 1.1 新增了 ETag 和 If-None-Match 字段进行解决。
### HTTP 1.1  的缓存技术
#### 1. Cache-Control
可以说 HTTP 1.1 使用了 Cache-Control 对 HTTP1.0 中的 Pragma 及 Expires 字段功能进行了整合和优化。
如 <code>Cache-Control: no-cache</code> 可以禁止缓存， <code>Cache-Control: max-age=3600</code> 启用缓存并设置了缓存过期时间为3600秒之后。
此外，Cache-Control 还可以同时配置多个值实现提供对缓存条件的设置。
如 <code>Cache-Control: max-age=3600, public </code> 不仅规定了缓存将在1小时后过期，还通过 <code>public</code> 规定了可以在代理服务器上进行缓存。
##### 2.  ETag 和 If-None-Match
ETag 及 If-None-Match 功能和 Last-Modified 及 If-Modified-Since 功能相同，都是服务器判断客户端内容是否已为最新内容的方法。
服务器将 ETag 字段值设置为对请求的文件内容进行 Hash 之后的值，客户端再次向服务器请求这个数据时，在请求头加上 If-None-Match 字段并将其设置为之前收到的 ETag 字段值。
同样的，服务器收到请求后通过判断 If-None-Match 字段值的内容和对最新的文件内容进行 Hash 之后的值是否一样即可知道是否需要返回文件内容还是只是返回 304 状态码的响应头部。
需要注意的是，使用 ETag 是根据对文件内容进行 Hash 运算判断是否文件变化，这会给服务器带来一定的运算负担。
### 总结
HTTP 提供了实现缓存的多种方式，HTTP 1.1 协议中也保留了 HTTP 1.0 中的缓存字段，不仅可以解决兼容性问题，一般服务器也会优先考虑 Last-Modified 及 If-Modified-Since 字段以减少使用 ETag Hash运算对服务器性能的消耗。    
