title: http.js 在Ajax单页面中的应用
date: 2014-10-17 23:37:55
tags: [javascript]
category: 开发
---

## 要解决的问题

1. 简化api使用;
2. 为页面中Ajax请求设置全局拦截器;
3. 页面中所有Ajax请求向服务器发送全局参数;

## 实现

### 简化API

我个人喜欢链式调用，以及简单明了的API设计。因此，我主要在以下几个部分做了简化:

1. 为每种请求类型增加一个函数。如:  `$http.get();` `$http.post();` `$http.del();` `$http.put();`

2. 可以动态设置请求参数; 

 * req.push(key,val); or req.push(jsonObj); //设置请求参数, 可同时增加一个或多个参数
 * req.data(jsonObj); //会覆盖push函数设置过的值
 * req.header(key,val); or req.header(jsonObj); //请求头, 可同时增一个或多个
 * req.headers(jsonObj); //会覆盖header函数设置过的值

3. 同样, 其他的功能也可以拆分成独立函数来设置, 比如: 

 * req.type(typeStr); //设置响应数据类型
 * req.cache(boolean); //是否缓存
 
总之，你完全可以按照你自己最喜爱的方式定制API。

<!-- more -->
### 拦截器实现

之所以会有拦截器的需求，是因为我之前所做的一个应用的数据响应格式如下: 

```javascript
{
  code: 1,
  message: '响应信息'
}
```

其中code值会有这样几种值: 
* 0 failed
* 1 success
* 2 session timeout

因此我需要统一处理会话过期情况，比如通知用户重新登录等。

我使用`req.interceptor(type,func(res){});` 来增加一个拦截器。当服务器响应数据时，将先调用设置的全局拦截器，之后再去调用业务相关的回调函数。如果我http.js去服务器获取html模板代码时，即响应的数据类型是`text/html`，而不是`application/json`，这时候再去做拦截就不好玩了。因此，我用了一个type参数来给interceptor分组。

### 发送全局参数

我做单页面应用时，会有这样一个情景: 

当用户登录后，服务器会返回一个token值，之后的大部分api请求都需要传递这个token参数。因此，如果在每次ajax请求时都手动增加一个token参数值，就感觉有些效率低下。同样，你的同事们可能也会遇到和你同样的情况，这时候还要一个个告诉他们怎么设置token参数就很不好玩儿了。

因此，我在http.js中增加`$http.global(key,val);` or `$http.global(jsonObj);`这样一个函数来解决上面的问题。

## 结语

至于基于什么ajax库来封装，你可以使用原生ajax，jquery ajax，或者其他你喜欢的ajax library。同时，欢迎各种讨论 ^_^

这是我写的[http.js代码片段](https://gist.github.com/lenbo-ma/0213bfea8e3f08ff8d66)，不太全面，也许可供你参考。


