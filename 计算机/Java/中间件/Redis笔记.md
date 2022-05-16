# Redis

## 一、概述

### 1.1 Redis特性

+ Redis致命命令的速度快
+ 基于键值对的数据结构服务
+ 丰富的功能
  + 提供了键过期功能，可以用来实现缓存
  + 提供了发布订阅功能，可以用来实现消息系统
  + ...
+ 简单稳定
  + 代码量少
+ 客户端语言多
  + Redis提供了简单的TCP通信协议，很多编程语言可以很方便的接入到Redis
+ 持久化
  + 通常将数据存在在内存中是不安全的，一旦发生断电或者机器故障，重要的数据可能就会丢失，Redis提供了两种持久化方式：`AOF`和`RDB`，可以用这两种策略将内存的数据保存到硬盘中，这样就保证了数据的可持久性
+ 主从复制
  + Redis提供了复制功能，实现了多个相同的数据的Redis副本
+ 高可用和分布式



### 1.2 Redis使用场景

一些需要频繁操作的数据可以放在redis中



## 二、基本命令
准备工作：插入三条数据

```shell
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> set java jedis
OK
127.0.0.1:6379> set python redis-py
OK
```

（1）`keys *`：查看所有键（**线上环境禁止使用**）

```shell
127.0.0.1:6379> keys *
1) "hello"
2) "python"
3) "java"
```

> 注：
>
> keys命令会遍历所有键，时间复杂度为*O(n)*，线上环境**禁止使用**



（2）`dbsize`：获取键的总数

```shell
127.0.0.1:6379> dbsize
(integer) 3
```

> 注：
>
> dbsize命令在计算键总数时不会遍历所有键，而是直接获取Redis内置的键总数变量，所以dbsize命令的时间复杂度为*O(1)*



（3）`exists key`：检查key是否存在，存在返回1，不存在返回0

```shell
127.0.0.1:6379> exists java
(integer) 1
127.0.0.1:6379> exists python
(integer) 1
127.0.0.1:6379> exists C
(integer) 0
```



（4）`del key [key ...]`：删除键，支持删除多个键，返回值为删除的个数

```shell
127.0.0.1:6379> del java
(integer) 1
127.0.0.1:6379> del C
(integer) 0
127.0.0.1:6379> del hello python
(integer) 2
```



（5）`expire key seconds`：设置键的过期时间

```shell
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> expire hello 10
(integer) 1
```



通过`ttl key`命令返回键的剩余过期时间，它有三种返回值

+ 大于等于0的整数：键剩余的过期时间
+ -1：键没有设置过期时间
+ -2：键不存在



（6）`type key`：键的数据类型

```shell
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> type hello
string
```

> 注：如果键不存在，会返回none



## 三、键的管理

### 3.1 单个键管理



### 3.2 遍历键



### 3.3 数据库管理



## 四、基本数据类型

### 4.1 字符串

#### 4.1.1 概述

所有的键都是字符串类型，字符串类型的值实际可以是字符串（简单的字符串、复杂的字符串（例如JSON、XML））、数字（整数、浮点数），甚至是二进制（图片、音频、视频），**但是最大值不能超过512MB**



#### 4.1.2 命令

（1） `set key value [ex seconds] [px milliseconds] [nx|xx]`

set命令有几个选项（option）：

+ `ex seconds`：为键设置秒级过期时间
+ `px milliseconds`：为键设置毫秒级过期时间

+ `nx`: 键必须不存在才可以添加
+ `xx`：与nx相反，键必须存在才可以更新键的值



除此之外，提供了`setex`和`setnx`两个命令，其作用和搭配了`ex`和`nx`选项的set命令一样，用法如下

```shell
setex key seconds value
setnx key value
```



命令演示：

```java
127.0.0.1:6379> exists hello
(integer) 0
127.0.0.1:6379> setnx hello redis
(integer) 1
127.0.0.1:6379> set hello jedis xx
OK
```



> 注：
>
> `setnx`和`set xx`的应用场景：拿`setnx`举例，因为Redis是单线程命令处理机制，如果有多个客户端同时执行`setnx key value`，根据`setnx`的特性可知，只有一个客户端可以设置成功，`setnx`可以作为分布式锁的一种实现



（2）`get key`：获取值

获取键hello的值

```shell
127.0.0.1:6379> get hello
"jedis"
```



如果键不存在，则返回`nil`

```shell
127.0.0.1:6379> get not_exist_key
(nil)
```



（3）`mset key value [key value ...]`：批量设置值

```shell
127.0.0.1:6379> mset key1 1 key2 2 key3 3
OK
```



（4）`mget key [key ...]`：批量获取值

```shell
127.0.0.1:6379> mget key1 key2 key3
1) "1"
2) "2"
3) "3"
```

> 注：如果键不存在，返回值为`nil`



批量操作可以提高开发效率，相对于get命令，mget批量操作可以减少多余的网络请求时间，大大提升了效率

> 注：
>
> 使用`get`批量操作：n次get时间 = n次网络时间 + n次命令时间
>
> 使用`mget`批量操作：n次get时间 = 1次网络时间 + n次命令时间

（5）`incr key`：自增操作

返回结果有三种情况：

+ 值不是整数，返回错误

```shell
127.0.0.1:6379> incr hello
(error) ERR value is not an integer or out of range
```

+ 键不存在，按照值为0自增，返回结果为1


```shell
127.0.0.1:6379> incr key
(integer) 1
```

+ 值是整数，返回自增后的结果

```shell
127.0.0.1:6379> incr key
(integer) 2
```



除了incr命令，Redis提供了`desr`（自减）、`incrby`（自增指定数字）、`decrby`（自减制定数字）、`incrbyfloat`（自增浮点数）:

`decr key`

`incrby key increment`

`decrby key decrement`

`incrbyfloat key increment`



（6）`append key value`：向字符串尾部追加值

```shell
127.0.0.1:6379> set key redis
OK
127.0.0.1:6379> get key
"redis"
127.0.0.1:6379> append key world
(integer) 10
127.0.0.1:6379> get key
"redisworld"
```

（7）`strlen key`：获取字符串长度

```shell
127.0.0.1:6379> get key
"redisworld"
127.0.0.1:6379> strlen key
(integer) 10
```



> 注：每个中文字占3个字节



（8）`getset key value`：设置并返回原值

```shell
127.0.0.1:6379> getset hello world
(nil)
127.0.0.1:6379> get hello
"world"
```



（9）`setrange key offset value`：设置制定位

```shell
127.0.0.1:6379> set redis pest
OK
127.0.0.1:6379> setrange redis 0 b
(integer) 4
127.0.0.1:6379> get redis
"best"
```



（10）`getrange key start end`：获取部分字符串

start 和 end 分别是开始和结束的偏移量，偏移量从0开始计算

```shell
127.0.0.1:6379> getrange redis 0 1
"be"
```



#### 4.1.3 字符串命令时间复杂度

| 命令          | 时间复杂度 |
| :-------: | :--------: |
| `set key value` | O(1) |
| `get key` | O(1) |
| `del key [key ...]` | O(k)，k是键的个数 |
| `mset key value [key value ...]` | O(k)，k是键的个数 |
| `mget key [key ...]` | O(k)，k是键的个数 |
| `incr key` | O(1) |
| `decr key` | O(1) |
| `incrby key increment` | O(1) |
| `decrby key decrement` | O(1) |
| `incrbyfloat key increment` | O(1) |
| `append key value` | O(1) |
| `strlen key` | O(1) |
| `setrange key offset value` | O(1) |
| `getrange key start end` | O(n)，n是字符串长度，由于获取字符串速度非常快，所以如果字符串不是很长，可以视同为**O(1)** |



#### 4.1.4 内部编码

字符串类型的内部编码有3种：

+ int：8个字节的长整型
+ embstr：小于等于39个字节的字符串
+ raw：大于39个字节的字符串

Redis会根据当前值的类型和长度决定使用哪种内部编码实现



#### 4.1.5 使用场景

+ 缓存功能

![Redis+MySQL组成的缓存存储架构](.\image\字符串类型缓存功能.png)



图中是字符串典型的使用场景，其中Redis作为缓存层，MySQL作为存储层，绝大部分请求的数据都是从Redis中获取。由于Redis具有支撑高并发的特点，所以缓存通常能起到加速读写和降低后端压力的作用，例如获取用户数据可以先从Redis中获取数据，如果Redis中没有用户数据，再从MySQL中获取，并将结果写到Redis，添加过期时间



+ 计数

许多应用都会使用Redis作为计数的基础工具，它可以实现快速计数、查询缓存的功能，同时数据可以异步的落地到其他数据源，例如视频播放计数可以使用Redis作为基础组件，用户没播放一次视频，相应的视频播放数就会自增1



+ 共享session

![Session分散管理](.\image\session分散管理.png)

如图所示，一个分布式Web服务将用户的Session信息（例如用户登录信息）保存在各自服务器，这样会造成一个问题，处于负载均衡的考虑，分布式服务器会将用户的访问均衡到不同服务器上，用户刷新一次访问可能会发现需要重新登录，这个问题是用户无法容忍的。

为了解决这个问题，可以使用Redis将用户的Session进行集中管理，如下图所示

![Redis集中管理Session](C:\Users\Ecifics\Desktop\Recent Files\笔记\计算机\Java\中间件\image\Redis集中管理Session.png)

在这种模式下只要保证Redis是高可用和扩展性，每次用户更新或者查询登录信息直接从Redis中集中获取



+ 限速

很多应用处于安全考虑，会在每次进行登录时，让用户输入手机验证码，从而确定是否是用户本人。但是为了短信接口不被频繁访问，会限制用户每分钟获取验证码的频率，例如一分钟不能超过5次，此功能可以用Redis来使用



### 4.2 哈希

#### 4.2.1 概述

![哈希](C:\Users\Ecifics\Desktop\Recent Files\笔记\计算机\Java\中间件\image\哈希.png)

#### 4.2.2

（1）`hset key field value`设置值

```shell
127.0.0.1:6379> hset user:1 name tom
(integer) 1
```



（2）`hget key field`获取值

获取键`user:1`的`name`域对应的值

```shell
127.0.0.1:6379> hget user:1 name
"tom"
```

> 注：如果键或者域不存在，会返回nil



（3）`hdel key field [field ...]`删除域





### 4.3 列表





### 4.4 集合



### 4.5 有序集合



## 五、事务



## 六、持久化操作

### 6.1 概述

Redis支持`RDB`和`AOF`两种持久化机制，持久化功能可以有效地避免因进程退出而造成的数据丢失的问题，当下次重启时利用之前持久化的文件即可实现数据恢复

### 6.2 RDB（Redis Database）

#### 6.2.1 相关概念

RDB持久化是把当前进程数据生成快照保存到硬盘的过程，触发RDB持久化过程分为手动触发和自动触发

#### 6.2.2 触发机制

##### 6.2.2.1 手动触发

+ `save`命令（已废弃，不建议在线上环境使用）
  + 阻塞当前的服务器，直到RDB过程执行完成为止，对于内存比较大的实例会造成长时间的阻塞
+ `bgsave`命令（对`save`命令阻塞问题做的优化）
  + Redis进程执行fork操作创建子进程，RDB吃就会过程由子进程负责，完成后自动结束。阻塞只发生在fork创建子进程阶段，一般时间很短
  + 执行流程：
    + ![bgsave命令执行流程](.\image\bgsave命令执行流程.png)
    + 执行`bgsave`命令，Redis父进程判断当前是否存在正在执行的子进程，如RDB/AOF子进程，如果存在`bgsave`命令直接返回
    + 父进程执行fork操作创建子进程，fork操作过程中父进程会阻塞，通过`info status`命令查看`last_fork_usec`选项，可以获取最近一个fork操作的耗时，单位为微秒
    + 父进程fork完成后，`bgsave`命令返回 "Background saving started" 信息并不再阻塞父进程，可以继续响应其他命令
    + 子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换。执行`lastsave`命令可以获取最后一次生成RDB的时间，对于info统计的`rdb_last_save_time`选项。
    + 进程发送信号给父进程表示完成，父进程更新统计信息 

##### 6.2.2.2 自动触发

  + 配置文件中使用save相关配置，例如`save m n`，表示在m秒内数据集存在n次修改时，自动触发`bgsave`
  + 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点
  + 执行`debug reload`命令重新加载Redis时，会自动触发save操作
  + 默认情况下执行`shutdown`命令时，如果没有开启AOF持久化功能则自动执行`bgsave`

#### 6.2.3 RDB文件的处理

+ 保存
  + RDB文件保存在dir配置指定的目录下，文件名通过dbfilename配置指定。
+ 压缩
  + Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后文件远远小于内存大小，默认开启，可以通过参数`config set rdbcompression {yes|no}`动态修改
+ 校验
  + 如果Redis加载损坏的RDB文件时拒绝启动，并打印如下日志`# Short read or OOM loading DB. Unrecoverable error, aborting now.`，这时可以使用Redis提供的 redis-check-dump 工具检测RDB文件并获取对应的错误报告


> 注：
>
> + 当遇到坏盘或者磁盘写满的情况时，可以通过`config set dir {newDir}`在线修改文件路径到可用的磁盘路径，之后执行`bgsave`进行磁盘切换，同样适用于AOF持久化文件
> + 虽然压缩RDB会消耗CPU，但可以大幅降低文件的体积，方便保存到硬盘或通过网络发送给从节点，因此线上建议开启



#### 6.2.4 RDB优缺点

+ RDB优点
  + RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。非常适用于备份，全量复制等场景。比如每6小时执行`bgsave`备份，并把RDB文件拷贝到远程机器或者文件系统中，用于灾难恢复
  + Redis加载RDB恢复数据远远快于AOF的方式
+ RDB缺点
  + RDB方式数据没办法做到实时持久化/秒级持久化。因为`bgsave`每次运行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高
  + RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题

### 6.3 AOF

#### 6.3.1 概述
Redis针对RDB不适合持久化的问题，Redis提供了AOF持久化方式来解决
AOF（append only file）持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的
AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

#### 6.3.2 AOF的配置
开启AOF功能需要在配置文件中配置：`appendonly yes`，默认不开启。
AOF文件名通过`appendfilename`配置设置，默认文件名是`appendonly.aof`，保存路径同RDB持久化方式一直，通过dir配置指定

#### 6.3.3 AOF工作流程
##### 6.3.3.1 大概流程
![AOF工作流程](../../../assets/AOF执行流程.png)

+ 所有的写入命令会追加到`aof_buf`（缓冲区）中
+ AOF缓冲区根据对应的策略向硬盘做同步操作
+ 随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩的目的
+ 当Redis服务器重启时，可以加载AOF文件进行数据恢复

##### 6.3.3.2 命令写入
AOF命令写入的内容直接是文本协议格式，采用文本协议格式的目的
+ 文本协议具有很好的兼容性
+ 开启AOF后，所有写入命令都包含追加操作，直接采用协议格式，避免了二次处理开销
+ 文本协议具有可读性，方便直接修改和处理

AOF把命令先写入`aof_buf`中的目的是可以提供多种缓冲区同步硬盘的策略，在性能和安全性方面做出平衡。由于Redis使用单线程相应命令，如果每次写入AOF文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘负载

##### 6.3.3.3 文件同步
Redis提供了多种AOF缓冲区同步文件策略，由参数`appendfsync`控制，不同值的含义如下表所示
| 可配置值 | 说明 |
|:-----:|:-----|
| always | 命令写入`aof_buf`后调用系统`fsync`操作同步到AOF文件，`fsync`完成后线程返回 |
| everysec | 命令写入`aof_buf`后调用系统`write`操作，`write`操作完成后线程返回。fsync同步文件操作由专门线程每秒调用一次 |
| no | 命令写入`aof_buf`后调用系统`write`操作，不对AOF文件做`fsync`同步，同步硬盘操作由操作系统负责，通常同步周期最长30秒 |



+ 配置为`always`时，每次写入都要同步AOF文件，在一般的SATA硬盘上，Redis只能支持大约几百TPS写入，显然跟Redis高性能特性背道而驰，不建议配置
+ 配置为`no`，由于操作系统每次同步AOF文件的周期不可控，而且会加大每次同步硬盘的数据量，虽然提升了性能，但数据安全性无法保证
+ 配置为`everysec`，**是建议的同步策略**，也是默认配置，做到兼顾性能和数据安全性。理论上只有在系统突然宕机的情况下丢失1秒的数据（严格来说最多丢失1秒数据是不准确的）

> 注：
>
> + `write`操作会触发延迟写机制（delayed write）。Linux在内核提供页缓冲区用来提高硬盘IO性能。`write`操作在写入系统缓冲区后直接返回。同步硬盘操作依赖于系统调度机制，例如：缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失
> + `fsync`针对单个文件操作（比如AOF文件），做强制硬盘同步，`fsync`将阻塞直到写入硬盘完成后返回，保证了数据持久化