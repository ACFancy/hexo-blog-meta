---
title: 善用Git Pages
date: 2016-04-13 08:15:00
category: 闲话
---

你写博客吗？你是前端开发者吗？

----

我大概14年初开始玩儿独立博客，最开始是搭建在自己的 VPS 上，后来开始用 Github Pages 服务，越用越觉得它是个好东西。搭配 Hexo 写完文章，提交就可以了。没那么多麻烦事儿，而且还免费，支持绑定自定义域名。你说到哪里找这么好的服务。

> Websites for you and your projects.

简单来说是托管静态文件的服务。因此，你可以建个人主页、独立博客和项目主页。再比如你是前端攻城狮，那它就是一个极好的作品展示服务嘛。

#### 一个实例

1. 在Github创建一个仓库；
2. `git clone https://github.com/lenbo-ma/hello-world`
3. `cd hello-world`
4. ` echo "Hello World" > index.html`
5. `echo "mlongbo.com" > CNAME`
6. `git add --all`
7. `git commit -m "Initial commit"`
8. `git push -u origin gh-pages`

然后，DNS 要新建一条 CNAME 记录，指向username.github.com（请将username换成你的用户名）。实际上除了Github Pages之外，国内的 coding.net 也提供同样的服务。所以，我通常是把国内的请求解析到coding, 国外的解析到github，这样速度会快些。

----

扫描二维码，关注我。

内容大多会是后端技术、前端工程、DevOps，偶尔会有一些大数据相关，会推荐一些好玩的东西。希望你会喜欢~

![长按二维码关注我](http://ww4.sinaimg.cn/large/b196a42dgw1f2r0uqcno4j209k09kwef.jpg)

一切，源于喜欢。