title: 基于 jQuery Ajax 定制自己喜爱的 API
date: 2014-10-17 23:37:55
category: 开发
---

![封面](http://ww1.sinaimg.cn/large/b196a42dgw1f2t2s0fkzoj20fu0643yd.jpg)

你觉得 jQuery的 Ajax API 怎么样，想不想定制更适合自己的丝滑 API。

----

编写常规的 AJAX 代码并不容易，因为不同的浏览器对 AJAX 的实现并不相同。不过 jQuery 已经帮忙处理掉了这些兼容，但代码写多了还是觉得不爽，因为它不能给我以下的体验:
1. 更简单的链式 API；
2. 页面中的所有请求要附加一些公共参数；
3. 不能统一处理服务返回的自定义错误；

## 新的 API
 不爱动手的程序员不是好攻城狮。😄  因此，我基于 jQuery，重新封装了一层。
### 这是一个例子

```javascript
//附加公共参数
$http.global("token","abcd321"); 
$http.use("/api",  (res) =>  {
    if (res.success) {
        return true;
    }
    var err = res.error;
    if (err.code === 1001) {
        location.href = "/login.html";
    } else {
        alert(err.message);
    }
    return false;
});

$http.get("/api/user").type("json")
.param("id", 110) .send( (res) =>  { 
       // 过滤器处理完才会执行到这一步
       var datum = res.datum;
       alert("Name: " + datum.name);
       console.log(res.datum);
    }, (err) => {
       alert("查询失败了额");
       console.error(err);
     });
```

### 这是详细的API说明
#### 设置公共参数；
* `$http.global(key, value);`
#### 设置过滤器；
* `$http.use(urlPrefix, option [optinal], callback);`
#### 设置请求类型；
* `$http.get(url);`
* `$http.getJson(url);`
* `$http.post(url);`
* `$http.put(url);`
* `$http.delete(url);`
#### 增加请求参数；
* `$http.param(key, value);`
* `$http.params(paramJson);`
#### 设置请求头；
* `$http.header(key, value);`
* `$http.headers(paramJson);`
#### 其它扩展选项；
* `$http.cache(boolean);`
* `$http.type("json/xml/html");`
* `$http.async(boolean);`
* `$http.timeout(number);`
#### 发送请求并处理回调；
* `$http.send(options [optional], successCallback, errorCallback);`   

### Gist 地址
 https://gist.github.com/lenbo-ma/0213bfea8e3f08ff8d66

----

长按二维码，关注我。

内容大多会是后端技术、前端工程、DevOps，偶尔会有一些大数据相关，会推荐一些好玩的东西。希望你会喜欢~

![长按二维码关注我](http://ww4.sinaimg.cn/large/b196a42dgw1f2r0uqcno4j209k09kwef.jpg)

一切，源于喜欢。