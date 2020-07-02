---
title: "重排与重绘"
date: 2015-09-01
thumbnail: https://images.unsplash.com/photo-1565100474581-da5b25b235c0?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2100&q=80
categories: FrontEnd
toc: true
tags:
  - DOM
---

浏览器下载完页面中的组件后，会解析并生成两个内部数据结构－－DOM 树(表示页面结构)、渲染树(表示 DOM 节点如何显示)。每当 DOM 元素的几何属性发生变化时（宽高发生变化、内容改编、位置发生变化等等），浏览器就需要重新计算变化元素和受影响的其他元素的几何属性。浏览器会使渲染树中受影响的部分失效，并重新构建渲染树，这个过程叫重排。重排完成后，浏览器会根据渲染树，把受影响的部分重新绘制到屏幕上，这个过程叫重绘。

重排和重绘都是相当消耗性能的操作，它们会导致 Web 页面 UI 反应迟钝。应当尽量减少重排和重绘。

<!--more-->

## 何时重排

1. 页面渲染器初始化的时候
2. 浏览器窗口发生尺寸变化时
3. 删除或添加新的可见 DOM 元素
4. 元素的位置改变
5. 元素的盒子模型发生变化
6. 元素内容发生变化（引起盒子模型发生变化）

## 减少重排和重绘

每次重绘和重排都会产生大量消耗，我们在编码时应当避免。

### 渲染树队列与刷新

大多数浏览器都会优化这个队列，通过批量执行等方式来优化重排过程，然而你可能会不经意间强制要求浏览器放弃优化，立即执行队列并刷新。

调用下面这些方法时，浏览器为了能返回正确的数据，会立即触发重排：

```javascript
offsetTop, offsetLeft, offsetWidth, offsetHeight;
scrollTop, scrollLeft, scrollWidth, scrollHeight;
clientTop, clientLift, clientWidth, clientHeight;
getComputedStyle();
```

### 优化代码

考虑这个例子：

```javascript
var el = document.getElementById("header");
el.style.borderLeft = "1px";
el.style.borderRight = "2px";
el.style.margin = "3px";
```

一般情况下，浏览器会优化这个，只发生一次重排。但是，如果在这期间有其他代码请求布局信息，会导致浏览器三次重排。一个更高效的方法就是，使用 cssText，只修改一次 DOM：

```javascript
var el = document.getElementById("header");
el.style.cssText += "border-left:1px;border-right:2px;margin:3px;";
```

另一个方法是修改元素的 class 名称，这种方法更清晰，更易维护，但是会带来轻微的性能影响，因为改变类时需要检查及联样式。

```javascript
var el = document.getElementById("header");
el.className = "active";
```

### 批量修改 DOM

当需要对 DOM 元素进行一连串的操作时，可以先让元素脱离文档流，然后对其应用多重改变，再把元素加入文档流。第一步和第三步会触发两次重排，如果不这么做，第二步做的任何修改都会触发一次重排。

那么怎么才能让元素脱离文档流哪？ 有三种基本方法：

1. 隐藏元素，应用修改，重新显示
2. 使用文档片段 (document fragment) 在当前 DOM 之外构建一个子树，再把它拷贝回文档
3. 创建一个需要修改节点的镜像，然后修改镜像，再用镜像替换原节点

比如我们需要向列表中添加一些项，如下

```
<ul id="test">
    <li><a href="url1">Name1</a></li>
    <li><a href="url2">Name2</a></li>
<ul>
var data = [
    {url: "url1", name: "Name1"},
    {url: "url2", name: "Name2"}
]
```

我们写一个添加的函数

```javascript
function appendDataToElement(to, data) {
  var link, li, item;
  for (var i = 0, len = data.length; i < len; i++) {
    item = data[i];
    link = document.createElement("a");
    link.href = item.url;
    link.appendChild(document.createTextNode(item.name));
    li = document.createElement("li");
    li.appendChild(link);
    to.appendChild(li);
  }
}

var ul = document.getElementById("test");
appendDataToElement(ul, data);
```

如果使用这种方法， data 数组中的每一个新条目被增加到 DOM 树中都会导致重绘，我们可以此用第一种方法，改变需要重排元素的 display 属性，从而使其脱离文档流，添加完成后，在修改 display 属性，使其添加回文档流中，代码如下：

```javascript
var ul = document.getElementById("test");
ul.style.display = "none";
appendDataToElementById(ul, data);
ul.style.display = "block";
```

第二种减少重排的方法就是使用文档片段，代码如下：

```javascript
var fragment = document.createDocumentFragment();
appendDataToElement(fragment, data);
var ul = document.getElementById("test");
ul.appendChild(fragment);
```

第三种方式，需要为原节点创建一个镜像，添加完成后，再替换原来的 DOM 节点，代码如下：

```javascript
var ul = document.getElementById("test");
var cloneUl = ul.cloneNode(true);
appendDataToElement(cloneUl, data);
ul.parentNode.replaceChild(cloneUl, ul);
```

推荐使用第二种方案，它操作 DOM 的次数最少。

### 缓存布局信息

我们在用定时器写动画的时候，经常会出现这种写法：

```javascript
var es = el.style;
es.left = 1 + el.offsetLeft + "px";
es.top = 1 + el.offsetTop + "px";
```

前面说过，这种写法会使浏览器强制重排获取正确的属性值，每次 ＋ 1 都会触发重排，我们可以将移动信息做缓存，如下

```javascript
var es = el.style,
  positionNum = el.offsetLeft;
es.left = ++positionNum;
```

这样，我们就避免了，多次重排!
