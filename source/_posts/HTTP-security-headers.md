---
title: HTTP-Security-Headers
date: 2021-11-04 22:35:08
tags: ['HTTP-Header']
categories: ['学习笔记',"HTTP"]
description:
photos:
---

## Content Security Policy (CSP)

> 内容安全策略  ([CSP](https://developer.mozilla.org/zh-CN/docs/Glossary/CSP)) 是一个额外的安全层，用于检测并削弱某些特定类型的攻击，包括跨站脚本 ([XSS (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting)) 和数据注入攻击等。无论是数据盗取、网站内容污染还是散发恶意软件，这些攻击都是主要的手段。
>
> CSP 被设计成完全向后兼容（除CSP2 在向后兼容有明确提及的不一致; 更多细节查看[这里](https://www.w3.org/TR/CSP2)）。不支持CSP的浏览器也能与实现了CSP的服务器正常合作，反之亦然：不支持 CSP 的浏览器只会忽略它，如常运行，默认为网页内容使用标准的同源策略。如果网站不提供 CSP 头部，浏览器也使用标准的[同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。
>
> **为使CSP可用, 你需要配置你的网络服务器返回  [{% label primary@Content-Security-Policy %}](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy) HTTP头部 **
>
> ( 有时你会看到一些关于{% label primary@X-Content-Security-Policy %}头部的提法, 那是旧版本，你无须再如此指定它)。
>
> 除此之外, **[`<meta>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta) 元素也可以被用来配置该策略** 
>
> ——MDN. 内容安全策略( CSP ). 

<!--more-->

### 语法及指令 

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
```

- 一个策略由一系列策略指令组成，每个策略指令都描述了一个针对某个特定类型资源以及生效范围的策略。
- default-src 是 CSP 指令，多个指令之间使用英文分号分割。
- self 是指令值，多个指令值用英文空格分割

具体语法及指令及其意义可以参考这里 [MDN. Content-Security-Policy. ](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy)

更多详细配置方案可以参考这里 [web.dev Security headers quick reference](https://web.dev/security-headers/)

### strict CSP

可以使用以下两种方式启用严格CSP

- 如果HTML在服务端渲染，选用 ** nonce-based strict CSP**
- 如果HTML必须静态提供或缓存，例如单页面富应用，选用 ** hash-based strict CSP**

#### nonce-based

{% note warning %}

nonce 是一个只使用一次的随机数。nonce-based strict CSP 只在能为每次响应生成不同的nonce的情况下是安全的。如果做不到选用hash-based strict CSP

{% endnote %}

服务器配置

```http
Content-Security-Policy:
  script-src 'nonce-{RANDOM1}' 'strict-dynamic' https: 'unsafe-inline';
  object-src 'none';
  base-uri 'none';
```

HTML文件中，需要为所有<script\>标签添加相同值 `{RANDOM1}` 字符串的nonce属性

```html
<script nonce="{RANDOM1}" src="https://example.com/script1.js"></script>
<script nonce="{RANDOM1}">
  // Inline scripts can be used with the {% label primary@nonce %} attribute.
</script>
```

#### hash-based

服务器配置

```http
Content-Security-Policy:
  script-src 'sha256-{HASH1}' 'sha256-{HASH2}' 'strict-dynamic' https: 'unsafe-inline';
  object-src 'none';
  base-uri 'none';
```

HTML文件中，需要把所有的js文件写在<script\>标签中(inline)，因为大多数浏览器都不支持对外部引用脚本进行哈希

```html
<script>
...// your script1, inlined
</script>
<script>
...// your script2, inlined
</script>
```

可以动态引用

```html
<script>
var scripts = [ 'https://example.org/foo.js', 'https://example.org/bar.js'];
scripts.forEach(function(scriptUrl) {
  var s = document.createElement('script');
  s.src = scriptUrl;
  s.async = false; // to preserve execution order
  document.head.appendChild(s);
});
</script>
```

## HTTP Public Key Pinning (HPKP)

> **Public-Key-Pins** 是一个响应首部，其包含该Web 服务器用来进行加密的 public [key (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/Key) （公钥）信息 ，以此来降低使用伪造证书进行 [MITM](https://developer.mozilla.org/zh-CN/docs/Glossary/MitM) （中间人攻击）的风险。如果锚定的加密串与服务器返回的公钥不匹配，那么浏览器将会认定响应不合法，并且不会将结果展示给用户 
>
> ——MDN. Public-Key-Pins.

### 语法

```http
Public-Key-Pins: pin-sha256="<pin-value>";
                 max-age=<expire-time>;
                 includeSubDomains;
                 report-uri="<uri>"
```

### 指令

- pin-sha256：即证书指纹，允许出现多次，实际上应用最少指定两个
- max-age：过期时间
- includeSubdomains：是否包含子域
- report-uri：验证失败时上报的地址

## HTTP Strict Transport Security(HSTS)

> {% label primary@HTTP Strict Transport Security %}（通常简称为 [HSTS](https://developer.mozilla.org/zh-CN/docs/Glossary/HSTS)）是一个安全功能，它告诉浏览器只能通过HTTPS访问当前资源，而不是 [HTTP](https://developer.mozilla.org/en/HTTP)。
>
> ——MDN. HTTP Strict Transport Security.

### 语法

```http
Strict-Transport-Security: max-age=<expire-time>
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; preload
```

### 指令

- max-age=<expire-time\>

  设置在浏览器收到这个请求后的<expire-time>秒的时间内凡是访问这个域名下的请求都使用HTTPS请求。

- {% label primary@includeSubDomains %} 可选

  如果这个可选的参数被指定，那么说明此规则也适用于该网站的所有子域名。

- {% label primary@preload %} 可选

  查看 [预加载 HSTS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Strict-Transport-Security#预加载_hsts) 获得详情。不是标准的一部分。

### 预加载 HSTS

谷歌维护着一个 [HSTS 预加载服务](https://hstspreload.appspot.com/)。按照如下指示成功提交你的域名后，浏览器将会永不使用非安全的方式连接到你的域名。虽然该服务是由谷歌提供的，但所有浏览器都有使用这份列表的意向（或者已经在用了）。但是，这不是 HSTS 标准的一部分，也不该被当作正式的内容。

- Chrome & Chromium 的 HSTS 预加载列表： https://www.chromium.org/hsts
- Firefox 的 HSTS 预加载列表：[nsSTSPreloadList.inc](https://hg.mozilla.org/mozilla-central/raw-file/tip/security/manager/ssl/nsSTSPreloadList.inc)

## Referrer-Policy

> **{% label primary@Referrer-Policy %}** 首部用来监管哪些访问来源信息——会在 [{% label primary@Referer %}](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referer) 中发送——应该被包含在生成的请求当中
>
> ——MDN. Referrer-Policy.

### 语法

```http
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```

### 指令

- no-referrer：不允许被记录。
- origin：只记录 origin，即域名。
- strict-origin：只有在 HTTPS -> HTTPS 之间才会被记录下来。
- strict-origin-when-cross-origin：同源请求会发送完整的 URL；HTTPS->HTTPS，发送源；降级下不发送此首部。
- no-referrer-when-downgrade(default)：同 strict-origin。
- origin-when-cross-origin：对于同源的请求，会发送完整的 URL 作为引用地址，但是对于非同源请求仅发送文件的源。
- same-origin：对于同源请求会发送完整 URL，非同源请求则不发送 referer。
- unsafe-url：无论是同源请求还是非同源请求，都发送完整的 URL（移除参数信息之后）作为引用地址。（可能会泄漏敏感信息）。

### 集成至HTML

你也可以在 HTML 内设置 referrer 策略。例如，你可以用一个 name 为 referrer 的 [`<meta>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta) 元素为整个文档设置 referrer 策略。

```html
<meta name="referrer" content="origin">
```

或者用<a\>、<area\>、<img\>、<iframe\>、<script\>或者<link\>元素上的 {% label primary@referrerpolicy %} 属性为其设置独立的请求策略。

```
<a href="http://example.com" referrerpolicy="origin">
```

另外也可以在<a\>、<area\>、<link\>元素上将 {% label primary@rel %} 属性设置为 {% label primary@noreferrer %}。

```
<a href="http://example.com" rel="noreferrer">
```

## X-Content-Type-Options

> X-Content-Type-Options HTTP 消息头相当于一个提示标志，被服务器用来提示客户端一定要遵循在 Content-Type 首部中对  MIME 类型 的设定，而不能对其进行修改。这就禁用了客户端的 MIME 类型嗅探行为，换句话说，也就是意味着网站管理员确定自己的设置没有问题
>
> ——MDN. X-Content-Type-Options.

### 语法

```http
X-Content-Type-Options: nosniff
```

### 指令

{% label primary@nosniff %}

下面两种情况的请求将被阻止：

- 请求类型是{% label primary@style %} 但是 MIME 类型不是 {% label primary@text/css %}
- 请求类型是{% label primary@script %} 但是 MIME 类型不是 [JavaScript MIME 类型](https://html.spec.whatwg.org/multipage/scripting.html#javascript-mime-type)。

## X-Frame-Options

> The X-Frame-Options HTTP 响应头是用来给浏览器 指示允许一个页面 可否在 <frame\>, <iframe\>, <embed\> 或者 <object\> 中展现的标记。站点可以通过确保网站没有被嵌入到别人的站点里面，从而避免 clickjacking 攻击
>
> The added security is only provided if the user accessing the document is using a browser supporting X-Frame-Options. 
>
> {% label primary@Content-Security-Policy  %}HTTP 头中的 frame-ancestors 指令会替代这个非标准的 header。CSP 的 frame-ancestors 会在 Gecko 4.0 中支持，但是并不会被所有浏览器支持。然而 X-Frame-Options 是个已广泛支持的非官方标准，可以和 CSP 结合使用
>
> ——MDN. X-Frame-Options.

### 语法

```http
X-Frame-Options: deny
X-Frame-Options: sameorigin
X-Frame-Options: allow-from https://example.com/
```

### 指令

- **deny：** 表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。
- **sameorigin：**表示该页面可以在相同域名页面的 frame 中展示。
- **allow-from *uri* ：**表示该页面可以在指定来源的 frame 中展示。

<% note warning %>

设置 meta 标签是无效的！例如 `<meta http-equiv="X-Frame-Options" content="deny">` 没有任何效果。

[配置方法 MDN. X-Frame-Options. ](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/X-Frame-Options)

<% endnote %>

## X-XSS-Protection

> HTTP **{% label primary@X-XSS-Protection %}** 响应头是 Internet Explorer，Chrome 和 Safari 的一个特性，当检测到跨站脚本攻击 ([XSS (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting))时，浏览器将停止加载页面。若网站设置了良好的 [{% label primary@Content-Security-Policy %}](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy) 来禁用内联 JavaScript ({% label primary@'unsafe-inline' %})，现代浏览器不太需要这些保护， 但其仍然可以为尚不支持 [CSP](https://developer.mozilla.org/zh-CN/docs/Glossary/CSP) 的旧版浏览器的用户提供保护
>
> ——MDN. X-XSS-Protection.

### 语法

```http
X-XSS-Protection: 0
X-XSS-Protection: 1
X-XSS-Protection: 1; mode=block
X-XSS-Protection: 1; report=<reporting-uri>
```

### 指令

- 0： 禁止XSS过滤
- 1： 启用XSS过滤（通常浏览器是默认的）。 如果检测到跨站脚本攻击，浏览器将清除页面（删除不安全的部分）。
- 1;mode=block：启用XSS过滤。 如果检测到攻击，浏览器将不会清除页面，而是阻止页面加载
- 1; report=<reporting-URI\> (Chromium only)：启用XSS过滤。 如果检测到跨站脚本攻击，浏览器将清除页面并使用CSP [report-uri (en-US)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)指令的功能发送违规报告

## 参考

[1\][web.dev. Security headers quick reference](https://web.dev/security-headers/)

[2\][MDN. HTTP-Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)
