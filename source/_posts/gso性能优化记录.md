title: gso性能优化记录
date: 2014-10-17 23:38:09
category: 开发
description: 在gso上线后，使用了哪些优化的手段不断提升响应速度。
---

自gso发布上线，之后的一段时间内因为忙于工作的事情，没有再更新维护。

当初一时兴起，只用了很短的时间便完成了代码编写及上线部署。一直想着要再优化整理下代码，最近也一直在努力做这些事情。

直到今日，算是告一段落。接下来，我从几点简单说一下都做了哪些事情。

### CDN

不得不说，七牛的服务确实很赞，他们提供的免费目前已足够gso使用。
我主要是将css，图片放到了七牛上。gso中使用的js并不多，所以没有做缓存。

### 压缩

包括css压缩，以及使用node compression模块进行压缩(压缩效果正在测试中..)。

### HTML解析

对google查询结果的解析使用了node-jquery模块。优化之后速度提高了将近900ms, 主要改进了以下方面:
<!-- more -->
1. 改进选择器使用。 jquery中最快的选择器是: id选择器和标签选择器，因为jquery内部会调用js的原生方法(比如`getElementById()`)。
其次是: class选择器，同理，chrome中有原生`getElementByClassName()`, 所以速度也不会太慢，而我之前一直使用的是class属性选择器....
最慢的选择器当属: 伪类选择器和属性选择器, 比如`$(':hidden')`和`$('[attribute=value]')`。

2. 做好缓存。使用选择器的次数应该越少越好，并且尽可能缓存选中的结果，便于反复使用.

3. 正确使用循环。javascript原生的for和while，要比jquery的.each方法快。另外，你可以使用node运行一下下面的代码: 

```javascript
var arr = [];
while(arr.length < 5000000){
    arr.push(arr.length);
}

console.time("for in");
for(var i in arr) {
}
console.timeEnd("for in");
 
console.time("i++"); 
var length = arr.length;
for(var i=0; i<length; i++) {
}
console.timeEnd("i++"); 
 
console.time("i--");
var length = arr.length;
for(var i=length; i--;) {
}
console.timeEnd("i--");
 
console.time("map");
arr.map(function(){});
console.timeEnd("map");
 
console.time("filter");
arr.filter(function(){});
console.timeEnd("filter");
 
console.time("forEach");
arr.forEach(function(){});
console.timeEnd("forEach");
 
console.time("some");
arr.some(function(){});
console.timeEnd("some");
 
console.time("every");
arr.every(function(){return true;});
console.timeEnd("every");
```

---

微信扫描二维码，关注我。

我会写一些是后端技术、前端工程、DevOps相关的文章，偶尔会有一些大数据相关，也会推荐一些好玩的东西。希望你会喜欢~

![长按二维码关注我](http://ww4.sinaimg.cn/large/b196a42dgw1f2r0uqcno4j209k09kwef.jpg)

一切，源于喜欢。
