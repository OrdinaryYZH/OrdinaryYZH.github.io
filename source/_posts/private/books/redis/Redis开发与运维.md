##  第1章 Redis初识

### 1.2 Redis的特性

1. 速度快（基于内存）

2. 持久化

3. 多种数据结构
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frq7e4yzzyj21rv0z9tok.jpg)

   ![](https://ws1.sinaimg.cn/large/8747d788gy1frq7rg59aoj21980m7wjs.jpg)

4. 支持多种编程语言

5. 功能丰富

6. 简单

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

11. **zincrby (key, increment member)** ：增加member分数

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

## 第3章 Redis客户端的使用（这里只说明Java客户端）

### 3.1 Jedis直连

![](https://ws1.sinaimg.cn/large/8747d788gy1frsexbk4q5j21zd14de81.jpg)

### 3.2 Jedis简单实用

![](https://ws1.sinaimg.cn/large/8747d788gy1frseyg1ztsj21w80z91kx.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1frsezj9sonj21w80rjnoy.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1frsf0e4iitj21tm0qph77.jpg)

### 3.3 Jedis连接池使用

![](https://ws1.sinaimg.cn/large/8747d788gy1frsf1os2ioj21w20ewwxl.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1frsf2wsfyfj21ux1287tq.jpg)

### 3.4 方案对比

![](https://ws1.sinaimg.cn/large/8747d788gy1frsf458vpoj22750y4b29.jpg)



## 第4章 瑞士军刀Redis其他功能

### 4.1 慢查询



















