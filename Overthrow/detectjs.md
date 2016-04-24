# detect.js

### *用于嗅探是否原生支持局部scroll(包括桌面浏览器)*
```
// Features-first. iOS5 overflow scrolling property check - no UA needed here. thanks Apple :)

"WebkitOverflowScrolling" in docElem.style ||

// Test the windows scrolling property as well

"msOverflowStyle" in docElem.style ||

// Touch events aren't supported and screen width is greater than X
// ...basically, this is a loose "desktop browser" check.
// It may wrongly opt-in very large tablets with no touch support.

( !canBeFilledWithPoly && w.screen.width > 800 )
```
### *以及是否支持polyfill模拟局部scroll*
```
// Touch events are used in the polyfill, and thus are a prerequisite

canBeFilledWithPoly = "ontouchmove" in doc,
```

## 1. 产生overthrow对象
__在window下产生overthrow对象__
* __方法__
    - __set__
当native的overflow可用时在documentElement上设置class标识
    - __forget__
取消在documentElement上设置的class标识
    - __addClass__
    在documentElement上设置enabledClassName class标识
    - __removeClass__
    在documentElement上移除enabledClassName class标识
* __属性__
    - __enabledClassName: overthrow-enabled__
    是否支持动量scroll的class标识
    - __canBeFilledWithPoly__
    浏览器是否支持scroll polyfill
    - __support__
    'native' or 'none'
