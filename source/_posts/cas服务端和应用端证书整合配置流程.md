title: cas服务端和应用端证书整合配置流程
date: 2015-01-09 23:14:09
tags: [java,sso,cas]
category: 开发
---

## 证书的配置
主要分为两大步：服务端生成配置证书，应用端导入证书。
### SSO服务端
1. 生成keystore, 此文件用于tomcat/conf/server.xml中配置及导出证书；
`keytool -genkey -keyalg RSA -alias mlongbosso -dname "cn=passport.mlongbo.com" -keystore /home/ndoc/test/cas/mlongbosso.keystore -storepass 123654`

	说明：指定使用RSA算法，生成别名为mlongbosso的证书，口令为123654，证书的DN为"cn=passport.mlongbo.com" ，这个DN必须同当前主机完整名称一致!!)

2. 导出mlongbosso.crt证书
`keytool -export -alias mlongbosso -file /home/ndoc/test/cas/mlongbosso.crt -keystore /home/ndoc/test/cas/mlongbosso.keystore -storepass 123654`
	(注释： 从mlongbosso.keystore中导出别名为mlongbosso的证书，生成文件mlongbosso.crt)

3. 配置Tomcat的HTTPS服务
	keystoreFile属性值为mlongbosso.keystore文件路径, keystorePass属性值为证书存贮口令
```
		<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
		               maxThreads="150" scheme="https" secure="true"
		               clientAuth="false" sslProtocol="TLS"
		                keystoreFile="/home/ndoc/test/cas/mlongbosso.keystore" 
		                keystorePass="123654"
		                 />
```

### 应用端
应用端即SSO客户端.

**注释：** Windows下为`%JAVA_HOME%` , Linux下为`$JAVA_HOME`

1. 将mlongbosso.crt导入到应用服务器所使用的jre的可信任证书仓库中
`keytool -import -alias mlongbosso -file /home/ndoc/test/cas/mlongbosso.crt -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass 123654`

2. 列出jre可信任证书仓库中证书名单，验证导入是否成功，如果导入成功，应该在列表中能找到mlongbosso这个别名
`keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass 123654`

注意：如果此处导入失败，或者要重新导入，需要先删除%JAVA_HOME%/jre/lib/security/cacerts文件(删除前请备份)
<!-- more -->
## 为应用服务器开启CAS
应用服务器即SSO客户端。修改web.xml文件，增加如下filter(需要添加在其他filter之前，如struts2)：

```xml
<!-- cas 客户端登录验证 -->
<filter>
    <filter-name>CAS Authentication Filter</filter-name>
    <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>
    <init-param>
        <param-name>casServerLoginUrl</param-name>
        <param-value>https://passport.mlongbo.com:8443/login</param-value>
    </init-param>
    <init-param>
        <param-name>serverName</param-name>
        <param-value>http://cas.mlongbo.com:8080/</param-value>
    </init-param>

</filter>

<filter-mapping>
    <filter-name>CAS Authentication Filter</filter-name>
    <url-pattern>/account/*</url-pattern>
</filter-mapping>

<!-- cas 凭证认证 -->
<filter>
    <filter-name>CAS Validation Filter</filter-name>
    <filter-class>org.jasig.cas.client.validation.Cas10TicketValidationFilter</filter-class>
    <init-param>
        <param-name>casServerUrlPrefix</param-name>
        <param-value>https://passport.mlongbo.com:8443/</param-value>
    </init-param>
    <init-param>
        <param-name>serverName</param-name>
        <param-value>http://cas.mlongbo.com:8080/</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CAS Validation Filter</filter-name>
    <url-pattern>/account/*</url-pattern>
</filter-mapping>

<!-- HttpServletRequet的包裹类
让他支持getUserPrincipal，getRemoteUser方法来取得用户信息-->

<filter>
    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
    <filter-class>org.jasig.cas.client.util.HttpServletRequestWrapperFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
    <url-pattern>/account/*</url-pattern>
</filter-mapping>

<!-- Assertion信息放在ThreadLocal变量中 -->
<filter>
    <filter-name>CAS Assertion Thread Local Filter</filter-name>
    <filter-class>org.jasig.cas.client.util.AssertionThreadLocalFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>CAS Assertion Thread Local Filter</filter-name>
    <url-pattern>/account/*</url-pattern>
</filter-mapping>
```

几个配置项说明：

* serverName指当前应用的域名地址和端口(80端口可不写)
* casServerLoginUrl配置sso登录地址
* casServerUrlPrefix设置sso应用地址
* url-pattern配置需要sso保护的资源地址

## 测试客户端配置
测试客户端即测试人员所使用的浏览器端。

1. 在测试浏览器中(受信任的根证书颁发机构项)导入mlongbosso.crt证书
2. 访问任意一个需要sso验证的地址
3. 跳转到sso登录界面后，输入正确的用户名和密码
4. 正常返回到原页面
5. 成功!!!!!!!

**附：命令记录*

![证书生成, 导出, 和导入](http://mlongbo-blog.qiniudn.com/images/cas_cert_demo.png)
