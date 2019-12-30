# redis

## install

### sentinel

**源码包中提供了sentinel.conf配置示例，可以参考配置**

### Cluster

拓展性，单个redis内存不足时，使用cluster分片存储

## 基本数据结构

- string
- list
- set
- hash
- zset(有序集合)

## string

### 键值对

> set key value

### 批量设置和获取

> mset name1 value name2 value
>
> mget name1 name2

### 2.6.12 set 命令增加了一系列选项

- EX seconds  - 设置过期时间，单位 秒
- PX milliseconds - 设置过期时间，单位毫秒
- NX - 当key值不存在时设置值
- XX - 当key值存在时设置值

> set key value [EX seconds] [PX milliseconds] [NX|XX]

### 计数

如果value值是整数(或者是整数字符串)，可以进行自增/减操作

> set age 1
>
> incr age (当变量没有定义时，默认为0)
>
> incrby age 5
>
> 
>
> decr age
>
> decrby age

## list 列表

redis里面的list类似Java中的LinkedList。也就是插入删除操作很快（O(1)），但是索引速度很慢（O(n)）

![list](redis.assets/1645918c2cdf772e)





### 先进先出：队列

- 右边进左边出
- 左边出右边近



### 后进先出：栈

- 右边进右边出
- 左边进左边出

### 慢操作

- lindex

> O(n)

- ltrim

> O(n)

- lrange

> lrange books 0 -1  # 获取所有元素，O(n) 慎用

### 实现原理

快速队列

![img](redis.assets/164d975cac9559c5)

## hash 字典

### 实现细节

类似Java 中 HashMap(jdk 8 之前的版本)

### rehash策略

一次全部hash很耗时，采用渐进式rehash策略

![img](redis.assets/164dc873b2a899a8)

### 相关命令

> hset books java "think in java"
>
> hset books golang "think in java"
>
> hgetall books
>
> hlen books
>
> hset books golang
>
> hget books golang



## set 集合

特殊的字典，value = null

不重复

### 相关命令

> http://www.redis.cn/commands.html#set
>
> 添加
>
> sadd books python
>
> 获取所有元素（无序）
>
> smembers books
>
> 查询某个元素是否存在
>
> sismember books java
>
> 获取books的长度
>
> scard books
>
> 弹出一个
>
> spop books
>
> 删除一个或多个元素
>
> srem books member

## zset 有序集合

- set集合，保证的value的唯一性
- 每个value有一个权重，代表value的排序权重

### 主要命令

> 添加
>
> zadd books score member
>
> 按照score排序
>
> zrange books 0 -1
>
> 按照score逆序
>
> zrevrange books 0 -1
>
> 获取score
>
> zscore books "java concurrency"
>
> 删除
>
> zrem books "xx"
>
> 根据分值区间遍历 zset
>
> zrangebyscore books -inf 8.91