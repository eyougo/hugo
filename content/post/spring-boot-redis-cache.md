---
title: "如何使用Redis作为Spring Boot缓存"
date: 2021-07-05T19:26:22+08:00
tags:
 - Spring Data
 - Caching
 - Redis
---

### 1、简介
在这个简短的教程中，我们将看看如何将Redis配置为Spring Boot缓存的数据存储。

### 2、依赖
首先，让我们在pom.xml中添加`spring-boot-starter-cache`和`spring-boot-starter-data-redis`组件：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.4.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.4.3</version>
</dependency>
```
这些添加了缓存支持并引入了所有必需的依赖项。

### 3、配置
通过添加上述依赖项和`@EnableCaching`注解，Spring Boot 将使用默认缓存配置自动配置一个`RedisCacheManager`。但是，我们可以在缓存管理器初始化之前以几种有用的方式修改此配置。

首先，让我们创建一个RedisCacheConfiguration bean：
```java
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
      .entryTtl(Duration.ofMinutes(60))
      .disableCachingNullValues()
      .serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
}
```
这使我们可以更好地控制默认配置——例如，我们可以设置所需的生存时间 (TTL) 值并自定义用于动态缓存创建的默认序列化策略。

接下来，为了完全控制缓存设置，让我们注册我们自己的RedisCacheManagerBuilderCustomizer bean：
```java
@Bean
public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
      .withCacheConfiguration("itemCache",
        RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(10)))
      .withCacheConfiguration("customerCache",
        RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(5)));
}
``` 
在这里，我们使用`RedisCacheManagerBuilder`和`RedisCacheConfiguration`分别为`itemCache`和`customerCache`配置了10分钟和5分钟的TTL值。这有助于在每个缓存的基础上进一步微调缓存行为，包括空值、键前缀和二进制序列化。

值得一提的是，Redis实例的默认连接详细信息是localhost:6379。Redis配置可用于进一步调整更底层的连接细节以及主机和端口。

### 4、例子
在我们的示例中，我们有一个ItemService组件，用于从数据库中检索项目信息。实际上，假设这是一个潜在的代价高昂的操作，是一个很好的使用缓存场景。

首先，让我们使用[嵌入式Redis服务器](/post/spring-embedded-redis)为该组件创建集成测试：
```java
@Import({ CacheConfig.class, ItemService.class})
@ExtendWith(SpringExtension.class)
@EnableCaching
@ImportAutoConfiguration(classes = { 
  CacheAutoConfiguration.class, 
  RedisAutoConfiguration.class 
})
class ItemServiceCachingIntegrationTest {

    @MockBean
    private ItemRepository mockItemRepository;

    @Autowired
    private ItemService itemService;

    @Autowired
    private CacheManager cacheManager;

    @Test
    void givenRedisCaching_whenFindItemById_thenItemReturnedFromCache() {
        Item anItem = new Item(AN_ID, A_DESCRIPTION);
        given(mockItemRepository.findById(AN_ID))
          .willReturn(Optional.of(anItem));

        Item itemCacheMiss = itemService.getItemForId(AN_ID);
        Item itemCacheHit = itemService.getItemForId(AN_ID);

        assertThat(itemCacheMiss).isEqualTo(anItem);
        assertThat(itemCacheHit).isEqualTo(anItem);

        verify(mockItemRepository, times(1)).findById(AN_ID);
        assertThat(itemFromCache()).isEqualTo(anItem);
    }
}
```
在这里，我们为缓存行为创建一个测试切面并调用getItemForId两次。第一次调用应该从存储库中获取结果，但第二次调用应该从缓存中返回结果而不调用存储库。

最后，让我们使用Spring的`@Cacheable`注解启用缓存行为：
```java
@Cacheable(value = "itemCache")
public Item getItemForId(String id) {
    return itemRepository.findById(id)
      .orElseThrow(RuntimeException::new);
}
```
这就应用了缓存逻辑，同时依赖于我们之前配置的Redis缓存的基础架构。有关控制Spring缓存对象的属性和行为（包括数据更新和清除）的更多详细信息，可以参见Spring缓存指南文章。

### 5、结论
在本文章中，我们已经看到了如何使用Redis进行Spring Boot缓存。