---
title: 使用Redis实现文章阅读量、收藏、点赞数量记录功能
tags: java redis

---

## 方案1:三个数据缓存都分别使用 字符串 结构计数器存储对应数量值

鉴于上面所说，这里我们将阅读量、收藏、点赞数量使用三个字符串结构存储，不同业务操作对应key即可，还要注意这些数据入数据库操作。

1、Key定义规则

阅读量key：article:readcount:文章id
收藏数量key：article:collectcount:文章id
点赞数量key：article:likecount:文章id
2、代码实现
这里直接使用计数器操作即可，每次有对应操作只要对具体key做自增或者自减即可，假设文章id=1

```
## 文章每次被阅读，文章阅读量自增1
127.0.0.1:6379> incr article:readcount:1
## 文章每次被收藏，文章收藏数量自增1
127.0.0.1:6379> incr article:collectcount:1
## 文章每次被点赞，文章点赞数量自增1
127.0.0.1:6379> incr article:likecount:1

## 文章每次被取消收藏，文章收藏数量自减1
127.0.0.1:6379> decr article:collectcount:1
## 文章每次被取消点赞，文章点赞数量自减1
127.0.0.1:6379> decr article:likecount:1

## 获取文章阅读量、收藏、点赞数量
127.0.0.1:6379> get article:readcount:1
127.0.0.1:6379> get article:collectcount:1
127.0.0.1:6379> get article:likecount:1

```

- 3、优点
  - 实现简单、能减轻统计查询时的数据库压力
- 4、不足点
  - 需要使用非常多的key，一篇文章就要使用三个key，而且每次查看文章都需要请求Redis三次才能获取到这三个数量值，对Redis资源开销比较大，可以使用Hash结构优化。



## 方案2:三个数据缓存使用一个 Hash 结构存储

针对3.1中因为需要使用非常多的key，一篇文章就要使用三个key，而且每次查看文章都需要请求Redis三次才能获取到这三个数量值，对Redis资源开销比较大，这里可以使用Hash结构优化。

1、Key定义规则

文章阅读、收藏、点赞Hash结构 key：article:count:文章id
Hash结构中阅读量key：readcount
Hash结构中收藏数量key：collectcount
Hash结构中点赞数量key：likecount
2、代码实现
这里直接Hash 结构操作即可，每次有对应操作只要对具体Hash结构中的key做自增或者自减即可，假设文章id=1

```
## 文章每次被阅读，文章阅读量自增1
127.0.0.1:6379> hincrby article:count:1 readcount 1
## 文章每次被收藏，文章收藏数量自增1
127.0.0.1:6379> hincrby article:count:1 collectcount 1
## 文章每次被点赞，文章点赞数量自增1
127.0.0.1:6379> hincrby article:count:1 likecount 1

## 文章每次被取消收藏，文章收藏数量自减1
127.0.0.1:6379> hincrby article:count:1 collectcount -1
## 文章每次被取消点赞，文章点赞数量自减1
127.0.0.1:6379> hincrby article:count:1 likecount -1

## 获取文章阅读量、收藏、点赞数量
127.0.0.1:6379> hgetall article:count:1

```

- 3、优点
  - 相比于3.1中实现简单、能有效减低Redis资源开销
- 4、不足点
  - 还是有一点问题没有解决，如果要查询阅读用户和文章的收藏和点赞关系，那么还是要查询数据库，数据库压力还是比较大，在下面3.3中会针对这个问题使用 Set 集合解决。



## 方案3:阅读量使用字符串结构计算器，收藏和点赞分别使用 Set 集合存储

为了解决查询阅读用户和文章的收藏和点赞关系，还需要查询数据库问题，这里采用Set集合来存储文章收藏点赞和用户的关联关系。

1、Key定义规则

阅读量key：article:readcount:文章id
收藏数量Set key：article:collectuser:文章id
点赞数量Set key：article:likeuser:文章id
2、代码实现
阅读量还是普通结构和3.1一致，收藏和点赞使用Set集合存储文章和用户对应关系，每次收藏点赞时将用户ID加入到对应Set集合中，假设文章id=1。

```
## 文章每次被阅读，文章阅读量自增1
127.0.0.1:6379> incr article:readcount:1
## 文章每次被收藏，将用户ID加入到文章收藏的Set集合中，假设用户ID为1001
127.0.0.1:6379> sadd article:collectuser:1 1001
## 文章每次被点赞，将用户ID加入到文章点赞的Set集合中，假设用户ID为1001
127.0.0.1:6379> sadd article:likeuser:1 1001

## 文章每次被取消收藏，将用户ID从文章收藏的Set集合中删除
127.0.0.1:6379> srem article:collectuser:1 1001
## 文章每次被取消点赞，将用户ID从文章点赞的Set集合中删除
127.0.0.1:6379> srem article:likeuser:1 1001

## 获取文章阅读量、收藏、点赞数量
### 获取阅读量
127.0.0.1:6379> get article:readcount:1
### 获取收藏数量
127.0.0.1:6379> scard article:collectuser:1
### 获取点赞数量
127.0.0.1:6379> scard article:likeuser:1

## 获取当前阅读用户是否收藏、点赞当前文章
### 获取用户是否收藏当前文章
127.0.0.1:6379> sismember article:collectuser:1 1001
### 获取用户是否点赞当前文章
127.0.0.1:6379> sismember article:likeuser:1 1001

```

3、优点

对于阅读量、收藏、点赞相关所有的查询操作都可以通过Redis获取，大大降低了数据库压力，并且还会再Redis中维护文章和用户的收藏、点赞关系，查询用户是否收藏点赞也不要再去查询数据库。
4、不足点

事情都有两面性，使用Redis来缓存这些信息虽然能大大提升系统整体性能，但是可能出现缓存与数据库数据不一致问题，还有缓存过期时间问题，如果一篇文章一个月才被访问一次那么不可能一直缓存这篇文章的信息，需要设置一个删除时间，要想比较完善的实现那么设计这个功能复杂度是成倍增长的。

## 后续优化

### 1.查询优化

为了进一步提升性能力，来做下一步优化，判断不存在之后，先不急于更新数据库，先在 Redis 里给这篇文章的浏览量+1，Key 为 viewCount:articleId，value 为缓存的浏览量。然后设置一个定时任务，定时更新 Redis 缓存数据到数据库。

### 2.使用Redis HyperLogLog

她是用来做基数统计的算法，优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

redis工具类

```java
/**
 * 计数
 *  https://blog.52itstyle.vip
 * @param key
 * @param value
 */
public void add(String key, Object... value) {
	redisTemplate.opsForHyperLogLog().add(key,valu);
}
/**
  * 获取总数
  *  https://blog.52itstyle.vip
  * @param key
  */
public Long size(String key) {
	return redisTemplate.opsForHyperLogLog().size(key);
}

```

aop

```java
@Around("ServiceAspect()")
public  Object around(ProceedingJoinPoint joinPoint) {
     Object[] object = joinPoint.getArgs();
     Object blogId = object[0];
     Object obj = null;
     try {
         String value = IPUtils.getIpAddr();
         String key = "viewCount:" + blogId;
		 // key 为 文章ID，Value 为请求IP地址
         redisUtil.add(key,value);
         obj = joinPoint.proceed();
     } catch (Throwable e) {
         e.printStackTrace();
     }
     return obj;
}

```

定时任务

```java
@Scheduled(cron = "0 30 23 * * ?")
public void createHyperLog() {
     logger.info("计数落库开始");
     String nativeSql = "SELECT id FROM blog";
     List<Object> list = dynamicQuery.query(nativeSql,new Object[]{});
     list.forEach(blogId ->{
         String key  = "viewCount:"+blogId;
         Long views = redisUtil.size(key);
         if(views>0){
             String updateSql = "UPDATE blog SET views=views+? WHERE id=?";
                dynamicQuery.nativeExecuteUpdate(updateSql,new Object[]{views,blogId});
                redisUtil.del(key);
         }
     });
     logger.info("计数落库结束");
}

```

博文请求

```java
/**
  * 博文
  *  https://blog.52itstyle.vip
  */
@RequestMapping("{id}.shtml")
public String page(@PathVariable("id") Long id, ModelMap model) {
     try{
         Blog blog = blogService.getById(id);
         String key = "viewCount:"+id;
         Long views = redisUtil.size(key);
		 //直接从缓存中获取并与之前的数量相加
         blog.setViews(views+blog.getViews());
         model.addAttribute("blog",blog);
     } catch (Throwable e) {
         return  "error/404";
     }
     return  "article";
}

```

业务代码

```java
/**
  *  https://blog.52itstyle.vip
  * 执行顺序
  * 1）限流
  * 2）布隆
  * 3）计数
  * 4) 缓存
  * @param id
  * @return
  */
@Override
@ServiceLimit(limitType= ServiceLimit.LimitType.IP)
@BloomLimit
@HyperLogLimit
@Cacheable(cacheNames ="blog")
public Blog getById(Long id) {
     String nativeSql = "SELECT * FROM blog WHERE id=?";
     return dynamicQuery.nativeQuerySingleResult(Blog.class,nativeSql,new Object[]{id});
}

```

