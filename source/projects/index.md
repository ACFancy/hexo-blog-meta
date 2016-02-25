title: 我的项目
date: 2014-10-17 23:18:41
comment: true
description: 我的个人作品和做过的一些开源项目。
---

### gso
取名自谷搜(谷歌搜索)谐音。顾名思义，这是一个在线谷歌搜索服务,使用Node.js编写。[查看详情](/aboutgso/) | [查看主页](http://gso.mlongbo.com/)

*****

### markdown在线编辑服务
这是一个免费的在线Markdown文档编辑服务，前端采用Editor.md作为编辑器，同时使用了bootstrap和backbone开源项目，后端使用NodeJs搭建。[查看详情](http://md.mlongbo.com/about.html) | [查看主页](http://md.mlongbo.com)

*****

### sunflow-ioc
这是一个使用Java编写的基于注解的ioc工具, 为什么要重复造轮子呢, 只是为了好玩。[查看主页](http://ioc.mlongbo.com/)

*****

### jfinal-api-scaffold
实际上这个项目更像一个脚手架，使用Java编写，是我多次开发HTTP API应用的经验总结。其中包含了常用的模块（如账户相关，版本更新等），以及本人认为比较好的开发方式和规范。目前已做好了一些基础的东西(如配置，响应规范等)和公共的模块，主要应用于APP(Android,IOS,WebApp)接口快速开发。[查看主页](https://github.com/lenbo-ma/jfinal-api-scaffold)

*****

### actvitystream

引用维基百科对Activity Streams协议的介绍

> **Activity Streams** is an open format specification for activity stream protocols, which are used to syndicate activities taken in social web applications and services, similar to those in Facebook[1]'s Newsfeed, FriendFeed, the Movable Type Action Streams plugin, etc.

一个活动流是指用户最近执行地一系列的活动。比如在豆瓣发了一篇日记、一个广播等等都是一个用户活动。另一个比较典型的应用场景是QQ空间，当你特别关注了某一位好友，在该好友发了一条说说或照片，你都将收到消息提醒。

因此，这个项目是对此协议的部分实现。服务器端使用NodeJs+Mongodb，之所以选用这两项技术主要是看中了JSON的灵活性，以及NodeJs开发Restful API的快速，同时还可以使用socket.io这项技术实现实时消息提醒功能。

客户端使用Java，用于生成符合协议的json内容。目前只做了部分实现，一般是应用于业务系统。| [查看主页](https://github.com/lenbo-ma/actvitystream)