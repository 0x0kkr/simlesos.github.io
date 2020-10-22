---
title: "事件委托"
date: 2015-08-09
thumbnail: https://images.unsplash.com/photo-1551803920-92b1e354bfa6?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2110&q=80
categories: FrontEnd
toc: true
tags:
  - DOM
---

我们经常会遇到给一堆元素绑定事件的情况，比如说给每个 li 中的 a 元素添加 onclik 事件，每一个 a 都要绑定多次事件处理器，这会影响页面性能，每绑定一个页面处理器都是有代价的，访问的 DOM 元素越多，应用程序就越慢。特别是事件绑定通常发生在 onload 时，此时对每一个富交互应用的网页来说都是一个拥堵的时刻。事件绑定占用了处理事件，而且，浏览器需要跟踪每个事件处理器，这也会占用更多的内存。

一种简单的 DOM 事件处理技术就是事件委托。

<!--more-->

根据 DOM 标准，每个事件都要经历三个阶段

1. 捕获（IE 不支持）
2. 到达目标
3. 冒泡

对事件委托来说，冒泡就够了，比如说我们想要给下面这段代码中每个 a 标签增加一个点击事件

```html
<ul id="links">
  <li><a href="#">item1</a></li>
  <li><a href="#">item2</a></li>
  <li><a href="#">item3</a></li>
  <li><a href="#">item4</a></li>
  <li><a href="#">item5</a></li>
</ul>
```

可以把 onclick 事件绑定在 ul 元素上，然后通过 event.target 获得当前冒泡事件对象，再根据业务逻辑判断是否是目标元素的点击事件

```javascript
var list = document.getElementById("links");
var handler = function (e) {
  e = e || window.event;
  e.preventDefault();
  var target = e.target || e.srcElement;
  if (target.nodeName !== "A") return;
  console.log(target.nodeName + ": " + target.textContent);
};
if (list.addEventListener) {
  list.addEventListener("click", handler, false);
} else if (list.attachEvent) {
  list.attachEvent("onclick", handler);
} else {
  list["onclick"] = handler;
}
```

上面这段代码可以很好的运行，但是假如我们想要的不是 a 标签，而是 li 标签，尝试修改其中这段代码：

```javascript
if (target.nodeName !== "LI") return;
```

运行发现并没有得到想要的结果。如果把刚刚修改的那行代码去掉，再次执行，每次点击 li，还是弹出的 a 标签的信息。

我们这里解释下，因为 Event.target 指向的是事件处理程序当前正在处理的那个元素，事件在冒泡阶段经历的过程是 a &gt; li &gt; ul ，所以刚开始 Event.target 指向的就是 a，之后由于 li 上并没有注册事件处理程序，结果 click 事件就一路冒泡到了 ul 上了，搞明白这个后，咱们把程序修改下：

```javascript
var list = document.getElementById("links");
var handler = function (e) {
  e = e || window.event;
  e.preventDefault();
  var target = e.target || e.srcElement,
    currentTarget = e.currentTarget; //e.currentTarget 是指向绑定事件处理器的对象，这里就是 ul 元素
  while (target !== currentTarget) {
    if (target.nodeName === "LI") {
      console.log(target.nodeName + ": " + target.textContent);
    }
    target = target.parentNode; //如果不是目标元素，就让 target 等于父元素，再次检查
  }
};
if (list.addEventListener) {
  list.addEventListener("click", handler, false);
} else if (list.attachEvent) {
  list.attachEvent("onclick", handler);
} else {
  list["onclick"] = handler;
}
```

这次终于可以了，最后依照惯例，我们重构下，提取公共函数：

```javascript
var EventUtil = {
  addHandler: function (element, type, handler) {
    if (window.addEventListener) {
      this.addHandler = function (element, type, handler) {
        element.addEventListener(type, handler, false);
      };
    } else if (window.attchEvent) {
      this.addHandler = function (element, type, handler) {
        element.attchEvent("on" + type, handler);
      };
    }
    this.addHandler(element, type, handler);
  },

  removeHandler: function (element, type, handler) {
    if (window.removeEventListener) {
      this.removeHandler = function (element, type, handler) {
        element.removeEventListener(type, handler, false);
      };
    } else if (window.detachEvent) {
      this.removeHandler = function (element, type, handler) {
        element.detachEvent("on" + type, handler);
      };
    }
    this.removeHandler(element, type, handler);
  },

  getEvent: function (e) {
    return e || window.event;
  },

  getTarget: function (e) {
    return e.target || e.srcElement;
  },

  preventDefault: function (e) {
    if (e.preventDefault) e.preventDefault();
    else event.returnValue = false;
  },

  stopPropagation: function (e) {
    if (e.stopPropagation) e.stopPropagation();
    else e.cancelBubble = true;
  },
};
/**
 * 事件委托
 * @param  {[DOM]}   element  [事件绑定元素]
 * @param  {[String]}   type     [绑定事件类型，不加on]
 * @param  {[String]}   nodeName [需要触发的子元素]
 * @param  {Function} fn       [子元素事件函数]
 * @return {[Function]}            [事件handler,用来注销事件]
 */
var delegation = function (element, type, nodeName, fn) {
  var handler = function (e) {
    var event = EventUtil.getEvent(e),
      target = EventUtil.getTarget(event),
      currentTarget = event.currentTarget || element;

    EventUtil.preventDefault(event);

    while (target !== currentTarget) {
      if (nodeName.toUpperCase() === target.nodeName) {
        fn(target);
        EventUtil.stopPropagation(event);
        break;
      }
      target = target.parentNode;
    }
  };
  EventUtil.addHandler(element, type, handler);
  return handler;
};
```

使用起来，直接调用

```javascript
var ul = document.querySelector("#links");
var handler = delegation(ul, "click", "li", function (target) {
  console.log(target.nodeName);
});

//EventUtil.removeHandler(ul, "click", handler)   //注销事件
```

是不是看起来舒服多了那:-P
