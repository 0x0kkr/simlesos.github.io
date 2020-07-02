---
title: "HTML5新增元素介绍"
date: 2015-11-01
thumbnail: https://images.unsplash.com/photo-1542831371-29b0f74f9713?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2100&q=80
categories: FrontEnd
toc: true
tags:
  - DOM
---

HTML5 新增了一些元素为了用于更好的文档结构，使用这
些标签能让章节和标题特性更精确，使得文档大纲变的可预测，结构更 Semantic！

<!--more-->

### section

`section` 元素表示一般意义上的区域/章节，，比如内容中的一个专题，一般会与`h1`~`h6`等元素搭配使用，以标示文档层级。如果元素内容可以分为几个部分的话，应该使用 `article` 而不是 `section`。不要把 `section` 元素作为一个普通的容器来使用，这种情况下 `div` 元素比他更合适，特别是仅仅用作美化样式的情况下。通常来说 `section` 应该出现在文档的框架中。

### article

`article` 元素表示文档、页面、应用或网站中的独立结构，其意在成为可独立分配的或可复用的结构，它可能是论坛帖子、文章、博客、用户提交评论或者是其它的独立内容项目。当 `article` 元素嵌套使用时，则该元素代表与外层元素有关的文章，例如代表博客评论的 `article` 元素可嵌套在代表博客文章的 `article` 元素中。

### main

`main` 元素呈现了文档 `body` 或应用的主体部分，这部分内容在文档中应该是独一无二的，在一个文档中不能出现一个以上的 `main`,不能是以下元素的继承 `article`,`aside`,`footer`,`header`,`nav`。

### aside

`aside` 元素代表与了一些与页面其他部分关联性不是那么大的内容，比如广告。

### header

`header` 代表了一组介绍性或者导航性质的辅助内容，可能包含标题元素，也可以包含其他元素，像 logo、分节头部、搜索表单等。`header` 不是分节元素，不会引入分节到大纲中。

### footer

`footer` 元素表示最近的一个章节内容或者根节点元素的页脚。一个页脚通常包含该章节坐着、版权数据或者文档相关的链接信息。

### nav

`nav` 代表一个含有多个超链接的区域。

### figure

`figure` 代表一个独立的内容流，常与 `figcaption` 元素一起出现。

### figcaption

`figcaption` 用作 `figure` 的标题，类似于表格的 `caption` 元素。

### template

`template` 元素是一种机制，允许包含加载页面时不渲染，但又可以随后通过 Javascript 实力化的客户端内容，就像我们这样做一样`script type="text/temple"`。

### audio & video

`audio` 和 `video` 是多媒体元素，他们提供相对应的 API 用于开发者定制 UI，同时也提供了出发 UA 展示其默认控件的方式。

### embed

`embed` 元素表示一个插件，展用于插件内容。

### mark

`mark` 元素代表了一个文档中需要标记或者高亮的部分，比如说搜索引擎搜索后的关键词。不要为了语法高亮而去使用 mark，应该使用 `strong` 元素，`mark` 元素表示上下文的关联性。

### progress

`progress` 用来表示一个进度条,代表一个任务的完成进度。

### meter

`meter` 代表了一个度量，比如对磁盘空间的度量。

### time

`time` 表示一个时间。有两个属性,datetime(一个有效的时间格式，表示具体时间),pubdate(一般用于文章发布时间，评论回复时间等)。

### ruby、rt、rp

`ruby`、`rt`、`rp` 代表 Ruby 表达式。

### bdi

`bdi` 代表了一段隔绝于周围元素的双向书写文本格式。

### wbr

`bdi` 代表了可能断行的部分。

### canvas

`canvas` 用于渲染动态位图。

### datalist

`datalist` 与 input 的 list 属性共同使用，可以用于创建下拉选择框控件。

### keygen

`keygen` 代表生成的密钥对。

### output

`output` 代表了一种输出内容。
