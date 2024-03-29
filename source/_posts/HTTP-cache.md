---
title: HTTP-Cache
date: 2021-11-03 22:26:31
tags: ['HTTP-Header']
categories: ['学习笔记',"HTTP"]
description:
photos:
---

## HTTP 缓存

> HTTP 缓存存储与请求关联的响应，并将存储的响应复用于后续请求。
>
> 可复用性有几个优点。首先，由于不需要将请求传递到源服务器，因此客户端和缓存越近，响应速度就越快。最典型的例子是浏览器本身为浏览器请求存储缓存。
>
> 此外，当响应可复用时，源服务器不需要处理请求——因为它不需要解析和路由请求、根据 cookie 恢复会话、查询数据库以获取结果或渲染模板引擎。这减少了服务器上的负载。
>
> 缓存的正确操作对系统的稳定运行至关重要。
>
> ——MDN - HTTP缓存

<!-- more -->

### 缓存类型

-  首次请求[{% label success@Client 1 %}]：浏览器首次请求资源时，由于浏览器缓存中没有对应的缓存，此时需要去服务器请求，待返回数据后将其存储在缓存数据库中。当浏览器存在对应缓存数据后，下次请求可以根据需要决定是否向服务器发起请求

- 强缓存[{% label success@Client 2 %}]：用户请求数据，如果命中强缓存，则不向服务器请求，而直接从本地资源获取，返回200状态码，并提示from disk cache或from memory cache（比从disk快）

- 协商缓存[{% label success@Client 3 %}]：在用户请求资源时，浏览器直接向服务器发送请求，协商对比服务端和本地的资源，验证本地资源是否失效

![HTTPStaleness](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/HTTPStaleness.png)

强制缓存和协商缓存命中缓存资源后，都是从本地读取资源。如果强制缓存生效，则不需要再向服务器发出请求；而协商缓存，不管是否使用缓存，必须向服务器发送一个请求来协商。

两类缓存规则可以同时存在，强制缓存优先级高于协商缓存，也就是说，当执行强制缓存的规则时，如果缓存生效，直接使用缓存，不再执行协商缓存规则。如果强制缓存规则不生效，则需要进行协商缓存判断

### 缓存相关Header

#### 强缓存

强制缓存的response header中会有两个字段来表明失效规则（Expires/Cache-Control）

- Cache-Control：Cache-Control 是最重要的规则。常见的取值有private、public、no-cache、max-age [更多指令](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)
  - no-store：所有内容都不会缓存，强缓存，协商缓存都不会触发
  - no-cache： 需要使用协商缓存来验证缓存数据
  - private：客户端可以缓存
  - public：客户端和代理服务器都可以缓存
  - max-age = xxx： 缓存内容将在xxx秒后失效 
-  Expires：Expires的值为服务端返回的到期时间，即下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据。
- Pragma：一个HTTP1.0中规定的通用首部，如果{% label primary@Cache-Control %}不存在的话，它的行为与{% label primary@Cache-Control: no-cache %}一致

在HTTP1.0的环境下，Cache-Control不起作用，Expires起作用； 在HTTP1.1的环境之下， Expires不起作用，而Cache-Control起作用。当前一般都是http1.1的情况，所以Expires是作为一种向下兼容的形式而存在的。

{% note warning %}

Expires 是HTTP 1.0的东西，现在默认浏览器均默认使用HTTP 1.1，所以它的作用基本忽略。

Expires 到期时间是由服务端生成的，但是客户端时间可能跟服务端时间有误差，这会导致缓存命中的误差。 

Pragma 的值就只有一个，no-cache，并且它的优先级比 Cache-Control 高

{% endnote %}

#### 协商缓存

##### Last-Modified / If-Modified-Since

- Last-Modified：服务器响应请求时，告诉浏览器资源最后的修改时间。

- If-Modified-Since：浏览器再次请求资源时，浏览器通知服务器，上次请求时，返回的资源最后修改时间。

若最后修改时间小于等于If-Modified-Since，则response header返回304，告知浏览器继续使用所保存的cache。若大于If-Modified-Since，则说明资源被改动过，返回状态码200；

#####  **If-none-match / Etag**

- Etag：服务器响应请求时，告诉浏览器当前资源在浏览器的唯一标识（生成规则由服务器确定）

- If-None-Match：再次请求服务器时，通过此字段通知服务器客户端缓存数据的唯一标识。

  服务器收到请求后发现有If-None-Match 则与被请求资源的唯一标识进行比对：
  - 不同，说明资源又被改动过，则响应整片资源内容，返回状态码200；
  - 相同，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。

本地缓存时间到期后，浏览器向服务端发送请求报文，其中Request Header中包含If-none-match和Last-Modified-Since（与服务端Etag和Last-Modified对比，Etag优先级高），用以验证本地缓存数据验证是否与服务端保持一致。在服务器端会优先判断Etag。如果相同，返回304；如果不同，就继续比较Last-Modified，然后决定是否返回新的资源。若服务端验证本地缓存与服务端一致，返回304，浏览器加载本地缓存；否则，服务器返回请求的资源，同时给出新的Etag以及Last-Modified时间。

{% note warning %}

在精确度上，Etag优于Last-Modified。Last-Modified精确到s，如果1s内，资源多次改变，Etag是可以判断出来并返回最新的资源。

在性能上，Last-Modified优于Etag，因为Last-Modified只需要记录时间，而Etag需要服务器重新生成hash值，所以性能上略差。

在优先级上，Etag优于Last-Modified，Etag和Last-Modified可同时存在。

{% endnote %}

### 流程图

![](https://pic1.zhimg.com/80/v2-b52f314c86e3e95459845589d4d35570_720w.jpg)

### 用户行为对缓存的影响

| 用户操作        | Expires/Cache-Control | Last-Modied/Etag |
| --------------- | --------------------- | ---------------- |
| 地址栏回车      | √                     | √                |
| 页面链接跳转    | √                     | √                |
| 新开窗口        | √                     | √                |
| 前进回退        | √                     | √                |
| F5刷新          | ×                     | √                |
| Ctrl+F5强制刷新 | ×                     | ×                |

### 缓存更新

通过更新页面中引用的资源路径，让浏览器主动放弃缓存，加载新资源。

#### 用数字做版本号

![](https://pic1.zhimg.com/80/8a8676e933478d1a73777d84a5de55f5_720w.jpg?source=1940ef5c)

但是如果使用版本号的话，假设某次更新只更改了一个文件，其他未更改的文件为了保持版本一致都需要更新，会导致原有缓存失效，造成无效更新

#### 用文件摘要做版本号

采用**数据摘要要算法**对文件求摘要值，拼接到路径参数上，更新某一文件时，只需更新对应文件的路径即可。

![](https://pic3.zhimg.com/80/5276595f41d6276e21e5bc1d25741680_720w.jpg?source=1940ef5c)

但这样会导致无论**先部署页面**还是**先部署静态资源**都会导致部署过程中发生页面错乱的问题

**先部署页面**，在两者部署间隔中，假如有用户访问新页面，此时**页面中的资源路径已经更新，但资源仍未更新**。客户端看到这个新的路径，就会发起请求，然而依然会拿到旧资源并作为缓存，并且，即便后面资源更新了，在已拿到资源缓存过期前，一直都不会重新请求，**一直都会是旧资源**

**先部署资源**，在两者部署间隔中，假如有用户访问，此时 html 中资源路径仍未更新，资源已经更新。这时假如本地缓存未过期，则不会请求，一切安好。假如这时本地缓存过期了，会去协商缓存，就会出现**旧 html 引用新资源的情况，可能会导致页面出错**。

#### 更好的做法

用文件的摘要信息来对资源文件进行重命名，把摘要信息放到资源文件发布路径中，这样，内容有修改的资源就变成了一个新的文件发布到线上，不会覆盖已有的资源文件。上线过程中，先全量部署静态资源，再灰度部署页面

![](https://pic4.zhimg.com/80/9b3a9df114d14a14130a70abf5733837_720w.jpg?source=1940ef5c)



### 参考

\[1][MDN. HTTP 缓存. ](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching)

\[2][华为云开发者社区. 深入理解浏览器的缓存机制. 博客园](https://www.cnblogs.com/huaweiyun/p/13390041.html)

\[3][字节前端. 浅析HTTP缓存. 知乎专栏](https://zhuanlan.zhihu.com/p/360512798)

\[4][张云龙. 大公司里怎样开发和部署前端代码？问题下回答. 知乎](https://www.zhihu.com/question/20790576/answer/32602154)

