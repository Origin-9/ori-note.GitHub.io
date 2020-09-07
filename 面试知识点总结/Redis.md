## 介绍Redis

Redis是使用C语言开发的数据库，不同于传统数据库，Redis的数据存在内存中，读写速度非常快。

被广泛应用在缓存方向。提供多种数据类型来支持不同的业务场景，Redis还支持事务，集群方案，哨兵，持久化，Lua脚本等。

## Redis和Memcached区别

### 共同点

1. 基于内存的数据库，一般用来当作缓存使用
2. 都有过期策略
3. 两者的性能都高

### 区别

1. Redis支持更丰富的数据类型，Memcached只支持最简单的k/v数据类型
2. Redis支持持久化，Memcached数据存储在内存之中
3. Redis有灾难恢复机制
4. Redis服务器内存使用完之后，可以将不用的数据放在磁盘上，Memcached会抛异常
5. Memcached没有原生的集群模式
6. Memcached是多线程的，非阻塞IO复用的网络模型，Redis是单线程多路IO复用模型
7. Redis支持发布订阅模型，Lua脚本，事务等功能，支持更多的编程语言
8. Memcached过期数据删除策略只用了惰性删除，Redis使用了惰性删除和定期删除

## Redis数据结构

![image-20200905231746050](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905231746050.png)

![image-20200905224321609](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905224321609.png)

### string

**常用命令:** `set,get,strlen,exists,dect,incr,setex` 等等。

**应用场景** ：一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

- 整数值

  ![image-20200905234422123](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905234422123.png)

- embstr（字符串长度小于32字节）

  ![image-20200905234441954](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905234441954.png)

- raw（SDS动态字符串）

  ![image-20200905234433533](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905234433533.png)

int，embstr会在一定条件下，转换成raw编码的字符串（例如append）

### list

 **双向链表**，即可以支持反向查找和遍历，插入删除方便，随机访问困难

**常用命令:** `rpush,lpop,lpush,rpop,lrange、llen` 等。

- 压缩列表

![image-20200905235108902](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905235108902.png)

- 双端链表

![image-20200905235131308](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905235131308.png)

列表对象使用压缩链表条件：字符串元素长度小于64字节；元素个数小于512个

### hash

hash 类似于 JDK1.8 前的 HashMap，内部实现也差不多(数组 + 链表)

**常用命令：** `hset,hmset,hexists,hget,hgetall,hkeys,hvals` 等。

**应用场景:** 系统中对象数据的存储。

- 压缩列表

![image-20200905235705603](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905235705603.png)

![image-20200905235713286](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905235713286.png)

- 字典

![image-20200905235811175](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200905235811175.png)

压缩链表条件：字符串元素长度小于64字节；元素个数小于512个

### set

set 类似于 Java 中的 `HashSet` 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序

**常用命令：** `sadd,spop,smembers,sismember,scard,sinterstore,sunion` 等。

**应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景

- 整数集合（灵活性，数值的大小升级为32位，64位int数值）

![image-20200906000619911](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906000619911.png)

- 字典

![image-20200906000628079](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906000628079.png)

整数集合条件：元素都为整数，个数小于512个

### sorted set

和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列

**常用命令：** `zadd,zcard,zscore,zrange,zrevrange,zrem` 等

**应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

- 压缩列表

![image-20200906001118650](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906001118650.png)

![image-20200906001126397](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906001126397.png)

- 跳跃表和字典

![image-20200906105816418](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906105816418.png)

压缩列表使用条件：元素个数小于128，每个元素大小小于64字节

**同时使用跳跃表和字典的原因：**

只使用跳跃表，范围操作优点会被保留，查询成员score复杂度上升![image-20200906110537704](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906110537704.png)

只使用字典，![image-20200906110628119](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906110628119.png)复杂度查询成员score，范围操作![image-20200906110657438](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200906110657438.png)

### 跳跃表

跳跃表以有序的方式在层次化的链表中保存元素， 效率和平衡树媲美。插入，查找，删除操作的平均效率都为 O(logn)。

跳跃表主要结构：

- 表头：维护跳跃表节点指针
- 跳跃表节点：保存元素值，以及多个层
- 层：保存指向其他元素的指针，高层的指针越过底层的指针，查找总从高层次访问，然后随着元素值范围缩小，慢慢降低层次。
- 表尾：全部为null，表示表尾

> 插入节点层数：
>
> 跳跃列表采取一个随机策略来决定新元素可以兼职到第几层，首先L0层肯定是100%了，L1层只有50%的概率，L2层只有25%的概率，L3层只有12.5%的概率，一直随机到最顶层L31层。

Redis的跳跃表：

1. 允许重复score，重复之后比较member

2. 每个节点有一个后退指针，用于表尾向表头迭代

## Redis为什么快

1. 操作完全基于内存，速度快
2. 数据结构简单，对数据的操作也简单
3. 采用单线程，避免了不必要的上下文切换
4. 使用非阻塞的多路IO复用模型

## Redis缓存数据设置过期时间

### 有什么用

内存有限，缓存中所有数据一直保存，会导致 out of memory

Redis自带给缓存数据设置过期时间的功能

字符串类型独有的设置过期时间命令setex，其他都要依靠expire，pexpire，expireAt，pexpireAt等命令设置

我们的业务场景就是需要某个数据只在某一时间段内存在，比如我们的短信验证码可能只在1分钟内有效，用户登录的 token 可能只在 1 天内有效。

### Redis怎么判断键过期

Redis 通过一个叫做过期字典（可以看作是hash表）来保存数据过期的时间。过期字典的键指向Redis数据库中的某个key(键)，过期字典的值是一个long long类型的整数，这个整数保存了key所指向的数据库键的过期时间

### 过期键的删除策略

- 定期删除

  每隔一段时间，抽取一批 key 执行删除过期key操作

- 懒惰删除

  只在取出key的时候，判断key值是否过期

定期删除对内存友好，惰性删除对CPU良好。Redis采用定期删除+惰性删除

仅仅给key键设置过期时间是不够的，还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况，大量堆积还是会导致out of memory，**Redis 内存淘汰机制**可以解决这个问题

### Redis内存淘汰机制

- no-eviction：内存不足，报错
- allkeys-random：随机删除键
- allkeys-lru：内存不足，删除最近最少使用的key
- volatile-random：在设置过期键中随机删除
- volatile-ttl：在设置过期键中删除快过期的
- volatile-lru：在设置过期间中删除最近最少使用的key

4.0版本之后增加了：

- volatile-lfu：在设置过期间中最不经常使用的
- allkeys-lfu：在所有键中最不经常使用的

## 持久化

将内存中的数据写入到硬盘里面，大部分原因是为了之后**重用数据**（比如重启机器、机器故障之后恢复数据），或者是为了防止系统故障而将**数据备份**到一个远程位置。

### RDB

Redis 可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本，还可以将快照留在原地以便重启服务器的时候使用。

save命令阻塞服务器进程，bgsave子进程执行，不会阻塞，bgsave执行条件可以在Redis.conf中设置，例如多少秒多少key变化，执行bgsave。

### AOF

开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件。

命令请求会先保存到AOF缓冲区里，在定期写入AOF文件当中

三种AOF持久化方式：

- appendfsync always：每次数据发生修改都会重写
- appendfsync everysec：每分钟同步一次，显示将多个写命令同步到磁盘
- appendfsync no：让操作系统决定

#### AOF重写

AOF重写，会产生一个新的AOF文件，这个文件和原本AOF文件数据库状态一样，文件更小

AOF重写是通过读取数据库中键值对实现的，

**在执行 BGREWRITEAOF 命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子进程创建新 AOF 文件期间，记录服务器执行的所有写命令。当子进程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF 文件的末尾，使得新旧两个 AOF 文件所保存的数据库状态一致。最后，服务器用新的 AOF 文件替换旧的 AOF 文件，以此来完成 AOF 文件重写操作**

## 事务

MULTI, EXEC, DISCARD and WATCH 是Redis事务的基础

事务中的所有命令都会被序列化并按顺序执行，在整形Redis事务时，不会接受另一客户端的请求，保证任务队列作为一个单一的原子操作执行

### 用法

使用`MULTI`命令`显式开启`Redis事务。 该命令总是以OK回应。`此时用户可以发出多个命令，Redis不会执行这些命令，而是将它们排队`。`EXEC`被调用后，所有的命令都会被执行。而调用`DISCARD`可以`清除`事务中的`commands队列`并`退出事务`。

### 事务中的错误

#### 入队错误

命令不存在，命令格式问题

- 在`Redis 2.6.5之前`，这种情况下，在`EXEC`命令调用后，客户端会执行命令的子集（成功排队的命令）而忽略之前的错误。

- 从`Redis 2.6.5开始`，服务端会记住在累积命令期间发生的错误，当`EXEC`命令调用时，`将拒绝执行事务，并返回这些错误，同时自动清除命令队列`。

#### 执行错误

执行过程中发生错误，一般是对数据库键执行了错误类型的操作

服务器不会中断事务的执行，会继续执行下边的命令，之前执行成功的命令也不会被影响

### Redis不支持Rollback

`Redis命令`可能会执行失败，仅仅是由于错误的语法被调用（命令排队时检测不出来的错误），或者使用错误的数据类型操作某个`Key`： 这意味着，实际上失败的命令都是编程错误造成的，都是开发中能够被检测出来的，生产环境中不应该存在。

#### watch

watch监听任意数量的键， 如果任意一个被监视的键已经被其他客户端修改了， 那么整个事务不再执行， 直接返回失败。

#### watch命令实现

watched_keys字典，保存被监视的键，字典的值（链表）保存所有监视这个键的客户端

当客户端对数据库键修改之后，序将所有监视这个/这些被修改键的客户端的 `REDIS_DIRTY_CAS` 选项打开

当客户端发送 EXEC 命令、触发事务执行时，会检查REDIS_DIRTY_CAS，如果被打开，就放弃执行事务

## 缓存穿透

缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。

### 解决办法

1. 参数校验，不合法的参数，直接抛出异常信息返回给客户端

2. 缓存无效 key

   如果缓存和数据库都查不到，则在数据库中设置过期时间。这种方式可以解决请求的 key 变化不频繁的情况。

   如果大量不同的键，用这种方法的话，就将无效key缓存，设置较短过期时间

3. 布隆过滤器

   把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话再去查缓存，数据库

## 缓存雪崩

缓存在同一时间大面积失效，所有的请求落在数据库上，导致数据库崩掉

缓存同一时间失效可能是Redis宕机或者大量key设置了同样的过期时间，同时失效

### 解决方法

Redis集群，key值过期时间错开，限流，利用持久化机制恢复缓存

## 集群方案

https://segmentfault.com/a/1190000022028642

### 主从复制

主从复制模式中包含一个主数据库实例（master）与一个或多个从数据库实例（slave）

客户端可对主数据库进行读写操作，对从数据库进行读操作，主数据库写入的数据会实时自动同步给从数据库。

具体工作机制：

首先主从同步，slave向master发送SYNC命令，master接收到SYNC命令后，