---
title: "嵌入式Redis服务器在Spring Boot测试中的使用"
date: 2021-07-05T22:57:22+08:00
tags:
- Spring Boot
- Testing
- Redis
---

### 1、概述
   
`Spring Data Redis`提供了一种与Redis实例集成的简单方法。

但是，在某些情况下，使用嵌入式服务器比使用真实服务器创建开发和测试环境更方便。

因此，我们将学习如何设置和使用嵌入式Redis服务器。

### 2、依赖

让我们首先添加必要的依赖项：
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
  <groupId>it.ozimov</groupId>
  <artifactId>embedded-redis</artifactId>
  <version>0.7.2</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```
这个`spring-boot-starter-test`包含我们需要运行集成测试的各种依赖。

此外，`embedded-redis`包含我们将使用的嵌入式服务器。

### 3、设置

添加依赖项后，我们应该定义Redis服务器和我们的应用程序之间的连接设置。

让我们首先创建一个类来保存我们的属性：
```java
@Configuration
public class RedisProperties {
    private int redisPort;
    private String redisHost;

    public RedisProperties(
      @Value("${spring.redis.port}") int redisPort, 
      @Value("${spring.redis.host}") String redisHost) {
        this.redisPort = redisPort;
        this.redisHost = redisHost;
    }

    // getters
}
```

接下来，我们应该创建一个配置类来定义连接并使用我们的属性：
```java
@Configuration
@EnableRedisRepositories
public class RedisConfiguration {

    @Bean
    public LettuceConnectionFactory redisConnectionFactory(
      RedisProperties redisProperties) {
        return new LettuceConnectionFactory(
          redisProperties.getRedisHost(), 
          redisProperties.getRedisPort());
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate(LettuceConnectionFactory connectionFactory) {
        RedisTemplate<byte[], byte[]> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        return template;
    }
}
```
配置非常简单。这样我们的嵌入式服务器可以在其他的端口上运行。

### 4、嵌入式Redis服务器

现在，我们将配置嵌入式服务器并在我们的一项测试中使用它。

首先，让我们在测试的资源目录（src/test/resources）中创建一个application.properties文件：
```properties
spring.redis.host=localhost
spring.redis.port=6370
```

之后，我们将创建一个@TestConfiguration注解的配置类：
```java
@TestConfiguration
public class TestRedisConfiguration {

    private RedisServer redisServer;

    public TestRedisConfiguration(RedisProperties redisProperties) {
        this.redisServer = new RedisServer(redisProperties.getRedisPort());
    }

    @PostConstruct
    public void postConstruct() {
        redisServer.start();
    }

    @PreDestroy
    public void preDestroy() {
        redisServer.stop();
    }
}
```

当context上下文启动，服务器就跟着启动。它根据我们在属性中定义的端口运行在我们的机器上。有了它，我们现在可以在不停止实际Redis服务器的情况下运行测试了。

理想情况下，我们希望在随机可用端口上启动它，但嵌入式Redis尚不具备此功能。我们现在可以做的是通过ServerSocket API 获取随机端口。

此外，当上下文停止，服务器也跟着停止。

服务器也可以由我们自己的可执行文件来提供：
```java
this.redisServer = new RedisServer("/path/redis", redisProperties.getRedisPort());
```

此外，可执行文件可以按不同的操作系统来定义：
```java
RedisExecProvider customProvider = RedisExecProvider.defaultProvider()
.override(OS.UNIX, "/path/unix/redis")
.override(OS.Windows, Architecture.x86_64, "/path/windows/redis")
.override(OS.MAC_OS_X, Architecture.x86_64, "/path/macosx/redis");

this.redisServer = new RedisServer(customProvider, redisProperties.getRedisPort());
```

最后，让我们创建一个使用TestRedisConfiguration类的测试吧：
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = TestRedisConfiguration.class)
public class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void shouldSaveUser_toRedis() {
        UUID id = UUID.randomUUID();
        User user = new User(id, "name");

        User saved = userRepository.save(user);

        assertNotNull(saved);
    }
}
```
这样用户保存就到了我们的嵌入式Redis服务器。

此外，我们必须手动将`TestRedisConfiguration`添加到`SpringBootTest`。正如我们之前所说，服务器在测试之前启动并在测试之后停止。

### 5、结论
   
嵌入式Redis服务器是在测试环境中替换实际服务器的完美工具。我们已经看到了如何配置它以及如何在我们的测试中使用它。



