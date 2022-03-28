---
title: Javascript-动画与图形-下
date: 2021-10-28 22:36:59
tags: [JS红宝书]
categories: [学习笔记, Javascript]
description:
photos:
---

Canvas API 提供了一个通过 JavaScript 和 HTML 的<canvas\>元素来绘制图形的方式。它可以用于动画、游戏画面、数据可视化、图片编辑以及实时视频处理等方面。

Canvas API 主要聚焦于 2D 图形。而同样使用<canvas\>元素的 WebGL API 则用于绘制硬件加速的 2D 和 3D 图形

<!-- more -->

## 基本的画布功能

创建<canvas\>元素时至少要设置其 width 和 height 属性，这样才能告诉浏览器在多大面积上绘图。出现在开始和结束标签之间的内容是后备数据，会在浏览器不支持<canvas\>元素时显示

与其他元素一样，width 和 height 属性也可以在 DOM 节点上设置，因此可以随时修改。整个元 素还可以通过 CSS 添加样式，并且元素在添加样式或实际绘制内容前是不可见的

可以使用 toDataURL()方法导出元素上的图像。这个方法接收一个参数：要生成图像 的 MIME 类型（与用来创建图形的上下文无关）。

```javascript
/**
  <canvas id="drawing" width="200" height="200">A drawing of something.</canvas>
*/

let drawing = document.getElementById("drawing");

// 确保浏览器支持<canvas>
if (drawing.getContext) {
  // 取得图像的数据 URI
  let imgURI = drawing.toDataURL("image/png");

  // 显示图片
  let image = document.createElement("img");
  image.src = imgURI;
  document.body.appendChild(image);
}
```

## 2D 绘图上下文

2D 绘图上下文提供了绘制 2D 图形的方法，包括矩形、弧形和路径。2D 上下文的坐标原点(0, 0)在<canvas\>元素的左上角。所有坐标值都相对于该点计算，因此 x 坐标向右增长，y 坐标向下增长。默认情况下，width 和 height 表示两个方向上像素的最大值

### 填充与描边

2D 上下文有两个基本绘制操作：填充和描边。填充以指定样式（颜色、渐变或图像）自动填充形状，而描边只为图形边界着色。大多数 2D 上下文操作有填充和描边的变体，显示效果取决于两个属性：fillStyle 和 strokeStyle。

这两个属性可以是字符串、渐变对象或图案对象，默认值都为"#000000"。字符串表示颜色值，可以是 CSS 支持的任意格式：名称、十六进制代码、rgb、rgba、hsl 或 hsla。

```javascript
let drawing = document.getElementById("drawing");
// 确保浏览器支持<canvas>
if (drawing.getContext) {
  let context = drawing.getContext("2d");
  context.strokeStyle = "red";
  context.fillStyle = "#0000ff";
}
```

### 绘制矩形

矩形是唯一一个可以直接在 2D 绘图上下文中绘制的形状。与绘制矩形相关的方法有 3 个：fillRect()、strokeRect()和 clearRect()。这些方法都接收 4 个参数：矩形 x 坐标、矩形 y 坐标、矩形宽度和矩形高度。这几个参数的单位都是像素。

```javascript
let drawing = document.getElementById("drawing");
// 确保浏览器支持<canvas>
if (drawing.getContext) {
  let context = drawing.getContext("2d");

  // fillRect()方法用于以指定颜色在画布上绘制并填充矩形。填充的颜色使用 fillStyle 属性指定
  // 绘制红色矩形
  context.fillStyle = "#ff0000";
  context.fillRect(10, 10, 50, 50);

  // strokeRect()方法使用通过 strokeStyle 属性指定的颜色绘制矩形轮廓
  // 绘制半透明蓝色轮廓的矩形
  context.strokeStyle = "rgba(0,0,255,0.5)";
  context.strokeRect(30, 30, 50, 50);

  // 使用 clearRect()方法可以擦除画布中某个区域。该方法用于把绘图上下文中的某个区域变透明。
  // 在前两个矩形重叠的区域擦除一个矩形区域
  context.clearRect(40, 40, 10, 10);
}
```

### 绘制路径

2D 绘图上下文支持很多在画布上绘制路径的方法。通过路径可以创建复杂的形状和线条。要绘制路径，必须首先调用 beginPath()方法以表示要开始绘制新路径。然后，再调用下列方法来绘制路径

-   arc(x, y, radius, startAngle, endAngle, counterclockwise): 以坐标(x ,y)为圆心，以 radius 为半径绘制一条弧线，起始角度为 startAngle，结束角度为 endAngle(都是弧度)。最后一个参数 counterclockwise 表示是否逆时针计算其实角度和结束角度(默认为顺时针)
-   arcTo(x1, y1, x2, y2, radius): 以给定半径 radius，经由(x1, y1)绘制一条从上一点到(x2, y2)的弧线
-   bezierCurveTo(c1x, c1y, c2x, c2y, x, y): 以(c1x, c1y)和(c2x, c2y)为控制点，绘制一条从上一点到(x, y)的弧线(三次贝塞尔曲线)
-   lineTo(x, y): 绘制一条从上一点到(x, y)的直线
-   moveTo(x, y): 不绘制线条，只把绘制光标移动到(x, y)
-   quadraticCurveTo(cx, cy, x, y): 以(cx, cy)为控制点，绘制一条从上一点到(x, y)的弧线(二次贝塞尔曲线)
-   rect(x, y, width, height): 以给定宽度和高度在坐标点(x, y)绘制一个矩形。这个方法与 strokeRect()和 fillRect()的区别在于，它创建的是一条路径，而不是独立的图形

创建路径之后，可以使用 closePath()方法绘制一条返回起点的线。如果路径已经完成，则既可 以指定 fillStyle 属性并调用 fill()方法来填充路径，也可以指定 strokeStyle 属性并调用 stroke()方法来描画路径，还可以调用 clip()方法基于已有路径创建一个新剪切区域

路径是 2D 上下文的主要绘制机制，为绘制结果提供了很多控制。因为路径经常被使用，所以也有 一个 isPointInPath()方法，接收 x 轴和 y 轴坐标作为参数。这个方法用于确定指定的点是否在路径 上，可以在关闭路径前随时调用

```JavaScript
let drawing = document.getElementById("drawing");
// 确保浏览器支持<canvas>
if (drawing.getContext) {
 let context = drawing.getContext("2d");

 // 创建路径
 context.beginPath();

 // 绘制外圆
 context.arc(100, 100, 99, 0, 2 * Math.PI, false);

 // 绘制内圆
 context.moveTo(194, 100);
 context.arc(100, 100, 94, 0, 2 * Math.PI, false);

 // 绘制分针
 context.moveTo(100, 100);
 context.lineTo(100, 15);

 // 绘制时针
 context.moveTo(100, 100);
 context.lineTo(35, 100);

 // 描画路径
 context.stroke();

  // 判断指点的点是否在路径上
 if (context.isPointInPath(100, 100)) {
  alert("Point (100, 100) is in the path.");
 }
}
```

### 绘制文本

文本和图像混合也是常见的绘制需求，因此 2D 绘图上下文还提供了绘制文本的方法，即 fillText() 和 strokeText()。这两个方法都接收 4 个参数：要绘制的字符串、x 坐标、y 坐标和可选的最大像素 宽度。而且，这两个方法最终绘制的结果都取决于以下 3 个属性

-   font: 以 CSS 语法指定的字体样式、大小、字体族等，比如"10px Arial"
-   textAlign: 指定文本的对齐方式，可能的值包括"start"、"end"、"left"、"right"和"center"。
-   textBaseLine: 指定文本的基线，可能的值包括"top"、"hanging"、"middle"、"alphabetic"、"ideological"和"bottom"

这些属性都有相应的默认值，因此没必要每次绘制文本时都设置它们。fillText()方法使用 fillStyle 属性绘制文本，而 strokeText()方法使用 strokeStyle 属性。通常，fillText()方法是使用最多的，因为它模拟了在网页中渲染文本

2D 上下文提供了用于辅助确定文本大小的 measureText()方法。这个方法接收一个参数，即要绘制的文本，然后返回一个 TextMetrics 对象。这个返回的对象目前只有一个属性 width，不过将来应该会增加更多度量指标。

measureText()方法使用 font、textAlign 和 textBaseline 属性当前的值计算绘制指定文本后的大小。

```javascript
let fontSize = 100;
context.font = fontSize + "px Arial";
while (context.measureText("Hello world!").width > 140) {
  fontSize--;
  context.font = fontSize + "px Arial";
}
context.fillText("Hello world!", 10, 10);
context.fillText("Font size is " + fontSize + "px", 10, 50);
```

### 变换

上下文变换可以操作绘制在画布上的图像。2D 绘图上下文支持所有常见的绘制变换。在创建绘制上下文时，会以默认值初始化变换矩阵，从而让绘制操作如实应用到绘制结果上。对绘制上下文应用变换，可以导致以不同的变换矩阵应用绘制操作，从而产生不同的结果

- rotate(angle)：围绕**原点**把图像旋转 angle 弧度。

- scale(scaleX, scaleY)：通过在 x 轴乘以 scaleX、在 y 轴乘以 scaleY 来缩放图像。scaleX 和 scaleY 的默认值都是 1.0。

- translate(x, y)：把原点移动到(x, y)。执行这个操作后，坐标(0, 0)就会变成(x, y)。

- transform(m1_1, m1_2, m2_1, m2_2, dx, dy)：像下面这样通过矩阵乘法直接修改矩阵。
  $$
  \begin{matrix}
     m1_1 & m1_2 & dx \\
     m2_1 & m2_2 & dy \\
     0 & 0 & 1
  \end{matrix}
  $$

- setTransform(m1_1, m1_2, m2_1, m2_2, dx, dy)：把矩阵重置为默认值，再以传入的参数调用 transform()

所有这些变换，包括 fillStyle 和 strokeStyle 属性，会一直保留在上下文中，直到再次修改它们。虽然没有办法明确地将所有值都重置为默认值，但有两个方法可以帮我们跟踪变化。如果想着什么时候再回到当前的属性和变换状态，可以调用 save()方法。调用这个方法后，所有这一时刻的设置会被放到一个暂存栈中。保存之后，可以继续修改上下文。而在需要恢复之前的上下文时，可以调用 restore()方法。这个方法会从暂存栈中取出并恢复之前保存的设置。多次调用 save()方法可以在暂存栈中存储多套设置，然后通过 restore()可以系统地恢复

```javascript
context.fillStyle = "#ff0000";
context.save();

context.fillStyle = "#00ff00";
context.translate(100, 100);
context.save();

context.fillStyle = "#0000ff";
context.fillRect(0, 0, 100, 200); // 在(100, 100)绘制蓝色矩形

context.restore();
context.fillRect(10, 10, 100, 200); // 在(100, 100)绘制绿色矩形

context.restore();
context.fillRect(0, 0, 100, 200); // 在(0, 0)绘制红色矩形
```

### 绘制图像

2D 绘图上下文内置支持操作图像。如果想把现有图像绘制到画布上，可以使用 drawImage()方法。这个方法可以接收 3 组不同的参数，并产生不同的结果

```JavaScript
let image = document.images[0];

// 获取了文本中的第一个图像，然后在画布上的坐标(10, 10)处将它绘制了出来
context.drawImage(image, 10, 10);

// 图像会缩放到 20 像素宽、30 像素高
context.drawImage(image, 50, 10, 20, 30)

/**
  要绘制的图像、源图像 x 坐标、源图像 y 坐标、源图像宽度、源图像高度、目标区域 x 坐标、目标区域 y 坐标、目标区域宽度和目标区域高度
*/
context.drawImage(image, 0, 10, 50, 50, 0, 100, 40, 60);
/**
  原始图像中只有一部分会绘制到画布上。
  这一部分从(0, 10)开始，50 像素宽、50 像素高。
  而绘制到画布上时，会从(0, 100)开始，变成 40 像素宽、60 像素高
*/
```

### 阴影

2D 上下文可以根据以下属性的值自动为已有形状或路径生成阴影。

-   shadowColor：CSS 颜色值，表示要绘制的阴影颜色，默认为黑色。
-   shadowOffsetX：阴影相对于形状或路径的 x 坐标的偏移量，默认为 0。
-   shadowOffsetY：阴影相对于形状或路径的 y 坐标的偏移量，默认为 0。
-   shadowBlur：像素，表示阴影的模糊量。默认值为 0，表示不模糊

### 渐变

渐变通过 CanvasGradient 的实例表示，在 2D 上下文中创建和修改都非常简单。要创建一个新的 线性渐变，可以调用上下文的 createLinearGradient()方法。这个方法接收 4 个参数：起点 x 坐标、 起点 y 坐标、终点 x 坐标和终点 y 坐标。调用之后，该方法会以指定大小创建一个新的 CanvasGradient 对象并返回实例

有了 gradient 对象后，接下来要使用 addColorStop()方法为渐变指定色标。这个方法接收两 个参数：色标位置和 CSS 颜色字符串。色标位置通过 0 ～ 1 范围内的值表示，0 是第一种颜色，1 是最后 一种颜色

```javascript
let gradient = context.createLinearGradient(30, 30, 70, 70);
gradient.addColorStop(0, "white");
gradient.addColorStop(1, "black");

// 绘制红色矩形
context.fillStyle = "#ff0000";
context.fillRect(10, 10, 50, 50);
// 绘制渐变矩形
context.fillStyle = gradient;
context.fillRect(30, 30, 50, 50);
```

径向渐变（或放射性渐变）要使用 createRadialGradient()方法来创建。这个方法接收 6 个参数，分别对应两个圆形圆心的坐标和半径。前 3 个参数指定起点圆形中心的 x、y 坐标和半径，后 3 个参数指定终点圆形中心的 x、y 坐标和半径。在创建径向渐变时，可以把两个圆形想象成一个圆柱体的两个圆形表面。把一个表面定义得小一点，另一个定义得大一点，就会得到一个圆锥体。然后，通过移动两个圆形的圆心，就可以旋转这个圆锥体

```javascript
let gradient = context.createRadialGradient(55, 55, 10, 55, 55, 30);
gradient.addColorStop(0, "white");
gradient.addColorStop(1, "black");
// 绘制红色矩形
context.fillStyle = "#ff0000";
context.fillRect(10, 10, 50, 50);
// 绘制渐变矩形
context.fillStyle = gradient;
context.fillRect(30, 30, 50, 50);
```

### 图案

图案是用于填充和描画图形的重复图像。要创建新图案，可以调用 createPattern()方法并传入两个参数：一个 HTML <img\>元素和一个表示该如何重复图像的字符串。第二个参数的值与 CSS 的 background-repeat 属性是一样的，包括"repeat"、"repeat-x"、"repeat-y"和"no-repeat"

```javascript
et image = document.images[0],
 pattern = context.createPattern(image, "repeat");
// 绘制矩形
context.fillStyle = pattern;
context.fillRect(10, 10, 150, 150);
```

### 图像数据

2D 上下文中比较强大的一种能力是可以使用 getImageData()方法获取原始图像数据。这个方法 接收 4 个参数：要取得数据中第一个像素的左上角坐标和要取得的像素宽度及高度

返回的对象是一个 ImageData 的实例。每个 ImageData 对象都包含 3 个属性：width、height 和 data，其中，data 属性是包含图像的原始像素信息的数组。每个像素在 data 数组中都由 4 个值表 示，分别代表红、绿、蓝和透明度值。换句话说，第一个像素的信息包含在第 0 到第 3 个值中

过更改图像数据可以创建一个简单的灰阶过滤器

```JavaScript
let drawing = document.getElementById("drawing");
// 确保浏览器支持<canvas>
if (drawing.getContext) {
 let context = drawing.getContext("2d"),
 image = document.images[0],
 imageData, data,
 i, len, average,
 red, green, blue, alpha;

 // 绘制图像
 context.drawImage(image, 0, 0);

 // 取得图像数据
 imageData = context.getImageData(0, 0, image.width, image.height);
 data = imageData.data;

 for (i=0, len=data.length; i < len; i+=4) {
 red = data[i];
 green = data[i+1];
 blue = data[i+2];
 alpha = data[i+3];
 // 取得 RGB 平均值
 average = Math.floor((red + green + blue) / 3);
 // 设置颜色，不管透明度
 data[i] = average;
 data[i+1] = average;
 data[i+2] = average;
 }
 // 将修改后的数据写回 ImageData 并应用到画布上显示出来
 imageData.data = data;
 context.putImageData(imageData, 0, 0);
}
```

### 合成

2D 上下文中绘制的所有内容都会应用两个属性：globalAlpha 和 globalComposition Operation，其中，globalAlpha 属性是一个范围在 0~1 的值（包括 0 和 1），用于指定所有绘制内容的透明度，默认值为 0。如果所有后来的绘制都需要使用同样的透明度，那么可以将 globalAlpha 设置为适当的值，执行绘制，然后再把 globalAlpha 设置为 0

```JavaScript
// 绘制红色矩形
context.fillStyle = "#ff0000";
context.fillRect(10, 10, 50, 50);


// 修改全局透明度
context.globalAlpha = 0.5;


// 绘制蓝色矩形
context.fillStyle = "rgba(0,0,255,1)";
context.fillRect(30, 30, 50, 50);


// 重置
context.globalAlpha = 0;
```

globalCompositionOperation 属性表示新绘制的形状如何与上下文中已有的形状融合。这个属性是一个字符串，可以取下列值

-   source-over：默认值，新图形绘制在原有图形上面。
-   source-in：新图形只绘制出与原有图形重叠的部分，画布上其余部分全部透明。
-   source-out：新图形只绘制出不与原有图形重叠的部分，画布上其余部分全部透明。
-   source-atop：新图形只绘制出与原有图形重叠的部分，原有图形不受影响。
-   destination-over：新图形绘制在原有图形下面，重叠部分只有原图形透明像素下的部分可见。
-   destination-in：新图形绘制在原有图形下面，画布上只剩下二者重叠的部分，其余部分完全透明。
-   destination-out：新图形与原有图形重叠的部分完全透明，原图形其余部分不受影响。
-   destination-atop：新图形绘制在原有图形下面，原有图形与新图形不重叠的部分完全透明。
-   lighter：新图形与原有图形重叠部分的像素值相加，使该部分变亮。
-   copy：新图形将擦除并完全取代原有图形。
-   xor：新图形与原有图形重叠部分的像素执行“异或”计
