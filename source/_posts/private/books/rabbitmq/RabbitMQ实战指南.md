## 第1章 RabbitMQ简介

### 1.4 RabbitMQ的安装及简单使用

#### 1.4.1 安装Erlang

1. 解压安装包，并配置安装目录，这里将目录设置到/opt/erlang下

   ```shell
   tar -zxvf opt_src_ -C /opt/erlang
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
