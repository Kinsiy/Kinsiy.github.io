---
title: Javascript-BOM-2
date: 2021-04-20 10:11:29
categories: [学习笔记, Javascript]
tags: [ BOM]
description:
photos:
---

## location 对象

location 对象是最有用的 BOM 对象之一，提供了当前窗口中加载文档的信息，以及通常的导航功能。这个对象独特的地方在于，它既是 window 的属性，也是 document 的属性。
假设当前加载的 URL 是 http://foouser:barpassword@www.wrox.com:80/WileyCDA/?q=javascript#contents, location 对象的内容如下表所示。

<!-- more -->

| 属性              | 值                                                     | 说明                                                         |
| :---------------- | :----------------------------------------------------- | :----------------------------------------------------------- |
| location.hash     | #contents                                              | URL 散列值(井号后跟零个或多个字符)，如果没有则为空字符串     |
| location.host     | www.wrox.com:80                                        | 服务器名及端口号                                             |
| location.hostname | www.wrox.com                                           | 服务器名                                                     |
| location.href     | http://www.wrox.com:80/WileyCDA/?q=javascript#contents | 当前加载页面的完整 URL。location 的 toString()方法返回这个值 |
| location.pathname | /WileyCDA/                                             | URL 中的路径和(或)文件名                                     |
| location.port     | 80                                                     | 请求的端口。如果 URL 中没有端口，则返回空字符串              |
| location.protocol | http:                                                  | 页面使用的协议。通常是"http:"或"https:"                      |
| location.search   | ?q=javascript                                          | URL 的查询字符串。这个字符串通常以问号开头                   |
| location.username | foouser                                                | 域名前指定的用户名                                           |
| location.password | barpassword                                            | 域名前指定的密码                                             |
| location.origin   | http://www.wrox.com                                    | URL 的源地址。只读                                           |

### 查询字符串

location 的多数信息都可以通过上面的属性获取。但是 URL 中的查询字符串并不容易使用。虽然 location.search 返回了从问号直到 URL 末尾的所有内容，但没有办法逐个访问每个查询参数。

```javascript
let getQueryStringArgs = function () {
  // 取得没有开头问号的查询字符串
  let qs = location.search.length > 0 ? location.search.substring(1) : "";
  // 保存数据的对象
  args = {};

  // 把每个参数添加到args对象
  for (let item of qs.split("&").map((kv) => kv.split("="))) {
    let name = decodeURIComponent(item[0]),
      value = decodeURIComponent(item[1]);
    if (name.length) {
      args[name] = value;
    }
  }

  return args;
};
```

#### URLSearchParams

URLSearchParams 提供一组标准 API 方法，通过它们可以检查和修改查询字符串。给 URLSearchParams 构造函数传入一个查询字符串，就可以创建一个实例。这个实例上暴露了 get()、set()和 delete()等方法，可以对查询字符串执行相应操作。

```javascript
// qs = name=kinsiy&age=24

let qs = location.search;
let searchParams = new URLSearchParams(qs);

console.log(searchParams.has("name")); // true
console.log(searchParams.get("age")); // 24

searchParams.set("lover", "someone");

console.log(searchParams.toString()); // name=kinsiy&age=24&lover=someone
// 能够迭代
for (let param of searchParams) {
  console.log(param); // ["name", "kinsiy"] ["age", "24"] ["lover", "someone"]
}
```

### 操作地址

可以通过 location 对象修改浏览器的地址。

```javascript
// 效果相同
location.assign("https://www.baidu.com");
window.location = "https://www.baidu.com";
location.href = "https://www.baidu.com";
```

修改 location 对象的属性也会修改当前加载的页面。其中，hash、search、hostname、pathname 和 port 属性被设置为新值之后都会修改当前 URL。除了 hash 之外，只要修改 location 的一个属性，就会导致页面重新加载新 URL。

如果不希望增加历史记录，可以使用 replace()方法。这个方法接收一个 URL 参数，但重新加载后不会增加历史记录。调用 replace()之后，用户不能回到前一页。

最后一个修改地址的方法是 reload()，它能重新加载当前显示的页面。调用 reload()而不传参数，页面会以最有效的方式重新加载。

```javascript
location.reload(); // 重新加载，可能是从缓存加载
location.reload(true); // 重新加载，从服务器加载
```

## navigator 对象

navigator 对象的属性通常用于确定浏览器的类型

| 属性/方法                     | 说明                                                             |
| :---------------------------- | :--------------------------------------------------------------- |
| activeVrDisplays              | 返回数组，包含 ispresenting 属性为 true 的 VRDisplay 实例        |
| appCodeName                   | 即使在非 Mozilla 浏览器中也会返回"Mozilla"                       |
| appName                       | 浏览器全名                                                       |
| appVersion                    | 浏览器版本。通常与实际的浏览器版本不一致                         |
| battery                       | 返回暴露 Batter Status API 的 BatteryManager 对象                |
| buildId                       | 浏览器的构建编号                                                 |
| connection                    | 返回暴露 Network Information API 的 NetworkInformation 对象      |
| cookieEnabled                 | 返回布尔值，表示是否启用了 cookie                                |
| credentials                   | 返回暴露 Credentials Management API 的 CredentialsContainer 对象 |
| deviceMemory                  | 返回单位为 GB 的设备内存容量                                     |
| doNotTrack                    | 返回用户"不跟踪"(do-not-track)设置                               |
| geolocation                   | 返回暴露 Geolocation API 的 Geolcation 对象                      |
| getVRDisplays()               | 返回数组，包含可用的每个 VRDisplay 实例                          |
| getUserMedia()                | 返回与可用媒体设备硬件关联的流                                   |
| hardwareConcurrency           | 返回设备的处理器核心数量                                         |
| javaEnabled                   | 返回布尔值，表示浏览器是否启用了 Java                            |
| language                      | 返回浏览器的主语言                                               |
| languages                     | 返回浏览器偏好的语言数组                                         |
| locks                         | 返回暴露 Web Locks API 的 LockManager 对象                       |
| mediaCapabilities             | 返回暴露 Media Capabilities API 的 MediaCapabilities 对象        |
| mediaDevices                  | 返回可用的媒体设备                                               |
| maxTouchPoints                | 返回设备触摸屏支持的最大触点数                                   |
| mimeTypes                     | 返回浏览器中注册的 MIME 类型数组                                 |
| onLine                        | 返回布尔值，表示浏览器是否联网                                   |
| oscpu                         | 返回浏览器运行设备的操作系统和(或)CPU                            |
| permissions                   | 返回暴露 Permissions API 的 Permissions 对象                     |
| platform                      | 返回浏览器运行的操作平台、                                       |
| plugins                       | 返回浏览器安装的插件数组                                         |
| product                       | 返回产品名称(通常是"Geoko")                                      |
| productSub                    | 返回产品的额外信息(通常是 Geoko 的版本)                          |
| registerProtocolHandler()     | 将一个网站注册为特定协议的处理程序                               |
| requestMediaKeySystemAccess() | 返回一个期约，解决为 MediaKeySystemAccess 对象                   |
| sendBeacon()                  | 异步传输一些小数据                                               |
| serviceWorker                 | 返回用来与 ServiceWorker 实例交互的 ServiceWorkerContainer       |
| share()                       | 返回当前平台的原生共享机制                                       |
| storage                       | 返回暴露 Storage API 的 StorageManager 对象                      |
| userAgent                     | 返回浏览器的用户代理字符串                                       |
| vendor                        | 返回浏览器的厂商名称                                             |
| vendorSub                     | 返回浏览器厂商的更多信息                                         |
| vibrate()                     | 触发设备振动                                                     |
| webdriver()                   | 返回浏览器当前是否被自动化程序控制                               |

### 插件检测

plugins 数组的每一项都包含如下属性

-   name: 插件名称
-   description：插件介绍
-   filename: 插件的文件名
-   length: 由当前插件处理的 MIME 类型数量

```javascript
let hasPlugin = function (name) {
  name = name.toLowerCase();
  for (let plugin of window.navigator.plugins) {
    if (plugin.name.toLowerCase().indexOf(name) > -1) {
      return true;
    }
  }
  return false;
};

// 检测QuickTime
console.log(hasPlugin("QuickTime")); // false
```

### 注册处理程序

现代浏览器支持 navigator 上的(在 HTML5 中定义的)registerProtocolHandler()方法。可以借助这个方法将 web 应用程序注册为像桌面应用程序一样的默认应用程序。
要使用 registerProtocolHandler()方法，必须传入 3 个参数：要处理的协议(如"mailto"或"ftp")、处理该协议的 URL，以及应用程序。

```javascript
navigator.registerProtocolHandler("mailto", "http://www.somedailclient.com?cmd=%s", "Some Mail Client");
```

## screen 对象

这个对象中保存的纯粹是客户端能力信息，也就是浏览器窗口外面的客户端显示器的信息，比如像素宽度和像素高度

| 属性        | 说明                                       |
| :---------- | :----------------------------------------- |
| availHeight | 屏幕像素高度减去系统组件高度(只读)         |
| availLeft   | 没有被系统组件占用的屏幕的最左侧像素(只读) |
| availTop    | 没有被系统组件占用的屏幕的最顶端像素(只读) |
| availWidth  | 屏幕像素宽度减去系统组件高度(只读)         |
| colorDepth  | 表示屏幕颜色的位数；多数系统是 32(只读)    |
| height      | 屏幕像素高度                               |
| left        | 当前屏幕左边的像素距离                     |
| pixelDepth  | 屏幕的位深(只读)                           |
| top         | 当前屏幕顶端的像素距离                     |
| width       | 屏幕像素宽度                               |
| orientation | 返回 Screen Orientation API 中屏幕的朝向   |

## history 对象

history 对象表示当前窗口首次使用以来用户的导航历史记录。因为 history 是 window 的属性。所以每个 window 都有自己的 history 对象。出于安全考虑，这个对象不会暴露用户访问过的 URL，但可以通过它在不知道实际 URL 的情况下前进和后退

### 导航

go()方法可以在用户历史记录中沿任何方向导航，可以前进也可以后退。

```javascript
// 后退一页
history.go(-1);
history.back();

// 前进一页
history.go(1);
history.forward();

// 导航到最近的页面，可能后退也可能前进。若无匹配项，则什么也不做
history.go("wrox.com");
```

history 对象还有一个 length 属性，表示历史记录中有多个条目。对于窗口或标签页加载的第一个页面，history.length 等于 1.

```js
if (history.length == 1) {
  // 这是用户窗口的第一个页面
}
```

### 历史状态管理

略

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
