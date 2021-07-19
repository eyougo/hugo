---
title: "Caffeine缓存简单介绍"
date: 2021-07-06T10:18:22+08:00
tags:
 - Java
 - Caching
---

### 1、简介

在本文中，我们将了解Caffeine，一个用于Java的高性能缓存库。

缓存和Map之间的一个根本区别是缓存会清理存储的项目。

一个清理策略会决定在某个给定时间哪些对象应该被删除，这个策略直接影响缓存的命中率——缓存库的一个关键特性。

Caffeine使用`Window TinyLfu`清理策略，它提供了接近最佳的命中率。
<!--more-->

### 2、依赖

我们需要将Caffeine依赖添加到我们的pom.xml中：
```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.5.5</version>
</dependency>
```
您可以在Maven Central上找到最新版本的Caffeine。

### 3、写入缓存

让我们关注Caffeine的三种缓存写入策略：手动、同步加载和异步加载。

首先，让我们编写一个类，作为要存储在缓存中的值的类型：
```java
class DataObject {
    private final String data;

    private static int objectCounter = 0;
    // standard constructors/getters
    
    public static DataObject get(String data) {
        objectCounter++;
        return new DataObject(data);
    }
}
```

#### 3.1、手动写入

在此策略中，我们手动将值写入缓存并稍后读取它们。

我们先初始化缓存：
```java
Cache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .maximumSize(100)
  .build();
```

现在，我们可以使用`getIfPresent`方法从缓存中获取一些值。如果缓存中不存在该值，则此方法将返回`null`：
```java
String key = "A";
DataObject dataObject = cache.getIfPresent(key);

assertNull(dataObject);
```

我们可以使用`put`方法手动写入缓存：
```java
cache.put(key, dataObject);
dataObject = cache.getIfPresent(key);

assertNotNull(dataObject);
```

我们还可以使用`get`方法获取值，该方法接受一个函数和一个键作为参数。如果缓存中不存在该键，则此函数将用于提供兜底值，该值将在执行后写入缓存：
```java
dataObject = cache
  .get(key, k -> DataObject.get("Data for A"));

assertNotNull(dataObject);
assertEquals("Data for A", dataObject.getData());
```
这个GET方法执行是原子性的。这意味着即使多个线程同时请求该值，执行只会进行一次。这就是为什么使用`get`比`getIfPresent`更好。

有时我们需要手动使一些缓存的值失效：
```java
cache.invalidate(key);
dataObject = cache.getIfPresent(key);

assertNull(dataObject);
```

#### 3.2、同步加载

这种加载缓存的方法需要一个`Function`，用于初始化写入值，类似于手动写入策略的get方法，让我们看看如何使用它。

首先，我们需要初始化我们的缓存：
```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

现在我们可以使用`get`方法读取值：
```java
DataObject dataObject = cache.get(key);

assertNotNull(dataObject);
assertEquals("Data for " + key, dataObject.getData());
```

我们还可以使用`getAll`方法获取一组值：
```java
Map<String, DataObject> dataObjectMap 
  = cache.getAll(Arrays.asList("A", "B", "C"));

assertEquals(3, dataObjectMap.size());
```

值从传递给`build`方法的底层后端初始化`Function`中读取到，这样就可以使用缓存作为访问值的主要入口了。

#### 3.3、异步加载

此策略的工作原理与前一个相同，但是会异步执行操作并返回一个`CompletableFuture`来保存实际的值：
```java
AsyncLoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .buildAsync(k -> DataObject.get("Data for " + k));
```
我们可以以相同的方式使用`get`和`getAll`方法，考虑到它们的返回是`CompletableFuture`：
```java
String key = "A";

cache.get(key).thenAccept(dataObject -> {
    assertNotNull(dataObject);
    assertEquals("Data for " + key, dataObject.getData());
});

cache.getAll(Arrays.asList("A", "B", "C"))
  .thenAccept(dataObjectMap -> assertEquals(3, dataObjectMap.size()));
```
CompletableFuture具有很多有用的API，您可以在本文中阅读更多相关信息。

### 4、缓存值的清理
Caffeine有三种缓存值的清理策略：基于大小、基于时间和基于引用。

#### 4.1、基于大小的清理

这种类型的清理设计为在超出缓存配置的大小限制时发生清理。有两种获取大小的方法——计算缓存中的对象数，或者获取它们的权重。

让我们看看如何计算缓存中的对象数。缓存初始化时，其大小为零：
```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(1)
  .build(k -> DataObject.get("Data for " + k));

assertEquals(0, cache.estimatedSize());
```

当我们添加一个值时，大小明显增加：
```java
cache.get("A");

assertEquals(1, cache.estimatedSize());
```

我们可以将第二个值添加到缓存中，这会导致删除第一个值：
```java
cache.get("B");
cache.cleanUp();

assertEquals(1, cache.estimatedSize());
```

值得一提的是，我们在获取缓存大小之前调用了`cleanUp`方法。这是因为缓存清理是异步执行的，该方法有助于等待清理完成。

我们还可以传入一个`weigher`的Function来定义缓存大小的获取：
```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumWeight(10)
  .weigher((k,v) -> 5)
  .build(k -> DataObject.get("Data for " + k));

assertEquals(0, cache.estimatedSize());

cache.get("A");
assertEquals(1, cache.estimatedSize());

cache.get("B");
assertEquals(2, cache.estimatedSize());
```

当权重超过 10 时，这些值将从缓存中删除：
```java
cache.get("C");
cache.cleanUp();

assertEquals(2, cache.estimatedSize());
```

#### 4.2、基于时间的清理

这种清理策略基于条目的过期时间，分为三种：

* 访问后过期——自上次读取或写入以来，条目在经过某段时间后过期
* 写入后过期——自上次写入以来，条目在经过某段时间后过期
* 自定义策略——由`Expiry`的实现来为每个条目单独计算到期时间

让我们使用`expireAfterAccess`方法配置访问后过期策略：
```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterAccess(5, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

要配置写入后过期策略，我们使用`expireAfterWrite`方法：
```java
cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .weakKeys()
  .weakValues()
  .build(k -> DataObject.get("Data for " + k));
```

要初始化自定义策略，我们需要实现Expiry接口：
```java
cache = Caffeine.newBuilder().expireAfter(new Expiry<String, DataObject>() {
    @Override
    public long expireAfterCreate(
      String key, DataObject value, long currentTime) {
        return value.getData().length() * 1000;
    }
    @Override
    public long expireAfterUpdate(
      String key, DataObject value, long currentTime, long currentDuration) {
        return currentDuration;
    }
    @Override
    public long expireAfterRead(
      String key, DataObject value, long currentTime, long currentDuration) {
        return currentDuration;
    }
}).build(k -> DataObject.get("Data for " + k));
```
#### 4.3、基于引用的清理

我们可以配置我们的缓存，允许缓存的键或值或二者一起的垃圾收集。为此，我们需要为键和值配置`WeakReference`的使用，并且我们可以配置`SoftReference`仅用于值的垃圾收集。

`WeakReference`的使用允许在没有对对象的任何强引用时对对象进行垃圾回收。`SoftReference`允许基于JVM的全局LRU（最近最少使用）策略对对象进行垃圾回收。可以在此处找到有关Java中引用的更多详细信息。

我们使用Caffeine.weakKeys()、Caffeine.weakValues()和Caffeine.softValues()来启用每个选项：
```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .weakKeys()
  .weakValues()
  .build(k -> DataObject.get("Data for " + k));

cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .softValues()
  .build(k -> DataObject.get("Data for " + k));
```

### 5、缓存刷新

可以将缓存配置为在定义的时间段后自动刷新条目。让我们看看如何使用refreshAfterWrite方法做到这一点：
```java
Caffeine.newBuilder()
  .refreshAfterWrite(1, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```
在这里，我们应该明白expireAfter和refreshAfter的一个区别：当请求过期条目时，执行会阻塞，直到build函数计算出新值。但是如果该条目符合刷新条件，则缓存将返回一个旧值并异步重新加载该值。

### 6、统计

Caffeine提供了一种记录缓存使用统计信息的方法：
```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .recordStats()
  .build(k -> DataObject.get("Data for " + k));
cache.get("A");
cache.get("A");

assertEquals(1, cache.stats().hitCount());
assertEquals(1, cache.stats().missCount());
```

我们还可以创建一个`StatsCounter`的实现作为参数来传入`recordStats`。每次与统计相关的更改，这个实现对象都将被调用。

### 7、结论

在本文中，我们熟悉了Java的Caffeine缓存库。我们看到了如何配置和存入缓存，以及如何根据需要选择合适的过期或刷新策略。