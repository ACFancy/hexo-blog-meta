title: 我是如何建立一个Google代理搜索服务(GuSou)的
date: 2014-10-17 23:36:26
tags: [NodeJs,gso]
category: 开发
description: GuSou是本人使用NodeJs编写的在线Google代理搜索服务，这篇文章介绍了原理和实现代码。
---


## 前言

当初要写gso的初衷是因为我女朋友她要写论文查资料，但我又不在她身边没办法给她翻墙用Google，所以就有了写一个Google搜索代理的想法。

如果刚好你也有个VPS，完全可以自己部署一个服务，这样大家就有更多的Google可以用。希望我们能够通过这种方式让更多的人获取知识，并且认识这个世界。

GitHub地址: https://github.com/lenbo-ma/gso.git

## 原理

前提是要一定将其部署在GFW之外，能正常访问Google的地方。gso的实现原理非常简单，分别为以下几步: 

1. 发起查询请求；
2. gso服务器接收到查询关键词;
3. gso向Google发起查询请求;
4. 接收到Google响应的数据;
5. 解析数据并重新渲染响应给用户;

<!-- more -->
## 实现
Node.js版:

```javascript
/**
 * @author Longbo Ma
 */
var request = require('request');
var zlib = require('zlib');
var $ = require('jQuery');

var gsearch = function (options, cb) {
    options = options || {};
    this.start = options.start || 0;
    this.q = options.q;
    this.callback = cb || function(s){};
    this.result = {
        q: this.q,
        start: this.start
    };
    this.config = {
        base_url: "https://www.google.com",
        lang: "zh-CN"
    };
};

gsearch.prototype.parseResponse = function (body) {
    var arr = [];
    var search = $(body).find("#search").find("li[class='g']");
    var resultStats = $(body).find("#resultStats").html() || "";
    search.each(function () {
        var h3 = $(this).find("h3[class='r']");
        var link = h3.find('a');
        var url = link.attr("href");
        if (url && url.indexOf('/search?') != 0) {
            var title = link.html();
            var content = $(this).find("span[class='st']").text();
            var cite = $(this).find("cite").html();
            arr.push({
                title: title, //搜索结果项标题
                url: url, 
                cite: cite, 
                content: content //搜索结果简要说明
            });
        }
    });
    this.result['resultStats'] = resultStats; //“搜索结果用时”文字
    this.result['data'] = arr;
};

gsearch.prototype.render = function (body) {
    this.parseResponse(body);
    this.callback(this.result);
};

/**
	 + 发起查询请求
 */
gsearch.prototype.request = function () {
    var self = this;
    if (!this.q) {
        cb([]);
        return;
    }
    var qs = {
        start: this.start,
        q: this.q,
        hl: this.config.lang
    };
    var headers = {
        'connection': 'keep-alive',
        'accept-encoding': 'gzip,deflate,sdch',
        'accept-language': 'zh-CN,zh;q=0.8,zh-TW;q=0.6',
        'referer': this.config.base_url
    };

    var options = {
        url: this.config.base_url+'/search',
        encoding: null,
        headers: headers,
        qs: qs
    };

    request(options, function (err, res, body) {
        if (!err && res.statusCode==200) {
            var res_encoding  = res.headers['content-encoding'];
            if (res_encoding.indexOf("gzip")  >= 0) { 
                zlib.unzip(body, function(err, buffer) { //gzip解压缩
                    body =  buffer.toString();
                    self.render(body);
                });
            } else {
                self.render(body);
            }
        }
    });
};

module.exports = function (options,cb) {
    new gsearch(options,cb).request();
};
```

使用:

查询第二页关于“java”的搜索结果

```javascript
gsearch({
    q: 'java',
    start: 10
}, function (res){
    console.log(res);
});
```

## 结语

整个实现过程非常简单，甚至会让人觉得没有技术含量。但我认为衡量一个应用的价值并非在于技术含量，而要看它是否有用，是否能够产生价值。

希望我们都能做出很多有意思并且有用的应用出来。

你如果对这个项目感兴趣，可以看看[这里][1]。

## 相关知识

*资料整理自互联网*

以下是一些可能会用到的请求参数:

* q: 查询关键词，必需的参数，其它都是可选;

* start: 显示搜索结果的起始端，缺省为0;

    如果你想直接看搜索结果第2页，让start=10即可，由于Google只显示1000条搜索结果记录，start理论取值范围在0--999之间。

* num: 搜索结果显示条数，取值范围在10--100条之间，缺省设置num=10;

* hl: Google搜索的界面语言,建议设置为zh-CN;

    因为我的vps在日本，起初我没有设置这个参数，当查询中文时，便出现了大量日文搜索结果。
   
    hl=zh-CN简体中文语言界面
    
    hl=zh-TW繁体中文语言界面
    
    hl=en-英文语言界面

* lr: 搜索内容的语言限定，限定只搜索某种语言的网页。缺省为搜索所有网页。

    常用的有:
    
    lr=lang_zh-CN  只搜索简体中文网页 
    
    lr=lang_zh-TW  只搜索繁体中文网页 
    
    lr=lang_zh-CN|lang_zh-TW  搜索所有中文网页
    
    lr=lang_en  只搜索英文网页 

* ie: 查询关键词的编码，缺省设置为utf-8，也就是说请求Google搜索时参数q的值是一段utf-8编码的文字。

* oe: 搜索结果页面的网页编码，缺省设置oe=utf-8




  [1]: https://github.com/lenbo-ma/gso.git
