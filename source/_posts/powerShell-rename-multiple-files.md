---
title: PowerShell 批量重命名
date: 2021-11-11 23:17:00
tags: ['rename']
categories: ['windows','技巧']
description:
photos:
---

## 基础知识

```sh
// 声明变量
$h = "Hello" 
$w = "world"
$h + " " + $w

$_ // 指向当前遍历到的值，可以理解为js中 Array.each((e,i,a)=> e ) 的e
```

{% note primary  %}

{% label success@ls -filter *.EXTENSION | ...%}搭配 System.String 提供的方法可以满足所有需求。对字符串使用 Get-Member 可以获取所有方法

{% code %}"Hello world" | Get-Member{% endcode %}

{% endnote %}

<!-- more -->

## 同名带序号

### 语法

```sh
ls | %{Rename-Item $_ -NewName ("NEW-FILE-NAME-{0}.EXTENSION" -f $nr++)}

// 搭配 -filter 使用，只重命名特定后缀文件
ls -filter *.EXTENSION | %{Rename-Item $_ -NewName ("NEW-FILE-NAME-{0}.EXTENSION" -f $nr++)} 
```

### 指令

- NEW-FILE-NAME：新名字
- EXTENSION：文件拓展名

### 效果

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Snipaste_2021-11-12_01-03-07.png)

## 部分剪切

### 语法

```sh
ls | Rename-Item -NewName {$_.name.substring(0,$_.BaseName.length-N) + $_.Extension}
```

### 指令

关键在{% label primary@System.String.substring %}函数，第一个入参代表**提取起点**，第二个入参代表**要提取的长度**。包含起点，下标从0开始

### 效果

这里不知道什么原因多遍历了一次，导致报错

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Snipaste_2021-11-12_01-30-18.png)

## 部分替换

### 语法

```sh
ls | Rename-Item -NewName {$_.name -replace "OLD-FILE-NAME-PART","NEW-FILE-NAME-PART"}
```

### 指令

- OLD-FILE-NAME-PART：被替换的部分
- NEW-FILE-NAME-PART：替换进去的部分，为空就变成了替换删除

### 效果

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Snipaste_2021-11-12_01-34-34.png)

## 参考

[1\][Windows Central. How to batch rename multiple files on Windows 10.](https://www.windowscentral.com/how-rename-multiple-files-bulk-windows-10#rename-files-using-powershell)

[2\][4sysops. Strings in PowerShell - Replace, compare, concatenate, split, substring.](https://4sysops.com/archives/strings-in-powershell-replace-compare-concatenate-split-substring/)

[3\][Microsoft. powershell](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-item?view=powershell-7.2)

