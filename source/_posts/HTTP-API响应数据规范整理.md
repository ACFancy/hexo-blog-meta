title: HTTP API响应数据规范整理
date: 2014-10-17 23:24:50
category: 架构杂谈
description: 本文档为本人对长期开发API接口所整理的经验总结，目的为规范服务器端API接口，便于服务器端与客户端代码重用。
---

## 概述

本文档为本人对长期开发API接口所整理的经验总结，如有不完善或不合理的地方，望各位多提意见。

文档目的为规范服务器端API接口，便于服务器端与客户端代码重用。服务器端和客户端可根据实际所定义规范编写序列化和反序列化工具，以便减少一些开发时间。

本文档为个人观点，仅供参考。

## HTTP接口

### Execute(CUD)

用于client向server发起的POST、PUT和DELETE请求

#### JSON

##### 参考

```javascript
{
    "code": "value",   //结果码，必需。客户端应首先根据此项结果进行相应处理。
    "message": "value"    //文本消息提示。
}
```

#### XML格式

##### 参考

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code></code>
    <message></message>
</result>
```

<!-- more -->
### Query(R)

#### 单项数据查询

##### JSON

###### 参考

```javascript
{
    "code": "value",    //响应结果码，必需；客户端可根据此结果判断数据查询是否正常
    "datum": {
     }
}
```

###### 示例

```javascript
{
    "code": 1, 
    "datum": {
         "id":3,
         "name":"jennifer"
         "age": 23,
         "sex": "female"
     }
}
```

##### Xml格式

###### 参考

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code></code>
    <datum>
        <id></id>
        <name></name>
        <age></age>
        <sex></sex>
    </datum>
</result>
```

###### 示例

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code>1</code>
    <datum>
        <id>3</id>
        <name>jennifer</name>
        <age>23</age>
        <sex>female</sex>
    </datum>
</result>
```

#### 列表数据查询

##### Json格式

###### 参考

```javascript
{
    "code": "value", //响应结果码，必需。
    "data": [{    //数据列表，数组
     },{
     }]
}
```

###### 示例

```javascript
{
    "code": 1,
    "data": [{
         "id":3,
         "name":"jennifer"
         "age": 23,
         "sex": "female"
     },{
         "id":5,
         "name":"lenbo"
         "age": 21,
         "sex": "male"
     }]
}
```

##### Xml格式

###### 参考

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code></code>
    <data>
        <datum>
        </datum>
        <datum>
        </datum>
    </data>
</result>
```

###### 示例

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code>1</code>
    <data>
        <datum>
            <id>5</id>
            <name>lenbo</name>
            <age>21</age>
            <sex>male</sex>
        </datum>
    </data>
</result>
```

#### 分页数据查询

##### Json格式

###### 参考

```javascript
{
    "code": 1,    //响应结果码，必需。
    "page": {    //分页参数，必需。
        "totalpage": "value", //总页数
        "count":"value",    //每页记录条数
        "curr": "value",    //本页页码
        "totalcount": "value" //总记录数
    },
    "data": [{    //列表数据，数组。
     },{
     },{
     }]
}
```

###### 示例

```javascript
{
    "code": 1,
    "page": {
        "totalpage": 20, //总页数
        "count":10,    //每页记录条数
        "curr":1,    //本页页码
        "totalcount": 180 //总记录数
    },
    "data": [{
         "id":3,
         "name":"jennifer"
         "age": 23,
         "sex": "female"
     },{
         "id":5,
         "name":"lenbo"
         "age": 21,
         "sex": "male"
     }]
}
```

##### Xml格式

###### 参考

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code></code>
    <page>
        <totalpage></totalpage>
        <count></count>
        <curr></curr>
        <totalcount></totalcount>
    </page>
    <data>
        <datum>
        </datum>
        <datum>
        </datum>
    </data>
</result>
```

###### 示例

```xml
<?xml version="1.0" encoding="utf-8"?> 
<result>
    <code>1</code>
    <page>
        <totalpage>20</totalpage>
        <count>10</count>
        <curr>2</curr>
        <totalcount>180</totalcount>
    </page>
    <data>
        <datum>
            <id>5</id>
            <name>lenbo</name>
            <age>21</age>
            <sex>male</sex>
        </datum>
        <datum>
            <id>2</id>
            <name>jennifer</name>
            <age>23</age>
            <sex>female</sex>
        </datum>
    </data>
</result>
```

## 附录1：code对照表参考
仅供参考，API接口开发人员可根据实际情况自定义相应结果码或节点

- 200 ok  - 成功状态，对应，GET,PUT,PATCH,DELETE.
- 500 faild - 失败状态
- 304 not modified   - HTTP缓存有效。
- 400 bad request   - 请求格式错误。可以标识参数错误或参数缺失
- 401 unauthorized   - 未授权。
- 403 forbidden   - 鉴权成功，但是该用户没有权限。
- 404 not found - 请求的资源或接口不存在
- 405 method not allowed - 该http方法不被允许。
- 410 gone - 这个url对应的资源现在不可用。
- 415 unsupported media type - 请求类型错误。
- 422 unprocessable entity - 校验错误时用。
- 429 too many request - 请求过多。


---

微信扫描二维码，关注我。

我会写一些是后端技术、前端工程、DevOps相关的文章，偶尔会有一些大数据相关，也会推荐一些好玩的东西。希望你会喜欢~

![长按二维码关注我](http://ww4.sinaimg.cn/large/b196a42dgw1f2r0uqcno4j209k09kwef.jpg)

一切，源于喜欢。
