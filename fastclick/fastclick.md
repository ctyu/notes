# **fastclick**

# 在ios9.x上fastclick已经不适用(比如在一个可滚动元素上滑动点击会错误触发click，原生则不会)

## 1. 判断是否需要fastclick，不需要直接return

主要逻辑：
判断在Android和ios中是否有meta[name=viewport]，并且符合任一条件则不需要(还需要结合浏览器的版本)
* content里含有user-scalable=no
* document.documentElement.scrollWidth <= window.outerWidth (width=device-width)

如何阻止原生click:
1. 绑定touchstart事件，必要时候preventDefault，可以阻止触发原生click
2. 绑定touchend事件，必要时候preventDefault，可以阻止触发原生click
3. document上捕获(非冒泡)click事件，然后preventDefault以及stopPropagation

如何触发模拟click事件
1. 先blur被选中元素(在某些Android机器上需要先这么做，否则模拟的click无效)
2. createEvent('MouseEvents');initMouseEvent();targetElement.dispatchEvent();

标志位：
1. trackingClick touchstart是否在处于fastclick的处理中。在touchstart时设置为true
2. trackingClickStart trackingClick设置为true的事件戳
3. targetElement touchstart时的target
