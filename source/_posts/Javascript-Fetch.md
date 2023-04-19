---
title: Javascript - Fetch
date: 2021-12-03 21:04:51
tags: [ Fetch]
categories: [学习笔记, Javascript]
description:
photos:
---

Fetch API 能够执行 XMLHttpRequest 对象的所有任务，但更容易使用，接口也更现代化，能够在Web 工作线程等现代 Web 工具中使用。XMLHttpRequest 可以选择异步，而 Fetch API 则必须是异步

{% note primary %}


文章内容大部分出自 [现代Javascript教程. Fetch.](https://zh.javascript.info/fetch) 可直接跳转过去查看


{% endnote %}


## 基本用法


fetch()方法是暴露在全局作用域中的，包括主页面执行线程、模块和工作线程。调用这个方法， 浏览器就会向给定 URL 发送请求


### 分派请求


```javascript
let response = fetch(url, [options])
```


- {% label primary@url %}——要访问的URL
- {% label primary@options %}——可选参数: method, header 等；

具体参数参考 [MDN. Fetch.](https://developer.mozilla.org/zh-CN/docs/Web/API/fetch) 或 [Request class](https://fetch.spec.whatwg.org/#request-class);  没有{% label primary@options %}，那就是一个简单的GET请求，下载{% label primary@url %}的内容

<!-- more -->


### 读取响应


为了获取response body， 我们需要使用一个其他的方法调用


- response.text() ——读取response，并以文本形式返回response
- response.json() ——将response解析为JSON
- response.formData() ——以{% label primary@FromData %}对象的形式返回response
- response.blob() ——以  [Blob](https://zh.javascript.info/blob) (具有类型的二进制数据)形式返回response
- response.arrayBuffer()  —— 以 [ArrayBuffer](https://zh.javascript.info/arraybuffer-binary-arrays)（低级别的二进制数据）形式返回 response


response.body 是一个ReadableStream对象，它允许你逐块读取body


```javascript
let url = 'https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits';
let response = await fetch(url);


let commits = await response.json(); // 读取 response body，并将其解析为 JSON


alert(commits[0].author.login);
```


{% note warning %}


我们只能选择一种读取 body 的方法。


如果我们已经使用了 {% label primary@response.text() %} 方法来获取 response，那么如果再用{% label primary@response.json() %} 则不会生效，因为 body 内容已经被处理过了


要多次读取包含响应体的同一个 Response 对象，必须在第一次读取前调用 clone()


{% endnote %}


### 处理状态码和请求失败


Fetch API 支持通过 Response 的 status (状态码) 和 statusText (状态文本) 属性检查响应状态。


```JavaScript
fetch('/does-not-exist') 
 .then((response) => { 
   console.log(response.status); // 404 
   console.log(response.statusText); // Not Found 
 }); 
```


{% note warning %}


虽然请求可能失败（如状态码为 500），但都只执行了期约的解决处理函数。事实上，只要服务器返回了响应，fetch()期约都会解决。这个行为是合理的：系统级网络协议已经成功完成消息的一次往返传输。至于真正的“成功”请求，则需要在处理响应时再定义。


{% endnote %}


通常状态码为 200 时就会被认为成功了，其他情况可以被认为未成功。为区分这两种情况，可以在状态码非 200~299 时检查 Response 对象的 ok 属性:


```javascript
fetch('/bar') 
 .then((response) => { 
   console.log(response.status); // 200 
   console.log(response.ok); // true 
 }); 
fetch('/does-not-exist') 
 .then((response) => { 
   console.log(response.status); // 404 
   console.log(response.ok); // false 
 }); 
```


{% note primary %}


可以显式地设置 fetch()在遇到重定向时的行为，不过默认行为是跟随重定向 并返回状态码不是 300~399 的响应。跟随重定向时，响应对象的 redirected 属性会被设置为 true， 而状态码仍然是 200：


{% endnote %}


## Response header


Response header 位于 {% label primary@response.headers %} 中的一个类似于 Map 的 header 对象。它不是真正的 Map，但是它具有类似的方法，我们可以按名称（name）获取各个 header，或迭代它们：


```javascript
let response = await fetch('https://api.github.com/repos/javascripttutorial/en.javascript.info/commits');


// 获取一个 header
alert(response.headers.get('Content-Type')); // application/json; charset=utf-8


// 迭代所有 header
for (let [key, value] of response.headers) {
  alert(`${key} = ${value}`);
}
```


## Request header


要在 {% label primary@fetch %} 中设置 request header，我们可以使用 {% label primary@headers %} 选项。它有一个带有输出 header 的对象


```javascript
let response = fetch(protectedUrl, {
  headers: {
    Authentication: 'secret'
  }
});
```


但有一些我们无法设置的header [Forbidden HTTP headers](https://fetch.spec.whatwg.org/#forbidden-header-name)


## 常见Fetch 请求模式


### 发送JSON


```JavaScript
let payload = JSON.stringify({ 
 foo: 'bar' 
}); 
let jsonHeaders = new Headers({ 
 'Content-Type': 'application/json' 
}); 
fetch('/send-me-json', { 
 method: 'POST', // 发送请求体时必须使用一种 HTTP 方法
 body: payload, 
 headers: jsonHeaders 
}); 
```


### 发送参数


```JavaScript
let payload = 'foo=bar&baz=qux'; 
let paramHeaders = new Headers({ 
 'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8' 
}); 
fetch('/send-me-params', { 
 method: 'POST', // 发送请求体时必须使用一种 HTTP 方法
 body: payload, 
 headers: paramHeaders 
});
```


### 发送文件


```JavaScript
let imageFormData = new FormData(); 
let imageInput = document.querySelector("input[type='file'][multiple]"); 
for (let i = 0; i < imageInput.files.length; ++i) { 
 imageFormData.append('image', imageInput.files[i]); 
}
fetch('/img-upload', { 
 method: 'POST', 
 body: imageFormData 
}); 
```


### 加载Blob


```JavaScript
const imageElement = document.querySelector('img'); 
fetch('my-image.png') 
 .then((response) => response.blob()) 
 .then((blob) => { 
 imageElement.src = URL.createObjectURL(blob); 
 }); 
```


### 跨域请求


从不同的源请求资源，响应要包含 CORS 头部才能保证浏览器收到响应。没有这些头部，跨源请求会失败并抛出错误.头部设置参考 [Javascript-CORS](https://kinsiy.github.io/Javascript-CORS/)


如果代码不需要访问响应，也可以发送 no-cors 请求。此时响应的 type 属性值为 opaque，因此无法读取响应内容。这种方式适合发送探测请求或者将响应缓存起来供以后使用。


```javascript
fetch('//cross-origin.com', { method: 'no-cors' }) 
 .then((response) => console.log(response.type)); 
// opaque
```


### 中断请求


Fetch API 支持通过 AbortController/AbortSignal 对中断请求。调用 AbortController. abort()会中断所有网络传输，特别适合希望停止传输大型负载的情况。中断进行中的 fetch()请求会导致包含错误的拒绝


```JavaScript
let abortController = new AbortController(); 
fetch('wikipedia.zip', { signal: abortController.signal }) 
 .catch(() => console.log('aborted!'); 
// 10 毫秒后中断请求
setTimeout(() => abortController.abort(), 10); 
// 已经中断
```


## 更多用法


更多更详细的用法，像下载进度、断定续传等可以参考 [现代Javascript教程. 网络请求专题](https://zh.javascript.info/network)


## 参考


[1\][Javascript.info. Fetch](https://zh.javascript.info/fetch)

[2\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
