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

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B1%BB%E5%9E%8B%E7%BC%93%E5%AD%98%E5%8A%9F%E8%83%BD.png" align="left" alt="Redis+MySQL组成   的缓存存储架构">

图中是字符串典型的使用场景，其中Redis作为缓存层，MySQL作为存储层，绝大部分请求的数据都是从Redis中获取。由于Redis具有支撑高并发的特点，所以缓存通常能起到加速读写和降低后端压力的作用，例如获取用户数据可以先从Redis中获取数据，如果Redis中没有用户数据，再从MySQL中获取，并将结果写到Redis，添加过期时间



+ 计数

许多应用都会使用Redis作为计数的基础工具，它可以实现快速计数、查询缓存的功能，同时数据可以异步的落地到其他数据源，例如视频播放计数可以使用Redis作为基础组件，用户没播放一次视频，相应的视频播放数就会自增1



+ 共享session

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/session%E5%88%86%E6%95%A3%E7%AE%A1%E7%90%86.png" align="left" alt="session分散管理">

如图所示，一个分布式Web服务将用户的Session信息（例如用户登录信息）保存在各自服务器，这样会造成一个问题，处于负载均衡的考虑，分布式服务器会将用户的访问均衡到不同服务器上，用户刷新一次访问可能会发现需要重新登录，这个问题是用户无法容忍的。

为了解决这个问题，可以使用Redis将用户的Session进行集中管理，如下图所示

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/Redis%E9%9B%86%E4%B8%AD%E7%AE%A1%E7%90%86Session.png" align="left" alt="Redis集中管理Session">

在这种模式下只要保证Redis是高可用和扩展性，每次用户更新或者查询登录信息直接从Redis中集中获取



+ 限速

很多应用处于安全考虑，会在每次进行登录时，让用户输入手机验证码，从而确定是否是用户本人。但是为了短信接口不被频繁访问，会限制用户每分钟获取验证码的频率，例如一分钟不能超过5次，此功能可以用Redis来使用



### 4.2 哈希

#### 4.2.1 概述

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E5%93%88%E5%B8%8C.png" align="left" alt="哈希">

哈希类型中的映射关系叫做`field-value`，注意这里的value是指field对应的值，不是键对应的值

#### 4.2.2 命令

（1）`hset key field value`设置值

```shell
127.0.0.1:6379> hset user:1 name tom
(integer) 1
127.0.0.1:6379> hset user:1 age 28
(integer) 1
```



（2）`hget key field`获取值

获取键`user:1`的`name`域对应的值

```shell
127.0.0.1:6379> hget user:1 name
"tom"
127.0.0.1:6379> hget user:1 age
"28"
```

> 注：如果键或者域不存在，会返回nil



（3）`hdel key field [field ...]`删除域

hdel会删除一个或者多个field，返回结果为成功删除field的个数

```shell
127.0.0.1:6379> hdel user:1 name
(integer) 1
127.0.0.1:6379> hdel user:1 age
(integer) 1
```



（4）`hlen key`：计算field个数

```shell
127.0.0.1:6379> hset user:1 name tom
(integer) 1
127.0.0.1:6379> hset user:1 age 28
(integer) 1
127.0.0.1:6379> hset user:1 city chengdu
(integer) 1
127.0.0.1:6379> hlen user:1
(integer) 3
```



（5）批量设置或者过去field-key

```shell
hmget key field [field ...]
hmset key field value [field value ...]
```



```shell
127.0.0.1:6379> hmset user:1 name mike age 12 city chengdu
OK
127.0.0.1:6379> hmget user:1 name age city
1) "mike"
2) "12"
3) "chengdu"
```



（6）`hexists key field`：判断field是否存在

存在返回1，不存在返回0

```java
127.0.0.1:6379> hexists user:1 name
(integer) 1
```



（7）`hkeys key`：获取所有field

```shell
127.0.0.1:6379> hkeys user:1
1) "name"
2) "age"
3) "city"
```



（8）`hvals key`：获取所有的value

```shell
127.0.0.1:6379> hvals user:1
1) "mike"
2) "12"
3) "chengdu"
```



（9）`hgetall key`：获取所有的field-value

```shell
127.0.0.1:6379> hgetall user:1
1) "name"
2) "mike"
3) "age"
4) "12"
5) "city"
6) "chengdu"
```



（10）`hstrlen key field`：计算value的字符串长度

```shell
127.0.0.1:6379> hget user:1 name
"mike"
127.0.0.1:6379> hstrlen user:1 name
(integer) 4
```



#### 4.2.3 哈希命令时间复杂度

| 命令 | 时间复杂度 |
| :------------: | :----: |
| `hset key field value` | `O(1)` |
| `hget key field` | `O(1)` |
| `hdel key field [field ...]` | `O(k)，k是field个数` |
| `hlen key` | `O(1)` |
| `hgetall key` | `O(n)，n是field总数` |
| `hmget field [field ...]` | `O(k)，k是field的个数` |
| `hmset field value [field value ...]` | `O(k)，k是field的个数` |
| `hexists key field` | `O(1)` |
| `hkeys key` | `O(n)，n是field总数` |
| `hvals key` | `O(n)，n是field总数` |
| `hsetnx key field value` | `O(1)` |
| `hincrby key field increment` | `O(1)` |
| `hincrbyfloat key field increment` | `O(1)` |
| `hstrlen key field` | `O(1)` |



#### 4.2.4 内部编码

+ `ziplist(压缩列表)`：当哈希类型元素个数小于`hash-max-ziplist-entries`配置（默认512个）、同时所有值都小于`hash-max-ziplist-value`配置（默认64字节）时，Redis会使用ziplist作为哈希的内部实现，ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀
+ `hashtable(哈希表)`：当哈希类型无法满足ziplist的条件时，Redis会采用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)



#### 4.2.5 使用场景

哈希类型经常被用来存储用户相关信息。优化用户信息的获取，不需要重复从数据库当中读取，提高系统性能，下面是将用户信息存储在关系型数据库和Redis中的差异

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BF%9D%E5%AD%98%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF.png" align="left" alt="关系型数据库保存用户信息">

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E4%BD%BF%E7%94%A8%E5%93%88%E5%B8%8C%E7%B1%BB%E5%9E%8B%E7%BC%93%E5%AD%98%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF.png" align="left" alt="使用哈希类型缓存用户信息">

其中有两点不同之处：

+ 哈希类型是稀疏的，而关系型数据库是完全结构化的，例如哈希类型每个键可以有不同的field，而关系型数据库一旦添加新的列，所有行都要为其设置值（即使为NULL）
+ 关系型数据库可以做负责的关系查询，而Redis去模拟关系型复杂查询开发困难，维护成本高

### 4.3 列表

#### 4.3.1 概述

列表用来存储多个有序的字符串，一个列表**最多可以存储2的32次方-1个元素**



列表特点：

+ 列表是有序的，可以通过索引下标获取某个元素或者某个范围内的元素列表
+ 列表中的元素可以是重复的



#### 4.3.2 命令

（1）`rpush key value [value ...]`：从右插入元素

```shell
127.0.0.1:6379> rpush listkey c b a
(integer) 3

127.0.0.1:6379> lrange listkey 0 -1
1) "c"
2) "b"
3) "a"
```

> 注：`lrange 0 -1` 命令从左到右获取列表的所有元素

（2）`lpush key value [value ...]`：从左边插入元素

方法和rpush类似

```shell
127.0.0.1:6379> lpush listkey c b a
(integer) 3

127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
```



（3）`linsert key before|after pivot value`：向某个元素前或者后插入元素

linsert命令会从列表中找到等于pivot的元素，在其前或后插入一个新的元素，例如下元素b前插入java

```shell
127.0.0.1:6379> linsert listkey before b java
(integer) 4

127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "java"
3) "b"
4) "c"
```



（4）`lrange key start end`：获取指定范围内的元素列表

lrange操作会获取列表指定索引范围所有的元素，索引下标有两个特点

+ 索引下标从左到右分别是0到N-1，但是从右到左分别是-1到-N
+ lrange中的end包含了自身

```shell
127.0.0.1:6379> lrange listkey 1 3
1) "java"
2) "b"
3) "c"
```



（5）`lindex key index`：获取列表指定索引下标的元素

例如获取列表最后一个元素

```shell
127.0.0.1:6379> lindex listkey -1
"c"
```



（6）`llen key`：获取列表长度

```shell
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "java"
3) "b"
4) "c"

127.0.0.1:6379> llen listkey
(integer) 4
```



（7）`lpop key`：从列表左边弹出元素

```shell
127.0.0.1:6379> lpop listkey
"a"

127.0.0.1:6379> lrange listkey 0 -1
1) "java"
2) "b"
3) "c"
```



（8）`rpop key`：从列表右侧弹出

和lpop类似

```shell
127.0.0.1:6379> rpop listkey
"c"

127.0.0.1:6379> lrange listkey 0 -1
1) "java"
2) "b"
```



（9）`lrem key count value`：删除指定元素

lrem命令会从列表中找到value的元素进行删除，根据count的不同分为三种情况

+ count > 0，从左到右，删除最多count个数
+ count < 0，从右到左，删除最多count绝对值个元素
+ count = 0，删除所有



```shell
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "a"
3) "a"
4) "a"
5) "a"
6) "java"
7) "b"

127.0.0.1:6379> lrem listkey 4 a
(integer) 4

127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "java"
3) "b"
```



（10）`ltrim key start end`：按照索引范围修剪列表

例如只保留列表的第二到第四个元素

```shell
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "a"
3) "a"
4) "a"
5) "a"
6) "a"
7) "java"
8) "b"

127.0.0.1:6379> ltrim listkey 1 3
OK

127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "a"
3) "a"
```



（11）`lset key index newValue`：修改指定索引下标的元素

```shell
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "a"
3) "a"

127.0.0.1:6379> lset listkey 2 python
OK

127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "a"
3) "python"
```



（12）`blpop key [key ...] timeout` 和 `brpop key [key ...] timout` 

blpop和brpop是lpop和rpop的阻塞版本，它们除了弹出方向不同，使用方法基本相同，所以下面以brpop命令进行说明，brpop命令包含两个参数

timeout：阻塞时间（单位秒）

+ 列表为空：如果timeout=3，那么客户端要等到3秒后返回，如果timeout=0，那么客户端一直阻塞等下去，如果等待过程中有数据添加进来，会立刻返回，即使没有到timeout时间

```shell
127.0.0.1:6379> brpop list:test 3
(nil)
(3.04s)
```

+ 列表不为空：客户端会忽略timeout的值立刻返回

```shell
127.0.0.1:6379> brpop list:test 3
1) "list:test"
2) "a"

127.0.0.1:6379> brpop list:test 0
1) "list:test"
2) "b"
```



使用注意：

+ 如果是多个键，那么brpop会从左至右遍历建，一旦有一个键能弹出客户端会立刻返回

  ```shell
  127.0.0.1:6379> brpop list:1 list:2 list:3 0
  ..阻塞..
  ```

  此时另一个客户端分别向list:2和list:3插入元素

  ```shell
  127.0.0.1:6379> lpush list:2 element2
  (integer) 1
  
  127.0.0.1:6379> lpush list:3 element3
  (integer) 1
  ```
  客户端会立刻返回list:2中的element2，因为list:2最先有可以弹出的元素
  
  ```shell
  127.0.0.1:6379> brpop list:1 list:2 list:3 0
  1) "list:2"
  2) "element2"
  (111.14s)
  ```

+ 如果多个客户端对同一个键执行brpop，那么最先执行brpop命令的客户端可以获取弹出的值

  

#### 4.3.3 命令时间复杂度

|命令|时间复杂度|
|:-----------:|:---:|
| `rpush key value [value ...]` | `O(k)，k是元素个数` |
| `lpush key value [value ...]` | `O(k)，k是元素个数` |
| `linsert key before|after pivot value` | `O(n)，n是pivot距离表头或表尾的距离` |
| `lrange key start end` | `O(s+n)，s是start偏移量，n是start到end的范围` |
| `lindex key index` | `O(n)，n是索引的偏移量` |
| `llen key` | `O(1)` |
| `lpop key` | `O(1)` |
| `rpop key` | `O(1)` |
| `lremkey count value` | `O(n)，n是列表长度` |
| `ltrim key start end` | `O(n)，n是要剪裁的元素总数`|
| `lset key index value` | `O(n)，n是索引的偏移量` |
| `blpop brpop` | `O(1)` |



#### 4.3.4 内部编码

+ ziplist（压缩列表）
  + 当列表的元素个数小于`list-max-ziplist-entries`配置（默认512个），同时列表中每个元素的值都小于`list-max-ziplist-value`配置时，Redis会选用ziplist来作为列表的内部实现来减少内存的使用
+ linkedlist（链表）
  + 当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现



#### 4.3.5 使用场景

（1）消息队列

Redis的lpush+brpop命令组合即可实现阻塞队列，生产者客户端使用lpush从列表左侧插入元素，多个消息者客户端使用brpop命令阻塞式的抢列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性

（2）文章列表

每个用户有属于自己的文章列表，现需要分页展示文章列表。此时可以考虑使用列表，因为列表不但是有序，同时支持按照索引范围获取元素

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/Redis%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E6%A8%A1%E5%9E%8B.png" align="left" alt="Redis消息队列模型">



> 注：
>
> 使用列表类型保存和获取文章列表会存在两个问题：
>
> + 如果每次分页获取的文章个数较多，需要执行多次hgetall操作，此时可以考虑使用Pipeline批量获取，或者考虑文章数据序列化为字符串类型，使用mget批量获取
> + 分页获取文章列表时，lrange命令在列表两端性能较好，但是如果列表较大，获取列表中间范围的元素性能会变差，此时可以考虑做二次拆分，使用Redis3.2的quicklist内部编码实现，结合ziplist和linkedlist的特点，获取列表中间范围的元素时也可以高效完成



### 4.4 集合

#### 4.4.1 概述

集合（set）类型是用来保存多个的字符串元素，但和列表类型不一样的是，集合中不允许有重复元素，并且集合中的元素时无序的，不能通过索引下标获取元素。例如“user:1follow”包含着"it"、"music"、"his"和"sports"四个元素，**一个集合最多可以存储2^32次方-1个元素**

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E9%9B%86%E5%90%88%E7%B1%BB%E5%9E%8B.png" align="left" alt="集合类型">



#### 4.4.2 命令

##### 集合内操作

（1）`sadd key element [element ...]`：添加元素

返回添加成功的元素个数

```shell
127.0.0.1:6379> exists myset
(integer) 0
127.0.0.1:6379> sadd myset a b c
(integer) 3
127.0.0.1:6379> sadd myset a b
(integer) 0
```



（2）`srem key element [element ...]`：删除元素

返回删除成功元素的个数

```shell
127.0.0.1:6379> srem myset a b
(integer) 2
127.0.0.1:6379> srem myset hello
(integer) 0
```



（3）`scard key`：计算元素个数

`scard`的时间复杂度为`O(1)`，**它不会遍历集合所要元素**，而是直接利用Redis内部的变量

```shell
127.0.0.1:6379> scard myset
(integer) 1
```



（4）`sismember key element`：判断元素是否在集合中

如果给定元素element在集合内返回1，否者返回0

```shell
127.0.0.1:6379> sismember myset c
(integer) 1
```



（5）`srandmemeber key [count]`：随机从集合返回指定个数元素

[count]是可选参数，如果不写默认为1

```java
127.0.0.1:6379> sadd myset a d
(integer) 2
    
127.0.0.1:6379> srandmember myset 2
1) "a"
2) "d"
    
127.0.0.1:6379> srandmember myset
"c"
```



（6）`spop key`：从集合随机弹出元素

```java
127.0.0.1:6379> spop myset
"d"
```



（7）`smembers key`：获取所有元素，返回结果是 无序的

```shell
127.0.0.1:6379> smembers myset
1) "c"
2) "a"
```



#### 集合间操作

初始化两个集合，它们分别是user:1:follow和user:2:follow

```shell
127.0.0.1:6379> sadd user:1:follow it music his sports
(integer) 4

127.0.0.1:6379> sadd user:2:follow it news ent sport
(integer) 4
```



（8）`sinter key [key ...]`：求多个集合的交集

```shell
127.0.0.1:6379> sinter user:1:follow user:2:follow
1) "it"
```



（9）`sunion key [key ...]`：求多个集合的并集

```shell
127.0.0.1:6379> sunion user:1:follow user:2:follow
1) "news"
2) "sport"
3) "music"
4) "ent"
5) "his"
6) "it"
7) "sports"
```



（10）`sdiff key [key ...]`：求多个集合的差集

```shell
127.0.0.1:6379> sdiff user:1:follow user:2:follow
1) "his"
2) "music"
3) "sports"
```



（11）将交集、并集、差集的结果保存

`sinterstore destination key [key ...]`

`sunionstore destination key [key ...]`

`sdiffstore destination key [key ...]`

集合间的运算在元素较多的情况下会比较耗时，所以Redis提供了三个命令将集合间交集、并集、差集的结果保存在destination key中

```shell
127.0.0.1:6379> sinterstore user:1_2:inter user:1:follow user:2:follow
(integer) 1

127.0.0.1:6379> type user:1_2:inter
set

127.0.0.1:6379> smembers user:1_2:inter
1) "it"
```



#### 4.4.3 集合命令时间复杂度 

| 命令 | 时间复杂度 |
| :-----------: | :-----: |
| `sadd key element [element ...]` | `O(k)，k是元素个数` |
| `srem key element [element ...]` | `O(k)，k是元素个数` |
| `scard key` | `O(1)` |
| `sismember key element` | `O(1)` |
| `srandmemeber key [count]` | `O(count)` |
| `spop key` | `O(1)` |
| `smembers key` | `O(n)，n是元素总数` |
| `sinter key [key ...]` | `O(m * k)，k是多个集合中元素最少的个数` |
| `sunion key [key ...]` | `O(k)，k是多个集合元素个数和` |
| `sdiff key [key ...]` | `O(k)，k是多个集合元素个数和` |



#### 4.4.4 内部编码

+ intset（整数集合）：当集合中的元素都是整数且元素个数小于`set-max-intset-entries`配置（默认512个）时，Redis会选用intset来作为集合的内部实现，而减少内存的使用
+ hashtable（哈希表）：当即和无法满足intset条件时，Redis会使用hashtable作为集合的内部实现



#### 4.4.5 使用场景

集合类型比较典型的使用场景是标签（tag）。例如一个用户可能对娱乐、体育比较感兴趣，另一个用户可能对历史、新闻比较感兴趣，这些兴趣点就是标签。有了这些数据就可以得到喜欢同一个标签的人，以及用户的共同喜好的标签，这些数据对于用户体验以及增强用户黏度比较重要。

一个电子商务的网站会对不同标签的用户做不同类型的推荐，比如对数码产品比较感兴趣的人，在各个页面或者通过邮件的形式给他们推荐最新的数码产品，通常会对网站带来更多的利益。

### 4.5 有序集合

#### 4.5.1概述

有序集合保留了集合不能重复元素的特性，但不同的是，有序集合可以排序。但是它和列表使用索引作为排序依据不同的是，它给每个元素设置一个分数（score）作为排序的依据

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E6%9C%89%E5%BA%8F%E9%9B%86%E5%90%88.png" align="left" alt="有序集合">



| 数据结构 | 是否允许重复元素 | 是否有序 | 有序实现方式 | 应用场景 |
| :--------: | :---: | :---: | :-------: | :-------: |
| 列表 | 是 | 是 | 索引下标 | 时间轴、消息队列等 |
| 集合 | 否 | 否 | 无 | 标签、社交等 |
| 有序集合 | 否 | 是 | 分值 | 排行榜系统、社交等 |



#### 4.5.2 命令

##### 集合内

（1）`zadd key score member [score member ...]`：添加成员

返回结果代表成功添加成员的个数

```shell
127.0.0.1:6379> zadd user:ranking 251 tom
(integer) 1

127.0.0.1:6379> zadd user:ranking 1 kris 91 mike 200 frank 220 tim 250 martin
(integer) 5
```



Redis为zadd命令添加了nx、xx、ch、incr四个选项，使用zadd命令注意事项：

+ nx：member必须不存在，才可以设置成功，用于添加
+ xx：member必须存在，才可以设置成功，用于更新
+ ch：返回此次操作后，有序集合元素和分数发生变化的个数
+ incr：对score做增加，相当于后面介绍的zincrby

**有序集合相比集合提供了排序字段，也产生了相应的代价，zadd的时间复杂度为O(logn)，sadd的时间复杂度为O(1)**



（2）`zcard key`：计算成员个数

和集合类型的`scard`命令一样，`zcard`命令的时间复杂度为O(1)

```shell
127.0.0.1:6379> zcard user:ranking
(integer) 5
```



（3）`zscore key member`：计算某个成员的分数

如果成员不存在会返回nil

```shell
127.0.0.1:6379> zscore user:ranking kris
"1"

127.0.0.1:6379> zscore user:ranking test
(nil)
```



（4）`zrank key member`和`zrevrank key memeber`：计算成员的排名

`zrank`是从分数从低到高返回排名，`zrevrank`反之（排名从0开始）

```shell
127.0.0.1:6379> zrank user:ranking kris
(integer) 0

127.0.0.1:6379> zrevrank user:ranking kris
(integer) 4
```



（5）`zrem key member [member ...]`：删除成员

执行完后返回成功删除的个数

```shell
127.0.0.1:6379> zrem user:ranking mike
(integer) 1
```



（6）`zincrby key increment member`：增加成员的分数

```shell
127.0.0.1:6379> zincrby user:ranking 9 tom
"9"
127.0.0.1:6379> zincrby user:ranking 9 tom
"18"
127.0.0.1:6379> zincrby user:ranking 9 tom
"27"
```

返回结果为成员最终分数



（7）`zrange key start end [withscores]`和`zrevrange key start end [withscores]`：返回指定排名范围的成员

有序集合是按照分值排名，`zrange`是从低到高返回，`zrevrange`反之，加上`withscores`，会返回成员对应的分数

```shell
127.0.0.1:6379> zrange user:ranking 0 2 withscores
1) "kris"
2) "1"
3) "tom"
4) "27"
5) "frank"
6) "200"
```

```shell
127.0.0.1:6379> zrevrange user:ranking 0 2 withscores
1) "martin"
2) "250"
3) "tim"
4) "220"
5) "frank"
6) "200"
```



（8）`zrangebyscore key min max [withscores] [limit offset count]`和`zrevrangebyscore key max min [withscores] [limit offset count]`：返回指定分数范围的成员

其中`zrangebyscore`按分数从低到高返回，`zrevrangebyscore`反之，`withscores`选项会同时返回每个成员的分数，`[limit offset count]`选项可以限制输出的起始位置和个数

```shell
127.0.0.1:6379> zrangebyscore user:ranking 200 221 withscores
1) "frank"
2) "200"
3) "tim"
4) "220"
```

```shell
127.0.0.1:6379> zrevrangebyscore user:ranking 221 200 withscores
1) "tim"
2) "220"
3) "frank"
4) "200"
```



同时min和max还支持开区间（小括号）和闭区间（中括号），-inf和+inf分别代表无限小和无限大



（9）`zcount key min max`：返回指定分数范围成员个数

```shell
127.0.0.1:6379> zcount user:ranking 200 220
(integer) 2
```



（10） `zremrangebyrank key start stop`：删除指定排名内的升序元素

```shell
127.0.0.1:6379> zremrangebyrank user:ranking 0 2
(integer) 3
```



（11）`zremrangebyscore key min max`：删除指定分数范围的成员

```shell
127.0.0.1:6379> zadd user:ranking 400 ecifics
(integer) 1

127.0.0.1:6379> zremrangebyscore user:ranking (250 +inf
(integer) 1
```



##### 集合间操作

将下图两个集合导入Redis中

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E6%9C%89%E5%BA%8F%E9%9B%86%E5%90%88user_ranking_1%E5%92%8Cuser_ranking_2.png" align="left" alt="有序集合user_ranking_1和user_ranking_2.png">

```shell
127.0.0.1:6379> zadd user:ranking:1 1 kris 91 mike 200 frank 220 tim 250 martin 251 tom
(integer) 6

127.0.0.1:6379> zadd user:ranking:2 8 james 77 mike 625 martin 888 tom
(integer) 4
```



（12）`zinterstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate sum|min|max]`：交集

命令参数：

+ destination：交集计算结果保存到这个键
+ numkeys：需要做交集计算键的个数
+ key [key ...]：需要做交集计算的键
+ weights weight [weight ...]：每个键的权重，在做交集计算时，每个键中的每个member会将自己分数乘以这个权重，每个键的权重默认是1
+ aggregate sum|min|max：计算成员交集后，分值可以按照sum、min、max做汇总，默认是sum

下面操作对user:ranking:1和user:ranking:2做交集，weights和aggregate使用了默认配置

```shell
127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1 user:ranking:2
(integer) 3

127.0.0.1:6379> zrange user:ranking:1_inter_2 0 -1 withscores
1) "mike"
2) "168"
3) "martin"
4) "875"
5) "tom"
6) "1139"
```

下面然权重变为0.5，并且聚合效果使用max

```shell
127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1 user:ranking:2 weights 1 0.5 aggregate max
(integer) 3

127.0.0.1:6379> zrange user:ranking:1_inter_2 0 -1 withscores
1) "mike"
2) "91"
3) "martin"
4) "312.5"
5) "tom"
6) "444"
```



（13）`zunionstore destination numkeys key [key ...] [weights weight [weight ...]] [aggregate sum|min|max]`：并集

```shell
127.0.0.1:6379> zunionstore user:ranking:1_union_2 2 user:ranking:1 user:ranking:2
(integer) 7

127.0.0.1:6379> zrange user:ranking:1_union_2 0 -1 withscores
 1) "kris"
 2) "1"
 3) "james"
 4) "8"
 5) "mike"
 6) "168"
 7) "frank"
 8) "200"
 9) "tim"
10) "220"
11) "martin"
12) "875"
13) "tom"
14) "1139"
```



#### 4.5.3 命令时间复杂度

| 命令 | 时间复杂度 |
| :----: | :----:|
| `zadd key score member [score memeber ...]` | `O(k * logn)，k是添加成员的个数，n是当前有序集合成员个数` |
| `zcard key` | `O(1)` |
| `zscore key member` | `O(1)` |
| `zrank key member` `zrevrank key member` | `O(logn)，n是当前有序集合成员个数` |
| `zrem key member [member ...]` | `n(k * logn)，k是删除成员的个数，n是当前有序集合成员个数` |
| `zincrby key increment member` | `n(logn)，n是当前有序集合成员个数` |
| `zrange key start end [withscores]` `zrevrange key start end [withscores]` | `O(logn + k)，k是要获取的成员个数，n是当前有序集合成员个数` |
| `zrangebyscore key min max [withscores]` `zrevrangebyscore key max min [withscores]` | `O(logn + k)，k是要获取的成员个数，n是当前有序集合成员个数` |
| `zcount` | `O(logn)，n是当前有序集合成员个数` |
| `zremrangebyrank key start end` | `O(logn + k)，k是要删除的成员个数，n是当前有序集合成员个数` |
| `zremrangebyscore key min max` | `O(logn + k)，k是要删除的成员个数，n是当前有序集合成员个数` |
| `zinterstore destination numkeys key [key ...]` | `O(n * k) + O(m * logm)，n是成员数最小的有序集合成员个数，k是有序集合的个数，m是结果集中成员个数` |
| `zunionstore destination numkeys key [key ...]` | `O(n) + O(m * logm)，n是所有有序集合成员个数和，m是结果集中成员个数` |



#### 4.5.4 内部编码

+ ziplist（压缩列表）
  + 当有序集合的元素个数小于`zset-max-ziplist-entries`配置（默认128个），同时每个元素的值都小于`zset-max-ziplist-value`配置（默认64字节）时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存的使用
+ skiplist（跳跃表）：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降



#### 4.5.5 使用场景

有序集合比较典型的使用场景就是排行榜系统。例如视频网站需要对用户上传的视频做排行榜，榜单的维度可能是多个方面的：按照时间、按照播放数量、按照获得的赞



### 4.6 键管理

#### 4.6.1 单个键管理

（1）`rename key newkey`：键重命名

```shell
127.0.0.1:6379> set python jedis
OK

127.0.0.1:6379> get python
"jedis"

127.0.0.1:6379> rename python java
OK

127.0.0.1:6379> get python
(nil)

127.0.0.1:6379> get java
"jedis"
```

上述操作将键python重命名为java，**如果键java已经存在，那么它的值会被覆盖**

```shell
127.0.0.1:6379> set C cedis
OK

127.0.0.1:6379> rename C java
OK

127.0.0.1:6379> get java
"cedis"
```



为了防止被强行rename，Redis提供了renamenx命令，确保只有newkey不存在时候才被覆盖，例如下面操作renamenx时，newkey=python已经存在，返回结果是0代表没有完成重命名



使用重命名命令时，需要注意：

+ 由于重命名键期间会执行del命令删除旧的键，如果键对应的值比较大，会存在阻塞Redis的可能性
+ 如果`rename`和`renamenx`中的key和newkey如果是相同的，在Redis3.2和之前版本返回结果不同



（2）`randomkey`：随机返回一个键



#### 4.6.2 遍历键

（1）`keys pattern`：全量遍历键

pattern使用的是glob风格的通配符

+ `*`：代表匹配任意字符
+ `?`：代表匹配一个字符
+ `[]`：代表匹配部分字符，例如[1,3]代表匹配1或3，[1-10]匹配1到10的任意数字
+ `\x`：用来做转义，例如要匹配星号、问号需要进行转义

例如`keys [j,r]edis`会匹配以j或者r开头，紧跟edis字符串的所有键

`keys h?ll*`：可以匹配hello和hill两个键



如果Redis包含了大量的键，执行keys命令很可能会造成Redis阻塞，所以一般建议不要再生产环境下使用keys命令。但有时候有遍历键的需求时，可以：

+ 在一个不对外提供服务的Redis从节点上执行，这样不会阻塞到客户端的请求，但是会影响到主从复制
+ 如果确认键值总数确实比较少，可以执行命令
+ 使用scan命令渐进式的遍历所有键，可以有效防止阻塞



（2）`scan cursor [match pattern] [count number]`：渐进式遍历

与keys命令执行时会遍历所有键不同，scan采用渐进式遍历的方式来解决keys命令可能带来的阻塞问题，每次scan命令的时间复杂度是O(1)，但是要真正实现keys的功能，需要执行多次scan。Redis存储键值对实际使用的是hashtable的数据结构，如下图

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/hashtable%E7%A4%BA%E6%84%8F%E5%9B%BE.png" align="left" alt="hashtable示意图">



scan方法参数

+ cursor
  + cursor是**必需参数**，cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回当前游标的值，直到游标值为0，表示遍历结束
  + match pattern参数是可选参数，它的作用是做模式的匹配，这点和keys的模式很像
  + count number是可选参数，它的作用是表明每次要遍历的键个数，默认值是10，此参数可以适当增大

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

    + <img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/bgsave%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png" align="left" alt="bgsave命令执行流程">

    + 执行`bgsave`命令，Redis父进程判断当前是否存在正在执行的子进程，如RDB/AOF子进程，如果存在`bgsave`命令直接返回

    + 父进程执行fork操作创建子进程，fork操作过程中父进程会阻塞，通过`info status`命令查看`last_fork_usec`选项，可以获取最近一个fork操作的耗时，单位为微秒

    + 父进程fork完成后，`bgsave`命令返回 "Background saving started" 信息并不再阻塞父进程，可以继续响应其他命令

    + 子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换。执行`lastsave`命令可以获取最后一次生成RDB的时间，对于info统计的`rdb_last_save_time`选项。

    + 进程发送信号给父进程表示完成，父进程更新统计信息 

##### 6.2.2.2 自动触发

  + 配置文件中使用save相关配置，例如`save m n`，表示在m秒内数据集存在大于等于n次修改时，自动触发`bgsave`
  + 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点
  + 执行`debug reload`命令重新加载Redis时，会自动触发save操作
  + 默认情况下执行`shutdown`命令时，如果没有开启AOF持久化功能则自动执行`bgsave`

#### 6.2.3 RDB文件的处理

+ 保存
  + RDB文件保存在dir配置指定的目录下（默认是当前目录，如果在/usr/local目录下启动，那么会在这个目录下生成，如果在/usr/local/bin下启动，会在这个目录下生成），文件名通过dbfilename配置指定。
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
  + RDB方式数据没办法做到实时持久化/秒级持久化。因为`bgsave`每次运行都要执行fork操作创建子进程，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑，属于重量级操作，频繁执行成本过高
  + RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题
  + 最后一个持久化时间段内如果宕机，可能会发生数据丢失

### 6.3 AOF

#### 6.3.1 概述
Redis针对RDB不适合持久化的问题，Redis提供了AOF持久化方式来解决
AOF（append only file）持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的
AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

#### 6.3.2 AOF的配置
开启AOF功能需要在配置文件中配置：`appendonly yes`，默认不开启。

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E5%BC%80%E5%90%AFAOF.png" align="left" alt="开启AOF">

AOF文件名通过`appendfilename`配置设置，默认文件名是`appendonly.aof`，保存路径同RDB持久化方式一直，通过dir配置指定

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/AOF%E6%96%87%E4%BB%B6%E5%90%8D.png" align="left" alt="AOF文件名">

#### 6.3.3 AOF工作流程
##### 6.3.3.1 大概流程

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/AOF%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png" align="left" alt="AOF执行流程">

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



+ 配置为`always`时，每次写入都要同步AOF文件，在一般的SATA硬盘上，Redis只能支持大约几百TPS写入，性能差但数据完整度好，显然跟Redis高性能特性背道而驰，不建议配置
+ 配置为`no`，由于操作系统每次同步AOF文件的周期不可控，而且会加大每次同步硬盘的数据量，虽然提升了性能，但数据安全性无法保证
+ 配置为`everysec`，**是建议的同步策略**，也是默认配置，每秒记入日志一次，做到兼顾性能和数据安全性。理论上只有在系统突然宕机的情况下丢失1秒的数据（严格来说最多丢失1秒数据是不准确的）

> 注：
>
> + `write`操作会触发延迟写机制（delayed write）。Linux在内核提供页缓冲区用来提高硬盘IO性能。`write`操作在写入系统缓冲区后直接返回。同步硬盘操作依赖于系统调度机制，例如：缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失
> + `fsync`针对单个文件操作（比如AOF文件），做强制硬盘同步，`fsync`将阻塞直到写入硬盘完成后返回，保证了数据持久化



##### 6.3.3.4 重写机制

​	随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入AOF重写机制压缩文件体积。AOF文件重写是把Redis进程内的数据转化为写命令同步到新AOF文件的过程



AOF重写文件的目的

+ 降低文件的空间占用
+ 更小的AOF文件可以更快被Redis加载



重写后AOF文件的体积变小的原因

+ 进程内已经超时的数据不再写入文件
+ 旧的AOF文件含有无效命令，如` del key1`、`hdel key2`、`srem keys`、`set a 111`、`set a 222`等。重写使用进程内的数据直接生成，这样新的AOF文件**只保留最终数据的写入命令**
+ 多条写命令可以合成一个，如：`lpush list a`、`lpush list b`、`lpush list c` 可以转化为 `lpush list a b c`。为了防止单条命令过大造成客户端缓冲区溢出，对于 list 、set 、 hash、zset等操作类型，以64个元素为界拆分为多条。



AOF重写过程分为手动触发和自动触发：

+ 手动触发
  + 直接调用`bgrewriteaof`命令
+ 自动触发
  + 根据`auto-aof-rewrite-min-size`和`auto-aof-rewrite-percentage`参数确定自动触发时机
    + `auto-aof-rewrite-min-size`：表示AOF重写时文件的最小体积，默认为64MB
    + `auto-aof-rewrite-percentage`：代表当前AOF文件空间（`aof_current_size`）和上一次重写AOF文件空间（`aof_base_size`）的比值

> 注：
>
> 自动触发时机=`aof_current_size` > `auto-aof-rewrite-min-size` && (`aof_current_size` - `aof_base_size`) / `aof_base_size` >= `auto-aof-rewrite-percentage`
>
> 其中`aof_current_size`和`aof_base_size`可以在 info Persistence 统计信息中查看



AOF重写运行流程

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/AOF%E9%87%8D%E5%86%99%E6%B5%81%E7%A8%8B.png" align="left" alt="AOF重写流程">

+ 执行AOF重写请求
  + 如果当前进程正在进行AOF重写，请求不执行返回如下响应：`ERR Background append only file rewriting already in progress`
  + 如果当前进行正在执行`bgsave`操作，重写命令延迟到`bgsave`完成之后再执行，返回如下响应`Background append only file rewriting scheduled`

+ 父进程执行fork创建子进程，开销等于`bgsave`过程
+ 主进程fork操作完成后，继续响应其他命令。所有修改命令依然写入AOF缓冲区并根据`appendfsync`策略同步到硬盘，保证原有AOF机制正确性
+ 由于fork操作运用写时复制技术，子进程只能共享fork操作时的内存数据。由于父进程依然响应命令，Redis使用“AOF重写缓冲区”保存这部分数据，防止新AOF文件生成期间丢失这部分数据。
+ 子进程根据内存快照，按照命令合并规则写入到新的AOF文件。每次批量写入硬盘数据由配置`aof-rewrite-incremental-fsync`控制，默认为32MB，防止单次刷盘数据过多造成硬盘阻塞
+ 新的AOF文件写入完成后，子进程发送信号给父进程，父进程更新统计信息，具体见 info persistence 下的 aof_*相关统计
+ 父进程吧AOF重写缓存区的数据写入到新的AOF文件。
+ 使用新的AOF文件替换老文件，完成AOF重写



>写时复制
>
>前提：基于操作系统虚拟化这一特性，应用程序使用的是虚拟内存，而 虚拟内存地址需要映射到 物理内存地址才能使用，如果使用没有映射的虚拟内存地址，将会导致缺页异常。
>
>原理大概如下：
>
>- 创建子进程时，将父进程的虚拟内存与物理内存映射关系复制到子进程中（也就是子进程和父进程的虚拟内存映射到同一块物理内存上），并将内存设置为只读（设置为只读是为了当对内存进行写操作时触发缺页异常）。
>- 当子进程或者父进程对内存数据进行修改时，便会触发缺页异常。在缺页异常处理函数中，对物理内存页进行复制一份新的物理内存页，并且将子进程的虚拟内存页映射到刚刚复制好的物理内存页，同时将父子进程的虚拟内存页M设置为可读写，此时父子进程的虚拟内存映射到了不同的物理内存上

6.3.3.5 重启加载

AOF和RDB文件都可以用于服务器重启时的数据恢复

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/Redis%E6%8C%81%E4%B9%85%E5%8C%96%E6%96%87%E4%BB%B6%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B.png" align="left" alt="Redis持久化文件加载流程">

+ AOF持久化开启并且存在AOF文件时，优先加载AOF文件，打印如下日志：`* DB loaded from append only file: 5.841 seconds`
+ AOF关闭或者AOF文件不存在时，加载RDB文件，打印如下日志：`DB loaded from disk: 5.586 seconds`
+ 加载AOF/RDB文件成功后，Redis启动成功
+ AOF/RDB文件存在错误时，Redis启动失败并打印错误信息

##### 6.3.3.5 文件校验

当AOF文件损坏时，启动Redis会拒绝启动，此时对于损坏的AOF文件，可以使用`redis-check-aof --fix`命令进行修复，修复后使用`diff-u`对比数据的差异，找出丢失的数据，有些可以进行人工修改补全



如果突然宕机导致AOF文件尾部写入不全，Redis为我们提供了`aof-load-truncated`配置来兼容这种情况，默认开启。加载AOF时，当遇到此问题时会忽略并继续启动，同时打印如下警告日志:

  ```
# !!! Warning: short read while loading the AOF file !!!
# !!! Truncating the AOF at offset ...... !!!
# AOF loaded anyway because aof-load-truncated is enabled
  ```





#### 6.3.4 AOF追加阻塞

当开启AOF持久化时，常用的同步硬盘的策略是everysec，用于平衡性能和数据安全性。对于这种方式，Redis使用另一条线程每秒执行fsync同步硬盘。当系统硬盘资源繁忙时，会造成Redis主线程阻塞

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/redis/%E4%BD%BF%E7%94%A8everysec%E5%81%9A%E5%88%B7%E7%9B%98%E7%AD%96%E7%95%A5%E7%9A%84%E6%B5%81%E7%A8%8B.png" align="left" alt="使用everysec做刷盘策略的流程">

阻塞流程：

+ 主线程负责写入AOF缓冲区
+ AOF线程负责每秒执行一次同步磁盘操作，并记录最近一次同步时间
+ 主线程负责对比上次AOF同步时间
  + 如果距上次同步成功时间在2秒内，主线程直接返回
  + 如果距上次同步成功时间超过2秒主线程将会阻塞，直到同步操作完成





## 七、Jedis

### 7.1 Maven创建

#### 7.1.1 导入依赖

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.8.0</version>
</dependency>
```



#### 7.1.2 打开防火墙放行6379端口或者直接关闭防火墙

打开防火墙放行6379端口指令

```shell
# 1.查看是否放行6379端口
firewall-cmd --list-ports
# 2.如果没有放行6379端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent
# 3.重启防火墙
systemctl restart firewalld.service
```



关闭防火墙

```shell
systemctl stop firewalld.service 
```

#### 7.1.3 和远程服务器建立连接

```java
public class RedisUtil {
    public static Jedis getConnection() {
        Jedis jedis = new Jedis("服务器的ip地址", 6379);
        jedis.auth("Redis密码");

        return jedis;
    }
}
```

控制台会打印`PONG`表示建立连接成功



#### 7.1.4 Jedis相关操作



（1）`key *`操作获取所有的键

```java
@Test
public void test1() {
    Jedis jedis = RedisUtil.getConnection();

    Set<String> keys = jedis.keys("*");
    for (String key : keys) {
        System.out.println(key);
    }
}
```

运行结果

```java
k13
k12
k11
k14
```



（2）`set`和`get`操作添加数据

```java
@Test
public void test2() {
    Jedis jedis = RedisUtil.getConnection();

    jedis.set("name", "Ecifics");
    String name = jedis.get("name");

    System.out.println(name);
}
```

运行结果

```java
Ecifics
```



（3）`mset`操作

```java
@Test
public void test3() {
    Jedis jedis = RedisUtil.getConnection();

    jedis.mset("k1", "v1", "k2", "v2");
    List<String> values = jedis.mget("k1", "k2");

    System.out.println(values);
}
```

运行结果

```java
[v1, v2]
```



（4）list数据类型基本操作

```java
@Test
public void test4() {
    Jedis jedis = RedisUtil.getConnection();

    jedis.lpush("key1", "Ecific", "Jack", "Tom");
    List<String> values = jedis.lrange("key1", 0, -1);

    System.out.println(values);
}
```

运行结果

```java
[Tom, Jack, Ecific]
```



（5）set数据类型基本操作

```java
@Test
public void test5() {
    Jedis jedis = RedisUtil.getConnection();

    jedis.sadd("names", "lucy", "jack");
    Set<String> names = jedis.smembers("names");

    System.out.println(names);
}
```

运行结果

```java
[jack, lucy]
```



### 7.2 Jedis案例-手机验证码

#### 7.2.1 要求

+ 输入手机号，点击发送后随机生成6位数字码，2分钟内有效
+ 输入验证码，点击验证，返回成功或者失败
+ 每个手机号每天只能输入3次

#### 7.2.2 实现

+ 生成随机6位数字码---Random

+ 验证码在2分钟内有效---把验证码放入redis，并设置过期时间为120秒
+ 判断验证码是否一致
+ 限制手机号发送验证码次数---每次获取后执行incr操作，当期大于2时，不在发送



```java
public class PhoneCode {

    /**
     * 获取6位数字验证码
     */
    public String getCode() {
        Random random = new Random();

        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < 6; i++) {
            int rand = random.nextInt(10);
            stringBuilder.append(String.valueOf(rand));
        }

        return stringBuilder.toString();
    }

    /**
     * 每个手机每天只能发送三次
     * 验证码放入redis，并设置其过期时间
     */
    public void setVerifyCode(String phone) {
        Jedis jedis = RedisUtil.getConnection();

        // 拼接key
        String countKey = "VerifyCode" + phone + ":count";

        String codeKey = "VerifyCode" + phone + ":code";

        String count = jedis.get(countKey);
        //还没发送过验证码
        if (count == null) {
            jedis.setex(countKey, 24 * 60 * 60, "1");
        }

        if (count != null && Integer.parseInt(count) <= 2) {
            jedis.incr(countKey);
        }

        // 以及发送了三次
        if (count != null && Integer.parseInt(count) > 2) {
            System.out.println("今天发送次数超过三次，不能再发送了");
            jedis.close();
        }

        String vcode = getCode();
        jedis.setex(codeKey, 120, vcode);
        jedis.close();
    }

    /**
     * 验证码校验
     */
    public void getRedisCode(String phone, String code) {
        Jedis jedis = RedisUtil.getConnection();

        String codeKey = "VerifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);

        if (Objects.equals(redisCode, code)) {
            System.out.println("成功");
        } else {
            System.out.println("失败");
        }

        jedis.close();
    }
}
```



### 7.2 SpringBoot整合Redis

#### 7.2.1 导入相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.11.1</version>
</dependency>
```



#### 7.2.2 配置文件

```yaml
spring:
  redis:
    host:
    password:
    port: 6379
    # Redis 数据库索引（一共有16个数据库，默认为0）
    database: 0
    # 链接超时时间（单位毫秒）
    timeout: 1800000
    lettuce:
      pool:
        # 连接池最大连接数
        max-active: 20
        # 最大阻塞等待时间
        max-wait: -1
        # 连接池中的最大空闲连接
        max-idle: 5
        # 连接池中的最小空闲连接
        min-idle: 0
```



#### 7.2.3 Redis自定义配置类

```java
@Configuration
@EnableCaching
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        template.setConnectionFactory(factory);
        template.setKeySerializer(redisSerializer);

        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();

        return cacheManager;
    }
}
```





## 八、缓存设计
### 8.1 缓存的收益、成本以及适用场景

收益

+ 加速读写
  + 因为缓存通常是全内存的，而存储层（硬盘）通常读写性能不够强悍，通过缓存的使用可以有效的加速读写，优化用户体验
+ 降低后端负载
  + 帮助后端减少访问量和复杂计算，在很大程度降低了后端的负载



成本

+ 数据不一致性
  + 缓存层和存储层的数据存在着一定时间窗口的不一致性，时间窗口跟更新策略有关
+ 代码维护成本
  + 加入缓存后，需要同时处理缓存层和存储层的逻辑，增大了开发者维护代码的成本





适用场景

+ 开销大的复杂计算
  + 以MySQL为例，一些复杂的操作（例如大量联表查询操作，一些分组计算），如果不加缓存，不断无法满足高并发，同时也会给MySQL带来巨大的负担
+ 加速请求响应
  + 即使查询单条数据的速度够快，也可以使用缓存，Redis每秒可以完成数万次读写，并且提供的批量操作可以优化整个IO链的响应时间



### 8.2 缓存穿透

#### 8.2.1 概念

缓存传统是指查询一个根本不存在的数据，缓存层和存储层都不会命中，通常出于容错的考虑，如果从存储层查不到数据则不写入缓存层，步骤如下：

+ 缓存层不命中
+ 存储层不命中，不将空结果写回缓存
+ 返回空结果

缓存穿透将导致不存在的数据每次请求都要到存储层去查询，失去了缓存保护后端存储的意义

造成缓存穿透的原因：

+ 自身业务代码或者数据出现问题
+ 一些恶意攻击（例如访问对应服务器一些本来没有的页面，例如blog.ecifics.top/1423412412，或者一直查询一个id为-1（数据库中的id都是大于0的）的数据）、爬虫等造成大量空命中



#### 8.2.2 解决方法

##### （1）缓存空对象

如果缓存层不命中，并且存储层也无法命中，将空对象写入缓存，下次查询缓存直接返回空对象



这样也造成了一定的问题：

+ 空值作了缓存，意味着缓存层中存了更多的键，需要更多的内存空间，为了应对这个问题， 需要对这些数据类型设置一个较短的过期时间
+ 缓存层和存储层的数据有一段时间窗口的不一致，可能会对业务有一定的影响。例如对一个空结果进行了缓存，但此时存储层添加了这个数据，此时访问就会出现存储层和缓存层数据不一致的情况



##### （2）布隆过滤器拦截

在访问存储层和缓存层之前，将存在的key用布隆过滤器提前保存起来，做第一层拦截。如果布隆过滤器认为这个key不存在就不访问存储层



### 8.3 缓存击穿

#### 8.3.1 概念

key对应的数据存储在存储层中，但是在缓存层Redis中过期，此时如果有大量请求过来，这些请求在缓存中无法命中，就会去存储层中查询，而大量的并发请求可能会瞬间把后端数据库压垮



#### 8.3.2 解决方法

##### （1）预先设置热门数据

在Redis高峰访问之前，把一些热门数据提前存入到Redis里面，加大这些热门数据的过期时长

##### （2）实时调整

现场监控哪些数据热门，实时调整key的过期市场

##### （3）使用锁

double-check查询存储层中的数据到缓存中





### 8.4 缓存雪崩

#### 8.4.1 概念

缓存雪崩是指缓存层由于某些原因不能提供服务，这时所有的请求都会到达存储层，存储层的调用量会暴增，造成存储层也会级联宕机的情况



#### 8.4.2 解决方法

##### （1）保证缓存层服务高可用性

##### （2）依赖隔离组件为后端限流并降级

不去访问数据库，直接返回默认数据或访问服务的内存数据

##### （3）提前演练

在项目上线前，演练缓存层宕掉后，应用以及后端的负载情况以及可能出现的情况，在此基础上做一些预案

##### （4） 构建多级缓存

nginx缓存+redis缓存+其他缓存

##### （5）使用锁或者队列

用加锁或者队列的方式来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上，效率低，不适合高并发情况

##### （6）缓存失效时间分散开

将key值的过期时间生成随机生成
