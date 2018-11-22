## 第2章 RabbitMQ入门

### 2.1 相关概念介绍

RabbitMQ模型架构：

![](https://ws1.sinaimg.cn/large/8747d788gy1fxcpoojs6hj21b60m4ti4.jpg)

#### 2.1.1 生产者和消费者

Producer：生产者，投递消息的一方
发送的消息包括：消息体(payload)和标签(Lable)两部分。消息体：业务消息（例如JSON字符串）；标签：可理解为消息转发的机制（例如：一个交换机的名称和一个路由键）

Consumer：消费者，接收消息的一方
消费者订阅到的是队列。消费者消费的只是消息体，标签在路由的过程中被丢弃了，存到队列中的消息只有消息体。

Broker：消息中间件的服务节点

▲一个消费者的例子：消费者进程可以使用一个线程去接收消息，存到内存中，比如使用Java中的BlockingQueue。业务处理逻辑使用另一个线程从内存中读取，这样可以将应用进一步解耦，提高整个应用的处理效率。（问题，如果内存挂了咋整？）

#### 2.1.2 队列

Queue:队列，是RabbitMQ的内部对象，**用于存储消息**。

* RabbitMQ中的消息只能存在队列中（与Kafka不同）
* RabbitMQ的生产者消息最终投递到队列中，消费者从队列中获取消息并消费
* 多个消费者可订阅同一个队列，这时队列中的消息会**被平摊**（轮训）给各个消费者处理，不是各个消费者都收到消息并处理
* RabbitMQ**不支持队列层面的广播消费**。（如果需要广播消费，需要在其上进行二次开发，处理逻辑会变得异常复杂，不建议）

#### 2.1.3 交换器、路由键、绑定

Exchange：交换器。生产者将消息发送到Exchange，由交换器将消息路由到一个或者多个队列中。如果路由不到，可能返回给生产者，或许直接丢弃。

RoutingKey：路由键。

Binding：绑定。（可以有多个）

![](https://ws1.sinaimg.cn/large/8747d788gy1fx96ii46bkj21ox0xx19e.jpg)

#### 2.1.4 交换器类型 

> RabbitMQ常用的交换器类型有fanout、direct、topic、headers这四种。AMQP协议列还提到了另外两种：System和自定义，这里不说

##### 1. fanout(扇出)

> 将消息发送给所有绑定到该交换器的队列中（**散弹枪**）

![](https://ws1.sinaimg.cn/large/8747d788gy1fxcou3wk91j218g0mojuk.jpg)

##### 2. direct

>  把消息路由到那些BindingKey和RoutingKey完全匹配的队列中（**激光枪**）

![](https://ws1.sinaimg.cn/large/8747d788gy1fxcpqv85vhj20y70crwh0.jpg)

##### 3. topic

> 也是将消息路由到BindingKey和RoutingKey相匹配的队列中，但是存在匹配规则：

1. ".": RoutingKey和BindingKey都为一个"."分割的字符串(被"."分隔开的每一段独立的字符串成为一个单词）
   ![](https://ws1.sinaimg.cn/large/8747d788gy1fxcpkjk611j20i1082q2z.jpg)
2. "*": 模糊匹配，代表一个单词
3. "#": 模糊匹配，用于匹配多规则单词（包括0个）

![](https://ws1.sinaimg.cn/large/8747d788gy1fxcpulo12fj20yh0bp0uy.jpg)

##### 4. headers

> 该类型交换器不依赖于路由键的匹配规则，而是根据发送的消息内容中的headers属性进行匹配。又分any和all两种。（这个类型的交换器性能会很差，而且也不实用，所以很少用）

#### 2.1.5 RabbitMQ运转流程

##### 1. 生产者发送消息过程

1. 生产者连接到RabbitMQ Broker ， 建立一个连接( Connection) ，开启一个信道(Channel) 
2. 生产者声明一个交换器并设置相关属性
3. 生产者声明一个队列井设置相关属性
4. 生产者通过路由键将交换器和队列绑定起来
5. 生产者发送消息至RabbitMQ Broker，其中包含路由键、交换器等信息
6. 相应的交换器根据接收到的路由键查找相匹配的队列。
7. 如果找到，则将从生产者发送过来的消息存入相应的队列中。
8. 如果没有找到，则根据生产者配置的属性选择丢弃还是回退给生产者
9. 关闭信道。
10. 关闭连接。

##### 2. 消费者接受的过程

1. 消费者连接到RabbitMQ Broker ，建立一个连接(Connection ) ，开启一个信道(Channel) 。
2. 消费者向RabbitMQ Broker 请求消费相应队列中的消息，可能会设置相应的回调函数，以及做一些准备工作
3. 等待RabbitMQ Broker 回应并投递相应队列中的消息， 消费者接收消息。
4. 消费者确认( ack) 接收到的消息。
5. RabbitMQ 从队列中删除相应己经被确认的消息。
6. 关闭信道。
7. 关闭连接。

##### 3. Connection和Channel

###### 概念

> Connection：相当于一个TCP链接
>
> Channel：简历在Connection上的虚拟链接，每个Channel被指派一个唯一的ID

![](https://ws1.sinaimg.cn/large/8747d788gy1fxhbjs7cawj21ak0rqwqi.jpg)

###### 为什么需要Channel？

> 对于操作系统而言，建立和销毁TCP是非常昂贵的开销，遇上高峰，容易出现性能瓶颈。
>
> RabbitMQ采用类似NIO的做法，选择TCP连接复用，不仅可以减少性能开销，同时便于管理。

###### 只有一个Connection的性能问题

当流量不大的时候，1个Connectinon多个Channel可以节省TCP连接资源；

但是当流量大时，多个Channel复用一个Connection就会产生性能瓶颈（因为流量被限制了），此时就需要多个Connectinon，将这些Channel分摊到多个Connection中。（参考第9章）

### 2.2 AMQP协议介绍

#### 2.1.1 AMQP生产者流转过程

#### 2.1.2 AMQP消费者流转过程

#### 2.1.3 AMQP命令概览