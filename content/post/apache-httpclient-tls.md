---
title: "如何在Apache HttpClient中设置TLS版本"
date: 2021-07-20T00:18:22+08:00
tags:
 - HTTP Client-Side
 - Security
---

### 1、简介

Apache HttpClient是一个底层、轻量级的客户端HTTP库，用于与HTTP服务器进行通信。
在本教程中，我们将学习如何在使用HttpClient时配置支持的传输层安全（TLS）版本。
我们将首先概述TLS版本协商如何在客户端和服务器之间工作。
之后，我们将看看在*使用HttpClient时配置支持的TLS版本的三种不同方式*。
<!--more-->

### 2、TLS版本协商

TLS是一种互联网协议，可在两方之间提供安全、可信的通信。它封装了像HTTP这样的应用层协议。
TLS协议自1999年首次发布以来已多次修订。
因此，客户端和服务器在建立新连接时，首先就他们将使用的TLS版本达成一致非常重要。
TLS版本在客户端和服务器交换hello消息后达成一致：
 1. 客户端发送支持的 TLS 版本列表。
 2. 服务器选择一个并在响应中包含所选版本。
 3. 客户端和服务器使用所选版本继续连接设置。
 
由于存在降级攻击的风险，因此正确配置Web客户端支持的TLS版本非常重要。
*请注意，为了使用最新版本的TLS（TLS 1.3），我们必须使用Java 11或更高版本*。

### 3、固定设置TLS版本

#### 3.1、SSLConnectionSocketFactory

让我们通过`HttpClients#custom`定制构建方法提供的`HttpClientBuilder`，来定制我们的`HTTPClient`配置。
此构建器模式允许我们传入我们自己的`SSLConnectionSocketFactory`，它将根据一组所需支持的TLS版本进行实例化：
```java
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
  SSLContexts.createDefault(),
  new String[] { "TLSv1.2", "TLSv1.3" },
  null,
  SSLConnectionSocketFactory.getDefaultHostnameVerifier());

CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
```
返回的`Httpclient`对象现在可以执行HTTP请求了。
通过在`SSLConnectionSocketFactory`构造函数中显式设置支持的协议，客户端将仅支持通过TLS 1.2或TLS 1.3的通信。
请注意，在 Apache HttpClient 4.3 之前的版本中，该类称为`SSLSocketFactory`。


#### 3.2、Java运行时参数

另外，我们可以使用Java的`https.protocols`系统属性配置支持的TLS版本。
此方法可以防止必须将值硬编码到应用程序代码中。
相反，我们将配置`HttpClient`以在设置连接时使用该系统属性。
HttpClient API提供了两种方法来实现。第一个是通过`HttpClients#createSystem`：
```java
CloseableHttpClient httpClient = HttpClients.createSystem();
```
如果需要更多的客户端配置，我们可以使用builder方法代替：
```java
CloseableHttpClient httpClient = HttpClients.custom().useSystemProperties().build();
```
这两种方法都告诉`HttpClient`在连接配置期间使用系统属性。
这允许我们在应用程序运行时使用命令行参数设置所需的TLS版本。例如：
```shell script
$ java -Dhttps.protocols=TLSv1.1,TLSv1.2,TLSv1.3 -jar webClient.jar
```

### 4、动态设置TLS版本

还可以根据主机名和端口等连接详细信息设置TLS版本。我们将扩展`SSLConnectionSocketFactory`并覆盖`prepareSocket`方法。
客户端会在启动新连接之前调用`prepareSocket`方法，这将可以让我们在每个连接的基础上决定使用哪些TLS协议。
也可以启用对旧TLS版本的支持，但前提是远程主机具有特定的子域：
```java
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(SSLContexts.createDefault()){

    @Override
    protected void prepareSocket(SSLSocket socket) {

        String hostname = socket.getInetAddress().getHostName();
        if (hostname.endsWith("internal.system.com")){
            socket.setEnabledProtocols(new String[] { "TLSv1", "TLSv1.1", "TLSv1.2", "TLSv1.3" });
        }
        else {
            socket.setEnabledProtocols(new String[] {"TLSv1.3"});
        }
    }
};
CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
```

在上面的示例中，`prepareSocket`方法首先获取`SSLSocket`将连接到的远程主机名，然后使用主机名来确定要启用的TLS协议。
现在，我们的HTTP客户端将对每个请求强制执行TLS 1.3，除了目标主机名的格式为`*.internal.system.com`。
由于能够在创建新`SSLSocket`之前插入自定义逻辑，我们的应用程序现在可以自定义TLS通信的详细信息。

### 5、结论

在本文中，我们研究了在使用Apache HttpClient库时配置支持的TLS版本的三种不同方式。
我们已经了解了如何为所有连接或基于每个连接设置TLS版本。


> 原文：https://www.baeldung.com/apache-httpclient-tls
>
> 翻译：[码农熊猫](https://pinmost.com)