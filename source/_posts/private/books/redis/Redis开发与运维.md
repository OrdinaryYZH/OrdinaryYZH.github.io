##  第1章 Redis初识

### 1.2 Redis的特性

1. 速度快
   * 基于内存
   * 使用C语言实现，更接近底层，执行速度更快
   * 使用单线程架构，预防了多线程可能的竞争问题，没有线程间切换时的时间
   * 使用I/O多路复用模型
2. 基于键值对的数据结构服务器
3. 丰富的功能（提供5种数据结构外，还提供了许多额外的功能）
4. 简单稳定
   * 实现的源码不多；
   * 而且使用单线程模型，服务端跟客户端处理都变得简单；
   * Redis不依赖操作系统中的类库
5. 客户端语言多
6. 持久化
7. 主从复制
8. 高可用、分布式

### 1.3. Redis典型应用场景

#### 1.3.1 Redis 可以做什么

1. 缓存系统
2. 计数器
3. 消息队列系统
4. 排行榜
5. 社交网络
6. 实时系统

#### 1.3.2 Redis不能做什么 

1. 数据规模角度：数据规模可分为大规模数据和小规模数据，Redis存大规模数据显然不合适
2. 冷热数据角度：冷数据（操作不频繁的数据）不适合放Redis，对内存的一种浪费；但是对于热数据，放在Redis中可以加速读写，也可以减轻后端存储的负载，可以说是事半功倍

### 1.4 用好Redis的建议

> 切勿当做黑盒使用，开发与运维同样重要
>
> 1. 如果不了解Redis的单线程模型，有些开发者会在有上千万个键的Redis上执行keys *
> 2. 如果不了解持久化的相关原理，会在一个操作量很大的Redis上配置自动保存RDB



### 1.5 正确安装并启动Redis

#### 1.5.1 安装Redis

Linux下软件安装的2种方式：

* 通过各个操作系统的软件管理软件安装，例如CentOS的yum，Ubuntu的apt；但是管理工具不一定更新到最新版本
* 源码的方式安装

```
1) 下载Redis指定版本你的源码压缩包到当前目录
	wget http://download.redis.io/redis-3.0.7.tar.gz
2) 解压
	tar xvzf redis-3.0.7.tar.gz
3) 建立一个redis目录的软连接，指向redis-3.0.7
	ln -s redis-3.0.7 redis
4)进入redis目录
	cd redis-3.0.7
5)编译
	make
6)安装
	make install
```

有两点要注意：

1. 第3步建立了一个redis目录的软连接，是为了不把redis目录固定在指定版本上，有利于Redis未来版本升级
2. 第6步的安装是将Redis的相关运行文件放到/usr/local/bin下，这样就可以在任意目录下执行Redis的命令

装完后：redis-cli -v查看版本

#### 1.5.2 配置、启动、操作、关闭Redis

Redis可执行文件说明

| 可执行文件       | 作用              |
| ---------------- | ----------------- |
| redis-server     | Redis服务器       |
| redis-cli        | Redis命令行客户端 |
| redis-benchmark  | Redis性能测试工具 |
| redis-check-aof  | AOF文件修复工具   |
| redis-check-dump | RDB文件检查工具   |
| redis-sentinal   | Sentinal服务器    |

##### 1.5.2.1 启动Redis

1. 最简启动：redis-server
2. 动态参数启动
   redis-server -p 6380
3. 配置文件启动
   redis-server configPath
4. redis常用配置
   - daemonize：是否是守护进程
   - port：端口
   - logfile：Redis系统日志
   - dir：Redis工作目录
5. 三种启动方式的比较
   * 生产环境选择配置文件启动
   * 单机多实例 配置文件可以用端口区分开
6. 验证
   1. p -rd | grep redis
   2. netstat -antpl | grep redis
   3. redis-cli -h ip -p port ping

##### 1.5.2.2 redis客户端

1. 连接
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frq7paa05mj21kc0gxws7.jpg)

2. 返回值
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frq7syb774j21if0y9e11.jpg)
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frq7vh6smvj21m20jswqu.jpg)

##### 1.5.2.3 停止Redis服务

   ```
redis-cli shutdown
   ```

   注意三点：

   1. 关闭的过程：断开与和客户端的连接，持久化文件生成，是一种优雅的关闭方式

   2. 也可以通过kill进程号关闭，但是不要用kill -9强制杀死Redis服务

   3. shutdown还有个参数：代表关闭前是否生成持久化文件
      ``` redis -cli shutdown nosave|save```




## 第2章 API的理解和使用

### 2.1 预备

#### 2.1.1 全局命令

##### 1. keys

1. keys *: 遍历所有key

2. keys [pattern]

   > keys命令一般不再生产环境使用

3. keys * 怎么用

   1. 热备从节点
   2. scan

##### 2. dbsize

> 计算key的总数

##### 3. exists key

> 检查key是否存在

##### 4. del key [key ...]

> 删除指定key-value，可删除多个键

##### 5. expire key seconds

> key在seconds秒后过期
>
> 1. ttl key：查看key剩余的过期时间
>    返回值：
>    -1：key存在，并且没有过期时间
>
>    -2：key已经不存在了
>
>    ≥0：还有n秒过期
>
> 2. persist key：去掉key的过期时间

##### 6. type key

> 返回key的类型
>
> 1. string
> 2. hash
> 3. list
> 4. set
> 5. zset
> 6. none

##### 7. 时间复杂度

![](https://ws1.sinaimg.cn/large/8747d788gy1frqbf4x5e7j218m0r04fy.jpg)

#### 2.1.2 数据结构和内部编码

>  redis有5种常用的数据结构：string、hash、list、set、zset，通过type [key]命令可以查看当前键的数据结构类型；每种数据结构都有不止一种相应的内部编码实现，redis会在合适的场景选择合适的内部编码，通过object encoding [key] 可以查看内部编码。
>
>  这样设计的好处：
>
>  ```
>  1.改进内部编码，对外的数据结构和命令没有影响，外部用户无感知。
>  2.多种内部编码可在不同场景下发挥各自优势。如：ziplist比较节约内存，但在元素比较多的时候，性能会下降，此时转换为linkedlist。
>  ```
命令：

```
object encoding hello
"embstr"
```

![](https://sfault-image.b0.upaiyun.com/186/825/1868251457-5ac3a4d996c3e)

#### 2.1.3 单线程

> Redis使用单线程架构和I/I多路复用模型来实现高性能的内存数据库服务；

##### 1. 引出单线程模型

例子：三个客户端执行命令，Redis执行的顺序不一定，但是不会有并发问题

##### 2. 为什么这么快

1. 纯内存

2. 非阻塞IO，Redis使用~~epoll~~作为I/O多路复用技术的实现，再加上Redis自身的事件处理模型将epoll中的链接、读写、关闭都转换为事件，不在网络I/O上浪费过度的时间。

   > ★参考：
   > [Redis 和 I/O 多路复用](https://draveness.me/redis-io-multiplexing)
   > [Redis 中的事件循环](https://draveness.me/redis-eventloop)
   > Redis设计与实现第12章-事件
   >
   > 结论：不看源码的话，很难明白整个流程；文章中东截一块，西截一块，很难看出整个流程；

3. 避免线程切换和静态消耗

注意：
1. 一次只运行一条命令
2. 拒绝长（慢）命令
   keys, flushall, flushdb, slow lua script, mutil/exec, operate big vale(collection)
3. 其实不是单线程(网上搜不到)
   1. fsync file descriptor
   2. close file descriptor

### 2.2 字符串

#### 2.2.1 API

##### 1. 常用命令

###### (1) 设置值

1. `SET key value [EX seconds] [PX milliseconds] [NX|XX] `

   - `EX seconds*` – 设置键key的过期时间，单位时秒
   - `PX milliseconds*` – 设置键key的过期时间，单位时毫秒
   - `NX` – 只有键key不存在的时候才会设置key的值，add
   - `XX` – 只有键key存在的时候才会设置key的值，updae
     不管key是否存在，都设置

2. **setnx key value** 
   「**SET** if **N**ot e**X**ists」key不存在，才设置（相当于add）

3. `SETEX key seconds value`

   设置过期

![](https://ws1.sinaimg.cn/large/8747d788gy1frqe0ow5v1j20zd0m2aj8.jpg)

###### (2) 获取值

`get key`

###### (3) 批量设置值

`mset key value [key value ...]`

###### (4) 批量获取值

`mget key [key ...]`

批量操作命令可以有效的提高开发效率：

n次get和1次get多个对比：
![](https://ws1.sinaimg.cn/large/8747d788gy1frqg1mqivjj21lj13ydya.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1frqg0s3m3ij21mp15sh13.jpg)

###### (5) 计数

`incr key`

- `incr key`:key自增1，如果key不存在，自增后得到1;如果值不是整数，返回错误
  其他类似，key不存在也操作

  ![](https://ws1.sinaimg.cn/large/8747d788gy1frqe3ttvflj21bw0w17ux.jpg)

- 其他类似api：

  - `decr key`
  - `incrby key increment`
  - `incrby key increment`
  - `increbyfloat key increment`

- 实战

  - 记录网站每个用户个人主页的访问量
    incr userid:pageview（单线程，无竞争）
  - 缓存视频的基本信息（数据源在MySQL中）
    伪代码：
    ![](https://ws1.sinaimg.cn/large/8747d788gy1frqe9fyigoj21np1164qp.jpg)
  - 分布式id生成器
    incr id（原子操作）

##### 2. 不常用命令

###### (1) 追加值

可以向字符串尾部追加值

`append key value `

###### (2) 字符串长度

`strlen key`

redisworld：返回10

世界：返回6（每个中文占用3个字节）

###### (3) 设置并返回原值

`getset key value`：getset和set一样会设置值，但是getset会返回键原来的值

```shell
> getset hello world
(nil)
> getset hello redis
"world"
```

###### (4) 设置指定位置的字符

`setrange key offeset value`

```shell
> set redis pest
OK
> setrange redis 0 b
4
> get redis
"best"
```

###### (5) 获取部分字符串

`getrange key start end`：偏移量从0开始

```shell
> getrange redis 0 1
"be"
```

##### 3. 时间复杂度

![](http://ww1.sinaimg.cn/large/8747d788gy1fu9ijx1p7zj21kw0vonb3.jpg)

#### 2.2.2 内部编码

> 可以是字符串、数字或者二进制
> 最大容量512MB
>
> 字符串类型的内部编码有3种：
>
> - int：8个字节的长整型。
> - embstr：小于等于39个字节的字符串。
> - raw：大于39个字节的字符串。
>
> Redis会根据当前值的类型和长度决定使用哪种内部编码实现。
> 整数类型示例如下：

```shell
127.0.0.1:6379> set key 8653
OK
127.0.0.1:6379> object encoding key
"int"
```

短字符串示例如下：

小于等于39个字节的字符串：embstr

```shell
127.0.0.1:6379> set key "hello,world"
OK
127.0.0.1:6379> object encoding key
"embstr"
```

长字符串示例如下：

大于39个字节的字符串：raw

```shell
127.0.0.1:6379> set key "one string greater than 39 byte........."
OK
127.0.0.1:6379> object encoding key
"raw"
127.0.0.1:6379> strlen key
(integer) 40
86
```



#### 2.2.3 场景

1. 缓存
2. 计数器
3. 分布式锁、共享Session
4. 限速

键名的建议：业务名：对象名：id：[属性]，e.g. vs:user:1:name

### 2.3 hash

![](https://ws1.sinaimg.cn/large/8747d788gy1frrcajetczj21tl0in456.jpg)



![](https://ws1.sinaimg.cn/large/8747d788gy1frrcb9ptopj21gz0tpwpt.jpg)

#### 2.3.1 API

| 命令                                              | 解释                                                  |
| ------------------------------------------------- | ----------------------------------------------------- |
| **hset(key, field, value)**                       | 向名称为key的hash中添加元素field<—>value              |
| **hget(key, field)**                              | 返回名称为key的hash中field对应的value                 |
| **hmset(key, field1, value1,…,field N, value N)** | 向名称为key的hash中添加元素field i<—>value i          |
| **hmget(key, field1, …,field N)**                 | 返回名称为key的hash中field i对应的value               |
| **hincrby(key, field, integer)**                  | 将名称为key的hash中field的value增加integer            |
| **hincrbyfloat(key, field, float)**               | 将名称为key的hash中field的value增加float              |
| **hexists(key, field)**                           | 名称为key的hash中是否存在键为field的域                |
| **hdel(key, field)**                              | 删除名称为key的hash中键为field的域                    |
| **hlen(key)**                                     | 返回名称为key的hash中元素个数                         |
| **hkeys(key)**                                    | 返回名称为key的hash中所有键                           |
| **hvals(key)**                                    | 返回名称为key的hash中所有键对应的value                |
| **hgetall(key)**                                  | 返回名称为key的hash中所有的键（field）及其对应的value |
| **hstrlen key field**                             | 计算value的字符串长度                                 |

#### 2.3.2 内部编码

* **ziplist**(压缩列表)：当field个数小于hash-max-ziplist-entries配置（默认512个）、同时所有valued的size小于hash-max-ziplist-value配置（默认64 Byte）时，使用ziplist作为内部实现，zuolist更加节省内存
* **hashtable**（哈希表）：当无法满足ziplist时就是用hashtable，因为zuolist的读写效率下降了，hashtable的读写效率更高为$O(1)$

#### 2.3.3 使用场景

##### **缓存对象**

例如缓存用户：

![](http://ww1.sinaimg.cn/large/8747d788gy1fudqp7q3zij20sg0osqcz.jpg)

要注意的：

1. 哈希类型是稀疏的，而关系型数据库是完全结构化的，例如哈希类型每个键可以有不同的field，而关系型数据库一旦添加新的列，所有行都要为其设置值（即使为NULL），如图所示。
2. 关系型数据库可以做复杂的关系查询，而Redis去模拟关系型复杂查询开发困难，维护成本高。

![](http://ww1.sinaimg.cn/large/8747d788gy1fudqqwf1w4j21gy10zto9.jpg)

目前缓存用户信息的方法（想一下各种方法的利弊）：

1. 字符串类型，每个属性一个键（不推荐）
2. 字符串类型，序列化用户信息后保存
3. Hash类型

### 2.4 list

列表结构：

![](https://ws1.sinaimg.cn/large/8747d788gy1fuf72xobggj21kw0l7gth.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1fuf71ofzaaj21kw0mhjz8.jpg)

注意：一个列表最多可以存储$2^{32}-1$个元素

#### 2.4.1 API

![](https://ws1.sinaimg.cn/large/8747d788gy1fuf76o9m48j20sg0fcwjh.jpg)

##### 1. 添加

| 命令                                         | 解释                                                   |
| -------------------------------------------- | ------------------------------------------------------ |
| **rpush (key, value,  [value...])**          | 右边插入                                               |
| **lpush (key, value,  [value...])**          | 左边插入                                               |
| **linsert (key, beforeafter, pivot, value)** | 在某个元素之前/之后插入元素，在pivot之前/之后插入value |

##### 2. 查找

| 命令                         | 解释                                                         |
| ---------------------------- | :----------------------------------------------------------- |
| **lrange (key, start, end)** | 索引为0~n-1，从右到左为-1 ~ -n；end包括自身；lrange 0 -1为查询全部 |
| **lindex (key, index**)      | 索取索引下标的元素                                           |
| **llen (key)**               | 获取列表长度                                                 |

##### 3. 删除

| 命令                         | 解释                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| **lpop (key)**               | 左边弹出                                                     |
| **rpop (key)**               | 右边弹出                                                     |
| **lrem (key, count, value)** | 删除指定元素(value)，个数为count<br />count>0：从左到右删除count个；<br />count<0：从右到左删除count个；<br />count=0：删除所有 |
| **ltrim (key, start, end)**  | 删除下标[start, end]的元素                                   |

##### 4. 修改

| 命令                         | 解释              |
| ---------------------------- | ----------------- |
| lset (key, index, newValue ) | 修改index下标的值 |

##### 5. 阻塞操作

| 命令                    | 解释         |
| ----------------------- | ------------ |
| blpop (key..., timeout) | 阻塞式左弹出 |
| brpop (key..., timeout) | 阻塞式右弹出 |

timeout为0：立即返回，如果为空则阻塞

timeout>0：阻塞timeout秒，如果为空则返回空

注意：

1. 如果是多个键，brpop会从左到右便利键，一旦有一个键能弹出，客户端立即返回（应该是timeout为0时）
2. 如果多个客户端对同意个键执行brpop，那么最先执行的客户端可以获取到弹出的值

#### 2.4.2 内部编码

1. **ziplist**（压缩列表）：当列表元素个数小于list-max-ziplist-entries配置（默认512），同时列表每个元素大小小于list-max-ziplist-value配置（默认64字节）时，Redis会使用该结构
2. **linkedlist**(链表)：ziplist不满足时，就是用该结构
3. quicklist：Redis 3.2提供的

#### 2.4.3 使用场景

1. 消息队列：lpush + brpop
2. 文章列表：值得参考，看书P45

列表实现栈和队列：

栈：lpush + lpop

队列：lpush + rpop

### 2.5 set

结构：

![](https://ws1.sinaimg.cn/large/8747d788gy1fuhqfjl7ljj21770ns15a.jpg)

注意：一个集合最多可以存储$2^{32}-1$个元素

特点:

1. 不可重复
2. 无序

#### 2.5.1 API

##### 2.5.1.1 集合内操作

| 命令                       | 解释                                           |
| -------------------------- | ---------------------------------------------- |
| sadd (key, element...)     | 添加元素                                       |
| srem (key, element...)     | 删除元素                                       |
| scard (key)                | 计算元素个数，card：cardinality                |
| sismember (key, element)   | 判断元素是否在集合中                           |
| srandmember (key, [count]) | 随机从集合返回count个元素，默认是1             |
| spop (key)                 | 从集合内弹出元素，Redis 3.2开始也支持count参数 |
| smembers (key)             | 获取所有元素                                   |

##### 2.5.1.1 集合间操作

| 命令                                                         | 解释                                        |
| ------------------------------------------------------------ | ------------------------------------------- |
| sinter (key...)                                              | 求多个集合的交集                            |
| sunion (key...)                                              | 求多个集合的并集                            |
| sdiff (key...)                                               | 求多个集合的差集                            |
| sinterstore（distination, key...)<br />sunionstore（distination, key...)<br />sdiffstore（distination, key...) | 将交集、并集和差集的结果保存到distination中 |

#### 2.5.2 内部编码

1. intset（整数集合）：当集合中的元素**都是整数**并且**元素个数小于set-max-intset-entries配置**（默认512）时
2. hashtable（哈希表）：不满足intset时使用该结构

#### 2.5.3 使用场景

* 标签

总的来说有以下几种:

1. sadd = Tagging（标签）
2. spop/srandmember = Random item（生成随机数，比如抽奖）
3. sadd + sinter = Social Graph（社交需求）

### 2.6 zset（有序集合）

结构

![](https://ws1.sinaimg.cn/large/8747d788gy1fuis4tmkw0j21kw0uv7lh.jpg)

#### 2.6.1 API

##### 2.6.1.1 集合内

###### 1) 添加元素

1. **zadd (key, score, member [score, member...])**

   可选参数：

   1. nx：member不存在才添加， = add
   2. xx：member必须存在，用于更新
   3. ch：返回此次操作后，有序集合元素和分数发生变化的个数
   4. incr：对score做增加，相当于后面介绍的zincrby

###### 2) 获取元素/信息

2. **zcard (key)**：获取成员个数

3. **zscore (key, member)**：获取某成员分数

4. **zrank (key, member)**：返回menber排名，分数从低到高，排名从0开始

5. **zrevrank (key, member)**：返回member倒数排名，分数从高到低

6. **zrange (key, start end, [withscores])**：返回第[start, end]名成员，排名从0开始，[withscores]一并返回分数

7. **zrevrange (key, start end, [withscores])**：与zrange相反

8. **zrangebyscore (key, min, max, [withscores], [limit offset count])**：返回分数在[min, max]之间的元素，[withscores]一并返回分数，[limit offset count]可以限制输出的起始位置和个数

   同时min和max还支持开区间（小括号）和闭区间（中括号），-inf和+inf分别代表无限小和无限大

   ```shell
   127.0.0.1:6379> zrangebyscore user:ranking (200 +inf withscores
   1) "tim"
   2) "220"
   3) "martin"
   4) "250"
   5) "tom"
   6) "260"
   ```

9. **zrevrangebyscore (key, min, max, [withscores], [limit offset count])**：和zrangebyscore 相反

10. **zcount (key, min, max)**：返回分数在[min, max]内的元素个数

###### 3) 修改元素

11. **zincrby (key, increment,  member)** ：增加member分数

###### 4) 删除元素

12. **zrem (key, member...)**：删除成员,返回成功个数
13. **zremrangebyrank (key, start,  end)**：删除指定排名内的升序元素
14. **zremrangebyscore (key, min, max)**：删除指定分数范围内的成员

##### 2.6.1.2 集合间

###### 1) 交集

`zinterstore destination numkeys key [key ...][weights weight [weight ...]] [aggregate sum|min|max]`

- destination：交集计算结果保存到这个键。
- numkeys：需要做交集计算键的个数。
- key[key...]：需要做交集计算的键。
- weights weight[weight...]：每个键的权重，在做交集计算时，每个键中的每个member会将自己分数乘以这个权重，每个键的权重默认是1。
- aggregate sum|min|max：计算成员交集后，分值可以按照sum（和）、min（最小值）、max（最大值）做汇总，默认值是sum

e.g.如果想让user：ranking：2的权重变为0.5，并且聚合效果使用max，可以执行如下操作：

```shell
127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1
user:ranking:2 weights 1 0.5 aggregate max
(integer) 3
127.0.0.1:6379> zrange user:ranking:1_inter_2 0 -1 withscores
1) "mike"
2) "91"
3) "martin"
4) "312.5"
5) "tom"
6) "444"
```

###### 2) 并集

`zunionstore destination numkeys key [key ...][weights weight [weight ...]] [aggregate sum|min|max]`

#### 2.6.2 内部编码

1. **ziplist**（压缩列表）：当有序集合的元素个数小于zset-max-ziplistentries配置（默认128个），同时每个元素的值都小于zset-max-ziplist-value配置（默认64字节）时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存的使用。
2. **skiplist**（跳跃表）：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降。

#### 2.6.3 使用场景

排行榜系统

主要需要实现以下4个功能。

（1）添加用户赞数
例如用户mike上传了一个视频，并获得了3个赞，可以使用有序集合的zadd和zincrby功能：

`zadd user:ranking:2016_03_15 mike 3`

如果之后再获得一个赞，可以使用zincrby：

`zincrby user:ranking:2016_03_15 mike 1`

（2）取消用户赞数
由于各种原因（例如用户注销、用户作弊）需要将用户删除，此时需要将用户从榜单中删除掉，可以使用zrem。例如删除成员tom

`zrem user:ranking:2016_03_15 mike`

（3）展示获取赞数最多的十个用户此功能使用zrevrange命令实现：

`zrevrangebyrank user:ranking:2016_03_15 0 9`

（4）展示用户信息以及用户分数

此功能将用户名作为键后缀，将用户信息保存在哈希类型中，至于用户的分数和排名可以使用zscore和zrank两个功能：

`hgetall user:info:tom`
`zscore user:ranking:2016_03_15 mike`
`zrank user:ranking:2016_03_15 mike`



### 2.7 键管理

#### 2.7.1 单个键管理

##### 1. 键重命名

**rename (key, newkey)**

注意：如果newkey已经存在了，那newkey的值就被覆盖了；为了防止这种情况，使用**renamenx**命令，确保只有newKey不存在是才有效，该命令在newkey已经存在的情况下返回0.

还有2点要注意:

1. 重命名会执行del命令删除旧的键，如果对应的value比较大，可能会阻塞
2. rename和renamenx中的key和newkey相同时，Redis 3.2和之前的处理不一样：
   * Redis 3.2会返回 OK
   * Redis 3.2之前会有错误提示

##### 2. 随机返回一个 键

**randomkey**

##### 3. 键过期

###### 1) 设置过期

1. **expire key seconds**：键在seconds后过期
2. **expireat key timestamp**：键在秒级时间戳timestamp后过期

例如如果需要将键hello在2016-08-01 00：00：00（秒级时间戳为1469980800）过期，可以执行如下操作：

```shell
127.0.0.1:6379> expireat hello 1469980800
(integer) 1
```

对应的毫秒级别如下：

1. **pexpire key milliseconds**
2. **pexpireat key milliseconds-timestamp**

不管是过期时间，还是秒级别或者毫秒级别的设置，Redis内部都是使用pexpireat命令

###### 2) 查看过期时间

1. **ttl**：秒级别
2. **pttl**：毫秒级别

返回说明：

- ≥ 0的整数：键剩余的过期时间（ttl是秒，pttl是毫秒）。
- -1：键没有设置过期时间。
- -2：键不存在

---

**以下是要注意的点**：

1. 如果expire key的键不存在，返回结果为0：
2. 如果过期时间为负值，键会立即被删除，犹如使用del命令一样
3. persist命令可以将键的过期时间清除
4. 对于字符串类型键，执行set命令会去掉过期时间
5. Redis不支持二级数据结构（例如哈希、列表）内部元素的过期功能，例如不能对列表类型的一个元素做过期时间设置
6. setex命令作为set+expire的组合，不但是原子执行，同时减少了一次网络通讯的时间

##### 4. 迁移键

###### 1） move(不建议生产中使用)

**move key db**：用于Redis内部数据库之间使用

###### 2）dump + restore

**dump key**	// 原客户端中执行

**resore key ttl value**	// 目标客户端中执行

用于在不同的Redis实例间迁移。

###### 3）migrate

`migrate host port key|"" destination-db timeout [copy][replace] [keys key [key...] ]`

该命令也用户不同Redis实例间迁移，但是是原子操作

参数说明：

- host：目标Redis的IP地址。
- port：目标Redis的端口。
- key|""：在Redis3.0.6版本之前，migrate只支持迁移一个键，所以此处是要迁移的键，但Redis3.0.6版本之后支持迁移多个键，如果当前需要迁移多个键，此处为空字符串""。
- destination-db：目标Redis的数据库索引，例如要迁移到0号数据库，这里就写0。
- timeout：迁移的超时时间（单位为毫秒）。
- [copy]：如果添加此选项，迁移后并不删除源键。
- [replace]：如果添加此选项，migrate不管目标Redis是否存在该键都会正常迁移进行数据覆盖。
- [keys key[key...]]：迁移多个键，例如要迁移key1、key2、key3，此处填写“keys key1 key2 key3”。

迁移的情况:

1. **源Redis有hello**，目标Redis没有（理想情况），返回OK
2. **源Redis有hello**，目标Redis也有，如果命令没有加replacecanshu ,会报错。
3. **源Redis没有hello**，那么会报错

#### 2.7.2 遍历键

##### 1. keys

`keys pattern`

pattern使用的是glob风格的通配符：

- *代表匹配任意字符。
- ?代表匹配一个字符。
- []代表匹配部分字符，例如[1，3]代表匹配1，3，[1-10]代表匹配1到10的任意数字。
- \x用来做转义，例如要匹配星号、问号需要进行转义。

知识点，删除video字符串开头的键：

**redis-cli keys video* | xargs redis-cli del**

**键太多，注意阻塞**

##### 2. scan，渐进式遍历

`scan cursor [match pattern][count number]`

- cursor是必需参数，实际上cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回当前游标的值，直到游标值为0，表示遍历结束。
- match pattern是可选参数，它的作用的是做模式的匹配，这点和keys的模式匹配很像。
- count number是可选参数，它的作用是表明每次要遍历的键个数，默认值是10，此参数可以适当增大。

除了scan以外，还有：`hscan`, `sscan`, `zscan`

>  详细用法看书P69.

总结：

渐进式遍历可以有效的解决keys命令可能产生的阻塞问题，但是scan并非完美无瑕，如果在scan的过程中如果有键的变化（增加、删除、修改），那么遍历效果可能会碰到如下问题：新增的键可能没有遍历到，遍历出了重复的键等情况，也就是说scan并不能保证完整的遍历出来所有的键，这些是我们在开发时需要考虑的。

#### 2.7.3 数据库管理

##### 1. 切换数据库

Redis实例有16个数据库，以数字为标记，从0开始。**但是一般不使用多个数据库**

`select dbindex`

##### 2. flushdb/flushall

`flushdb`：清除当前数据库

`flushall`：清除所有数据库

两个问题：

- flushdb/flushall命令会将所有数据清除，一旦误操作后果不堪设想，第12章会介绍rename-command配置规避这个问题，以及如何在误操作后快速恢复数据。
- 如果当前数据库键值数量比较多，flushdb/flushall存在阻塞Redis的可能性。





## 第3章 小功能大用处

### 3.1 慢查询分析

> Redis客户端执行一条命令分为以下4个部分：
>
> ![](https://ws1.sinaimg.cn/large/8747d788gy1fuv6ycesxyj215o0oi122.jpg)
>
> **慢查询统计的是第3个步骤的时间**。

#### 3.1.1 慢查询的两个配置参数

> **Redis的慢查询存在内存中，使用List存储，List有长度，超过长度的日志会被会删掉，类似队列。**

可通过配置以下两个参数:

1. `slowlog-log-slower-than`：超过xx微秒的命令会被记录
2. `slowlog-max-len`：存储日志的List长度

配置方法有：

1. 修改配置文件

2. 使用config set命令

   ```shell
   config set slowlog-log-slower-than 20000 # =0会记录所有命令
   config set slowlog-max-len 1000		 	 # <0会不记录任何命令
   config rewrite # 该命令可将配置持久化到本地配置文件
   ```

管理慢查询的命令：

1. 获取慢查询日志

   `slowlog get [n]`：n指定条数

   e.g.

   ```shell
   slowlog get
   1)  1) (integer) 666		#id
       2) (integer) 1456786500 #发生时间戳
       3) (integer) 11615		#耗时
       4) 1) "BGREWRITEAOF"	#执行命令和参数
   2)  1) (integer) 665
       2) (integer) 1456718400
       3) (integer) 12006
       4)  1) "SETEX"
           2) "video_info_200"
           3) "300"
           4) "2"
   ...
   ```

2. 获取慢查询List长度

   `slowlog len`

3. 慢查询日志重置

   `slowlog reset`：实际是对列表做清理操作

#### 3.1.2 最佳实践

1.  slowlog-max-len配置建议：Redis记录并不会占用大量内存，线上可以调大一点，1000以上，防止慢查询过多被剔除过多命令。
2.  slowlog-log-slower-than配置建议：可以根据并发量配置该选项，注：默认10毫秒即为慢查询，OPS为100000。如果查询在1毫秒以上，那么Redis的OPS最多1000...
3.  客户端超时，一种可能是其他慢查询导致命令队列阻塞
4.  因为慢查询日志是一个在内存中的队列， 那么如果慢查询过多，那么会丢失部分日志。最好能够将其持久化起来（例如MySQL）

### 3.2 Redis Shell

#### 3.2.1 redis-cli 详解

| 参数              | 解释                                                         |
| ----------------- | ------------------------------------------------------------ |
| -r                | 将命令执行多次                                               |
| -i                | 命令每个几秒执行一次，配合-r使用                             |
| -x                | 选项代表从标准输入（stdin）读取数据作为redis-cli的最后一个参数 |
| -c                | Redis Cluster相关                                            |
| -a                | 设置了密码时可使用                                           |
| --scan和--pattern | 扫描指定模式的键                                             |
| --slave           | 将当前客户端模拟成当前Redis节点的从节点                      |
| --rdb             | 请求Redis实例生成并发送RDB持久化文件，保存在本地             |
| --pipe            | 将命令封装成Redis通信协议定义的数据格式，批量发送给Redis执行 |
| --bigkeys         | 选项使用scan命令对Redis的键进行采样，从中找到内存占用比较大的键值，这些键可能是系统的瓶颈 |
| --eval            | 执行指定Lua脚本                                              |
| --latency         | 有三个选项，分别是--latency、--latency-history、--latency-dist。它们都可以检测网络延迟，对于Redis的开发和运维非常有帮助。 |
| --stat            | 可以实时获取Redis的重要统计信息，虽然info命令中的统计信息更全，但是能实时看到一些增量的数据（例如requests）对于Redis的运维还是有一定帮助的 |
| --raw和--no-raw   | --no-raw选项是要求命令的返回结果必须是原始的格式，--raw恰恰相反，返回格式化后的结果 |

#### 3.2.2. redis-server详解

redis-server --test-memory xxxx

可以用来检测当前操作系统能否稳定地分配指定容量的内存给Redis，通过这种检测可以有效避免因为内存问题造成Redis崩溃。

通常无需每次开启Redis实例时都执行--test-memory选项，该功能更偏向于调试和测试，例如，想快速占满机器内存做一些极端条件的测试，这个功能是一个不错的选择。

#### 3.2.3 redis-benchmark详解

> redis-benchmark可以为Redis做基准性能测试，它提供了很多选项帮助开发和运维人员测试Redis的相关性能

| 参数          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| -c            | -c（clients）选项代表客户端的并发数量（默认是50）            |
| -n  <request> | -n（num）选项代表客户端请求总量（默认是100000）              |
| -q            | -q选项仅仅显示redis-benchmark的requests per second信息       |
| -r            | 想向Redis插入更多的键，可以执行使用-r（random）选项，可以向Redis插入更多随机的键 |
| -P            | 代表每个请求pipeline的数据量（默认为1）                      |
| -k <boolean>  | 代表客户端是否使用keepalive，1为使用，0为不使用，默认值为1。 |
| -t            | 可以对**指定命令**进行基准测试。                             |
| --csv         | 会将结果按照csv格式输出，便于后续处理，如导出到Excel等       |



### 3.3 Pipeline

#### 3.3.1 Pipeline概念

Redis客户端执行一条命令分为如下四个过程：

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

其中 1和4 称为Round Trip Time（RTT，往返时间）

**问题：**虽然Redis提供了批量操作命令（例如mget、mset等），有效地节约RTT。**但大部分命令是不支持批量操作的**，例如要执行n次**hgetall**命令，并没有mhgetall命令存在，需要消耗n次RTT

**解决：**Pipeline（流水线）机制能改善上面这类问题，它能将一组Redis命令进行组装，通过一次RTT传输给Redis，再将这组Redis命令的执行结果按顺序返回给客户端

![](https://ws1.sinaimg.cn/large/8747d788gy1fuvbe0q9crj22fe0x5wwa.jpg)

#### 3.3.2 性能测试

在不同网络下测试，得出以下结论：

1. Pipeline执行速度一般比逐条执行要快
2. 客户端和服务端的网络延时越大， Pepeline效果越明显

![](https://ws1.sinaimg.cn/large/8747d788gy1fuvbjbxc92j21h3089diq.jpg)

#### 3.3.3 原生批量命令与Pipeline对比

1. 原生批量命令是原子的，Pipeline是非原子的。
2. 原生批量命令是一个命令对应多个key，Pipeline支持多个命令。
3. 原生批量命令是Redis服务端支持实现的，而Pipeline需要服务端和客户端的共同实现

#### 3.3.4 最佳实践

**不要一次传输太多**，因为会增加客户端的等待时间，而且会造成一定的网络阻塞。

Pepeline只能操作一个Redis实例，但是在分布式Redis中，也可以作为批量操作的重要优化手段

### 3.4 事务与Lua

#### 3.4.1 事务

Redis提供了简单的事务功能，将一组命令放在multi和exec两条命令之间。

如果要停止事务的执行，使用discard代替exec。

事务中出现错误的情况：

1. 命令错误，可能将set写成了sett，那么整个事务无法执行

2. 运行时错误，例如用户B在添加粉丝列表时，误把sadd命令写成了zadd命令，这种就是运行时命令，因为语法是正确的

   ```shell
   127.0.0.1:6379> multi
   OK
   127.0.0.1:6379> sadd user:a:follow user:b
   QUEUED
   127.0.0.1:6379> zadd user:b:fans 1 user:a # key已经存在
   QUEUED
   127.0.0.1:6379> exec
   1) (integer) 1
   2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
   127.0.0.1:6379> sismember user:a:follow user:b
   (integer) 1
   ```

   Redis不支持回滚，可以看出第一条命令已经执行了

有些场景需要在事务之前，确保事务中的key没有被其他客户端修改过，才执行事务，否则不执行（类似乐观锁）。Redis提供了watch命令来解决这类问题，如下图：

![](https://ws1.sinaimg.cn/large/8747d788gy1fuvcuvgmy6j21hy0etadp.jpg)

#### 3.4.2 Lua用法简述

#### 3.4.3 Redis与Lua

#### 3.4.4 案例

#### 3.4.5 Redis如何管理Lua脚本

### 3.5 Bitmaps

#### 3.5.1 数据结构模型

Bitmaps本身不是一种数据结构，实际上她就是字符串，但可以对字符串的位进行操作

![](https://ws1.sinaimg.cn/large/8747d788gy1fuyh1kes34j21hc07jgpg.jpg)

#### 3.5.2 命令

下面的例子以用户访问网站记录为例，访问过的为1，id为偏移量，从0开始

**感觉位图还是比较适合做签到功能，统计UV使用HyperLogLog比较省空间**

##### 1. 设置值

**setbit key offset value**

e.g. setbit unique:users:2018-09-05 10241 // id为1024的用户在2018-09-05这天访问了网站

##### 2. 获取值

**getbit key offset**

##### 3. 获取Bitmaps指定范围值为1的个数

`bitcount key [start][end]`

##### 4. Bitmaps间的运算

**bitop op destkey key [key ...]**

该操作中的op可以为and(交集)、or(并集)、not(非)、xor(异或)，并将其结果放到destkey中

e.g. 统计某2天都访问过（任意一天）网站的用户数量

##### 5. 计算Bitmaps中第一个值为targetBit的偏移量

`bitpos key targetBit [start][end]` ：start和end表示字节，不是位

#### 3.5.3 BitMaps分析

用户数量的多少决定是用那种数据结构存储统计信息：

1. 假设网站有1亿用户，每天独立访问的用户有5千万，如果每天用集合类型和Bitmaps分别存储活跃用户可以得到如下表

   ![](https://ws1.sinaimg.cn/large/8747d788gy1fuyja9901mj21ik07jdjo.jpg)

   很明显，这种情况下使用Bitmaps能节省很多的内存空间，尤其是随着时间推移节省的内存还是非常可观的：

   ![](https://ws1.sinaimg.cn/large/8747d788gy1fuyjan8ynzj21j507ltad.jpg)

2. 但Bitmaps并不是万金油，假如该网站每天的独立访问用户很少，例如只有10万（大量的僵尸用户），那么两者的对比如表所示，很显然，这时候使用Bitmaps就不太合适了，因为基本上大部分位都是0。

   ![](https://ws1.sinaimg.cn/large/8747d788gy1fuyjb5uaprj21id08dae4.jpg)

### 3.6 HyperLogLog

> **HyperLogLog**是一个基数估计算法，**只需要极少空间就可以计算超大基数**
>
> 简单来说，基数（cardinality，也译作势），是指一个集合（这里的集合允许存在重复元素）中不同元素的个数。例如看下面的集合：
> {1,2,3,4,5,2,3,9,7}
> 这个集合有9个元素，但是2和3各出现了两次，因此不重复的元素为1,2,3,4,5,9,7，所以这个集合的基数是7。
>
> PS：**pf**是HyperLogLog 这个数据结构的发明人 Philippe Flajolet 的首字母缩写

#### 1. 添加

**pfadd key element [element ...]**

#### 2. 计算独立用户数

**pfcount key [key...]**

#### 3. 合并

**pfmerge destkey sourcekey [sourcekey ...]**

#### 4. 总结

实际应用：统计网页的UV（**Unique visitor**）

选型时注意以下2点：

1. 只为了计算独立总数，不需要获取单条数据
2. 可以容忍一定误差率（官方的数字是0.81%），毕竟非常节省内存，它需要占据一定 12k 的存储空间，所以它不适合统计单个用户相关的数据

### 3.7 发布订阅

> Redis提供了的发布订阅机制，发布者跟订阅者不通过直接通信，而通过频道（channel）转发：
>
> ![](https://ws1.sinaimg.cn/large/8747d788gy1fuyonpoolbj21o00q6ds9.jpg)

#### 3.7.1 命令

##### 1. 发布消息

**publish channel message**

##### 2. 订阅消息

**subscribe channel [channel ... ]**

redis-cli：使用该命令后会一直监听消息，不处理其他命令，包括unsubscribe；而且要退出的话只是推出redis-cli，返回shell，而不是取消订阅

参考：[Redis的Pub/Sub模式](https://my.oschina.net/itblog/blog/601284)

##### 3. 取消订阅

**unsubscribe [channel [channel ...] ]**

##### 4. 按照模式订阅和取消订阅

**psubscribe pattern [pattern ... ]** 

**punsubscribe [pattern [pattern ...] ]**

##### 5. 查询订阅

1. 查看活跃的频道（至少有一个订阅者）
   **pubsub channels [pattern]**

2. 查看频道订阅数
   **pubsub numsub [channel ...]**

3. 查看模式订阅数

   这个命令返回的不是订阅模式的客户端的数量， 而是客户端订阅的所有模式的数量总和。

   **pubsub numpat**

参考：http://redisdoc.com/pub_sub/pubsub.html

#### 3.7.2应用场景

个人想到的可能是弹幕系统。

因为不支持持久化，没有很好的场景。

### 3.8 GEO

> Redis 3.2提供了GEO（地理信息定位）功能。

#### 1. 增加地理位置信息

**geoadd key longitude latitude member [longitude latitude member....]**

longitude：经度

latitude：维度

member：成员

注意返回的结果：

1. 如果新添加成功，返回1
2. 如果已经存在了，则是修改成功，返回0（如果要更新位置信息，也使用geoadd，虽然返回结果为0。）

结论：返回值是新增的数量。

#### 2. 获取地理位置信息

**geopos key menber [member...]**

```shell
127.0.0.1:6379> geopos cities:locations beijing
1) 1) "116.28000229597091675"
   2) "39.51099897006833572"
```

#### 3. 获取两个地理之位置的距离

**geodist key member1 member2 [unit]**

- m（meters）代表米。
- km（kilometers）代表公里。
- mi（miles）代表英里。
- ft（feet）代表尺。

#### 4. 获取指定位置范围内的地理信息集合

`georadius key longitude latitude radiusm|km|ft|mi [withcoord][withdist] [withhash][COUNT count] [asc|desc][store key] [storedist key]`

``georadiusbymember key member radiusm|km|ft|mi [withcoord][withdist][withhash][COUNT count] [asc|desc][store key] [storedist key]`

georadius和georadiusbymember两个命令的作用是一样的，都是以一个地理位置为中心算出指定半径内的其他地理信息位置，不同的是georadius命令的中心位置给出了具体的经纬度，georadiusbymember只需给出成员即可。其中radiusm|km|ft|mi是必需参数，指定了半径（带单位），这两个命令有很多可选参数，如下所示：

- withcoord：返回结果中包含经纬度。
- withdist：返回结果中包含离中心节点位置的距离。
- withhash：返回结果中包含geohash，有关geohash后面介绍。
- COUNT count：指定返回结果的数量。
- asc|desc：返回结果按照离中心节点的距离做升序或者降序。
- store key：将返回结果的地理位置信息保存到指定键。
- storedist key：将返回结果离中心节点的距离保存到指定键。

例子：距离北京150公里以内的城市

```shell
127.0.0.1:6379> georadiusbymember cities:locations beijing 150 km
1) "beijing"
2) "tianjin"
3) "tangshan"
4) "baoding"
```

#### 5. 获取geohash

**geohash key member [member...]**

Redis使用geohash将二维经纬度转换为一维字符串，下面操作会返回beijing的geohash值。

geohash的特点：

* GEO的数据类型为zset，Redis将所有地理位置信息的geohash存放在zset中
* 字符串越长，表示的位置更精确，表3-8给出了字符串长度对应的精度，例如geohash长度为9时，精度在2米左右
* 两个字符串越相似，它们之间的距离越近，Redis利用字符串前缀匹配算法实现相关的命令。
* geohash编码和经纬度是可以相互转换的。

![](https://ws1.sinaimg.cn/large/8747d788gy1fuytrkm8p4j20u00okaea.jpg)

#### 6. 删除地理位置信息

因为底层实现是zset，所以使用的是zrem。

**zem key member**



















