## 安装Linux

1. 使用虚拟机安装，virtualbox
2. 安装的版本为CentOS7：CentOS-7-x86_64-DVD-1804

安装完要记得配置：

1. 修改 /etc/sysconfig/network-scripts/ifcfg-xxx的onboot

   ```properties
   ONBOOT=yes                #系统启动时是否自动加载 
   ```

   修改后重启`service network restart`

   ![](https://ws1.sinaimg.cn/large/8747d788gy1ful6k64j56j21kw0k1k1l.jpg)

2. `yum install net-tools`，安装后可使用`ifconfig`

3. 使用桥接模式、修改为静态IP，防止每次都变动，参考：[Centos 7 学习之静态IP设置](https://blog.csdn.net/johnnycode/article/details/40624403),桥接模式后xshell可以连得上

4. 替换默认源

   参考：http://mirrors.163.com/.help/centos.html

5. 安装vim，`yum install vim`



## SSH

### SSH服务端安装

1. 安装ssh

   `yum install openssh-server`

2. 启动SSH

   `service sshd start`

3. 设置开机运行

   `chkconfig sshd on`

### SSH config

为了方便管理多个ssh，给不同的服务器起别名

config存放在 ~/.ssh/config，编辑如下：

```
host "linux"
	HostName 192.168.0.111
	User root
	Port 22
```

编辑完后，ssh linux即可连接

### SSH 免密码登陆 和 端口安全

#### 免密码登陆

客户端生成私钥和公钥，把公钥配置放到服务器的`~/.ssh/authorized_keys`中。

#### 端口安全

修改`/etc/ssh/sshd_config`配置