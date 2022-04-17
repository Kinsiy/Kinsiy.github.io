---
title: 一个基于Node.js的简易局域网图床
mathjax: false
date: 2022-04-16 21:47:48
tags:
categories: ['nodeJs']
description:
photos:
---

最近有需要向别人分享markdown文档的需求,然而文档上面插入了一些本地图片,这些图片又不好上传到外网图床上.导致在非本机环境中打开文档,均无法查看图片.

既然无法使用外网图床,在本机搭建一个局域网图床不就行了? 在github上面找了一圈只找到了一个Python+Flask的实现.看了一下它的代码,非常简单,核心就是一个web服务.[Github. My-Easy-Pic-Bed](https://github.com/fslongjin/My-Easy-Pic-Bed)

我寻思着这也没必要上Flask,只要能起web服务的就行,而且我并不需要网页上传入口,通过`curl`命令行上传图片就行,这下`html`页面也省了

<!--more-->

## 代码实现

利用`http.createServer()`创建web服务,我们只需要处理图片的上传(`post`)和下载(`get`)就行.

{% note info %}

环境: node -v12.21.0

代码已上传至GitHub，clone下来`npm run start`即可食用[Github. Kinsiy.Simple-pic-bed](https://github.com/Kinsiy/simple-pic-bed/tree/main)

{% endnote %}

### Post

文件上传核心就一件事,将文件以唯一文件名保存到指定文件夹.代码也很简单,使用`formidable`库获取请求体中的文件,使用`uuid`库生成唯一文件名,保存文件,并返回路径即可.

```javascript
if (req.url === "/upload" && req.method.toLowerCase() === "post") {
  // parse a file upload
  const form = formidable({});

  form.parse(req, (err, fields, files) => {
    if (err) {
      res.statusCode = err.httpCode || 400;
      res.end(String(err));
      return;
    }

    const file = files.file[0];

    // checks if the file is valid
    const isValid = isFileValid(file);

    // rename file be unique
    const fileName = file.originalFilename.replace(/^.*(?=\.)/, uuidv4());

    if (!isValid) {
      // throes error if file isn't valid
      res.statusCode = 400;
      res.end("The file type is not a valid type");
      return;
    }

    let httpCode, result;
    try {
      // store file
      fs.copyFile(file.filepath, path.join(uploadFolder, fileName), function (err) {
        if (err) throw new Error(err);
      });
      httpCode = 200;
      result = `http://${hostname}:${port}/upload/${fileName}`;
    } catch (error) {
      httpCode = 500;
      result = "Image save failed! " + error.message;
    }
    res.statusCode = httpCode;
    res.end(result);
    return;
  });
}
```

### Get

获取图片,也很简单.根据文件名去上传路径找到改文件,正确返回即可

```Javascript
if (/\/upload\/[^\/]+/.test(req.url) && req.method.toLowerCase() === "get") {
  const resFileName = req.url.split("/").pop();
  let imagePath = path.join(uploadFolder, resFileName);

  if (fs.existsSync(imagePath)) {
    // Content-type is very interesting part that guarantee that
    // Web browser will handle response in an appropriate manner.
    res.writeHead(200, {
      "Content-Type": mime.lookup(imagePath),
      "Content-Disposition": "inline;",
    });
    fs.createReadStream(imagePath).pipe(res);
    return;
  }
  res.statusCode = 404;
  res.end("ERROR File does not exist");
}
```

### 示例

通过`curl`上传.`curl`用法自行查阅

![image-20220417011951482](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220417011951482.png)

终端无法查看图片，在浏览器中打开/Markdown中插入即可

{% note  info %}

贴几个`curl`的参考文档

- [阮一峰的网络日志. curl的用法指南](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)
- [catonamat.net. Curl CookBook](https://catonmat.net/cookbooks/curl)
- [curl.se](https://curl.se/)

{% endnote %}

## Typora自动上传

图床搭建好了，比较大的缺点就是一次只能上传一张图片并且需要手动复制返回的连接。Typora提供了自动上传的功能，我们只需要编写一个小脚本来自动上传即可. 脚本没考虑出错的情况，局域网内一般也不会有啥错，需要错误处理的可以自己改下

![image-20220417124706484](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220417124706484.png)

### upload-image.sh

`"C:/Program Files/Git/bin/bash.exe" "D:\Blog\picBed\upload-image.sh"`

```sh
result=()

function upload() {
# 上传文件
    t=$(curl -X POST -s -F "file=@$1"  http://192.168.123.54:7070/upload)
    result=(${result[@]} "${t}")
}
while [ $# -gt 0 ]; do
    upload "$1" $index
    shift
done
# 输出格式
echo "Upload Success:"
for i in ${result[*]}; do
echo $i;
done
```

### upload-image.ps1

`"C:\Program Files\PowerShell\7\pwsh.exe" "D:\Blog\picBed\upload-image.ps1"`

```powershell
$ipv4 = (Get-NetIPAddress | Where-Object { $_.AddressState -eq "Preferred" -and $_.ValidLifetime -lt "24:00:00" }).IPAddress
$Port = '7070'

function Publish-Image {
  param (
    $ImagePath
  )
  Write-Output (curl.exe -X POST -s -F ('"file=@' + $ImagePath + '"') ('http://' + $ipv4 + ':' + $Port + '/upload'))
}

Write-Output 'Upload Success:'
for ($i = 0; $i -lt $args.Count; $i++) {
  Publish-Image $args[$i]
}
```

### 验证

![image-20220417124921514](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220417124921514.png)



## 参考

[1\][CSDN.Net Typora 配置自定义图床 upload-image.sh](https://blog.csdn.net/bklydxz/article/details/116562345)

