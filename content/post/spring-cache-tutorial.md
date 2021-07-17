---
title: "Spring缓存指南"
date: 2021-07-07T18:18:22+08:00
tags:
 - Spring
 - Caching
---

### 1、缓存抽象？

在本教程中，我们将学习如何*在Spring中使用缓存抽象*，并从总体上改进我们系统的性能。

我们将为一些现实的方法示例启用简单的缓存，还将讨论如何通过智能的缓存管理来实际改进这些调用的性能。
<!--more-->
 
### 2、入门

Spring提供的核心缓存抽象位于 [`spring-context`](https://search.maven.org/search?q=g:org.springframework%20a:spring-context) 模块中。
所以在使用Maven时，我们的`pom.xml`应该包含如下依赖：
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.3</version>
</dependency>
```

有趣的是，还有另一个名为[`spring-context-support`](https://search.maven.org/search?q=g:org.springframework%20a:spring-context-support) 的模块，
它位于`spring-context`模块的顶层，并提供了一些`CacheManagers`，由类似EhCache或Caffeine等提供支持。如果我们想将它们用作我们的缓存存储，那么我们需要改用`spring-context-support`模块：
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.3.3</version>
</dependency>
```
由于`spring-context-support`模块传递依赖于`spring-context`模块，因此不需要为`spring-context`单独声明依赖项。

#### 2.1、Spring Boot

如果我们使用Spring Boot，那么我们可以利用[`spring-boot-starter-cache`](https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-cache) 启动包来轻松添加缓存依赖项：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.4.0</version>
</dependency>
```

在这个封装下，启动包会引入`spring-context-support`模块。

### 3、启用缓存

为了启用缓存，Spring充分利用了注解，就像启用框架中的任何其他配置级别功能一样。

我们可以简单地通过在任何配置类中添加@EnableCaching注解来启用缓存功能：

```java
@Configuration
@EnableCaching
public class CachingConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("addresses");
    }
}
```

当然，我们也可以使用XML配置启用缓存管理：
```xml
<beans>
    <cache:annotation-driven />

    <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
        <property name="caches">
            <set>
                <bean 
                  class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" 
                  name="addresses"/>
            </set>
        </property>
    </bean>
</beans>
```

注意：启用缓存后，对于最基本的设置，我们必须注册一个cacheManager。

#### 3.1、使用Spring Boot

使用Spring Boot时，依赖的starter包与`EnableCaching`注释一起存在将注册相同的`ConcurrentMapCacheManager`，所以不需要单独的bean声明。

此外，我们可以使用一个或多个`CacheManagerCustomizer<T>`bean来定制[自动配置](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.java)的`CacheManager`：
```java
@Component
public class SimpleCacheCustomizer 
  implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
        cacheManager.setCacheNames(asList("users", "transactions"));
    }
}
```

这个[`CacheAutoConfiguration`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.java)自动配置会找到这些定制类，并把它们应用到当前的`CacheManager`，在其完整初始化之前。

### 4、使用注解缓存

启用缓存后，下一步是将缓存行为绑定到声明了注解的方法。

#### 4.1、@Cacheable

为方法启用缓存行为的最简单方法是使用`@Cacheable`对其进行标注，并使用要存储结果的缓存名称作为其参数：
```java
@Cacheable("addresses")
public String getAddress(Customer customer) {...}
```
这个`getAddress()`被调用时，将在实际调用方法之前首先检查缓存`addresses`，之后缓存方法的结果。

虽然在大多数情况下一个缓存就足够了，但Spring框架也支持多个缓存作为参数传递：
```java
@Cacheable({"addresses", "directory"})
public String getAddress(Customer customer) {...}
```
在这种情况下，如果任何缓存包含所需的结果，则返回结果并且不调用实际方法。

#### 4.2、@CacheEvict

现在，如果让所有方法都`@Cacheable`会有什么问题？

问题是缓存大小。不是经常需要读取的值，我们并不需要存入缓存。缓存会增长得非常大，非常快，我们可能会保留大量陈旧或不使用的数据。

我们可以使用`@CacheEvict`注解来指示删除一个或多个/所有的值，以便可以再次将新值加载到缓存中：
```java
@CacheEvict(value="addresses", allEntries=true)
public String getAddress(Customer customer) {...}
```
在这里，我们添加了额外参数`allEntries`与要清空的缓存一起使用，这将清除`addresses`缓存中的所有条目并为新数据做好准备。

#### 4.3、@CachePut

虽然`@CacheEvict`通过删除陈旧和未使用的条目来减少在大型缓存中查找条目的开销，但我们希望避免从缓存中清理太多数据。

相反，每当我们更改条目时，我们都可以选择更新条目。

使用`@CachePut`注解，我们可以在不干扰方法执行的情况下更新缓存的内容。也就是说，该方法将始终执行并缓存结果：
```java
@CachePut(value="addresses")
public String getAddress(Customer customer) {...}
```
`@Cacheable`和`@CachePut`之间的区别是`@Cacheable`将跳过运行的方法，而`@CachePut`将实际运行方法，然后把它的结果存在缓存中。

#### 4.4、@Cache

如果我们想使用多个相同类型的注解来标注一个方法的缓存怎么办？让我们看一个不正确的例子：
```java
@CacheEvict("addresses")
@CacheEvict(value="directory", key=customer.name)
public String getAddress(Customer customer) {...}
```
上面的代码将无法编译，因为Java不允许为给定的方法声明多个相同类型的注解。

上述问题的解决方法是：
```java
@Caching(evict = { 
  @CacheEvict("addresses"), 
  @CacheEvict(value="directory", key="#customer.name") })
public String getAddress(Customer customer) {...}
```
如上面的代码片段所示，我们可以使用`@Caching`将多个缓存注解分组，并使用它来实现我们自己自定义的缓存逻辑。

#### 4.5、@CacheConfig

使用@CacheConfig注解，我们可以在类级别将一些缓存配置简化到一个地方，这样我们就不必多次声明：
```java
@CacheConfig(cacheNames={"addresses"})
public class CustomerDataService {

    @Cacheable
    public String getAddress(Customer customer) {...}
```

### 5、有条件的缓存

有时，缓存可能无法对一个方法在所有情况下都适用。

重用我们在`@CachePut`注解中的示例，这将每次都执行该方法并缓存结果：
```java
@CachePut(value="addresses")
public String getAddress(Customer customer) {...}
```

#### 5.1、Condition参数

如果我们想要更多地控制注解何时被激活，我们可以使用一个condition参数作为参数传递给`@CachePut`，该参数接收了一个SpEL表达式并基于该表达式的计算来对结果进行缓存：
```java
@CachePut(value="addresses", condition="#customer.name=='Tom'")
public String getAddress(Customer customer) {...}
```

#### 5.2. Unless参数

我们还可以通过unless参数根据方法的输出而不是输入来控制缓存：
```java
@CachePut(value="addresses", unless="#result.length()<64")
public String getAddress(Customer customer) {...}
```
上述注解将缓存addresses，除非它们短于64个字符。

重要的是要知道Condition和unless参数可以与所有缓存注解结合使用。

事实证明，这种条件缓存对于管理大型结果非常有效。它也可用于根据输入参数自定义行为，而不是对所有操作强制执行通用行为。

### 6、基于XML的声明式缓存

如果我们无法访问应用程序的源代码，或者想要在外部注入缓存行为，我们还可以使用基于XML的声明性缓存。

这是我们的 XML 配置：
```xml
<!-- the service that you wish to make cacheable -->
<bean id="customerDataService" 
  class="com.your.app.namespace.service.CustomerDataService"/>

<bean id="cacheManager" 
  class="org.springframework.cache.support.SimpleCacheManager"> 
    <property name="caches"> 
        <set> 
            <bean 
              class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" 
              name="directory"/> 
            <bean 
              class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" 
              name="addresses"/> 
        </set> 
    </property> 
</bean>
<!-- define caching behavior -->
<cache:advice id="cachingBehavior" cache-manager="cacheManager">
    <cache:caching cache="addresses">
        <cache:cacheable method="getAddress" key="#customer.name"/>
    </cache:caching>
</cache:advice>

<!-- apply the behavior to all the implementations of CustomerDataService interface->
<aop:config>
    <aop:advisor advice-ref="cachingBehavior"
      pointcut="execution(* com.your.app.namespace.service.CustomerDataService.*(..))"/>
</aop:config>
```

### 7、基于Java的缓存

这是等效的Java配置：
```java
@Configuration
@EnableCaching
public class CachingConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
          new ConcurrentMapCache("directory"), 
          new ConcurrentMapCache("addresses")));
        return cacheManager;
    }
}
```

这是我们的CustomerDataService：
```java
@Component
public class CustomerDataService {
 
    @Cacheable(value = "addresses", key = "#customer.name")
    public String getAddress(Customer customer) {
        return customer.getAddress();
    }
}
```

### 8、总结

在本文中，我们讨论了Spring中缓存的基础知识，以及如何通过注解恰当地使用这个抽象。