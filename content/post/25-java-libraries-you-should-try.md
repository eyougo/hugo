---
title: "你应该尝试的25个不那么知名的Java库"
date: 2021-07-26T16:50:22+08:00
tags:
 - Java
---

之前的文章，我们列出了10个最重要的第三方Java库，每个Java程序员都应该知道它们：

[每个Java程序员都应该知道的10个顶级库](/post/java-top10-libraries/)

在这篇文章中，我想介绍25个另外的Java库。
这些库也许不那么知名，但也很成熟，能够在Java开发中对一些常见的问题提供非常有效的解决方案。

当然，这里不会包含上一篇文章中已经列出的那些库。而且在这里主要列出的是库而不是框架。
<!--more-->

### 1、RxJava

Reactive Extensions (ReactiveX) 是一种流行的软件开发范式，用于处理异步和事件驱动的编程。

`RxJava`是使用`Observables`的`Reactive Extensions`的 Java VM 实现。
它通过以声明方式在一系列事件或数据上添加可组合的操作符来扩展观察者模式以支持事件驱动编程。
它还隐藏了底层的复杂性，如线程、线程安全、同步和并发的数据结构等。

如果你想用Java进行响应式编程，它是一个必备的库。

链接：<https://github.com/ReactiveX/RxJava>

### 2、OkHttp

HTTP是迄今为止最常用的应用层协议。有许多优秀的基于Java的HTTP客户端库。

但 `OkHttp` 是 JVM 中最简单但功能强大的 HTTP 客户端库。
它提供了流畅和干净的 API 来用 Java 开发 HTTP 客户端。
它还支持一些高级功能：连接池、GZIP压缩、响应缓存、现代TLS功能等等。

链接：<https://github.com/square/okhttp>

### 3、MyBatis

在大多数软件开发项目中，我们需要存储数据。尽管数据存储的类型很多，但 SQL 仍然是最常用的数据存储类型。

作为 Java 开发人员，我们需要将 Java 对象与 SQL 表进行匹配。实现映射的一种方法是使用ORM（例如，Hibernate）。
但是当你想要完全控制对象表的映射（比如性能）时，有很多实现用例。在这些情况下，你可以直接使用`JDBC`编写`SQL`查询。
另一种方式是使用`MyBatis`将`Java Object`映射到`Stored Procedure`或`SQL`语句。它提供基于注解或基于XML描述符的映射。
我更喜欢`MyBatis`而不是普通的`JDBC`，尤其是在较大的项目中，因为它让关注点的分离更加优化。

链接：<https://github.com/mybatis/mybatis-3>

### 4、HikariCP

`HikariCP`是此列表中与数据库相关的第二个库。建立`JDBC`连接是资源昂贵的。
如果你每次访问数据库时都创建一个新连接并在完成后关闭它，会严重影响程序性能，甚至未能正确关闭连接或无限的数据库连接可能会使程序崩溃。

使用连接池意味着每次请求连接时都会重用而不是创建连接。`HikariCP`是JVM中一个非常快速但轻量级的数据库连接池。它非常可靠并且是一个“零开销”的JDBC连接池。

链接：<https://github.com/brettwooldridge/HikariCP>

### 5、Lombok

在最新的趋势里，Java经常被批评是一种冗长和臃肿的编程语言。
与其他流行语言（JavaScript、Python、Scala、Kotlin）相比，开发人员需要用Java编写大量模版代码。
尽管 Java 在`JDK 15`中引入了`Records` 以减少 Java 中的模版代码，但它还不是LTS版本。

幸运的是，一个库已经可以显着减少 Java 中的样板代码：`Lombok`。
你可以通过添加一些简单的注解来生成`getter、setter、hashcode、equals、toString、Builder`的代码。
此外，它还提供空指针检查、记录器等。

链接：<https://github.com/rzwitserloot/lombok>

### 6、VAVR

Java 终于在第8版中发布了期待已久的通过`Lambda`和`Streaming`进行的函数式编程。
如果你习惯了函数式编程，或者想深入研究函数式编程，那么你可能会发现Java的函数式编程还远远不够好。
与许多其他函数式编程语言（Haskell、Scala）相比，Java还是显得苍白无力。

`VAVR`是一个可以填补Java中函数式编程特性空白的库。它提供持久化集合、错误处理的功能抽象、并发编程、模式匹配等等。

链接：<https://www.vavr.io/>

### 7、Gson

多年来，JSON 已成为事实上的数据交换格式。在 Java 中，也有几个优秀的库来处理 JSON。
其中之一是`Jackson`，在之前文章中已经介绍过。

另一个优秀的库是 Google 的`Gson`。与`Jackson`不同的是，它是一个简约的库，并且只支持 JSON。它提供数据绑定、广泛的通用支持、灵活的定制。
`Gson`的主要优点（也可能是缺点这取决于你的喜好）之一是它不需要注解。

链接：<https://github.com/google/gson>

### 8、jsoup

如果需要使用 Java 处理`HTML`，你应该使用`jsoup`。

它是一个用于处理真实世界`HTML`的 Java 库。它提供了一个非常方便的 API 来获取 URL 以及提取和操作数据。
它实现了`WHATWG` `HTML5`规范并使用最好的`HTML5` `DOM`方法解析`HTML`。它支持从URL/字符串解析`HTML`，查找和提取数据，操作`HTML`元素，清理`HTML`，输出`HTML`。

链接：<https://github.com/jhy/jsoup>

### 9. JIB

如果你正在开发的是企业级的应用程序，它至少应该是准备上云的。
使你的应用程序为云做好准备的第一步是将你的应用程序容器化，即将你的制品二进制文件放入`Docker`镜像中。
对 Java 应用程序进行`Docker`化是一项有点乏味的工作：你需要对`Docker`有深入的了解，需要创建一个`Dockerfile`，并且需要`Docker Daemon`。

对于 Java 开发人员来说幸运的是，谷歌使用现有工具创建了一个开源 Java 容器化工具。你可以使用`JIB`作为Java库来构建优化的`Docker`和`OCI`镜像。

链接：<https://github.com/GoogleContainerTools/jib>

### 10、Tink

`Tink`是这个列表中Google出品的另一个方便的Java库。

密码学和安全性在软件开发中变得越来越重要，密码技术用于保护用户数据，而正确实施密码学需要大量的专业知识和努力。
Google 的一组密码学家和安全工程师编写了多语言密码库`Tink`。它提供易于使用但难以滥用的安全 API。
`Tink`通过不同的原语提供加密功能。它提供了对称密钥加密、流式对称密钥加密、确定性对称密钥加密、数字签名、混合加密和许多其他加密功能。

链接：<https://github.com/google/tink>

### 11、Webmagic

假如你需要网络爬虫，可以自己编写，但这既费时又乏味。

在 Java 中，`Webmagic`是一个优秀的网络爬虫库，涵盖了爬虫的完整生命周期：下载、URL管理、内容提取、持久化。
它提供了一个简单而灵活的核心、注解支持、多线程和易于使用的 API。

链接：<https://github.com/code4craft/webmagic>

### 12、ANTLR 4

如果你致力于解析和处理数据，那么`ANTLR`库可能会很方便。
它是一个强大的解析器生成器，用于读取、处理、执行或翻译结构化文本或二进制文件。
它通过让我们访问语言处理原语（如词法分析器、语法、解析器和运行时）来处理文本来实现这一点。

它通常用于构建工具和框架。

链接：<https://github.com/antlr/antlr4>

### 13、Caffeine

如果你的应用程序需要大量读取操作，那么缓存可以显着提高数据访问性能。

Java 有很多很棒的缓存库，`Caffeine`是其中最好的之一。它是一个基于 Java 的高性能、近乎最优的缓存库。
它提供了流畅的缓存API和一些高级功能，如条目的异步加载、异步刷新、弱引用键等。

链接：<https://github.com/ben-manes/caffeine>

### 14. Metrics

一旦你的Java应用程序投入生产，你将希望深入了解应用程序的关键组件。
来自`Dropwizard`框架的`Metrics`是一个简单但引人注目的 Java 库，它提供对应用程序和`JVM`的 KPI 的洞察，例如事件率、待处理作业、服务健康检查等。
它是模块化的，并为其他库/框架提供模块。

链接：<https://github.com/dropwizard/metrics>

### 15、gRPC-Java

谷歌在2015年创建了`gRPC`作为现代的RPC系统。从那时起，`gRPC`变得非常流行，并且是现代软件开发中最常用的RPC系统之一。

`gRPC-Java`库是`gRPC`客户端的 Java 实现。如果你想在 Java 中使用`gRPC`，那么这个库对你来说很方便。

链接：<https://github.com/grpc/grpc-java>

### 16、Java WebSocket

传统的客户端-服务器通信是单向的。`WebSocket`是一种基于单个TCP连接的双向通信协议。

`Java WebSocket`是Java中的准系统`WebSocket`服务器和客户端实现。
如果你是Java开发者并希望使用`WebSocket`，强烈推荐使用此库。

链接：<https://github.com/TooTallNate/Java-WebSocket>

### 17、JJWT

JSON Web Token (JWT) 是现代软件开发中事实上的授权和安全信息交换格式。无论你是使用简单的基于会话的授权还是基于 OAuth2 的高级授权，你都可能会使用 JWT。

`JJWT`是一个简单的Java库，用于在 Java 和 JVM 环境中创建和验证 JWT。它完全符合所有实现的功能的 RFC 规范。它支持可读和方便的流式 API。

链接：<https://github.com/jwtk/jjwt>

### 18、Swagger-Core

OpenAPI 是用于描述、生成、使用和可视化 RESTful Web 服务的机器可读接口文件的规范。

`Swagger-Core` 是 OpenAPI 规范的 Java 实现。如果你在 Java/JavaEE 应用程序中公开 REST API，你可以使用 Swagger-Core 自动提供和公开你的 API 定义。

链接：<https://github.com/swagger-api/swagger-core>

### 19、Async Http Client

由于其非阻塞的特性，异步编程最近变得越来越流行。大多数流行的 Java HTTP 客户端库都提供有限的异步 HTTP 响应处理。

`Async Http Client`是一个流行的 Java 库，提供异步 HTTP 响应处理。作为一项额外功能，该库还支持 WebSocket 协议。

链接：<https://github.com/AsyncHttpClient/async-http-client>

### 20、Liquibase

作为软件开发人员，我们都知道代码的版本控制、DevOps 和 CI/CD 的重要性。
伟大的Martin Fowler在他的博文[Evolutionary Database Design]<https://martinfowler.com/articles/evodb.html>里认为我们还需要数据库的版本控制和 CI/CD。

`Liquibase`是一个支持在 Java 应用程序中跟踪、版本控制和部署 SQL 数据库更改的工具。
如果你正在使用数据库不断发展的 SQL 数据库，此工具可以极大地简化你的数据库迁移。

链接：<https://www.liquibase.org/>

### 21、Springfox

这个列表中已经列出了`Swagger-Core`，它可以为普通的 Java 或 Java EE 应用程序自动生成 REST API 文档。

在企业应用开发方面，`Spring MVC`已经超越 Java EE 成为第一的应用开发平台。
在基于`Spring`的 Java 应用程序中，`Springfox`库可以从源代码自动生成 REST API 文档。

链接：<https://github.com/springfox/springfox>

### 22、JavaCV

`OpenCV`是一个计算机视觉和机器学习软件库。它是开源的，旨在为计算机视觉应用程序提供通用基础架构。

`JavaCV`是`OpenCV`和计算机视觉领域许多其他流行库（FFmpeg、libdc 1394、PGR FlyCapture）的包装器。
`JavaCV`还带有硬件加速的全屏图像显示、使用简单方法在多核上并发执行代码、用户友好的几何参数以及相机和投影仪的颜色校准、特征点的检测和匹配等特征。

链接：<https://github.com/bytedeco/javacv>

### 23、Joda Time

在 Java8 之前的核心库中，Java 的日期和时间功能很差。Java8 在其java.time包中发布了急需的高级日期和时间功能。

如果你使用的是旧版本的 Java（Java8 之前），`Joda Time`可以为你提供高级日期和时间功能。但是，如果你使用的是较新版本的 Java，则可能不需要此库。

链接：<https://github.com/JodaOrg/joda-time>

### 24、Wiremock

HTTP 是现代应用程序开发中最受欢迎的传输协议，而 REST 是基于微服务的应用程序开发中事实上的通信协议。

在编写单元测试时，最好将重点放在 SUT（被测系统）上，并模拟 SUT 中使用的服务。
`Wiremock`是 REST API 的模拟器，使开发人员能够针对不存在或不完整的 API 编写代码。在基于微服务的软件开发中，Wiremock 可以显着提高开发速度。

链接：<https://github.com/tomakehurst/wiremock>

### 25、MapStruct

在 Java 应用程序开发中，你经常需要将一种类型的 POJO 转换为另一种类型的 POJO。实现这种 POJO 或 Bean 转换的一种方法是对转换进行显式编码，这很乏味。

聪明的方法是使用专门开发的库来转换 POJO/Bean。`MapStruct` 是一个代码生成器，它以约定优于配置的方式实现 POJO/Bean 之间的映射。生成的映射代码使用简单的方法调用，因此速度快、类型安全且易于理解。

链接：<https://github.com/mapstruct/mapstruct>

### 总结

本文中列出了25个 Java 库，可以通过它们测试库里的常见任务来更好地帮助你进行软件开发。

这些库不是特定于领域的，无论你是在为商业应用程序、机器人、Android 应用程序还是个人项目开发软件，都可以为你提供帮助。

当然，对于像 Java 这样庞大而广泛的生态系统，这个列表并不是一定的。有很多优秀的 Java 库没有在这里列出但值得一试，但是这个库列表可以让你快速了解 Java 生态系统的世界。