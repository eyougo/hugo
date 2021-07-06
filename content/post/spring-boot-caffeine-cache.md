---
title: "在Spring Boot中使用Caffeine缓存"
date: 2021-07-06T10:18:22+08:00
tags:
 - Spring Boot
 - Caching
---

### 1、概述
[Caffeine缓存](/post/java-caching-caffeine)是Java的高性能缓存库。在这个简短的教程中，我们将看到如何在Spring Boot中使用它。

### 2、依赖
要开始使用Caffeine和Spring Boot，我们首先添加`spring-boot-starter-cache`和`caffeine`依赖项：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
    </dependency>
</dependencies>
```
这些导入基本的Spring缓存支持以及Caffeine库。

### 3、配置
现在我们需要在我们的Spring Boot应用程序中配置缓存。

首先，我们创建一个caffeine的bean，用来控制缓存行为的主要配置，例如过期、缓存大小限制等：
```java
@Bean
public Caffeine caffeineConfig() {
    return Caffeine.newBuilder().expireAfterWrite(60, TimeUnit.MINUTES);
}
```
接下来，我们需要使用Spring `CacheManager`接口创建另一个bean。Caffeine提供了这个接口的实现，它需要我们上面创建的Caffeine对象：
```java
@Bean
public CacheManager cacheManager(Caffeine caffeine) {
    CaffeineCacheManager caffeineCacheManager = new CaffeineCacheManager();
    caffeineCacheManager.setCaffeine(caffeine);
    return caffeineCacheManager;
}
```
最后，我们需要使用@EnableCaching注解在Spring Boot中启用缓存支持，这个注解可以添加到应用程序中的任何@Configuration类。

### 4、例子
启用缓存并配置为使用Caffeine后，让我们看几个示例，说明如何在 Spring Boot 应用程序中使用缓存。

在 Spring Boot 中使用缓存的主要方法是使用@Cacheable注解。此注解适用于 Spring bean（甚至整个类）的任何方法，它指示注册的缓存管理器将方法调用的结果存储在缓存中。

一个典型的用法是在服务类中：
```java
@Service
public class AddressService {
    @Cacheable
    public AddressDTO getAddress(long customerId) {
        // lookup and return result
    }
}
```
使用不带参数的@Cacheable注解，Spring会为缓存和缓存的key使用默认名称。

我们可以通过向注解添加一些参数来覆盖这两个做法：
```java
@Service
public class AddressService {
    @Cacheable(value = "address_cache", key = "customerId")
    public AddressDTO getAddress(long customerId) {
        // lookup and return result
    }
}
```
上面的示例告诉Spring使用名为address_cache的缓存和以customerId参数作为缓存key。

最后，因为缓存管理器本身就是一个Spring bean，我们也可以将它自动装配到任何其他bean中并直接使用它：
```java
@Service
public class AddressService {

    @Autowired
    CacheManager cacheManager;

    public AddressDTO getAddress(long customerId) {
        if(cacheManager.containsKey(customerId)) {
            return cacheManager.get(customerId);
        }
        
        // lookup address, cache result, and return it
    }
}
```

### 5、结论
在本教程中，我们已经了解了如何配置Spring Boot以使用Caffeine缓存，以及如何在我们的应用程序中使用缓存的一些示例。

