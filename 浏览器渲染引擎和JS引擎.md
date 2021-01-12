# 主流浏览器内核及JS引擎
学习web前端，浏览器和编辑器是我们的好朋友
所以有必要了解浏览器的一些小知识
比如：什么是主流浏览器

## 主流浏览器
主流浏览器是有一定市场份额且有自己独立研发内核的浏览器
也可以叫原生浏览器
这其中我们大家常见的有：
IE/Edge，Chrome，Safari，Opera，Firefox
至于我们大家也很常用的qq浏览器、uc浏览器等等都是壳浏览器
它们只是把原生浏览器的内核拿过来，精简一下、包装一下，


## 浏览器内核
浏览器内核名字有很多，渲染引擎、排版引擎、解释引擎，英文Rendering Engine
是用来渲染网页内容的，把你的网页代码转化为可见的页面
在早期内核也是包含js引擎的，而现在js引擎越来独立了，可以把它单独提出来
主流浏览器的内核及变化如下（面试题重点，现在网上写的好多都过时了，于是我总结了一下）

## 主流浏览器	内核
|浏览器|内核|
| ------ | ------ | ------ |
|IE -> Edge | trident->EdgeHTML|
|Chrome	| webkit->blink|
|Safari |	webkit|
|Firefox | Gecko|
|Opera | Presto->blink|

> Edge是微软随win10推出的（微软嫌弃IE了）

## 浏览器js引擎
js引擎用来解释执行js代码
当扩展了解一下吧，不用刻意记

|主流浏览器|js引擎|
| ------ | ---- |
|IE -> Edge	| JScript（IE3.0-IE8.0） / Chakra（IE9+之后）|
|Chrome | V8（大名鼎鼎）|
|Safari |	Nitro（4-）|
|Firefox | SpiderMonkey（1.0-3.0）/ TraceMonkey（3.5-3.6）/ JaegerMonkey（4.0-）|
|Opera |	Linear A（4.0-6.1）/ Linear B（7.0-9.2）/ Futhark（9.5-10.2）/ Carakan（10.5-）|

SpiderMonkey是第一款JS引擎，JavaScript之父Brendan Eich在网景的时候写的