---
title: window.requestAnimationFrame使用
tags:
  - JS
abbrlink: 6ff4728c
date: 2018-06-26 21:40:56
---
## 先看MDN文档：

window.requestAnimationFrame() 方法告诉浏览器您希望执行动画并请求浏览器在下一次重绘之前调用指定的函数来更新动画。该方法使用一个回调函数作为参数，这个回调函数会在浏览器重绘之前调用。

> 注意：若您想要在下次重绘时产生另一个动画画面，您的回调例程必须调用 requestAnimationFrame()。

当你需要更新屏幕画面时就可以调用此方法。在浏览器下次重绘前执行回调函数。回调的次数通常是每秒60次，但大多数浏览器通常匹配 W3C 所建议的刷新频率。在大多数浏览器里，当运行在后台标签页或者隐藏的&lt;iframe&gt; 里时，requestAnimationFrame() 会暂停调用以提升性能和电池寿命。

回调函数会被传入一个参数，DOMHighResTimeStamp，指示当前被 requestAnimationFrame() 排序的回调函数被触发的时间。即使每个回调函数的工作量的计算都花了时间，单个帧中的多个回调也都将被传入相同的时间戳。该时间戳是一个十进制数，单位毫秒，最小精度为1ms(1000μs)。   

## 语法

```
window.requestAnimationFrame(callback)
```

> 参数

callback   
一个指定函数的参数，该函数在下次重新绘制动画时调用。这个回调函数`只有一个传参`，`DOMHighResTimeStamp`，指示`requestAnimationFrame()` 开始触发回调函数的当前时间（`performance.now()` 返回的时间）。

> 返回值

一个 `long` 整数，请求 ID ，是回调列表中`唯一的标识`。是个非零值，没别的意义。你可以传这个值给 `window.cancelAnimationFrame()` 以取消回调函数。


比如： 写一个进度条
> html
```
<div id="requestAnimationFrame-test" style="width: 1px;height: 18px;background: #666;">0%</div>
<button onclick="run()">Run</button>
```

> javascript
```
window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame
var start = 0;
var ele = document.getElementById('requestAnimationFrame-test');
var progress = 0

function step(timestamp) {
    progress += 1;
    ele.style.width = progress+'%';
    ele.innerHTML = progress+'%';
    if(progress < 100) {
        requestAnimationFrame(step)
    }
}

function run() {
    ele.style.width = "1px";
    progress = 0;
    requestAnimationFrame(step);
}
```


demo: [点击查看](https://sansanshow.github.io/fe-notes/examples/html/requestAnimationFrame.html)

