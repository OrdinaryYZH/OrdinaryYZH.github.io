### 第1章 课程准备

1. 并发编程初体验，通过两个例子：
   1. 计数器
   2. HashMap
2. 并发与高并发基本概念



### 第2章 并发基础

1. CPU多级缓存-缓存一致性
   说的是个啥子啊
2. CPU多级缓存-乱序执行优化
   代码执行顺序可能变化，一个核使用另一个核的某个变量时，顺序不对会导致错误
3. JAVA内存模型
   1. Java内存模型抽象结构图
   2. 同步的8种操作
   3. 同步规则
      ![](https://ws1.sinaimg.cn/large/8747d788gy1fr8m4z108aj21320h7wgt.jpg)
4. 并发的优势与风险
   ![](https://ws1.sinaimg.cn/large/8747d788gy1fr8m9w1209j20yd0m60yx.jpg)



### 第3章 项目准备

1.  案例环境初始化
2.  案例准备工作
   定义4个注解：线程安全，线程不安全，推荐写法，不推荐写法
3.  并发模拟-工具
   1.  PostMan测试并发
   2.  Apache Bench
   3.  JMeter
4.  并发模拟-代码
   1.  CountDownLatch
   2.  Semaphore



### 第4章 线程安全性

1. 线程安全性
2. 原子性
   1. AtomicXXX
      Unsafe操作，CAS
   2. AtomicLong、LongAdder（说了一大堆，没图，没字，听不懂）
   3. AtomicReference（没看出来实际用处）、AtomicReferenceFieldUpdater：更新某个类的某个值
   4. AtomicStampReference：CAS的ABA问题
   5. 锁：
      1. synchronized：依赖JVM
         1. 修饰代码块，作用于对象
         2. 修饰方法，作用于对象
         3. 修饰静态方法，作用于类（所有对象）
         4. 修饰类，作用于类（所有对象）
      2. Lock依赖特殊的CPU指令，代码实现：ReentrantLock
      3. 对比：
         1. synchronized：不可中断锁，适合竞争不激烈，可读性好
         2. Lock：可中断锁，多样化同步，竞争激烈时能维持常态
         3. Atomic：竞争激烈时能维持常态，比Lock性能好；但只能同步一个值
3. 可见性
   1. 不可见性出现的原因，3个
   2. volatile，通过加入"内存屏障"和"禁止重排序""优化来实现;PS：没有原子性
4. 有序性
   1. happens-before原则
      1. 程序次序规则
      2. 锁定规则
      3. volatile变量规则
      4. 传递规则
      5. 程序启动规则
      6. 程序终端规则
      7. 程序终结规则
      8. 对象终结规则



### 第5章 安全发布对象

1. 安全发布对象-发布与逸出
   1. 举了个例子，一个方法，返回了一个对象的某个字段，但是这个字段可被修改
   2. 举了个逸出的例子，PS：看不懂，也听不懂
2. 安全的发布对象
   1. 懒汉模式 - 不安全模式
   2. 饥汉模式 - 安全
   3. 懒汉模式 - 安全 - synchronized
   4. 懒汉模式 - 不安全 - 双重检查 + synchronized
      指令重排列问题导致返回未初始化完成的对象
      PS：synchronized保证所标志的区块的互斥性和可见性
      1、互斥性：同时只能有一个线程
      2、可见性：指线程离开synchronized区块后，另一个线程接触到的就是上一个线程改变后的对象状态
   5. 懒汉模式 - 安全 - 双重检查 + synchronized + volatile
   6. 枚举方式实现



### 第6章 线程安全策略

1. 不可变对象

   1. 满足的条件：
      1. 对象创建以后其状态就不能修改
      2. 所有域都是final类型
      3. 对象是正确创建的（在对象创建时间，this引用没有一处）
   2. final关键字：类、方法、变量
   3. JDK: Collections.unmodifiableXXX()
      Guava: ImmutableXXX

2. 线程封闭

   1. Ad-hoc线程封闭：程序控制实现，最糟糕，忽略
   2. 堆栈封闭：局部变量，无并发问题
   3. ThreadLocal线程封闭：特别好的封闭方法
   4. 写了个Filter、Interceptor、ThreadLocal例子
      在Filter存，在Interceptor remove，PS：在Filterremove也可以吧

3. 线程不安全类与写法

   1. StringBuilder、StringBuffer
      例子：多个线程append
   2. SimpleDateFormate、JodaTime
      SimpleDateFormate要放到方法中new，别放到类中的field（线程不安全）
      JodaTime线程安全
   3. ArrayList、HashSet、HashMap
   4. 写法：先检查再执行：if( condition(a) ) { handler(a); }

4. 同步容器

   1. ArrayList -> Vector, Stack
      举个例子：Vector被两个线程remove和get，数组越界异常

      List遍历时remove()，ConcurrentModificationException
      参考：[Java ConcurrentModificationException异常原因和解决方法](http://www.cnblogs.com/dolphin0520/p/3933551.html)

   2. HashMap -> HashTable(key, value 不能为null)

   3. Collections.synchronized(List, Set, Map)

5. 并发容器

   1. ArrayList -> CopyOnWriteArrayList 适合读多写少
      读写分离、最终一致性、使用时另外开辟空间保证线程安全
   2. HashSet、TreeSet -> CopyOnWriteArraySet、ConcurrentSkipListSet
   3. HashMap、TreeMap -> ConcurrentHashMap、ConcurrentSkipListMap



### 第7章 J.U.C之AQS

1. AQS简介

2. CountDownLatch
   举个例子1：基本使用
   举个例子2：规定时间内完成，不完成也不管了

3. Semaphore，控制并发数
   Demo：线程调用时拿Semaphore的"资源"，可拿1个或多个，semaphore.acquire()
   Demo：线程尝试获取，获取不到直接结束；

   ```java
   if (semaphore.tryAcquire()) { // 尝试获取一个许可
     test(threadNum);
     semaphore.release(); // 释放一个许可
   }
   ```

4. CyclicBarrier：
   Demo：让线程n个n个的一起执行，并且线程等待n个都完成了，继续执行

   ```java
   private static CyclicBarrier barrier = new CyclicBarrier(5);

   private static void race(int threadNum) throws Exception {
     Thread.sleep(1000);
     log.info("{} is ready", threadNum);
     barrier.await();
     log.info("{} continue", threadNum);
   }
   ```

   Demo2：设置线程等待时间，超过时间后会抛：BarrierException

   ```java
   private static void race(int threadNum) throws Exception {
       Thread.sleep(1000);
       log.info("{} is ready", threadNum);
       try {
           barrier.await(2000, TimeUnit.MILLISECONDS);
       } catch (Exception e) {
           log.warn("BarrierException", e);
       }
       log.info("{} continue", threadNum);
   }
   ```

   Demo3：构造时，定义个线程执行

   ```java
   private static CyclicBarrier barrier = new CyclicBarrier(5, () -> {
           log.info("callback is running");
       });
   ```

5. ReentrantLock

   1. ReentrantLock和synchronized区别
   2. ReentrantLock独有的功能
      1. 可指定是公平锁还是非公平锁
      2. 提供Condition类，可以分组唤醒需要唤醒的线程
      3. 提供能够终端等待锁的线程的机制，lock.lockInterruptibly()
   3. ReentrantReadWriteRock
      Demo：读写HashMap，问题，读多写少，可能遇到饥饿问题，写不进
   4. StampedLock
   5. Condition

6. Condition



### 第8章 J.U.C组件拓展

1. Callable，有返回值
2. Future接口
   Demo
3. FutureTask类
   Demo
4. Fork/Join
5. BlockingQueue接口
   处理满队列的put()阻塞，和空队列的take()阻塞
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frayc7s2h7j21wu0dxdn2.jpg)
   1. ArrayBlockingQueue
   2. DelayQueue
   3. LinkedBlockingQueue
   4. PriorityBlockingQueue
   5. SynchronousQueue，只允许一个元素



### 第9章 线程调度-线程池

1. new Thread 弊端
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frayksbb3mj21q20jyncx.jpg)
2. 线程池的好处
   ![](https://ws1.sinaimg.cn/large/8747d788gy1fraym5tycij21qe0k5neo.jpg)
3. ThreadPoolExecutor
   1. corePoolSize：核心线程数量
   2. maximumPoolSize：最大线程数
   3. workQueue：阻塞队列
   4. keepAliveTime：
   5. unit
   6. threadFactory
   7. rejectHandler
4. 线程池状态
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frb0b4scztj21vr0nf15d.jpg)
   1. execute()
   2. submit()
   3. shutdown()
   4. shutdownNow()
   5. getTaskCount()
   6. getCompletedTaskCount()
   7. getPoolSize()
   8. getActiveCount()
5. 线程池类图
   ![](https://ws1.sinaimg.cn/large/8747d788gy1fraz6jowmrj218w0uxwyj.jpg)
6. Executor框架接口
   1. Executors.newCachedThreadPool
   2. Executors.newFixedThreadPool
   3. Executors.newScheduledThreadPool
   4. Executors.newSingleThreadExecutors



### 第10章 多线程并发拓展

1. 死锁
   1. 必要条件
      1. 互斥条件
      2. 请求和保持条件
      3. 不剥夺条件
      4. 环路等待条件
   2. 死锁Demo
   3. 死锁检测
2. 并发最佳实践
   1. 使用本地变量
   2. 使用不可变类
   3. 最小化锁的作用于范围：S = 1/(1 - a + a/n)
   4. 使用线程池
   5. 宁可使用同步也不要使用线程的wait和notify
   6. 使用BlockingQueue实现生产-消费你模式
   7. 使用并发集合而不是加了锁的同步集合
   8. 使用Semaphore创建有界的访问
   9. 宁可使用同步代码块，也不实用同步的方法
   10. 避免使用静态变量
3. Spring与线程安全
   1. 需要开发者保证
   2. Spring bean： singleton、prototype
   3. 无状态对象
4. HashMap与ConcurrentHashMap解析
   1. HashMap线程不安全，死循环分析
5. 多线程并发与线程安全总结
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frb7dpnhmmj224n14tkjl.jpg)



### 第11章 高并发之扩容思路

1. 扩容
   1. 垂直扩容：提高系统部件能力
   2. 水平扩容：增加增多的系统成员来实现
   3. 运用车子运送木材例子：
      1. 垂直扩容：换一辆装的更多木材的卡车
      2. 水平扩容：多辆卡车
2. 扩容-数据库
   1. 读操作扩展：memcache、redis、CDN等缓存
   2. 写操作扩展：Cassandra、Hbase等



### 第12章 高并发之缓存思路

1. 缓存特征
   1. 命中率：命中数/ (命中数 + 没有命中数)
   2. 最大元素（空间）
   3. 清空策略：FIFO, LFU（最少使用）, LRU（最近最少使用），过期时间，随机等
2. 缓存命中率影响因素
   1. 业务场景和业务需求：读多写少
   2. 缓存的设计（粒度和策略）
   3. 缓存容量和基础设施
3. 缓存分类和应用场景
   1. 本地缓存
   2. 分布式缓存
4. Guava Cache
5. Memcache
6. Redis
7. 高并发之缓存-高并发场景问题及实战讲解
   1. 高并发场景下缓存常见问题
      1. 缓存一致性
         ![](https://ws1.sinaimg.cn/large/8747d788gy1frb9s91lg5j20vp0gftak.jpg)
      2. 缓存并发问题
         ![](https://ws1.sinaimg.cn/large/8747d788gy1frbb2tkjicj20eu0czq82.jpg)
      3. 缓存穿透问题
         ![](https://ws1.sinaimg.cn/large/8747d788gy1frbb4s8l6dj20b40csmyv.jpg)
      4. 缓存的雪崩现象
         ![](https://ws1.sinaimg.cn/large/8747d788gy1frbb5ut9fmj20p80c3gr2.jpg)



### 第13章 高并发之消息队列思路

1. 消息队列
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frbc5uixkej21jb0nak5k.jpg)
   例子：系统发送短信，发送邮件之类；如果发送失败了，可以返回队列继续处理，或者做其他处理，做到异步解耦
   下订单例子
   消息队列的好处，网上搜
2. 消息队列特性
   1. 业务无关：制作消息分发
   2. FIFO
   3. 容灾：节点的动态增删和消息的持久化
   4. 性能：吞吐量提升，系统内部通讯效率提高
3. 为什么需要消息队列
   【生产】和【消费】的速度或稳定性等因素不一致
4. 好处
   1. 业务解耦，无需等待结果
   2. 最终一致性，记录 + 补偿
   3. 广播
   4. 错峰与流控
5. Kafka
6. RabbitMQ



### 第14章 高并发之应用拆分思路

1. 应用拆分
   1. 功能模块拆分，防止一个模块崩了，导致其他模块不能使用
      ![](https://ws1.sinaimg.cn/large/8747d788gy1frbu7tbm5aj21va0u67pm.jpg)
   2. 弊端：
      1. 拆分前管理一个应用就好了，拆分后需要管理多个应用，管理成本会提高很多，而且服务器成本也会提升
      2. 应用多了，带来更多的网络开销
2. 应用拆分原则
   1. 业务优先
   2. 循序渐进
   3. 兼顾技术：重构、分层
   4. 可靠测试
3. 应用拆分 - 思考
   1. 应用之间通信：RPC(dubbo等)、消息队列
      1. RPC实时性更高，一般不会使用http，webservice
      2. 消息队列：传输数据包小，数据量大，实时性要求不高的场景
   2. 应用之间数据库设计：每个应用都有独立的数据库
   3. 避免事务操作跨应用
4. Dubbo
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frbuitpj92j20pd0g4765.jpg)
5. 微服务
   ![](https://ws1.sinaimg.cn/large/8747d788gy1frburo42stj20mt0g9tbj.jpg)
   要解决的4个问题
   1. 1
   2. 2
   3. 3
   4. 4



### 第15章 高并发之应用限流思路

1. 应用限流

![](https://ws1.sinaimg.cn/large/8747d788gy1frbuwydjt7j20tb0dr75p.jpg)

2. 应用限流 - 算法
   1. 计数器法
      ![](https://ws1.sinaimg.cn/large/8747d788gy1frbv5irpg2j20ri0h3ta3.jpg)
   2. 滑动窗口
      ![](https://ws1.sinaimg.cn/large/8747d788gy1frbv5p3b33j20sg0g60vk.jpg)
   3. 漏桶算法
      ![](https://ws1.sinaimg.cn/large/8747d788gy1frbv5v0kbaj20rt0gtdi0.jpg)
   4. 令牌桶算法
      ![](https://ws1.sinaimg.cn/large/8747d788gy1frbv625jwej20rw0gr785.jpg)



### 第16章 高并发之服务降级与服务熔断思路

1. 服务降级：当请求处理不了， 返回个默认处理结果
   1. 分类
      1. 自动降级：超时、失败次数、故障、限流
      2. 主动降级秒杀、双11大促等人工降级
   2. 要考虑的问题
      1. 核心服务、非核心服务
      2. 是否支持降级，降级策略
      3. 业务放通场景，策略
2. 服务熔断
3. Hystrix
   1. 再通过第三方客户端访问（通常是通过网络）依赖服务出现高延迟或者失败时，位系统提供保护和控制
   2. 在分布式系统中防止级联失败
   3. 快速失败（Fail fast）同时能快速恢复
   4. 提供失败回退（Fallback）和优雅的服务降级机制



### 第17章 高并发之数据库切库分库分表思路

1. 数据库瓶颈
   1. 单个库数据量太大（1T ~ 2T）：多个库
   2. 单个数据库服务器压力过大、读写瓶颈：多个库
   3. 单个表数据量过大：分表
2. 数据库切库
   1. 切库的基础及实际运用：读写分离
   2. 自定义注解完成数据库切库
3. 数据库支持多个数据源与分库
   1. 支持多数据源吗、分库
   2. 数据库支持多个数据源 - 代码实现
4. 数据库分表
   1. 什么时候考虑分表
      1. 表的数据大到在优化sql和建立索引后，基本操作的速度还是非常慢
      2. 根据表的数据的快速增长，提前分表
   2. 分表策略
      1. 横向
      2. 纵向
   3. mubatis分表插件，shardbatis2.0



### 第18章 高并发之高可用手段介绍

1. 高可用的一些手段
   1. 任务调度系统分布式：elastic-job + zookeeper
   2. 主备切换：apache curator + zookeeper分布式锁实现
   3. 监控报警机制













