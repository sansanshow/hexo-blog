---
title: sessionStorage移动端兼容问题
date: 2018-09-13 11:37:45
tags:
  - JS
---

# 关于sessionStorage的移动端兼容问题
最近在开发移动端项目时，需要用到的本地存储的地方不少。都是一些只要记住当前打开窗口的用户数据就行，所以我选择用的sessionStorage。使用场景如下：

> A.html页面需要记录一条数据{a:1,b:2};
```
sessionStorage.setItem("data","{a:1,b:2}");
```
> B.html页面取出使用;

`sessionStorage.getItem("data"); // 获取结果为null`

问题：如果项目不是单页面复应用，AB是两个html文件，需要跳转href的。我们会发现有些Andiron系统的浏览器在B页获取是到的结果是null (如：vivo手机自带的世界之窗浏览器)。

经过分析，其实这并不是这个浏览器不支持sessionStorage，因为你还是能获取到sessionStorage这个对象的。而是**因为sessionStorage是一个当前窗口的数据存储格式，有些浏览器在跳转新页面的时候他系统是打开了一个新的webView，把原来的关了,也就相当于我们在浏览器打开了一个新窗口。这样他就跟我们的sessionStorage原理冲突了，在新页面当然就获取不到咯。**  所以建议大家做移动端的时候如果不是单页面复应用的的项目最好不要使用sessionStorage。慎用！


## sessionStorage 的数据会在同一网站的多个标签页之间共享吗？这取决于标签页如何打开

一直以来，我所以为的 sessionStorage 的生命周期是这样的：在 sessionStorage 中存储的数据会在当前浏览器的同一网站的多个标签页中共享，并在此网站的最后一个标签页被关闭后清除。注意：这是错误的。

我之所以会这么认为，是因为我写代码的时候，sessionStorage 给我的表现就是这样的。

假设我们有一个 index.html：

```
<!-- 使用一个新标签页打开自身，并设置一个 sessionStorage -->
<a href="index.html" target="_blank" onclick="sessionStorage.setItem('j', 's')">
  open myself
</a>
```

接下来：

1. 在浏览器中打开这个 index.html，我们称之为标签页 A。注意：需要用 http 协议打开！例如 `http://localhost/index.html`
2. 点击页面上的链接，此时会弹出来标签页 B。
3. 在标签页 B 中打开控制台并执行 sessionStorage.getItem('j')
控制台会输出 's'，这说明标签页 A 和 B 共享了 sessionStorage 中的数据；接下来，先关闭这两个标签页，然后再打开一个标签页 C，再读取一下 j 的值，得到的是 null。

这看起来跟本文一开始的说法是一致的，但今天我遇到了一个奇怪的事情……

我们给上面的步骤添加第四步：

1. 在浏览器中打开这个 index.html，我们称之为标签页 A。注意：需要用 http 协议打开！例如 http://localhost/index.html
2. 点击页面上的链接，此时会弹出来标签页 B。
3. 在标签页 B 中打开控制台并执行 sessionStorage.getItem('j')，得到 's'
4. 新建一个新标签页 D，然后在地址栏内输入 http://localhost/index.html 打开同样的页面， 然后执行 sessionStorage.getItem('j') 。

按照我的预期，标签页 D 得到的应该还是 's'，毕竟我认为 sessionStorage 的数据是在同一网站的多个标签页之间共享的。但是我错了，得到的结果是 null。

发生了什么？为什么标签页 B 中得到的是 's'，为什么标签页 D 中却是 null？

细心的同学可能已经发现了，**标签页 B 和标签页 D 之间唯一的不同就是它们被打开的方式：标签页 B 是通过在标签页 A 中点击链接打开的，但标签页 D 是在浏览器地址栏输入地址打开的。**

我赶紧上 MDN 查了一下，上面是这么说的：

> ...data stored in sessionStorage gets cleared when the page session ends...Opening a page in a new tab or window will cause a new session to be initiated, which differs from how session cookies work.

所以现在我明白了：通过点击链接（或者用了 window.open）打开的新标签页之间是属于同一个 session 的，但新开一个标签页总是会初始化一个新的 session，即使网站是一样的，它们也不属于同一个 session。