title: 【HTML5】Canvas 内部元素添加事件处理
date: 2016-12-06 09:58:34
tags: HTML5
categories: HTML5
---
## 前言
canvas 没有提供为其内部元素添加事件监听的方法，因此如果要使 canvas 内的元素能够响应事件，需要自己动手实现。实现方法也很简单，首先获得鼠标在 canvas 上的坐标，计算当前坐标在哪些元素内部，然后对元素进行相应的操作。配合自定义事件，我们就可以实现为 canvas 内的元素添加事件监听的效果。

[源码](https://github.com/zhangjikai/canvas-element-event) &nbsp;&nbsp;&nbsp;[演示](http://zhangjikai.com/canvas-event/)
<!-- more -->
## 自定义事件
为了实现javascript对象的自定义事件，我们可以创建一个管理事件的对象，该对象中包含一个内部对象（当作map使用，事件名作为属性名，事件处理函数作为属性值，因为可能有个多个事件处理函数，所以使用数组存储事件处理函数），存储相关的事件。然后提供一个激发事件的函数，通过使用 `call` 方法来调用之前绑定的函数。下面是代码示例：
```js
(function () {
    cce.EventTarget = function () {

        this._listeners = {};
        this.inBounds = false;

    };

    cce.EventTarget.prototype = {
        constructor: cce.EventTarget,

        // 查看某个事件是否有监听
        hasListener: function (type) {
            if (this._listeners.hasOwnProperty(type)) {
                return true;
            } else {
                return false;
            }
        },

        // 为事件添加监听函数
        addListener: function (type, listener) {
            if (!this._listeners.hasOwnProperty(type)) {
                this._listeners[type] = [];
            }

            this._listeners[type].push(listener);
            cce.EventManager.addTarget(type, this);
        },

        // 触发事件
        fire: function (type, event) {
            if (event == null || event.type == null) {
                return;
            }

            if (this._listeners[event.type] instanceof Array) {
                var listeners = this._listeners[event.type];
                for (var i = 0, len = listeners.length; i < len; i++) {
                    listeners[i].call(this, event);
                }
            }
        },

        // 如果listener 为null，则清除当前事件下的全部事件监听
        removeListener: function (type, listener) {
            if (listener == null) {
                if (this._listeners.hasOwnProperty(type)) {
                    this._listeners[type] = [];
                    cce.EventManager.removeTarget(type, this);
                }
            }
            if (this._listeners[type] instanceof Array) {
                var listeners = this._listeners[type];
                for (var i = 0, len = listeners.length; i < len; i++) {
                    if (listeners[i] === listener) {
                        listeners.splice(i, 1);
                        if (listeners.length == 0)
                            cce.EventManager.removeTarget(type, this);
                        break;
                    }
                }
            }

        }
    };
}());
```
在上面的代码中，`EventManager` 用来存储所有绑定了事件监听的对象，便于后面判断鼠标是否位于某个对象内部。如果一个自定义对象需要添加事件监听，只需要继承 `EventTarget`。

## 有序数组
在判断触发某个事件的元素时，需要遍历所有绑定了该事件的元素，判断鼠标位置是否位于元素内部。为了减少不必要的比较，这里使用了一个有序数组，使用元素区域的最小 x 值作为比较值，按照升序排列。如果一个元素区域的最小 x 值大于鼠标的 x 值，那么就无需比较数组中该元素后面的元素。具体实现可以看 [SortArray.js](https://github.com/zhangjikai/canvas-element-event/tree/master/js)

## 元素父类
这里设计了一个抽象类，来作为所有元素对象的父类，该类继承了 `EventTarget`，并且定义了三个函数，所有子类都应该实现这三个函数。 具体代码如下所示：
```js
(function () {

    // 抽象类，该类继承了事件处理类，所有元素对象应该继承这个类
    // 为了实现对象比较，继承该类时应该同时实现compareTo, comparePointX 以及 hasPoint 方法。
    cce.DisplayObject = function () {
        cce.EventTarget.call(this);
        this.canvas = null;
        this.context = null;

    };

    cce.DisplayObject.prototype = Object.create(cce.EventTarget.prototype);
    cce.DisplayObject.prototype.constructor = cce.DisplayObject;

    // 在有序数组中会根据这个方法的返回结果将对象排序
    cce.DisplayObject.prototype.compareTo = function (target) {
        return null;
    };

    // 比较目标点的x值与当前区域的最小 x 值，结合有序数组使用，如果 point 的 x 小于当前区域的最小 x 值，那么有序数组中剩余
    // 元素的最小 x 值也会大于目标点的 x 值，就可以停止比较。在事件判断时首先使用该函数过滤一下。
    cce.DisplayObject.prototype.comparePointX = function (point) {
        return null;
    };

    // 判断目标点是否在当前区域内
    cce.DisplayObject.prototype.hasPoint = function (point) {
        return false;
    };

}());
```

## 事件判断
以鼠标事件为例，这里我们实现了 `mouseover`, `mousemove`, `mouseout` 三种鼠标事件。首先对 canvas 添加 `mouseover`事件，当鼠标在 canvas 上移动时，会时时对比当前鼠标位置与绑定了上述三种事件的元素的位置，如果满足了触发条件就调用元素的 `fire` 方法触发对应的事件。下面是示例代码：
```js
_handleMouseMove: function (event, container) {

    // 这里传入container 主要是为了使用 _windowToCanvas函数
    var point = container._windowToCanvas(event.clientX, event.clientY);

    // 获得绑定了 mouseover, mousemove, mouseout 事件的元素对象
    var array = cce.EventManager.getTargets("mouse");
    if (array != null) {
        array.search(point);

        // 鼠标所在的元素
        var selectedElements = array.selectedElements;

        // 鼠标不在的元素
        var unSelectedElements = array.unSelectedElements;
        selectedElements.forEach(function (ele) {
            if (ele.hasListener("mousemove")) {
                var event = new cce.Event(point.x, point.y, "mousemove", ele);
                ele.fire("mousemove", event);
            }

            // 之前不在区域内，现在在了，说明鼠标进入了
            if (!ele.inBounds) {
                ele.inBounds = true;
                if (ele.hasListener("mouseover")) {
                    var event = new cce.Event(point.x, point.y, "mouseover", ele);
                    ele.fire("mouseover", event);
                }
            }
        });

        unSelectedElements.forEach(function (ele) {

            // 之前在区域内，现在不在了，说明鼠标离开了
            if (ele.inBounds) {
                ele.inBounds = false;
                if (ele.hasListener("mouseout")) {
                    var event = new cce.Event(point.x, point.y, "mouseout", ele);
                    ele.fire("mouseout", event);
                }
            }
        });
    }
}
```
## 其他
### 立即执行函数
诸如下面形式的函数称之为立即执行函数。
```js
(function() {
    // code
}());
```
使用立即执行函数的好处就是它限定了变量的作用域，使在立即执行函数中定义变量不会污染其他作用域，更加详细的讲解请看[这里](http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html)

### apply, call, bind
这三个函数的使用类似于java 反射中的 `Method.invoke`，方法作为一个主体，将执行方法的对象作为参数传入到方法里。其中 apply 和 call 作用一样，调用后都会立即执行，只是接受参数的形式不同。
```js
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2])
```
而 bind 会返回对应函数，不会立即执行，便于以后调用。 看下面的例子：
```js
function aa() {
    console.log(111);
    console.log(this);
}

var bb = aa.bind(Math);
bb();
```
更加详细的讲解请看[这里](http://www.cnblogs.com/coco1s/p/4833199.html)

### addEventListener 传参
如果给某个元素添加事件监听时需要传递参数，可以使用下面的方法
```js
var i = 1;
aa.addEventListener("click", function() {
    bb(i);
}, false);
```

### 调用父类的构造函数
使用 `call` 即可
```
Child = function() {
    Parent.call(this);
}
```

### 对象检测
* 判断对象为 null 或者 undefined
```js
// `null == undefined` 为true
if (variable == null) {
    // code
}
```
* 判断对象是否有某个属性
```js
if(myObj.hasOwnProperty("<property name>")){
    alert("yes, i have that property");
}
// 或者
if("<property name>" in myObj) {
    alert("yes, i have that property");
}
```

### isPointInPath
canvas中判断点是否在某个路径内部，可以用于多边形的检测。不过 `isPointInPath` 使用路径是最后一次绘制的图形，如果有多个图形需要判断，需要将前面的图形路径保存下来，判断时需要重新构造路径，不过不需要绘制，如下面
```js
this.context.save();
this.context.beginPath();

//console.log(this.points);
this.context.moveTo(this.points[0].x, this.points[0].y);

for (var i = 1; i < this.points.length; i++) {
    this.context.lineTo(this.points[i].x, this.points[i].y);
}

if (this.context.isPointInPath(target.x, target.y)) {
    isIn = true;
}
this.context.closePath();
this.context.restore();
```

---
**参考文章：**
* [Custom events in JavaScript](https://www.nczonline.net/blog/2010/03/09/custom-events-in-javascript/)
* [javascript中constructor的作用](http://www.ghugo.com/javascript-constructor/)
* [【优雅代码】深入浅出 妙用Javascript中apply、call、bind](http://www.cnblogs.com/coco1s/p/4833199.html)
* [深入理解JavaScript系列（4）：立即调用的函数表达式](http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html)
* [Using Super Constructors Is Critical In Prototypal Inheritance In Javascript](https://www.bennadel.com/blog/1566-using-super-constructors-is-critical-in-prototypal-inheritance-in-javascript.htm)
