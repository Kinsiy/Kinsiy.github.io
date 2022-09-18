---
title: Javascript-CORS
date: 2021-10-29 19:08:39
tags:
    - JS红宝书
    - CORS
categories: [学习笔记, Javascript]
description:
photos:
---

## 跨域资源共享(CORS)

跨源请求 —— 那些发送到其他域（即使是子域）、协议或端口的请求 —— 需要来自远程端的特殊 header

跨域资源共享(CORS，Cross-Origin Resource Sharing)定义了浏览器与服务器如果实现跨域通信。CORS 背后的基本思路就是使用自定义的 HTTP 头部允许浏览器和服务器相互了解，以确实请求或响应应该成功还是失败

有两种类型的跨域请求：

-   简单的请求
-   所有其他请求

<!-- more -->

### 简单的请求

一个简单的请求是指满足以下两个条件的请求：

-   简单的方法：GET，POST 或 HEAD
-   简单的 header——仅允许自定义下列 header
    -   Accept
    -   Accept-Language
    -   Content-Language
    -   Content-Type 的值为 application/x-www-from-urlencoded, multipart/from-data 或 text/plain

任何其他请求都被认为是"非简单请求"

本质区别在于，可以使用<from\>或<script\>经行"简单请求"，而无需任何其他特殊方法。

实际区别在于，简单请求会使用 {% label primary@Origin %} header 并立即发送，而对于其他请求，浏览器会发出初步的“预检”请求，以请求许可

### 用于简单请求的 CORS

如果一个请求是跨源的，浏览器会始终向其添加 Origin header; Origin 包含了确切的源（domain/protocol/port），没有路径。

Origin: {% label primary@http://www.nczonline.net %}

如果服务器决定响应请求，那么应该发送 Access-Control-Allow-Origin 头部，包含相同的源；或者如果资源是公开的，那么就包含"\*"。比如：

Access-Control-Allow-Origin: {% label primary@http://www.nczonline.net %}

如果没有这个头部，或者有但源不匹配，则表明不会响应浏览器请求。否则，服务器就会处理这个请求。注意，无论请求还是响应都不会包含 cookie 信息。

浏览器在这里扮演被信任的中间人的角色：

-   它确保发送的跨源请求带有正确的 Origin
-   它检查响应中的许可 Access-Control-Allow-Origin， 如果存在，则允许 Javascript 访问响应，否则将失败并报错

![](https://zh.javascript.info/article/fetch-crossorigin/xhr-another-domain.svg)

#### Response header

对于跨域请求，默认情况下，Javascript 只能访问"简单"response header：

-   Cache-control
-   Content-language
-   Content-Type
-   Expires
-   Last-Modified
-   Pargma

访问任何其他 response header 都将导致 error

要授予 Javascript 对任何其他 response header 的访问权限，服务器必须发送{% label primary@Access-Control-Expose-Headers  %} header。它包含一个以逗号分隔的应该被设置为可访问的非简单 header 名称列表

```javascript
200 OK
Content-Type:text/html; charset=UTF-8
Content-Length: 12345
API-Key: 2c9de507f2c54aa1
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Expose-Headers: Content-Length,API-Key
```

### "非简单"请求

我们可以使用任何 HTTP 方法：不仅仅是 {% label primary@GET/POST %}，也可以是 {% label primary@PATCH %}，{% label primary@DELETE %} 及其他

之前，没有人能够设想网页能发出这样的请求。因此，可能仍然存在有些 Web 服务将非标准方法视为一个信号：“这不是浏览器”。它们可以在检查访问权限时将其考虑在内。

因此，为了避免误解，任何“非标准”请求 —— 浏览器不会立即发出在过去无法完成的这类请求。即在它发送这类请求前，会先发送“预检（preflight）”请求来请求许可。

预检请求使用 {% label primary@OPTIONS %} 方法，它没有 body，但是有两个 header：

-   Access-Control-Request-Method： header 带有简单请求的方法
-   Access-Control-Request-Headers:：header 提供一个以逗号分隔的非简单 HTTP header 列表

如果服务器同意处理请求，那么他会进行响应，此响应的状态码应该为 200，没有 body，具有 header：

-   Access-Control-Allow-Origin: 必须为经行请求的源或 " \* "。才能允许此请求
-   Access-Control-Allow-Methods: 必须具有允许的方法
-   Access-Control-Allow-Headers: 必须具有一个允许的 header 列表
-   另外，header {% label primary@Access-Control-Max-Age %} 可以指定缓存此权限的秒数。因此，浏览器不是必须为满足给定权限的后续请求发送预检

![](https://zh.javascript.info/article/fetch-crossorigin/xhr-preflight.svg)

> **注意：**
>
> **预检请求发生在“幕后”，它对 JavaScript 不可见。**
>
> **JavaScript 仅获取对主请求的响应，如果没有服务器许可，则获得一个 error。**

## 代替性跨源技术

### 图片探测/使用表单

使用表单，提交一个<from\>到<iframe\>以停留在当前页面

```HTML
<!-- 表单目标 -->
<iframe name="iframe"></iframe>


<!-- 表单可以由 JavaScript 动态生成并提交 -->
<form target="iframe" method="POST" action="http://another.com/…">
  ...
</form>


```

图片探测是与服务器之间简单、跨域、单向的通信

```js
let img = new Image();
img.onload = img.onerror = function () {
  console.log("Done!");
};
img.src = "Http://www.example.com/test?name=Nicholas";
```

这两种方式都无法读取响应

### JSONP

使用 <script\> 标签。<script\> 可以具有任何域的 src，例如 <script src="{% label primary@http://another.com/ %}…"\>。也可以执行来自任何网站的 <script\>。

如果一个网站，例如 another.com 试图公开这种访问方式的数据，则会使用所谓的 “JSONP (JSON with padding)” 协议。

假设在我们的网站，需要以这种方式从 {% label primary@http://another.com %} 网站获取数据，例如天气：

1. 首先，我们先声明一个全局函数来接收数据，例如 gotWeather
    ```JavaScript
       // 1. 声明处理天气数据的函数
       function gotWeather({ temperature, humidity }) {
         alert(`temperature: ${temperature}, humidity: ${humidity}`);
       }
    ```
2. 然后我们创建一个特性（attribute）为 src="{% label primary@http://another.com/weather.json?callback=gotWeather %}" 的 <script\> 标签，使用我们的函数名作为它的 callback URL-参数。

    ```JavaScript
       let script = document.createElement('script');
       script.src = `http://another.com/weather.json?callback=gotWeather`;
       document.body.append(script);
    ```

3. 远程服务器 another.com 动态生成一个脚本，该脚本调用 gotWeather(...)，发送它想让我们接收的数据。

    ```JavaScript
    // 我们期望来自服务器的回答看起来像这样：
    gotWeather({
    temperature: 25,
    humidity: 78
    });

    ```

4. 当远程脚本加载并执行时，gotWeather`函数将运行，并且因为它是我们的函数，我们就有了需要的数据

JSONP 由于其简单易用，在开发者中非常流行。相比于图片探测，使用 JSONP 可以直接访问响应， 实现浏览器与服务器的双向通信。不过 JSONP 也有一些缺点

首先 JSONP 是从不同的域拉取可执行代码。如果这个域并不可信，则可能在响应中加入恶意内容。此时除了完全删除 JSONP 没有其他办法。在使用不受控的 Web 服务时，一定要保证是可以信任的。

第二个缺点是不好确定 JSONP 请求是否失败。虽然 HTML5 规定了<script\>元素的 onerror 事件处理程序，但还没有被任何浏览器实现。为此，开发者经常使用计时器来决定是否放弃等待响应。这种方式并不准确，毕竟不同用户的网络连接速度和带宽是不一样的

## 参考

[1\][Javascript.info. Fetch: 跨源请求.](https://zh.javascript.info/fetch-crossorigin )

