---
title: "Spring框架中一个有用的小组件：Spring Retry"
date: 2021-07-22T10:18:22+08:00
tags:
 - Spring
 - Spring Retry
---

### 1、概述

[Spring Retry](https://github.com/spring-projects/spring-retry) 是Spring框架中的一个组件，
它提供了自动重新调用失败操作的能力。这在错误可能是暂时发生的（如瞬时网络故障）的情况下很有帮助。

在本文中，我们将看到使用Spring Retry的各种方式：注解、RetryTemplate以及回调。
<!--more-->

### 2、Maven依赖

让我们首先将`spring-retry`依赖项添加到我们的`pom.xml`文件中：
```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>1.2.5.RELEASE</version>
</dependency>
```

我们还需要将Spring AOP添加到我们的项目中：
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

可以查看Maven Central来获取最新版本的[`spring-retry`](https://search.maven.org/search?q=spring-retry) 
和[`spring-aspects`](https://search.maven.org/search?q=a:spring-aspects) 依赖项。

### 3、开启Spring Retry

要在应用程序中启用Spring Retry，我们需要将`@EnableRetry`注释添加到我们的`@Configuration`类：
```java
@Configuration
@EnableRetry
public class AppConfig { ... }
```

### 4、使用Spring Retry

#### 4.1、`@Retryable`而不用恢复

我们可以使用`@Retryable`注解为方法添加重试功能：
```java
@Service
public interface MyService {
    @Retryable(value = RuntimeException.class)
    void retryService(String sql);

}
```
在这里，当抛出RuntimeException时尝试重试。

根据@Retryable的默认行为，重试最多可能发生3次，重试之间有1秒的延迟。

#### 4.2、`@Retryable`和`@Recover`

现在让我们使用`@Recover`注解添加一个恢复方法：
```java
@Service
public interface MyService {
    @Retryable(value = SQLException.class)
    void retryServiceWithRecovery(String sql) throws SQLException;
        
    @Recover
    void recover(SQLException e, String sql);
}
```
这里，当抛出`SQLException`时重试会尝试运行。 当`@Retryable`方法因指定异常而失败时，`@Recover`注解定义了一个单独的恢复方法。

因此，如果`retryServiceWithRecovery`方法在三次尝试之后还是抛出了`SQLException`，那么`recover()`方法将被调用。

恢复处理程序的第一个参数应该是`Throwable`类型（可选）和相同的返回类型。其余的参数按相同顺序从失败方法的参数列表中填充。

#### 4.3、自定义`@Retryable`的行为

为了自定义重试的行为，我们可以使用参数`maxAttempts`和`backoff`：
```java
@Service
public interface MyService {
    @Retryable( value = SQLException.class, 
      maxAttempts = 2, backoff = @Backoff(delay = 100))
    void retryServiceWithCustomization(String sql) throws SQLException;
}
```
这样最多将有两次尝试和100毫秒的延迟。

### 4.4、使用Spring Properties

我们还可以在`@Retryable`注解中使用properties。

为了演示这一点，我们将看到如何将`delay`和`maxAttempts`的值外部化到一个properties文件中。

首先，让我们在名为`retryConfig.properties`的文件中定义属性：
```properties
retry.maxAttempts=2
retry.maxDelay=100
```

然后我们指示`@Configuration`类加载这个文件：
```java
@PropertySource("classpath:retryConfig.properties")
public class AppConfig { ... }
// ...
```

最后，我们可以在`@Retryable`的定义中注入`retry.maxAttempts`和`retry.maxDelay`的值：
```java
@Service 
public interface MyService { 
  @Retryable( value = SQLException.class, maxAttemptsExpression = "${retry.maxAttempts}",
            backoff = @Backoff(delayExpression = "${retry.maxDelay}")) 
  void retryServiceWithExternalizedConfiguration(String sql) throws SQLException; 
}
```
请注意，我们现在使用的是`maxAttemptsExpression`和`delayExpression`而不是`maxAttempts`和`delay`。

### 5、`RetryTemplate`

#### 5.1、`RetryOperations`

Spring Retry提供了`RetryOperations`接口，它提供了一组`execute()`方法：
```java
public interface RetryOperations {
    <T> T execute(RetryCallback<T> retryCallback) throws Exception;

    ...
}
```

`execute()`方法的参数`RetryCallback`，是一个接口，可以插入需要在失败时重试的业务逻辑：
```java
public interface RetryCallback<T> {
    T doWithRetry(RetryContext context) throws Throwable;
}
```

#### 5.2、`RetryTemplate`配置

`RetryTemplate`是`RetryOperations`的一个实现。

让我们在`@Configuration`类中配置一个`RetryTemplate`的bean：
```java
@Configuration
public class AppConfig {
    //...
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
		
        FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(2000l);
        retryTemplate.setBackOffPolicy(fixedBackOffPolicy);

        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(2);
        retryTemplate.setRetryPolicy(retryPolicy);
		
        return retryTemplate;
    }
}
```
这个`RetryPolicy`确定了何时应该重试操作。

其中`SimpleRetryPolicy`定义了重试的固定次数，另一方面，`BackOffPolicy`用于控制重试尝试之间的回退。

最后，`FixedBackOffPolicy`会使重试在继续之前暂停一段固定的时间。

#### 5.3、使用`RetryTemplate`

要使用重试处理来运行代码，我们可以调用`retryTemplate.execute()`方法：
```java
retryTemplate.execute(new RetryCallback<Void, RuntimeException>() {
    @Override
    public Void doWithRetry(RetryContext arg0) {
        myService.templateRetryService();
        ...
    }
});
```

我们可以使用lambda表达式代替匿名类：
```java
retryTemplate.execute(arg0 -> {
    myService.templateRetryService();
    return null;
});
```

### 6、监听器

监听器在重试时提供另外的回调。我们可以用这些来关注跨不同重试的各个横切点。

#### 6.1、添加回调

回调在`RetryListener`接口中提供：

```java
public class DefaultListenerSupport extends RetryListenerSupport {
    @Override
    public <T, E extends Throwable> void close(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onClose");
        ...
        super.close(context, callback, throwable);
    }

    @Override
    public <T, E extends Throwable> void onError(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onError"); 
        ...
        super.onError(context, callback, throwable);
    }

    @Override
    public <T, E extends Throwable> boolean open(RetryContext context,
      RetryCallback<T, E> callback) {
        logger.info("onOpen");
        ...
        return super.open(context, callback);
    }
}
```
`open`和`close`的回调在整个重试之前和之后执行，而`onError`应用于单个`RetryCallback`调用。

#### 6.2、注册监听器

接下来，我们将我们的监听器`（DefaultListenerSupport）`注册到我们的`RetryTemplate` bean：
```java
@Configuration
public class AppConfig {
    ...

    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
        ...
        retryTemplate.registerListener(new DefaultListenerSupport());
        return retryTemplate;
    }
}
```
### 7、测试结果

为了完成我们的示例，让我们验证一下结果：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = AppConfig.class,
  loader = AnnotationConfigContextLoader.class)
public class SpringRetryIntegrationTest {

    @Autowired
    private MyService myService;

    @Autowired
    private RetryTemplate retryTemplate;

    @Test(expected = RuntimeException.class)
    public void givenTemplateRetryService_whenCallWithException_thenRetry() {
        retryTemplate.execute(arg0 -> {
            myService.templateRetryService();
            return null;
        });
    }
}
```

从测试日志中可以看出，我们已经正确配置了`RetryTemplate`和`RetryListener`：
```
2020-01-09 20:04:10 [main] INFO  c.p.s.DefaultListenerSupport - onOpen 
2020-01-09 20:04:10 [main] INFO  c.pinmost.springretry.MyServiceImpl - throw RuntimeException in method templateRetryService() 
2020-01-09 20:04:10 [main] INFO  c.p.s.DefaultListenerSupport - onError 
2020-01-09 20:04:12 [main] INFO  c.pinmost.springretry.MyServiceImpl - throw RuntimeException in method templateRetryService() 
2020-01-09 20:04:12 [main] INFO  c.p.s.DefaultListenerSupport - onError 
2020-01-09 20:04:12 [main] INFO  c.p.s.DefaultListenerSupport - onClose
```

### 8、结论

在本文中，我们看到了如何使用注解、`RetryTemplate`和回调监听器来使用Spring Retry。