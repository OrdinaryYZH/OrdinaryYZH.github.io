## 第1章 RabbitMQ简介

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


















