---
title: 前端工程之CDN文件上传
date: 2016-04-12 08:00:00
category: 前端工程
---

你们是怎么上传静态网页资源的，还是手动处理？有考虑过前端妹子和运维哥们的感受吗

----

很久以前，一个前端妹子写完了代码要上线，扔了一个压缩包过来。我把 HTML 文件传到服务器配置好 Nginx 后，就把其它的静态文件扔 CDN 了。没过多久，妹子来找我，说产品经理刚让改了一个样式，你给更新下吧。我说，好，没问题..  过了大概半小时，妹子又跑过来找我，产品说那个按钮有点点大，又让改了下样式，你再更新下吧。我微微皱了下眉，说好吧，没问题。然后有一次大晚上的，产品又他妈改需求，虽然我很喜欢妹子找我，但经常这样，我也不能愉快地和自己的妹子玩耍了啊。于是，我动起了手....

![grunt-cdn-upyun](http://ww3.sinaimg.cn/large/b196a42dgw1f2s0kmm8osj20ic07c3z6.jpg)

是的，这是一个 Grunt 插件，当使用`grunt build`命令构建完成后，再执行`grunt cdn-upyun`，它可以很迅速的把你生成好的目录中的所有文件扔到CDN 上去。然后我再结合 webhook，当 master 分支代码有提交时，自动执行`grunt build`和`grunt cdn-upyun`命令，很爽有木有~ 😄
后来我的同事城管哥也用同样方式写了一个npm script 版本的一键上传工具，叫 npm-scripts-cdn，不依赖于Grunt，执行`npm run cdn-upyun`就可以轻松搞定。

虽然我写的都是基于upyun的插件，但比如七牛提供了很完善的 NodeJs版SDK，根据自己的需要稍微扩展下就可以了。

我把代码放在了 Github 上，你可以到访问以下地址:
* [grunt-cdn-upyun](https://github.com/xiaoenai/grunt-cdn-upyun)
* [npm-scripts-cdn](https://github.com/xiaoenai/npm-scripts-cdn)

----

扫描二维码，关注我。
内容大多会是后端技术、前端工程、DevOps，偶尔会有一些大数据相关，会推荐一些好玩的东西。希望你会喜欢~

![扫描二维码关注我](http://ww4.sinaimg.cn/large/b196a42dgw1f2r0uqcno4j209k09kwef.jpg)

一切，源于喜欢。