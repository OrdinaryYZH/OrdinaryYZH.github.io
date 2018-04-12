---
title: Nginx反向代理后，ServerName最终结果分析
date: 2018-3-8 00:00:00
toc: true
categories: 
- server
tags:
- springboot
- nginx
---

> 说明：
>
> Web容器：tomcat-embed-8.5.16

## 1. 问题

​	项目现场Web服务器被Nginx代理后，Server端通过`Request.getRequestUrl()`中获得的ServerName为何是客户端发送的域名，而不是Nginx代理后发送的内网ip？

## 2. 知识背景

### 2.1 Tomcat使用Facade模式封装Request

先看下类的关系：

![](https://ws1.sinaimg.cn/mw690/8747d788gy1fp7nk2v8cmj213k0xo3zu.jpg)

`org.apache.catalina.connector.Request`类，封装了`org.apache.coyote.Request`类，实现了`HttpServletRequest`接口，已经具备了实际使用能力，不过它还包含了很多Catalina的方法，这些方法不应该暴露给应用层，以免引起与其他容器实现的兼容性问题。

`org.apache.catalina.connector.RequestFacade`类实现了`HttpServletRequest`接口，**并在其中包含了一个`org.apache.catalina.connector.Request`对象，将所有`HttpServletRequest`接口的调用**，都代理给**`org.apache.catalina.connector.Request`对象来处理**，这样就屏蔽了Catalina的相关的内部方法，使用户可以专注于servlet的标准方法。

### 2.2 Nginx配置说明

> 参考：http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header

| Syntax:  | `proxy_set_header field value;`          |
| -------- | ---------------------------------------- |
| Default: | `proxy_set_header Host $proxy_host;` <br />`proxy_set_header Connection close;` |
| Context: | `http`, `server`, `location`             |

```
proxy_set_header Host       $http_host;
```
>  However, if this field is not present in a client request header then nothing will be passed.
>  所以配置$http_host就是用来传递客户端的Host头的

## 3. 问题分析

有了以上背景之后，就可以很容易的分析出该问题的原因了。

1. 因为tomcat中实际处理的是`org.apache.catalina.connector.Request`，所以看下`getRequestURL()`是如何处理的：

   ```java
   @Override
   public StringBuffer getRequestURL() {

       StringBuffer url = new StringBuffer();
       String scheme = getScheme();
       int port = getServerPort();
       if (port < 0)
        {
           port = 80; // Work around java.net.URL bug
       }

       url.append(scheme);
       url.append("://");
       // 获取ServerName
       url.append(getServerName());
       if ((scheme.equals("http") && (port != 80))
           || (scheme.equals("https") && (port != 443))) {
           url.append(':');
           url.append(port);
       }
       url.append(getRequestURI());

       return url;
   }
   ```

   再看下`getServerName()`

   ```java
   /**
    * @return the server name responding to this Request.
    */
   @Override
   public String getServerName() {
       return coyoteRequest.serverName().toString();
   }
   ```

   继续进入`org.apache.coyote.Request#serverName()`

   ```java
   /**
    * Get the "virtual host", derived from the Host: header associated with
    * this request.
    *
    * @return The buffer holding the server name, if any. Use isNull() to check
    *         if there is no value set.
    */
   public MessageBytes serverName() {
       return serverNameMB;
   }
   ```

   **从注释中可以看到，`serverName()`拿的是Host请求头**

2. Nginx中的配置应该是`proxy_set_header Host       $http_host;`，（PS：为什么说应该？因为是客户现场配置的Nginx，无法直接查看配置；而且从程序中获得的requestUrl以及Nginx的默认配置说明中推测出来）

## 4. 结论

所以说程序中`request.getRequestUrl()`后获得的url中的serverName是由Host-Header决定的，

而我们的Web服务器经过了Nginx代理，Nginx中配置了`proxy_set_header Host       $http_host`,所以最终获得的还是客户端的serverName，而不是Nginx服务器的内网ip。



## 5. 参考资源

1. [HTTP头字段](https://zh.wikipedia.org/zh-cn/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5)
2. [Tomcat为什么要使用Facde模式对Request对象进行包装？](https://www.zhihu.com/question/39872707)
3. [HTTP 请求头中的 X-Forwarded-For](https://imququ.com/post/x-forwarded-for-header-in-http.html)
4. [http头中的host字段详解](http://www.6san.com/463/)