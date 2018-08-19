##  第1章 Redis初识

### 1.2 Redis的特性

1. 速度快（基于内存）

2. 持久化

3. 多种数据结构
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frq7e4yzzyj21rv0z9tok.jpg)
   ​

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
3) 简历一个redis目录的软连接，指向redis-3.0.7
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

2. setnx key value 
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

1. **hset(key, field, value)**：向名称为key的hash中添加元素field<—>value
2. **hget(key, field)**：返回名称为key的hash中field对应的value
3. **hmset(key, field1, value1,…,field N, value N)**：向名称为key的hash中添加元素field i<—>value i
4. **hmget(key, field1, …,field N)**：返回名称为key的hash中field i对应的value
5. **hincrby(key, field, integer)**：将名称为key的hash中field的value增加integer
6. **hincrbyfloat(key, field, float)**：将名称为key的hash中field的value增加float
7. **hexists(key, field)**：名称为key的hash中是否存在键为field的域
8. **hdel(key, field)**：删除名称为key的hash中键为field的域
9. **hlen(key)**：返回名称为key的hash中元素个数
10. **hkeys(key)**：返回名称为key的hash中所有键
11. **hvals(key)**：返回名称为key的hash中所有键对应的value
12. **hgetall(key)**：返回名称为key的hash中所有的键（field）及其对应的value
13. **hstrlen key field**：计算value的字符串长度

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

### 4. list

#### 4.1列表结构

![](https://ws1.sinaimg.cn/large/8747d788gy1frrhxit3s6j225x0t1dpc.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1frrk2uvvbsj22680uxk1g.jpg)

#### 4.2 重要的API

##### 4.2.1 增（rpush、lpush、linsert）

1. rpush
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frrk6ayrmkj21xh0qzk5w.jpg)
2. lpush
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frrk73f3uwj21x20q5175.jpg)
3. linsert
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frrk8tw8fjj22590ulnk2.jpg)

##### 4.2.2 删（lpop、rpop、lrem ）

1. lpop
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frrkaw7ewjj21xt0ztds0.jpg)

2. rpop
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frrkdcw7f1j21y80yzgxp.jpg)

3. lrem
   ![1527528777006](C:\Users\yao\AppData\Local\Temp\1527528777006.png)

4. ltrim
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frrkptxgewj21z91k81d0.jpg)

 ##### 4.2.3 查（lrange）

   ###### 4.3.1 lrange

![](https://ws1.sinaimg.cn/large/8747d788gy1frrksycggwj21ze1hp7st.jpg)

###### 4.3.2 lindex

![](https://ws1.sinaimg.cn/large/8747d788gy1frrl2pmjl9j21xr11gh1d.jpg)

###### 4.3.3 llen![](https://ws1.sinaimg.cn/large/8747d788gy1frrl3mt73fj21xc13jqe3.jpg)

##### 4.2.4 改

###### 4.4.1 lset

![](https://ws1.sinaimg.cn/large/8747d788gy1frrl9tdlx9j21ym10zney.jpg)

#### 4.3 实战-TimeLine

![](https://ws1.sinaimg.cn/large/8747d788gy1frrlcnwf2vj223814eh2v.jpg)

#### 4.4 查漏补缺（blpop、brpop）

> 它是 [LPOP](http://redisdoc.com/list/lpop.html#lpop) 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 [BLPOP](http://redisdoc.com/list/blpop.html#blpop) 命令阻塞，直到等待超时或发现可弹出元素为止。
>
> 当给定多个 `key` 参数时，按参数 `key` 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。

![](https://ws1.sinaimg.cn/large/8747d788gy1frrle5ufccj21v40xr4qp.jpg)

#### 4.5 Tips（命令组合使用组成不同的数据结构）

![](https://ws1.sinaimg.cn/large/8747d788gy1frrlexmn5uj21p50kgn5s.jpg)





### 5. set

#### 5.1 集合结构

##### 5.1.1 图解

![](https://ws1.sinaimg.cn/large/8747d788gy1frs83fvavcj21j80ueqdb.jpg)

##### 5.1.2 特点

1. 无序
2. 无重复
3. 集合间操作

#### 5.2 集合内API

##### 5.2.1 sadd srem

![](https://ws1.sinaimg.cn/large/8747d788gy1frs87bnhzgj21x70w2hcf.jpg)

##### 5.2.2 scard sismember srandmember smembers

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8awwtwnj2215161hdt.jpg)

> 注意：smembers 集合太大小心使用

#### 5.3 实战

##### 5.3.1 抽奖系统

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8ikpgfcj21fw0fv46a.jpg)

##### 5.3.2 Like、赞、踩

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8j7c2u6j21sf0px7hw.jpg)

##### 5.3.3 标签（tag）

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8klangoj22650u1tz4.jpg)

#### 5.4 集合间API(sdiff sinter sunion)

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8mx4wjjj21xp157e81.jpg)

##### 5.4.1 例子：共同关注

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8nwa6jwj20te1337i6.jpg)

#### 5.5 TIPS

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8qay6vxj21ip0fptfv.jpg)



### 6. zset（有序集合）

#### 6.1 有序集合结构

![](https://ws1.sinaimg.cn/large/8747d788gy1frs8t1wpxtj21wp11ak7r.jpg)

#### 6.2 API

##### 6.2.1 zadd

![](https://ws1.sinaimg.cn/large/8747d788gy1frs9hx761sj21yb16e7su.jpg)

##### 6.2.2 zrem

![](https://ws1.sinaimg.cn/large/8747d788gy1frs9miavfvj21z5169ker.jpg)

##### 6.2.3 zscore

![](https://ws1.sinaimg.cn/large/8747d788gy1frs9p6hsqzj21yg16iavn.jpg)

##### 6.2.4 zincrby

![](https://ws1.sinaimg.cn/large/8747d788gy1frs9qwvranj21y61737vb.jpg)

##### 6.2.5 zcard

![](https://ws1.sinaimg.cn/large/8747d788gy1frs9zg9ti0j21zr16s1dq.jpg)

##### 6.2.6 zrank

`ZRANK key member `

```
redis> ZRANK salary tom                     # 显示 tom 的薪水排名，第二
(integer) 1
```

##### 6.2.7 demo

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa119rk6j21od18mkjl.jpg)

##### 6.2.8 zrange

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa20hjkqj2203132qtl.jpg)

##### 6.2.9 zrangebyscore

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa3t5hqxj220j13h7wh.jpg)

##### 6.2.10 zcount

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa4m4210j220d13k1kx.jpg)

##### 6.2.11 zremrangebyrank

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa7jv1l3j220l13w1kx.jpg)

##### 6.2.12 zremrangebyscore

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa8eox2hj220g13r1kx.jpg)

##### 6.2.13 demo

![](https://ws1.sinaimg.cn/large/8747d788gy1frsa9fhf2oj21jq18ohdt.jpg)

#### 6.3 实战 - 排行榜

![](https://ws1.sinaimg.cn/large/8747d788gy1frsacovfxlj21sh1434qp.jpg)

#### 6.4 查漏补缺（zrevrank、zrevrange、zrevrangebyscore、zinterstore、zunionstore）

#### 6.5 总结

![](https://ws1.sinaimg.cn/large/8747d788gy1frsam5sklij217410ye0c.jpg)



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



















