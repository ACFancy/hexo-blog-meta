title: Javascript中变量的有效性判断
date: 2014-12-25 11:12:03
tags: [javascript]
category: 开发
---

javascript语言设计的不严谨，有时候很容易把人给搞晕，比如说这个变量有效性判断。

先举几个例子：

```javascript
var a;

console.log(a == undefined); //true

console.log(a == null); //true

console.log(a === null); //false

console.log(typeof a == 'undefined'); //true

console.log(null == undefined); //true

console.log(null === undefined); //false
```

想要理解为什么得出上面的结果，首先得明白undefined和null在javascript中所表示的不同含义。

这里借用下阮一峰老师博客中的一个结论:

> null和undefined基本是同义的，只有一些细微的差别。

> **null表示"没有对象"，即该处不应该有值。**典型用法是：
> （1） 作为函数的参数，表示该函数的参数不是对象。
> （2） 作为对象原型链的终点。

> **undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义。**典型用法是：
> （1）变量被声明了，但没有赋值时，就等于undefined。
>（2) 调用函数时，应该提供的参数没有提供，该参数等于undefined。
>（3）对象没有赋值的属性，该属性的值为undefined。
>（4）函数没有返回值时，默认返回undefined。

在Java中，if条件中必须使用boolean表达式, 所以使用javascript时，也会习惯性的认为如此。实际上，javascript中null,undefined,0,"",false作为if的条件的时候，都被认为是false.

所以，再判断变量时，可以用以下几种方式: 

1. 精确判断一个变量是否为undefined

```javascript
var a;
console.log(a === undefined); //true
console.log(typeof a === 'undefined'); //true
```

2. 精确判断一个变量是否为null

```javascript
var a = null;
console.log(a === null); //true
```

3. 判断一个变量是否为null或undefined

```javascript
console.log(a == null); 
或
console.log(a == undefined);
```

4. 判断值是否有效, 可以直接在if表达式中使用变量名称, 我常用来判断一个输入框值是否有效(注意: 如果变量值为0, 会被当作无效)

```javascript
var a = 'a';
if (a) {
   console.log('有效');
}
``` 
