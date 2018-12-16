## 第1章 RabbitMQ简介

### 1.1 什么是消息中间件

消息队列中间件：是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并给予数据通信来进行分布式系统的集成。通过提供消息传递和消息排队模型，它可以在分布式环境下扩展进程间的通信。

### 1.2 消息中间件的作用

| 业务方面                      | 特色功能               | 高并发场景下 |
| ----------------------------- | ---------------------- | ------------ |
| 1. 解耦                       | 4. 冗余（存储/持久化） | 7. 削峰      |
| 2. 扩展性（相对于消费端来说） | 5. 可恢复性            | 8. 缓冲      |
| 3. 异步通信                   | 6. 顺序保证            |              |



1. 解耦：用户登陆的例子：

   业务逻辑需要用户在登录完成之后，为用户发送推送短信，同时在安全系统中进行记录，甚至还需要给用户进行消息推送等功能。如果使用直接服务调用的方式，就会使得登陆系统的越来越臃肿，逻辑与其他服务紧密耦合在一起。如果要增加新的功能，需要改动的地方也会比较多

   ![](http://intheworld.win/wordpress/wp-content/uploads/2017/05/image189.png)

   作为对比，贴出一张使用消息中间件的结构图:

   ![](http://intheworld.win/wordpress/wp-content/uploads/2017/05/image-2.png)

   > 参考：[理解消息中间件](http://intheworld.win/2017/05/13/%E7%90%86%E8%A7%A3%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6/)

2. 冗余（存储）：某些情况下，处理数据的过程会失败。消息中间件可以把数据进行持久化直到他们已经被完全处理，通过这一方式规避了数据丢失的风险。

3. 扩展性

   消息中间件本身可以分布式部署，所以其本身具有比较强的扩展性。同时，使用消息中间件的架构本身也具有很好的可扩展性。
   ![](http://intheworld.win/wordpress/wp-content/uploads/2017/05/mq-1.jpg)

   其原因其实简单，看上面这张图就能一目了然了。对于基于消息系统的架构来说，消息生产者与消费者的水平扩展非常简单，只是**简单地增加消息队列的Publisher和Subscriber**而已

4. 削峰

   消息中间件能够使关键组件支撑突发访问压力，不会因为突发的超负荷请求而完全奔溃

5. 可恢复性（跟存储性类似）

   一个消费者挂掉时，加入消息中间件中的消息仍然可以在系统恢复后进行处理

6. 顺序保证

7. 缓冲

   消息中间件通过一个**缓冲层**来帮助任务最高效率地执行，写入消息中间件的处理会尽可能快速。该缓冲层有助于控制和优化数据流经过系统的速度。

8. 异步通信

   在不需要立即处理的消息，消息中间件提供了异步处理机制，允许应用把一些消息放进消息中间件中，但并不立即处理，在之后需要的时候慢慢处理。

### 1.3 RabbitMQ的起源

RabbitMQ的特点：

1. **可靠性**：RabbitMQ使用一些机制来保证可靠性，如持久化、传输确认及发布确认等。
2. **灵活的路由**
3. **扩展性**：多个RabbitMQ节点可以组成一个集群，也可以根据需求动态地扩展集群中节点
4. **高可用性**：队列可以在集群中的机器上设置镜像，使得在部分节点出现问题的情况下队列仍然可用。
5. **多种协议**：除了AMQP，还支持STOMP、MQTT等多种协
6. **多语言客户端**
7. **管理界面**
8. **插件机制**

### 1.4 RabbitMQ的安装及简单使用

#### 1.4.1 安装Erlang

1. 解压安装包，并配置安装目录，这里将目录设置到/opt/erlang下（解压路径不放到/opt/erlang下也可以，记得配置./configure文件就好）

   ```shell
   tar -zxvf opt_src_....tar.gz -C /opt/erlang（tar -Jxvf opt_src_....tar.xz -C /opt/erlang）
   cd /opt/erlang
   ./configure --prefix=/opt/erlang
   ```

   如果出现：No curses library functions found。需要安装ncurses：
   `yum install ncurses-devel`

2. 安装Erlang

   ```shell
   make
   make install
   ```

   如果安装过程中出现"NO *** found"的提示，需要安装对象的包

3. 修改`/etc/profile`配置文件

   ```properties
   ERLANG_HOME=/opt/erlang
   export PATH=$PATH:$ERLANG_HOME/bin
   export ERLANG_HOME
   ```

   执行source /etc/profile 让配置文件生效

4. 检验是否安装成功：

   `erl`

注意：make之后会有提示哪些部件没安装，需要把它们安装完，不然后面RabbitMQ启动会报各种错误。

#### 1.4.2 RabitMQ的安装

RabbitMQ的安装简单很多，解压后，配置下即可：

1. 解压：

    ```shell
    tar zxvf rabbitmq-server-generic-unix-xxx.tar.gz -C /opt
    cd /opt
    #建立软连接
    ln -sv rabbitmq_server-xxx rabbitmq
    ```

    > 注意：为 rabbitmq_server-3.7.5 目录创建软件链接时，rabbitmq_server-3.7.5 后面一定不要加上 `/` ，一般习惯用 TAB 键来补全命令的，都会自动把 `/` 给补上去。这样后面使用 rabbitmqctl 等命令时就会报错：
    >
    > [root@node05 ~]# rabbitmqctl status escript: exception error: undefined function rabbitmqctl_escript:main/1 in function  escript:run/2 (escript.erl, line 759) in call from escript:start/1 (escript.erl, line 277) in call from init:start_em/1 in call from init:do_boot/3
    >
    > 链接：https://juejin.im/post/5b3dff6c51882519a42661ca

2. 修改/etc/profile文件：

    ```properties
    export PATH=$PATH:/opt/rabbitmq/sbin
    export RABBITMQ_HOME=/opt/rabbitmq
    ```

    执行source /etc/profile

#### 1.4.3 RabbitMQ的运行

后台启动RabbitMQ：

```shell
rabbitmq-server -detached
```

查看状态：

```shell
rabbitmqctl status
rabbitmqctl cluster_status #查看集群状态
```



#### 1.4.4 生产者与消费消息

##### 添加账号

> 默认情况下访问RabbitMQ的用户名和密码都是"guest"，这个账号有限制，只能通过本地localhost访问，所以我们需要添加一个用户

1. 添加新用户：

   ```shell
   [root@localhost ~]# rabbitmqctl add_user root root123
   Adding user "root" ...
   ```

2. 为root用户设置所有权限

   ```shell
   [root@localhost ~]# rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
   Setting permissions for user "root" in vhost "/" ...
   ```

3. 设置root用户为管理员角色

   ```shell
   [root@localhost ~]# rabbitmqctl set_user_tags root administrator
   Setting tags for user "root" to [administrator] ...
   ```

##### 一个生产者跟消费者例子（具体看代码）


















