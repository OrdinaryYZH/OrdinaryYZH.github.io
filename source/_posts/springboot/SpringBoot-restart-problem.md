---
title: SpringBoot项目添加日志打包后，无限重启
date: 2018-3-5 00:00:00
toc: true
---

> 本篇文章主要叙述遇到这种情况的原因以及解决方法

**说明：**

**项目基于`SpringBoot 1.5.9.RELEASE`、日志：`Slf4j `+ `log4j2`**

**打包使用`maven-jar-plugin` + `maven-assembly-plugin`**

## 1.  起因

​	公司项目需要添加本地文件存储日志，所以很自然的在log4j2的配置文件中(`log4j2-*.xml`)添加`RollingFile Appender`:

```xml
...
    <Appenders>
        ...
        <RollingFile name="RollingFileInfo" fileName="../logs/info.log"
                     filePattern="../logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
        ...
    </Appenders>
	...
    <Loggers>
        <root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RollingFileInfo"/>
        </root>
    </Loggers>
...
```

**注意：这里选择的日志路径是`../logs/...`，即当前目录的上一层**

> 补充说明下打包后的目录结构：

![](https://ws1.sinaimg.cn/large/8747d788gy1fp3d6hpzz5j20p106x0tl.jpg)

config目录中：

![](https://ws1.sinaimg.cn/large/8747d788gy1fp3ecc0rikj20uw057wf9.jpg)

添加成功后，启动bin中的脚本，然后不可思议的一幕发生了：**日志是成功生成了，但是启动成功后，一直重启**

![](http://7xrjzo.com1.z0.glb.clouddn.com/uploads%2Ffull_emotion%2Femotion_pic%2F10091%2F1-1F622210432.jpg)

这时候试过的方法：

- [x] 换了绝对路径到桌面

是成功的，启动后不重启了，这时候就怀疑log4j2的`RollingFile`配置路径只能使用绝对路径？显然并不大可能

这时候叫来身边的小明帮忙看下...

半小时过去了...没发现原因

最后把配置文件发给小明，小明回到工位后一顿乱操作，尝试许久之后，说了一句：是不是日志被修改了，`devtools`发现文件被修改了，就自动重启了，然后就死循环了...

于是google下`devtools`原理：`devtools`会监听`classpath`下的文件变动，并且会立即重启应用（发生在保存时机），注册：因为其采用的虚拟机制，重启是很快的。

这时候再确认下打包的jar中的`classpath`中配置：

![](https://ws1.sinaimg.cn/large/8747d788gy1fp3e9m98jij213t07odh3.jpg)

果然把打包后jar包的当前目录加到了`classpath`中

到这里其实已经破案了：就是因为打的日志`../logs/info.log`在`classpath`下，每次写进日之后，`devtools`就会触发重启..

## 2. 解决

1. 日志别输出到`classpath`中
2. ~~不使用`devtools`插件~~
3. 添加`devtools`忽略的监听路径/文件：`spring.devtools.restart.additional-exclude`

