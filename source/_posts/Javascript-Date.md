---
title: Javascript-Date
date: 2021-01-13 00:48:35
description: 基本引用类型, Date 对象的使用
categories: [学习笔记, Javascript]
tags: [JS红宝书, Date]
---

引用值(或者对象)是某个特定引用对象的实例。

<!-- more -->

## Date

要创建日期对象，就使用 new 操作符来调用 Date 构造函数：

```javascript
// 在不给Date()传参数的时候，创建的对象将保存当前的日期和时间。
let now = new Date();

// 要基于其他日期和时间建立日期对象，必须传入其毫秒表示(UNIX纪元1970年1月1日午夜之后的毫秒数)

/* Date.parse() */
// 月/日/年

let someDate = new Date(Date.parse("3/18/1998"));
let otherDate = new Date("3/18/1998"); // 等价

/* Date.UTC() */
// *年-*月(0-11)-日(1-31)-时(0-23)-分-秒-毫秒
// GMT 时间 2021年1月13日1时06分55秒
let nowDate_1 = new Date(Date.UTC(2021, 0, 13, 1, 05, 55)); // 无引号
let nowDate_2 = new Date(2021, 0, 13, 1, 05, 55); // 等价

/* Date.now() */
// 返回表示方法执行时日期和时间的毫秒数
let start = Date.now();
let stop = Date.now();
let result = stop - start;
console.log(result); // 0
```

### toLocaleString()-toString()-valueof()-...

```javascript
/* Date类型的
toLocaleString()方法 返回与浏览器运行的本地环境一致的时间和日期。这通常意味着格式中包含针对时间的AM或PM，但不包含时区信息(具体格式  可能因浏览器而不同)
toString()方法 返回带时区信息的日期和时间，而时间也是24小时制表示的
valueOf()方法 不返回字符串，这个方法被重写后返回的是日期的毫秒表示
toDateString() 显示日期中的周几、月、日、年(格式特定于实现)
toTimeString() 显示日期中的时、分、秒和时区(格式特定与实现)
toLocaleDateString() 显示日期中的周几、月、日、年(格式特定于实现和地区)
toLocaleTimeString() 显示日期中的时、分、秒和时区(格式特定与实现)
toUTCString() 显示完整UTC日期(格式特定于实现)
*/

let now = new Date();
console.log(now.toLocaleString()); // 2021/1/13 上午1:25:05
console.log(now.toString()); // Wed Jan 13 2021 01:29:11 GMT+0800 (中国标准时间)
console.log(now.valueOf()); // 1610472551838
console.log(now.toDateString()); // Wed Jan 13 2021
console.log(now.toTimeString()); // 01:39:22 GMT+0800 (中国标准时间)
console.log(now.toLocaleDateString()); // 2021/1/13
console.log(now.toLocaleTimeString()); // 上午1:39:22
console.log(now.toUTCString()); // 1610472Tue, 12 Jan 2021 17:39:22 GMT551838
```

### 日期/时间组件方法

| 函数                           | 说明                                                   |
| :----------------------------- | :----------------------------------------------------- |
| getTime()                      | 返回日期的毫秒数；与 valueOf()相同                     |
| setTime(milliseconds)          | 设置日期的毫秒数，从而修改整个日期                     |
| getFullYear()                  | 返回四位年数(即 2021 而不是 21)                        |
| getUTCFullYear()               | 返回 UTC 日期的四位年数                                |
| setFullYear(year)              | 设置日期的年(year 必须是 4 位数)                       |
| setUTCFullYear(year)           | 设置 UTC 日期的年(year 必须是四位数)                   |
| getMonth()                     | 返回日期的月(0 表示 1 月，11 表示 12 月)               |
| getUTCMonth()                  | 返回 UTC 日期的月(0 表示 1 月，11 表示 12 月)          |
| setMonth(month)                | 设置日期的月(month 为大于 0 的数值，大于 11 加年)      |
| setUTCMonth(month)             | 设置 UTC 日期的月(month 为大于 0 的数值，大于 11 加年) |
| getDate()                      | 返回日期中的日(1~31)                                   |
| getUTCDate()                   | 返回 UTC 日期中的日(1~31)                              |
| setDate(date)                  | 设置日期的日(如果 date 大于该月天数，则加月)           |
| setUTCDate(date)               | 设置 UTC 日期的日(如果 date 大于该月天数，则加月)      |
| getDay()                       | 返回日期中表示周几的数值(0 表示周日，6 表示周六)       |
| getUTCDay()                    | 返回 UTC 日期中表示周几的数值(0 表示周日，6 表示周六)  |
| getHours()                     | 返回日期中的时(0~23)                                   |
| getUTCHours()                  | 返回 UTC 日期中的时(0~23)                              |
| setHours(hours)                | 设置日期中的时(如果 hours 大于 23，则加日)             |
| setUTCHours(hours)             | 设置 UTC 日期中的时(如果 hours 大于 23，则加日)        |
| getMinutes()                   | 返回日期中的分(0~59)                                   |
| getUTCMinutes()                | 返回 UTC 日期中的分(0~59)                              |
| setMinutes(minutes)            | 设置日期中的分(如果 minutes 大于 59，则加时)           |
| setUTCMinutes(minutes)         | 设置 UTC 日期中的分(如果 minutes 大于 59，则加时)      |
| getSeconds()                   | 返回日期中的秒(0~59)                                   |
| getUTCSeconds()                | 返回 UTC 日期中的秒(0~59)                              |
| setSeconds(seconds)            | 设置日期中的秒(如果 seconds 大于 59，则加分)           |
| setUTCSeconds(seconds)         | 设置 UTC 日期中的秒(如果 seconds 大于 59，则加分)      |
| getMillseconds()               | 返回日期中的毫秒                                       |
| getUTCMillseconds()            | 返回 UTC 日期中的毫秒                                  |
| setMillseconds(millseconds)    | 设置日期中的毫秒                                       |
| setUTCMillseconds(millseconds) | 设置 UTC 日期中的毫秒                                  |
| getTimezoneOffset()            | 返回以分钟计的 UTC 与本地时区的偏移量                  |

```javascript
let now = new Date();
console.log(now.getTime()); // 1610474569539
console.log(now.setTime(now.getTime() / 2)); // Sun Jul 09 1995 05:01:24 GMT+0800 (中国标准时间)
console.log(now.getTimezoneOffset()); // -480
```
