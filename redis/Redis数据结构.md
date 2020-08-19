# Redis数据结构

Redis定义了`redisObject`结构来表示string、hash、list、set、zset五种数据结构

redisObject C语言定义如下

```c
 typedef struct redisObject {
     // 数据类型，可以通过命令 type key查看
     unsigned type:4;
     // 底层的编码，可以通过命令 object encoding key查看
     unsigned encoding:4;
     // 用作内存回收
     unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */
     // 对象的引用次数
     int refcount;
     // 指向真正的存储结构
     void *ptr;
 } robj;
```



## string

 Redis 的字符串叫着「SDS」，也就是`Simple Dynamic String` ； 它的结构是一个带长度信息的字节数组；字符串是可以修改的。 

```c
struct SDS<T> {
  T capacity; // 数组容量
  T len; // 数组长度
  byte flags; // 特殊标识位，不理睬它
  byte[] content; // 数组内容
}
```

 ![img](Redis数据结构.assets/164db13445631ab4)

  Redis 规定字符串的长度不得超过 512M 字节 

- int 8个字节的长整形（long，2^63-1）
- embstr 存储 <=44个字节的字符串
- raw 存储>44个字节的字符串

### embstr vs raw

 ![img](Redis数据结构.assets/164db4dcdac7e7f9) 

 如图所示，`embstr` 存储形式是这样一种存储形式，它将 RedisObject 对象头和 SDS 对象连续存在一起，使用 `malloc` 方法一次分配。而 `raw` 存储形式不一样，它需要两次 `malloc`，两个对象头在内存地址上一般是不连续的。 

而内存分配器 jemalloc/tcmalloc 等分配内存大小的单位都是 2、4、8、16、32、64 等等，为了能容纳一个完整的 `embstr` 对象，`jemalloc` 最少会分配 32 字节的空间，如果字符串再稍微长一点，那就是 64 字节的空间。如果总体超出了 64 字节，Redis 认为它是一个大字符串，不再使用 `emdstr` 形式存储，而该用 `raw` 形式。

当内存分配器分配了 64 空间时，那这个字符串的长度最大可以是多少呢？这个长度就是 44。那为什么是 44 呢？

前面我们提到 SDS 结构体中的 `content` 中的字符串是以字节`\0`结尾的字符串，之所以多出这样一个字节，是为了便于直接使用 `glibc` 的字符串处理函数，以及为了便于字符串的调试打印输出。

 ![img](Redis数据结构.assets/164db590af5e8551)

 看上面这张图可以算出，留给 `content` 的长度最多只有 45(64-19) 字节了。字符串又是以`\0`结尾，所以 `embstr` 最大能容纳的字符串长度就是 44。 

 **注意**：emstr编码时只读的；当需要修改时会转换成raw，不管是否达到了44个字节；

 

## hash

- ziplist 压缩列表
- ht(hashtable)
  - 类似Java中hashmap
  - 有两个链表，方便一次进行部分rehash，减少rehash时间

### ziplist 压缩列表

- entry的 个数< 512 and value < 64 byte；可以在redis.confz中配置

- 示例图![img](https://user-gold-cdn.xitu.io/2018/7/28/164df01c1c7579e7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- C 语言定义的结构信息

  ```c
  struct ziplist<T> {
      int32 zlbytes; // 整个压缩列表占用字节数
      // 最后一个元素距离压缩列表起始位置的偏移量，用于快速定位到最后一个节点，方便倒着遍历对象
      int32 zltail_offset; 
      int16 zllength; // 元素个数
      T[] entries; // 元素内容列表，挨个挨个紧凑存储
      int8 zlend; // 标志压缩列表的结束，值恒为 0xFF
  }
  
  struct entry {
      // 前一个 entry 的字节长度，倒着遍历时会用到
      int<var> prevlen; 
      // 元素类型编码
      int<var> encoding;
      // 元素内容
      optional byte[] content;
  }
  ```

  

## list

redis 3.2版本以前  元素少时用  `ziplist` ，多时用 `linkedlist`

redis3.2版本之后统一使用  `quicklist`

### quicklist结构

```c
struct quicklist {
    quicklistNode* head;
    quicklistNode* tail;
    unsigned long count; // 元素总数
    unsigned long len; // quicklistNode 节点的个数
    int fill : 16;
    unsigned int compress : 16;
}

struct quicklistNode {
    quicklistNode* prev;
    quicklistNode* next;
    unsigned char* zl; // ziplist* zl 指向压缩列表
    unsigned int sz; // ziplist 的字节总数
    unsigned int count; // ziplist 中的元素数量
    unsigned int encoding; // 存储形式 2bit，原生字节数组还是 LZF 压缩存储
    ...
}
```

 ![img](Redis数据结构.assets/164e3b0b953f2fc7) 

#### 压缩深度

 ![img](Redis数据结构.assets/164e3d168aa62cc9) 

 quicklist 默认的压缩深度是 0，也就是不压缩。压缩的实际深度由配置参数`list-compress-depth`决定。为了支持快速的 push/pop 操作，quicklist 的首尾两个 ziplist 不压缩，此时深度就是 1。如果深度为 2，就表示 quicklist 的首尾第一个 ziplist 以及首尾第二个 ziplist 都不压缩。 



## set 无序的集合

- intset 存储整形 and 元素个数 < 512(可以在配置文件中配置)
- hashtable (value = null)

## zset 有序集合

- ziplist 总个数<128 and member< 64byte (可以在redis.config中修改)

  - 紧挨在一起的压缩列表节点保存，第一个节点保存member，第二个节点保存score

- skiplist编码底层命名为zet，它同时包含hash 和 跳跃表（skiplist）

  - 整体结构 ![img](Redis数据结构.assets/164d9cd9064b556e)

    

  - skiplist跳表结构 ![img](Redis数据结构.assets/164d9f96ed4e1a0d) 

