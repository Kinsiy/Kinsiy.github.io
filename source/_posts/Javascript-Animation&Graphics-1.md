---
title: Javascript-动画与图形-上
date: 2021-10-25 21:08:28
tags: [JS红宝书]
categories: [学习笔记, Javascript]
keywords:
description:
photos:
---

图形和动画已经日益成为浏览器中现代 Web 应用程序的必备功能，但实现起来仍然比较困难

## 早期定时动画

以前，在 JavaScript 中创建动画基本上就是使用 setInterval()来控制动画的执行

```javascript
(function () {
	function updateAnimations() {
		doAnimation1();
		doAnimation2();
		// 其他任务
	}
	setInterval(updateAnimations, 100);
})();
```

这种定时动画的问题在于无法准确知晓循环之间的延时。定时间隔必须足够短，这样才能让不同的动画类型都能平滑顺畅，但又要足够长，以便产生浏览器可以渲染出来的变化。一般计算机显示器的屏幕刷新率都是 60Hz，基本上意味着每秒需要重绘 60 次。大多数浏览器会限制重绘频率，使其不超出屏幕的刷新率，这是因为超过刷新率，用户也感知不到。

<!-- more -->

因此，实现平滑动画最佳的重绘间隔为 1000 毫秒/60，大约 17 毫秒。以这个速度重绘可以实现最平滑的动画，因为这已经是浏览器的极限了。如果同时运行多个动画，可能需要加以限流，以免 17 毫秒的重绘间隔过快，导致动画过早运行完。

虽然使用 setInterval()的定时动画比使用多个 setTimeout()实现循环效率更高，但也不是没有问题。无论 setInterval()还是 setTimeout()都是不能保证时间精度的。作为第二个参数的延时只能保证何时会把代码添加到浏览器的任务队列，不能保证添加到队列就会立即运行。如果队列前面还有其他任务，那么就要等这些任务执行完再执行。简单来讲，这里毫秒延时并不是说何时这些代码会执行，而只是说到时候会把回调加到任务队列。如果添加到队列后，主线程还被其他任务占用，比如正在处理用户操作，那么回调就不会马上执行。

## requestAnimationFrame

requestAnimationFrame()方法接收一个参数，此参数是一个要在重绘屏幕前调用的函数。这个函数就是修改 DOM 样式以反映下一次重绘有什么变化的地方。为了实现动画循环，可以把多个 requestAnimationFrame()调用串联起来，就像以前使用 setTimeout()时一样

```JavaScript
let requestId
function updateProgress() {
 var div = document.getElementById("status");
 div.style.width = (parseInt(div.style.width, 10) + 5) + "%";
 if (div.style.left != "100%") {
 requestId = requestAnimationFrame(updateProgress);
 }
}
requestId = requestAnimationFrame(updateProgress);


// 返回值 requestId 可用来取消回调


cancelAnimationFrame(requestId);
```

传给 requestAnimationFrame()的函数实际上可以接收一个参数，此参数是一个 DOMHighResTimeStamp 的实（比如 performance.now()返回的值），表示下次重绘的时间

### 结构化动画

在 requestAnimationFrame 基础上创建一个更通用的动画函数：

```javascript
function animate({ timing, draw, duration }) {
	let start = performance.now();

	requestAnimationFrame(function animate(time) {
		// timeFraction 从 0 增加到 1
		let timeFraction = (time - start) / duration;
		if (timeFraction > 1) timeFraction = 1;

		// 计算当前动画状态
		let progress = timing(timeFraction);

		draw(progress); // 绘制

		if (timeFraction < 1) {
			requestAnimationFrame(animate);
		}
	});
}
```

animate 函数接受 3 个描述动画的基本参数：

-   duration：动画总时间，比如 1000。

-   timing(timeFraction)：时序函数，类似 CSS 属性 transition-timing-function，传入一个已过去的时间与总时间之比的小数（0 代表开始，1 代表结束），返回动画完成度（类似 Bezier 曲线中的 y）。[时序函数对比](https://cubic-bezier.com/#.17,.67,.83,.67)
-   draw(progress)：获取动画完成状态并绘制的函数。值 progress = 0 表示开始动画状态，progress = 1 表示结束状态。这是实际绘制动画的函数

更多信息参考这里 [javascript 动画](https://zh.javascript.info/js-animation#jie-gou-hua-dong-hua)
