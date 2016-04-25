## iscroll Core源码解读，基于iscroll -v 5.2.0

**iscroll 官网 <http://iscrolljs.com/>**

**先学用法，再看源码**

### 是一个自执行闭包

    (function(window, document, Math){[...code]}})(window, document, Math);

### 获取刷新函数

    var rAF = window.requestAnimationFrame	||
    window.webkitRequestAnimationFrame	||
    window.mozRequestAnimationFrame		||
    window.oRequestAnimationFrame		||
    window.msRequestAnimationFrame		||
    function (callback) { window.setTimeout(callback, 1000 / 60); };

### utils工具

**一个自执行函数，形成闭包，缓存一些内部函数和变量**

    // 根据style返回带前缀的style名称
    function _prefixStyle (style) {
        if ( _vendor === false ) return false;
        if ( _vendor === '' ) return style;
        return _vendor + style.charAt(0).toUpperCase() + style.substr(1);
    }
    // 缓存transform样式名
    var _transform = _prefixStyle('transform');

#### 可用api：
**getTime** 获取时间戳

    Date.now || function getTime () { return new Date().getTime(); };

**extend** 简单的扩展函数

    function (target, obj) {
        for ( var i in obj ) {
            target[i] = obj[i];
        }
    };

**addEvent** 简单封装addEventListener

    function (target, obj) {
        for ( var i in obj ) {
            target[i] = obj[i];
        }
    };

**removeEvent** 简单封装removeEventListener

    function (el, type, fn, capture) {
        el.removeEventListener(type, fn, !!capture);
    };

**prefixPointerEvent** 跨平台获取pointer事件名称

    function (pointerEvent) {
        return window.MSPointerEvent ?
            'MSPointer' + pointerEvent.charAt(7).toUpperCase() + pointerEvent.substr(8):
            pointerEvent;
    };

**momentum** 动量函数，根据当前位置、起始位置、持续时间、加速度（这边相当于负值）、容器信息，返回目标位置和所需时间

    /**
     * 动量函数，根据当前位置、起始位置、持续时间、加速度（这边相当于负值）、容器信息，返回目标位置和所需时间
     * @method
     * @param  {[type]} current      [当前位置]
     * @param  {[type]} start        [起始位置]
     * @param  {[type]} time         [持续时间]
     * @param  {[type]} lowerMargin  [配置项]
     * @param  {[type]} wrapperSize  [容器尺寸]
     * @param  {[type]} deceleration [加速度]
     * @return {[type]}              [目标位置和所需时间]
     */
    function (current, start, time, lowerMargin, wrapperSize, deceleration) {
        // 计算距离和平均速度
		var distance = current - start,
			speed = Math.abs(distance) / time,
			destination,
			duration;
        // 加速度默认0.0006
		deceleration = deceleration === undefined ? 0.0006 : deceleration;
        // 匀加速直线运动，计算终点位置
        // 需要根据distance判断方向
		destination = current + ( speed * speed ) / ( 2 * deceleration ) * ( distance < 0 ? -1 : 1 );
        // 计算速度变成0时所需要的时间
		duration = speed / deceleration;
        // 根据wrapperSize和lowerMargin更新distance和duration
		if ( destination < lowerMargin ) {
			destination = wrapperSize ? lowerMargin - ( wrapperSize / 2.5 * ( speed / 8 ) ) : lowerMargin;
			distance = Math.abs(destination - current);
			duration = distance / speed;
		} else if ( destination > 0 ) {
			destination = wrapperSize ? wrapperSize / 2.5 * ( speed / 8 ) : 0;
			distance = Math.abs(current) + destination;
			duration = distance / speed;
		}

		return {
			destination: Math.round(destination),
			duration: duration
		};
	};

**hasClass** 判断dom上是否含有指定类名

    function (e, c) {
		var re = new RegExp("(^|\\s)" + c + "(\\s|$)");
		return re.test(e.className);
	}

**addClass** 给指定的dom元素加上指定类名

    function (e, c) {
		if ( me.hasClass(e, c) ) {
			return;
		}

		var newclass = e.className.split(' ');
		newclass.push(c);
		e.className = newclass.join(' ');
	}

**removeClass** 在指定的dom元素移除指定类名

    function (e, c) {
		if ( !me.hasClass(e, c) ) {
			return;
		}

		var re = new RegExp("(^|\\s)" + c + "(\\s|$)", 'g');
		e.className = e.className.replace(re, ' ');
	};

**offset** 获取指定元素相对于浏览器的定位

    function (el) {
		var left = -el.offsetLeft,
			top = -el.offsetTop;

		// jshint -W084
		while (el = el.offsetParent) {
			left -= el.offsetLeft;
			top -= el.offsetTop;
		}
		// jshint +W084

		return {
			left: left,
			top: top
		};
	};

**preventDefaultException** 判断元素是否需要强制不使用preventDefault

    // exceptions为{ tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT)$/ }
    // 在浏览器中有特殊的默认操作

    function (el, exceptions) {
		for ( var i in exceptions ) {
			if ( exceptions[i].test(el[i]) ) {
				return true;
			}
		}

		return false;
	};

****

#### 可用字段

**hasTransform** 是否支持transform

**hasPrespective** 是否支持prespective

**hasTouch** 是否支持touch事件

**hasPointer** 是否支持pointer事件

**hasTransition** 是否支持transition

**isBadAndroid**

    /*
    	This should find all Android browsers lower than build 535.19 (both stock browser and webview)
    	- galaxy S2 is ok
        - 2.3.6 : `AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1`
        - 4.0.4 : `AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30`
       - galaxy S3 is badAndroid (stock brower, webview)
         `AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30`
       - galaxy S4 is badAndroid (stock brower, webview)
         `AppleWebKit/534.30 (KHTML, like Gecko) Version/4.0 Mobile Safari/534.30`
       - galaxy S5 is OK
         `AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Mobile Safari/537.36 (Chrome/)`
       - galaxy S6 is OK
         `AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Mobile Safari/537.36 (Chrome/)`
      */
    	me.isBadAndroid = (function() {
    		var appVersion = window.navigator.appVersion;
    		// Android browser is not a chrome browser.
    		if (/Android/.test(appVersion) && !(/Chrome\/\d/.test(appVersion))) {
    			var safariVersion = appVersion.match(/Safari\/(\d+.\d)/);
    			if(safariVersion && typeof safariVersion === "object" && safariVersion.length >= 2) {
    				return parseFloat(safariVersion[1]) < 535.19;
    			} else {
    				return true;
    			}
    		} else {
    			return false;
    		}
    	})();

**style** style字段，缓存带浏览器前缀的样式名

    transform: _transform,
    transitionTimingFunction: _prefixStyle('transitionTimingFunction'),
    transitionDuration: _prefixStyle('transitionDuration'),
    transitionDelay: _prefixStyle('transitionDelay'),
    transformOrigin: _prefixStyle('transformOrigin')

**eventType** eventType字段，iscroll自己设定的事件类型，因为事件绑定的句柄都是一个，所以需要区分，防止混淆

    touchstart: 1,
	touchmove: 1,
	touchend: 1,

	mousedown: 2,
	mousemove: 2,
	mouseup: 2,

	pointerdown: 3,
	pointermove: 3,
	pointerup: 3,

	MSPointerDown: 3,
	MSPointerMove: 3,
	MSPointerUp: 3

**ease** 缓动函数和对应的css3属性，用于动量滚动

可参考：<http://easings.net/zh-cn#>

    quadratic: {
		style: 'cubic-bezier(0.25, 0.46, 0.45, 0.94)',
		fn: function (k) {
			return k * ( 2 - k );
		}
	},
	circular: {
		style: 'cubic-bezier(0.1, 0.57, 0.1, 1)',	// Not properly "circular" but this looks better, it should be (0.075, 0.82, 0.165, 1)
		fn: function (k) {
			return Math.sqrt( 1 - ( --k * k ) );
		}
	},
	back: {
		style: 'cubic-bezier(0.175, 0.885, 0.32, 1.275)',
		fn: function (k) {
			var b = 4;
			return ( k = k - 1 ) * k * ( ( b + 1 ) * k + b ) + 1;
		}
	},
	bounce: {
		style: '',
		fn: function (k) {
			if ( ( k /= 1 ) < ( 1 / 2.75 ) ) {
				return 7.5625 * k * k;
			} else if ( k < ( 2 / 2.75 ) ) {
				return 7.5625 * ( k -= ( 1.5 / 2.75 ) ) * k + 0.75;
			} else if ( k < ( 2.5 / 2.75 ) ) {
				return 7.5625 * ( k -= ( 2.25 / 2.75 ) ) * k + 0.9375;
			} else {
				return 7.5625 * ( k -= ( 2.625 / 2.75 ) ) * k + 0.984375;
			}
		}
	},
	elastic: {
		style: '',
		fn: function (k) {
			var f = 0.22,
				e = 0.4;

			if ( k === 0 ) { return 0; }
			if ( k == 1 ) { return 1; }

			return ( e * Math.pow( 2, - 10 * k ) * Math.sin( ( k - f / 4 ) * ( 2 * Math.PI ) / f ) + 1 );
		}
	}

**tap** 在指定dom上触发tap事件

    function (e, eventName) {
		var ev = document.createEvent('Event');
		ev.initEvent(eventName, true, true);
		ev.pageX = e.pageX;
		ev.pageY = e.pageY;
		e.target.dispatchEvent(ev);
	};

**click** 通过event对象，在target上触发click事件

    function (e) {
		var target = e.target,
			ev;

		if ( !(/(SELECT|INPUT|TEXTAREA)/i).test(target.tagName) ) {
			ev = document.createEvent('MouseEvents');
			ev.initMouseEvent('click', true, true, e.view, 1,
				target.screenX, target.screenY, target.clientX, target.clientY,
				e.ctrlKey, e.altKey, e.shiftKey, e.metaKey,
				0, null);
            // 标识该事件是否iscroll生成
			ev._constructed = true;
			target.dispatchEvent(ev);
		}
	};

### Iscroll 类

    /**
     * [IScroll description]
     * @method IScroll
     * @param  {string | dom} el      [需要滚动的元素的wrapper]
     * @param  {obj} options [配置参数]
     */
    function IScroll (el, options) {
        // 通过el获取wrapper
    	this.wrapper = typeof el == 'string' ? document.querySelector(el) : el;
        // iscroll会认为该wrapper下的第一个子元素是需要滚动的元素，忽略其他子元素
    	this.scroller = this.wrapper.children[0];
        // 缓存滚动元素的style属性
    	this.scrollerStyle = this.scroller.style;		// cache style for better performance
        // 默认配置
    	this.options = {

    		resizeScrollbars: true,

    		mouseWheelSpeed: 20,

    		snapThreshold: 0.334,

            // INSERT POINT: OPTIONS
    		disablePointer : !utils.hasPointer,
    		disableTouch : utils.hasPointer || !utils.hasTouch,
    		disableMouse : utils.hasPointer || utils.hasTouch,
    		startX: 0,
    		startY: 0,
    		scrollY: true,
    		directionLockThreshold: 5,
    		momentum: true,

    		bounce: true,
    		bounceTime: 600,
    		bounceEasing: '',

    		preventDefault: true,
    		preventDefaultException: { tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT)$/ },

    		HWCompositing: true,
    		useTransition: true,
    		useTransform: true,
    		bindToWrapper: typeof window.onmousedown === "undefined"
    	};

        // 覆盖默认配置
    	for ( var i in options ) {
    		this.options[i] = options[i];
    	}

    	// Normalize options
        // 如果开启硬件加速并且浏览器支持3d变换，则使用translateZ(0)开启硬件加速
    	this.translateZ = this.options.HWCompositing && utils.hasPerspective ? ' translateZ(0)' : '';

        // 如果浏览器支持transition并且配置为使用transition动画，那么使用transition动画
    	this.options.useTransition = utils.hasTransition && this.options.useTransition;
        // 根据浏览器支持能力，重置useTransform配置项
    	this.options.useTransform = utils.hasTransform && this.options.useTransform;
        // 定义哪个滚动方向需要使用native滚动
    	this.options.eventPassthrough = this.options.eventPassthrough === true ? 'vertical' : this.options.eventPassthrough;
        // 通过是否需要native滚动和配置项，重置preventDefault配置
    	this.options.preventDefault = !this.options.eventPassthrough && this.options.preventDefault;

    	// If you want eventPassthrough I have to lock one of the axes
    	this.options.scrollY = this.options.eventPassthrough == 'vertical' ? false : this.options.scrollY;
    	this.options.scrollX = this.options.eventPassthrough == 'horizontal' ? false : this.options.scrollX;

    	// With eventPassthrough we also need lockDirection mechanism
    	this.options.freeScroll = this.options.freeScroll && !this.options.eventPassthrough;
        // directionLockThreshold表示在一次移动中，两个轴的位移差距大于该值时，只滚动位移大的那个轴
    	this.options.directionLockThreshold = this.options.eventPassthrough ? 0 : this.options.directionLockThreshold;

        // 设置反弹缓动函数
    	this.options.bounceEasing = typeof this.options.bounceEasing == 'string' ? utils.ease[this.options.bounceEasing] || utils.ease.circular : this.options.bounceEasing;

        // resize 函数节流器
    	this.options.resizePolling = this.options.resizePolling === undefined ? 60 : this.options.resizePolling;

    	if ( this.options.tap === true ) {
    		this.options.tap = 'tap';
    	}

        // 设置在滚动超出scroller范围的时候scrollbar的行为。默认是clip。
        // 设置为scale将无法使用transition来做动画，对性能产生影响
    	if ( this.options.shrinkScrollbars == 'scale' ) {
    		this.options.useTransition = false;
    	}

        // 是否需要反转滚轮滚动方向
    	this.options.invertWheelDirection = this.options.invertWheelDirection ? -1 : 1;

        // INSERT POINT: NORMALIZATION

    	// Some defaults
    	this.x = 0; // 初始X轴位置
    	this.y = 0; // 初始Y轴位置
    	this.directionX = 0; // X轴滚动方向
    	this.directionY = 0; // Y轴滚动方向
    	this._events = {};

        // INSERT POINT: DEFAULTS
        // 初始化
    	this._init();
    	this.refresh();

    	this.scrollTo(this.options.startX, this.options.startY);
    	this.enable();
    }

#### prototype

**_init** 初始化事件，滚动条，滚轮，分页，绑定的键盘

    function () {
		this._initEvents();

		if ( this.options.scrollbars || this.options.indicators ) {
			this._initIndicators();
		}

		if ( this.options.mouseWheel ) {
			this._initWheel();
		}

		if ( this.options.snap ) {
			this._initSnap();
		}

		if ( this.options.keyBindings ) {
			this._initKeys();
		}

        // INSERT POINT: _init

	}

**_initEvents** 绑定事件

    // 是否remove
    function (remove) {
        // 根据bindToWrapper字段确定事件绑定对象，默认window
		var eventType = remove ? utils.removeEvent : utils.addEvent,
			target = this.options.bindToWrapper ? this.wrapper : window;

        // 绑定屏幕旋转和resize事件
		eventType(window, 'orientationchange', this);
		eventType(window, 'resize', this);

        // 需要支持click的话，捕获click事件
		if ( this.options.click ) {
			eventType(this.wrapper, 'click', this, true);
		}

        // 需要支持mouse事件的时候，绑定句柄
		if ( !this.options.disableMouse ) {
			eventType(this.wrapper, 'mousedown', this);
			eventType(target, 'mousemove', this);
			eventType(target, 'mousecancel', this);
			eventType(target, 'mouseup', this);
		}

        // 浏览器支持pointer事件并且需要绑定pointer事件
		if ( utils.hasPointer && !this.options.disablePointer ) {
			eventType(this.wrapper, utils.prefixPointerEvent('pointerdown'), this);
			eventType(target, utils.prefixPointerEvent('pointermove'), this);
			eventType(target, utils.prefixPointerEvent('pointercancel'), this);
			eventType(target, utils.prefixPointerEvent('pointerup'), this);
		}

        // 支持touch，并且需要绑定
		if ( utils.hasTouch && !this.options.disableTouch ) {
			eventType(this.wrapper, 'touchstart', this);
			eventType(target, 'touchmove', this);
			eventType(target, 'touchcancel', this);
			eventType(target, 'touchend', this);
		}

        // 绑定transitionend事件
		eventType(this.scroller, 'transitionend', this);
		eventType(this.scroller, 'webkitTransitionEnd', this);
		eventType(this.scroller, 'oTransitionEnd', this);
		eventType(this.scroller, 'MSTransitionEnd', this);
	}

**refresh**

    // 刷新wrapper的宽高和scroller的宽高
    function () {
		var rf = this.wrapper.offsetHeight;		// Force reflow

        // 获取wrapper的宽高
		this.wrapperWidth	= this.wrapper.clientWidth;
		this.wrapperHeight	= this.wrapper.clientHeight;

        /* REPLACE START: refresh */
        // 获取scroller的宽高
		this.scrollerWidth	= this.scroller.offsetWidth;
		this.scrollerHeight	= this.scroller.offsetHeight;

        // 更新最大可滚动宽高
		this.maxScrollX		= this.wrapperWidth - this.scrollerWidth;
		this.maxScrollY		= this.wrapperHeight - this.scrollerHeight;

        /* REPLACE END: refresh */

        // 更新是否应该有水平垂直滚动条
		this.hasHorizontalScroll	= this.options.scrollX && this.maxScrollX < 0;
		this.hasVerticalScroll		= this.options.scrollY && this.maxScrollY < 0;

        // 如果没有（可能有scrollX为false引起，用户强制不滚动）
		if ( !this.hasHorizontalScroll ) {
            // 重置
			this.maxScrollX = 0;
			this.scrollerWidth = this.wrapperWidth;
		}

        // 如果没有（可能有scrollY为false引起，用户强制不滚动）
		if ( !this.hasVerticalScroll ) {
            // 重置
			this.maxScrollY = 0;
			this.scrollerHeight = this.wrapperHeight;
		}

        // 重置touchend时间戳以及x,y轴滚动方向
		this.endTime = 0;
		this.directionX = 0;
		this.directionY = 0;

        // 更新wrapper位置
		this.wrapperOffset = utils.offset(this.wrapper);
        // 触发refresh事件
		this._execEvent('refresh');
        // 重置scroll位置
		this.resetPosition();

        // INSERT POINT: _refresh

	}

**scrollTo** 根据提供的缓动函数和持续时间，滚动到指定的X,Y坐标

    function (x, y, time, easing) {
		easing = easing || utils.ease.circular;

		this.isInTransition = this.options.useTransition && time > 0;
        // 如果可以使用transition，并且有对应的贝塞尔函数，使用transition动画
		var transitionType = this.options.useTransition && easing.style;
		if ( !time || transitionType ) {
				if(transitionType) {
                    // 设置transition动画
					this._transitionTimingFunction(easing.style);
                    // 设置transition duration time
					this._transitionTime(time);
				}
            // 设置translate，开始动画
			this._translate(x, y);
		} else {
            // 使用requestAnimateFrame进行动画
			this._animate(x, y, time, easing.fn);
		}
	}

**_animate** 不使用transition实现动画

    /**
     * 不使用transition实现动画
     * @method
     * @param  {[type]} destX    [最终的X轴坐标]
     * @param  {[type]} destY    [最终的Y轴坐标]
     * @param  {[type]} duration [动画用时]
     * @param  {[type]} easingFn [缓动函数]
     * @return {[type]}          [description]
     */
    function (destX, destY, duration, easingFn) {
		var that = this,
			startX = this.x,
			startY = this.y,
			startTime = utils.getTime(),
			destTime = startTime + duration;

		function step () {
			var now = utils.getTime(),
				newX, newY,
				easing;

			if ( now >= destTime ) {
                // 超过动画时间
				that.isAnimating = false;
                // 增加最后一帧
				that._translate(destX, destY);

                // 判断是否需要回弹，并更新数据
				if ( !that.resetPosition(that.options.bounceTime) ) {
                    // 不需要回弹，触发scrollEnd事件
					that._execEvent('scrollEnd');
				}

				return;
			}

            // 利用缓动函数更新滚动位置
			now = ( now - startTime ) / duration;
			easing = easingFn(now);
			newX = ( destX - startX ) * easing + startX;
			newY = ( destY - startY ) * easing + startY;
			that._translate(newX, newY);

			if ( that.isAnimating ) {
				rAF(step);
			}
		}

        // 设置标志位，跟isInTransition类似
		this.isAnimating = true;
		step();
	}

**_transitionEnd** iscroll自己的transitionEnd事件回调

    function (e) {
        // 如果该事件的target不是iscroll实例的scroller，或者iscroll不在transition动画中，直接返回
		if ( e.target != this.scroller || !this.isInTransition ) {
			return;
		}
        // 重置transitionDuration
		this._transitionTime();
        // 判断是否需要回弹，如果不需要，直接触发scrollEnd
		if ( !this.resetPosition(this.options.bounceTime) ) {
			this.isInTransition = false;
			this._execEvent('scrollEnd');
		}
	}

**_transitionTime** 设置transitionDuration css属性

    function (time) {
		time = time || 0;

        // 获取transitionDuration属性名
		var durationProp = utils.style.transitionDuration;
        // 设置属性
		this.scrollerStyle[durationProp] = time + 'ms';

        // 如果time为0，在badAndroid下需要特殊处理
		if ( !time && utils.isBadAndroid ) {
			this.scrollerStyle[durationProp] = '0.0001ms';
			// remove 0.0001ms
			var self = this;
			rAF(function() {
				if(self.scrollerStyle[durationProp] === '0.0001ms') {
					self.scrollerStyle[durationProp] = '0s';
				}
			});
		}

        // 如果有滚动条
		if ( this.indicators ) {
            // 设置滚动条的transitionDuration属性
			for ( var i = this.indicators.length; i--; ) {
				this.indicators[i].transitionTime(time);
			}
		}


        // INSERT POINT: _transitionTime

	}

**_transitionTimingFunction** 设置transitionTimingFunction css属性，用以指定动画函数

    function (easing) {
        // 设置缓动函数
		this.scrollerStyle[utils.style.transitionTimingFunction] = easing;


		if ( this.indicators ) {
            // 如果有滚动条，设置滚动条的动画函数
			for ( var i = this.indicators.length; i--; ) {
				this.indicators[i].transitionTimingFunction(easing);
			}
		}


        // INSERT POINT: _transitionTimingFunction

	}

**_translate** 设置translate属性

    function (x, y) {
		if ( this.options.useTransform ) {
            // 如果可用transform，设置transform属性

            /* REPLACE START: _translate */

            // 加上translateZ，启用硬件加速
			this.scrollerStyle[utils.style.transform] = 'translate(' + x + 'px,' + y + 'px)' + this.translateZ;

            /* REPLACE END: _translate */

		} else {
            // 否则使用left,top值控制滚动
			x = Math.round(x);
			y = Math.round(y);
			this.scrollerStyle.left = x + 'px';
			this.scrollerStyle.top = y + 'px';
		}
        // 更新X,Y轴坐标
		this.x = x;
		this.y = y;


    	if ( this.indicators ) {
            // 如果有滚动条，依次更新滚动条的位置
    		for ( var i = this.indicators.length; i--; ) {
    			this.indicators[i].updatePosition();
    		}
    	}


        // INSERT POINT: _translate

	}

**enable** 设置标志位

    function () {
        this.enabled = true;
    }


**destroy** 销毁事件，取消resize事件回调，触发destroy事件

    function () {
        this._initEvents(true);
        clearTimeout(this.resizeTimeout);
        this.resizeTimeout = null;
        this._execEvent('destroy');
    }

**getComputedPosition** 获取translateX translateY 或者top left值

    function () {
		var matrix = window.getComputedStyle(this.scroller, null),
			x, y;

		if ( this.options.useTransform ) {
			matrix = matrix[utils.style.transform].split(')')[0].split(', ');
			x = +(matrix[12] || matrix[4]);
			y = +(matrix[13] || matrix[5]);
		} else {
			x = +matrix.left.replace(/[^-\d.]/g, '');
			y = +matrix.top.replace(/[^-\d.]/g, '');
		}

		return { x: x, y: y };
	}



**_start** touchstart mousedown pointerdown 事件句柄

    function (e) {
		// React to left mouse button only
		if ( utils.eventType[e.type] != 1 ) {
            // 不是touch事件时，只处理左键
		  // for button property
		  // http://unixpapa.com/js/mouse.html
		    var button;
    	    if (!e.which) {
    	      /* IE case */
    	      button = (e.button < 2) ? 0 :
    	               ((e.button == 4) ? 1 : 2);
    	    } else {
    	      /* All others */
    	      button = e.button;
    	    }
			if ( button !== 0 ) {
				return;
			}
		}

        // 如果在一次touch事件没结束的时候再次出发了touchstart，判断跟上次的touchstart是否是同一类事件引起。
        // 如果不是，放弃这次touchstart
		if ( !this.enabled || (this.initiated && utils.eventType[e.type] !== this.initiated) ) {
			return;
		}

        // 是否需要preventDefault
		if ( this.options.preventDefault && !utils.isBadAndroid && !utils.preventDefaultException(e.target, this.options.preventDefaultException) ) {
			e.preventDefault();
		}

		var point = e.touches ? e.touches[0] : e,
			pos;
        // 设置initiated 为该次时间的类型
		this.initiated	= utils.eventType[e.type];
        // 设置标志位
		this.moved		= false;
        // 初始化移动距离
		this.distX		= 0;
		this.distY		= 0;
        // 重置滚动方向
		this.directionX = 0;
		this.directionY = 0;
        // 初始化锁定方向
		this.directionLocked = 0;
        // 初始化开始时间
		this.startTime = utils.getTime();

		if ( this.options.useTransition && this.isInTransition ) {
            // transition滚动中
            // 重置transitionDuration
			this._transitionTime();
            // 重置标志位
			this.isInTransition = false;
			pos = this.getComputedPosition();
            // 设置到目前位置的translate
			this._translate(Math.round(pos.x), Math.round(pos.y));
            // 触发scrollEnd事件
			this._execEvent('scrollEnd');
		} else if ( !this.options.useTransition && this.isAnimating ) {
            // requestAnimationFrame 动画中
			this.isAnimating = false;
			this._execEvent('scrollEnd');
		}

        // 重置滚动开始位置
		this.startX    = this.x;
		this.startY    = this.y;
        // 重置
		this.absStartX = this.x;
		this.absStartY = this.y;
        // 重置点击位置
		this.pointX    = point.pageX;
		this.pointY    = point.pageY;

		this._execEvent('beforeScrollStart');
	}


**_move** touchmove mousemove pointermove事件句柄

    function (e) {
        // iscroll 不可用的时候或者触发start事件的事件类型和move不一致，放弃此次move
		if ( !this.enabled || utils.eventType[e.type] !== this.initiated ) {
			return;
		}
        // 阻止native滚动
		if ( this.options.preventDefault ) {	// increases performance on Android? TODO: check!
			e.preventDefault();
		}

        // 获取移动距离，时间戳
		var point		= e.touches ? e.touches[0] : e,
			deltaX		= point.pageX - this.pointX,
			deltaY		= point.pageY - this.pointY,
			timestamp	= utils.getTime(),
			newX, newY,
			absDistX, absDistY;

        // 更新pointX|Y
		this.pointX		= point.pageX;
		this.pointY		= point.pageY;
        // 增加移动距离
		this.distX		+= deltaX;
		this.distY		+= deltaY;
		absDistX		= Math.abs(this.distX);
		absDistY		= Math.abs(this.distY);

		// We need to move at least 10 pixels for the scrolling to initiate
        // 用于处理end之后再次move导致跳动问题
		if ( timestamp - this.endTime > 300 && (absDistX < 10 && absDistY < 10) ) {
			return;
		}

		// If you are scrolling in one direction lock the other
        // 如果还没有locked，那么判断是否需要locked
		if ( !this.directionLocked && !this.options.freeScroll ) {
			if ( absDistX > absDistY + this.options.directionLockThreshold ) {
                // 只滚动X轴
				this.directionLocked = 'h';		// lock horizontally
			} else if ( absDistY >= absDistX + this.options.directionLockThreshold ) {
                // 只滚动Y轴
				this.directionLocked = 'v';		// lock vertically
			} else {
                // 不locked
				this.directionLocked = 'n';		// no lock
			}
		}

		if ( this.directionLocked == 'h' ) {
            // 只滚动X轴，如果Y轴需要native滚动，那么为了不然native滚动，需要阻止
			if ( this.options.eventPassthrough == 'vertical' ) {
				e.preventDefault();
			} else if ( this.options.eventPassthrough == 'horizontal' ) {
                // 使用native滚动， 重置initiated为false
				this.initiated = false;
				return;
			}
            // Y轴移动距离重置为0
			deltaY = 0;
		} else if ( this.directionLocked == 'v' ) {
			if ( this.options.eventPassthrough == 'horizontal' ) {
				e.preventDefault();
			} else if ( this.options.eventPassthrough == 'vertical' ) {
				this.initiated = false;
				return;
			}

			deltaX = 0;
		}

        // 再次根据配置项重置移动距离
		deltaX = this.hasHorizontalScroll ? deltaX : 0;
		deltaY = this.hasVerticalScroll ? deltaY : 0;

		newX = this.x + deltaX;
		newY = this.y + deltaY;

		// Slow down if outside of the boundaries
		if ( newX > 0 || newX < this.maxScrollX ) {
            // 如果超出scroll范围，有回弹效果则回弹，不能回弹则直接到零界点
			newX = this.options.bounce ? this.x + deltaX / 3 : newX > 0 ? 0 : this.maxScrollX;
		}
		if ( newY > 0 || newY < this.maxScrollY ) {
			newY = this.options.bounce ? this.y + deltaY / 3 : newY > 0 ? 0 : this.maxScrollY;
		}

        // 重置滚动方向
		this.directionX = deltaX > 0 ? -1 : deltaX < 0 ? 1 : 0;
		this.directionY = deltaY > 0 ? -1 : deltaY < 0 ? 1 : 0;

        // 如果还没有开始滚动，触发scrollStart
		if ( !this.moved ) {
			this._execEvent('scrollStart');
		}

        // 设置标志位
		this.moved = true;

        // 滚动到新位置
		this._translate(newX, newY);

        /* REPLACE START: _move */

        // 如果move触发的时间比start触发时间超过300ms
        // 重新设置start时间和startX|Y，用于更准确的计算速度
		if ( timestamp - this.startTime > 300 ) {
			this.startTime = timestamp;
			this.startX = this.x;
			this.startY = this.y;
		}

        /* REPLACE END: _move */

	}

**_end** touchend mouseup pointerup事件句柄

    function (e) {
        // 同move
		if ( !this.enabled || utils.eventType[e.type] !== this.initiated ) {
			return;
		}
        // 同start
		if ( this.options.preventDefault && !utils.preventDefaultException(e.target, this.options.preventDefaultException) ) {
			e.preventDefault();
		}

        // 获取start到end的事件，start到end的距离
		var point = e.changedTouches ? e.changedTouches[0] : e,
			momentumX,
			momentumY,
			duration = utils.getTime() - this.startTime,
			newX = Math.round(this.x),
			newY = Math.round(this.y),
			distanceX = Math.abs(newX - this.startX),
			distanceY = Math.abs(newY - this.startY),
			time = 0,
			easing = '';

        // 重置
		this.isInTransition = 0;
		this.initiated = 0;
		this.endTime = utils.getTime();

		// reset if we are outside of the boundaries
        // 如果需要回弹，直接回弹
		if ( this.resetPosition(this.options.bounceTime) ) {
			return;
		}
        // 滚动到新的位置，使用scrollTo平滑过渡到动量滚动
		this.scrollTo(newX, newY);	// ensures that the last position is rounded

		// we scrolled less than 10 pixels
		if ( !this.moved ) {
            // 没有触发move
			if ( this.options.tap ) {
                // 触发tap
				utils.tap(e, this.options.tap);
			}

			if ( this.options.click ) {
                // 触发click
				utils.click(e);
			}
            // 触发scrollCancel
			this._execEvent('scrollCancel');
			return;
		}
        // 一次快速滑动，触发flick事件
		if ( this._events.flick && duration < 200 && distanceX < 100 && distanceY < 100 ) {
			this._execEvent('flick');
			return;
		}

		// start momentum animation if needed
		if ( this.options.momentum && duration < 300 ) {
            // 使用300ms做阈值是为了处理move和end事件触发间隔过长，此时不应该动量滚动
			momentumX = this.hasHorizontalScroll ? utils.momentum(this.x, this.startX, duration, this.maxScrollX, this.options.bounce ? this.wrapperWidth : 0, this.options.deceleration) : { destination: newX, duration: 0 };
			momentumY = this.hasVerticalScroll ? utils.momentum(this.y, this.startY, duration, this.maxScrollY, this.options.bounce ? this.wrapperHeight : 0, this.options.deceleration) : { destination: newY, duration: 0 };
			newX = momentumX.destination;
			newY = momentumY.destination;
			time = Math.max(momentumX.duration, momentumY.duration);
			this.isInTransition = 1;
		}

        // 使用分页的时候的处理
		if ( this.options.snap ) {
			var snap = this._nearestSnap(newX, newY);
			this.currentPage = snap;
			time = this.options.snapSpeed || Math.max(
					Math.max(
						Math.min(Math.abs(newX - snap.x), 1000),
						Math.min(Math.abs(newY - snap.y), 1000)
					), 300);
			newX = snap.x;
			newY = snap.y;

			this.directionX = 0;
			this.directionY = 0;
			easing = this.options.bounceEasing;
		}

        // INSERT POINT: _end
        // 动量滚动
		if ( newX != this.x || newY != this.y ) {
			// change easing function when scroller goes out of the boundaries
			if ( newX > 0 || newX < this.maxScrollX || newY > 0 || newY < this.maxScrollY ) {
				easing = utils.ease.quadratic;
			}

			this.scrollTo(newX, newY, time, easing);
			return;
		}

		this._execEvent('scrollEnd');
	}
