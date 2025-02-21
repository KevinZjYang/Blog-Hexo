---
title: UWP WebView注入JS
tags:
  - UWP
  - WebView
categories:
  - - 开发笔记
date: 2020-01-01 23:03:21
---

\[post id=1372\]

接上面这篇文章，在修改UA还是失败后，感谢蓝火火大佬，告诉我是因为没有响应点击事件，只响应了触摸事件。并且亲自操刀写下了下面的js代码：

(function (d) {
    const fun = function (e) {
        if (e.touches === undefined) {
            e.touches = \[{ pageX: e.screenX, pageY: e.screenY }\]
        }
    }
    d.addEventListener('pointerdown', fun, true)
    d.addEventListener('pointermove', fun, true)
    d.body.style\["userSelect"\] = "none"
    d.body.style\["webkitUserSelect"\] = "none"
})(document)

然后我只需要在WebView里面无脑注入就可以了。

await \_webView.InvokeScriptAsync("eval", new string\[\]
             {
                 @"(function (d) {
     const fun = function (e) {
         if (e.touches === undefined) {
             e.touches = \[{ pageX: e.screenX, pageY: e.screenY }\]
         }
     }
     d.addEventListener('pointerdown', fun, true)
     d.addEventListener('pointermove', fun, true)
     d.body.style\['userSelect'\] = 'none'
     d.body.style\['webkitUserSelect'\] = 'none'
 })(document)"
             });