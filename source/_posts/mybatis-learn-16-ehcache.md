---
layout: post
title:  MyBatis学习笔记(16)-mybatis整合ehcache
date:   2016-03-08 10:39:16 +08:00
category: 入门系列笔记
tags: [MyBatis]
comments: true
---

ehcache是一个分布式缓存框架

<!-- more -->

## 分布缓存

我们系统为了提高系统并发，性能、一般对系统进行分布式部署（集群部署方式）

![分布缓存](http://blog.qiniu.brianway.site/mybatis_%E5%88%86%E5%B8%83%E7%BC%93%E5%AD%98.png)


不使用分布缓存，缓存的数据在各各服务单独存储，不方便系统开发。所以要使用分布式缓存对缓存数据进行集中管理。

mybatis无法实现分布式缓存，需要和其它分布式缓存框架进行整合。


## 整合方法(掌握)

mybatis提供了一个`cache`接口，如果要实现自己的缓存逻辑，实现`cache`接口开发即可。

mybatis和ehcache整合，mybatis和ehcache整合包中提供了一个cache接口的实现类。


```java
package org.apache.ibatis.cache;

import java.util.concurrent.locks.ReadWriteLock;

/**
 * SPI for cache providers.
 *
 * One instance of cache will be created for each namespace.
 *
 * The cache implementation must have a constructor that receives the cache id as an String parameter.
 *
 * MyBatis will pass the namespace as id to the constructor.
 *
 * <pre>
 * public MyCache(final String id) {
 *  if (id == null) {
 *    throw new IllegalArgumentException("Cache instances require an ID");
 *  }
 *  this.id = id;
 *  initialize();
 * }
 * </pre>
 *
 * @author Clinton Begin
 */

public interface Cache {

  /**
   * @return The identifier of this cache
   */
  String getId();

  /**
   * @param key Can be any object but usually it is a {@link CacheKey}
   * @param value The result of a select.
   */
  void putObject(Object key, Object value);

  /**
   * @param key The key
   * @return The object stored in the cache.
   */
  Object getObject(Object key);

  /**
   * Optional. It is not called by the core.
   *
   * @param key The key
   * @return The object that was removed
   */
  Object removeObject(Object key);

  /**
   * Clears this cache instance
   */  
  void clear();

  /**
   * Optional. This method is not called by the core.
   *
   * @return The number of elements stored in the cache (not its capacity).
   */
  int getSize();

  /**
   * Optional. As of 3.2.6 this method is no longer called by the core.
   *  
   * Any locking needed by the cache must be provided internally by the cache provider.
   *
   * @return A ReadWriteLock
   */
  ReadWriteLock getReadWriteLock();

}
```


mybatis默认实现cache类是：

```java
package org.apache.ibatis.cache.impl;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;

import org.apache.ibatis.cache.Cache;
import org.apache.ibatis.cache.CacheException;

/**
 * @author Clinton Begin
 */
public class PerpetualCache implements Cache {

  private String id;

  private Map<Object, Object> cache = new HashMap<Object, Object>();

  public PerpetualCache(String id) {
    this.id = id;
  }

  public String getId() {
    return id;
  }

  public int getSize() {
    return cache.size();
  }

  public void putObject(Object key, Object value) {
    cache.put(key, value);
  }

  public Object getObject(Object key) {
    return cache.get(key);
  }

  public Object removeObject(Object key) {
    return cache.remove(key);
  }

  public void clear() {
    cache.clear();
  }

  public ReadWriteLock getReadWriteLock() {
    return null;
  }

  public boolean equals(Object o) {
    if (getId() == null) throw new CacheException("Cache instances require an ID.");
    if (this == o) return true;
    if (!(o instanceof Cache)) return false;

    Cache otherCache = (Cache) o;
    return getId().equals(otherCache.getId());
  }

  public int hashCode() {
    if (getId() == null) throw new CacheException("Cache instances require an ID.");
    return getId().hashCode();
  }

}
```

### 整合ehcache

- 加入ehcache包
   - ehcache-core-2.6.5.jar
   - mybatis-ehcache-1.0.2.jar

配置mapper中`cache`中的`type`为ehcache对cache接口的实现类型

```xml
 <!-- 开启本mapper的namespace下的二级缓存
    type：指定cache接口的实现类的类型，mybatis默认使用PerpetualCache
    要和ehcache整合，需要配置type为ehcache实现cache接口的类型
    <cache />
    -->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

### 加入ehcache的配置文件

在classpath下配置ehcache.xml


```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	<diskStore path="F:\develop\ehcache" />
	<defaultCache
		maxElementsInMemory="1000"
		maxElementsOnDisk="10000000"
		eternal="false"
		overflowToDisk="false"
		timeToIdleSeconds="120"
		timeToLiveSeconds="120"
		diskExpiryThreadIntervalSeconds="120"
		memoryStoreEvictionPolicy="LRU">
	</defaultCache>
</ehcache>
```
