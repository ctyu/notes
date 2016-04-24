# polyfill.js

### *主逻辑，绑定touchstart, touchmove, touchend事件*


## touchstart

### 绑定

**在document上绑定touchstart事件**

### touchstart逻辑
1. **从事件对象中就近获取拥有overflow类名的dom元素作为elem**
2. **设置退出条件，无elem或者elem为docElem，或者为多指触控**
3. **设置elem中textarea 和 input的pointerEvents属性为none，防止无法拿到后续的touchmove事件**
4. **重置*lastTops lastDown* 和 *lastLefts lastRight***
5. **获取元素的宽高，元素的scrollTop,scrollLeft，元素的scrollHeight和scrollWidth用于后续计算处理。设置startY,startX(e.pageX)**
6. **为elem绑定touchmove和touchend事件**


## touchmove

### touchmove逻辑
1. **根据scrollTop、startY和e.touches[0].pageY和上次touchmove对应的值判断方向**
2. **如果此时elem是个可滚动的元素并且没有滚到头，则需要阻止touchmove的默认行为，防止多层滚动**
3. **如果elem非滚动元素或者已滚到头，则在该elem元素上面手动触发touchend, 在其父可滚动元素(含有overthrow)中使用该touchmove的事件对象触发touchstart事件，重走touchstart逻辑**
4. **如滚动方向改变，走touchstart的4逻辑**
5. **设置elem的scrollTop和left，并缓存，用于下次touchmove时判断方向。缓存数最多保存3个**

## touchend

### touchend逻辑
1. **设置elem中textarea 和 input的pointerEvents属性为auto，防止无法触发输入元素的focus**
2. **解绑touchmove touchend事件**
