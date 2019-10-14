# Redis 数据持久化

redis虽然数据都是放在内存中的，但是也支持持久化，这样可以防止数据丢失

有两种持久化方式

- rdb (Redis Database 是一种二进制存储方式，存储的是二进制数据) (默认的持久化机制)
- AOF 存储操作的命令

## rdb

默认仅开启了rdb方式，

### 配置文件说明

```
####### SNAPSHOTTING ######### 配置文件中SNAPSHOTTING包围部分
# 保存方式 如果900s内一个key改变了会触发保存
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
# 开启rdb文件压缩，会减小rdb文件大小，但是会消耗更多的cpu资源
rdbcompression yes
rdbchecksum yes
# rdb文件名称
dbfilename dump.rdb
# rdb文件目录
dir /var/lib/redis
```

### 自动触发保存

可以在配置文件中配置，参考上面的配置文件说明部分

```
save 900 1
save 300 10
save 60 10000
```



### 如何手动触发保存

- save 会阻塞redis服务。如果数据比较多，就不能处理其他的命令了，所以线上一般都不用
- flushall 清空数据命令也会触发数据保存(清空后的数据，也就是空内容)
- kill redis进程也会触发redis数据保存(-9 强制杀死进程除外)，当然这种方式不推荐
- shutdown save命令(在redis-cli中执行的关闭服务命令)也会触发数据保存
- bgsave 异步产生快照保存数据，不会阻塞redis服务；

#### busave原理

通过fock字进程的方式，保存的是fock时内存的快照；fock开始后的命令不会被存储



### rdb保存方式的优缺点

优势

- 可以压缩
- bgsave不会对主进程产生影响

劣势

- 不能精准的备份

### 查看最后生成快照的时间

lastsave

## aof

这种方式默认式关闭的

### 配置文件说明

```
#### APPEND ONLY MODE #####
# 是否开启aof方式
appendonly no
# aof文件名称，**保存路径和rdb相同**
appendfilename "appendonly.aof"
#aof保存方式，最多只会丢失1s的数据
appendfsync everysec
no-appendfsync-on-rewrite no
# aof文件重写配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
```

### 自动触发保存(持久化)

```
appendfsync everysec
```

参考上面的配置文件说明

### aof(Append Only File)文件重写

由于aof会保存用户所有的写操作，即使数据的值并不会发生改变，就导致了文件可能会变得很大；

系统提供了重写机制来减少文件的大小(不是整理aof文件，而是从redis中读取数据转换成aof指令的方式重写aof文件，在生成aof文件的过程中的操作会保存到aof文件缓存中，最后合并到重写的aof文件中)



BGREWRITEAOF 命令可以手动触发后台aof文件重写



**aof 和 rdb方式可以同时启用，aof方式更精确（因为只会丢失1s的数据），如果aof方式启用了，redis在启动时会优先使用aof方式恢复数据**

### aof保存方式的优缺点

优点

- 同步频率可以比较高，精准（最多相差1s的数据）

缺点

- 文本方式保存，文件会越来越大