## SpringBoot + RabbitMQ 配置json转换

> 如果不配置Converter的话，默认是使用SimpleMessageConverter

### 1. MessageConverter需知

```java
public interface MessageConverter {

    Message toMessage(Object object, MessageProperties messageProperties)
            throws MessageConversionException;

    Object fromMessage(Message message) throws MessageConversionException;

}
```

以下列出了AmqpTemplate中的相关消息发送方法。它们比以前讨论的方法简单，因为它们不需要Message实例。**相反，MessageConverter负责通过将提供的对象转换为消息体的字节数组，然后添加任何提供的MessageProperties来“创建”每个消息。**

```java
void convertAndSend(Object message) throws AmqpException;

void convertAndSend(String routingKey, Object message) throws AmqpException;

void convertAndSend(String exchange, String routingKey, Object message)
    throws AmqpException;

void convertAndSend(Object message, MessagePostProcessor messagePostProcessor)
    throws AmqpException;

void convertAndSend(String routingKey, Object message,
    MessagePostProcessor messagePostProcessor) throws AmqpException;

void convertAndSend(String exchange, String routingKey, Object message,
    MessagePostProcessor messagePostProcessor) throws AmqpException;
```

在接收端，只有两种方法：一种接受队列名称，一种依赖于模板的“队列”属性已被设置。

```java
Object receiveAndConvert() throws AmqpException;

Object receiveAndConvert(String queueName) throws AmqpException;
```

> “异步消费者”一节中提到的MessageListenerAdapter也使用MessageConverter。

### 2. 开始本文例子

#### 2.1 本文Gole

同样的消息会存放到两个队列中，然后由不同的接收方法处理

* 一个接受Message
* 一个接受Java Bean

![](https://ws1.sinaimg.cn/large/8747d788gy1fydkmiy8jtj21kw0j0n6u.jpg)

#### 2.2 Setting up the project

* 配置maven依赖

  ```xml
  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-amqp</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-test</artifactId>
          <scope>test</scope>
      </dependency>
      <dependency>
          <groupId>com.fasterxml.jackson.core</groupId>
          <artifactId>jackson-core</artifactId>
          <version>2.9.6</version>
      </dependency>
      <dependency>
          <groupId>com.fasterxml.jackson.core</groupId>
          <artifactId>jackson-annotations</artifactId>
          <version>2.9.6</version>
      </dependency>
      <dependency>
          <groupId>com.fasterxml.jackson.core</groupId>
          <artifactId>jackson-databind</artifactId>
          <version>2.9.6</version>
      </dependency>
  </dependencies>
  ```

* 创建传输Bean

  ```java
  public final class CustomMessage implements Serializable {
   
      private final String text;
      private final int priority;
      private final boolean secret;
   
      public CustomMessage(@JsonProperty("text") String text,
                           @JsonProperty("priority") int priority,
                           @JsonProperty("secret") boolean secret) {
          this.text = text;
          this.priority = priority;
          this.secret = secret;
      }
   
      public String getText() {
          return text;
      }
   
      public int getPriority() {
          return priority;
      }
   
      public boolean isSecret() {
          return secret;
      }
   
      @Override
      public String toString() {
          return "CustomMessage{" +
                  "text='" + text + ''' +
                  ", priority=" + priority +
                  ", secret=" + secret +
                  '}';
      }
  }
  ```

#### 2.3 Sending messages

```java
@Service
public class CustomMessageSender {
 
    private static final Logger log = LoggerFactory.getLogger(CustomMessageSender.class);
 
    private final RabbitTemplate rabbitTemplate;
 
    public CustomMessageSender(final RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
 
    @Scheduled(fixedDelay = 3000L)
    public void sendMessage() {
        final var message = new CustomMessage("Hello there!", new Random().nextInt(50), false);
        log.info("Sending message...");
        rabbitTemplate.convertAndSend(MessagingApplication.EXCHANGE_NAME, MessagingApplication.ROUTING_KEY, message);
    }
}
```

#### 2.4 Receiving messages

```java
@Service
public class CustomMessageListener {
 
    private static final Logger log = LoggerFactory.getLogger(CustomMessageListener.class);
 
    @RabbitListener(queues = MessagingApplication.QUEUE_GENERIC_NAME)
    public void receiveMessage(final Message message) {
        log.info("Received message as generic: {}", message.toString());
    }
 
    @RabbitListener(queues = MessagingApplication.QUEUE_SPECIFIC_NAME)
    public void receiveMessage(final CustomMessage customMessage) {
        log.info("Received message as specific class: {}", customMessage.toString());
    }
}
```

#### 2.5 Configuration

```java
@SpringBootApplication
@EnableScheduling
public class MessagingApplication {
```

配置Exchange，Queue和Bindings

```java
@Bean
public TopicExchange appExchange() {
    return new TopicExchange(EXCHANGE_NAME);
}
 
@Bean
public Queue appQueueGeneric() {
    return new Queue(QUEUE_GENERIC_NAME);
}
 
@Bean
public Queue appQueueSpecific() {
    return new Queue(QUEUE_SPECIFIC_NAME);
}
 
@Bean
public Binding declareBindingGeneric() {
    return BindingBuilder.bind(appQueueGeneric()).to(appExchange()).with(ROUTING_KEY);
}
 
@Bean
public Binding declareBindingSpecific() {
    return BindingBuilder.bind(appQueueSpecific()).to(appExchange()).with(ROUTING_KEY);
}
```

**重点是以下配置**

```java
@Bean
public RabbitTemplate rabbitTemplate(final ConnectionFactory connectionFactory) {
    final var rabbitTemplate = new RabbitTemplate(connectionFactory);
    rabbitTemplate.setMessageConverter(producerJackson2MessageConverter());
    return rabbitTemplate;
}
 
@Bean
public Jackson2JsonMessageConverter producerJackson2MessageConverter() {
    // 这里可以配置自定义ObjectMapper
    return new Jackson2JsonMessageConverter();
}
```

#### 2.6 运行效果

```
INFO 13100 --- [pool-4-thread-1] c.t.rabbitmqconfig.CustomMessageSender   : Sending message...
INFO 13100 --- [cTaskExecutor-1] c.t.r.CustomMessageListener              : Received message as specific class: CustomMessage{text='Hello there!', priority=40, secret=false}
INFO 13100 --- [cTaskExecutor-1] c.t.r.CustomMessageListener              : Received message as generic: (Body:'{"text":"Hello there!","priority":40,"secret":false}' MessageProperties [headers={__TypeId__=com.thepracticaldeveloper.rabbitmqconfig.CustomMessage}, timestamp=null, messageId=null, userId=null, receivedUserId=null, appId=null, clusterId=null, type=null, correlationId=null, correlationIdString=null, replyTo=null, contentType=application/json, contentEncoding=UTF-8, contentLength=0, deliveryMode=null, receivedDeliveryMode=PERSISTENT, expiration=null, priority=0, redelivered=false, receivedExchange=appExchange, receivedRoutingKey=messages.key, receivedDelay=null, deliveryTag=3, messageCount=0, consumerTag=amq.ctag-cZstJRR8omlFYGJDJ0zvcA, consumerQueue=appGenericQueue])
```

注意Message的headers的**contentType=application/json**

```properties
headers={__TypeId__=com.thepracticaldeveloper.rabbitmqconfig.CustomMessage}
timestamp=null
messageId=null
userId=null
receivedUserId=null
appId=null
clusterId=null
type=null
correlationId=null
correlationIdString=null
replyTo=null
contentType=application/json
contentEncoding=UTF-8
contentLength=0
deliveryMode=null
receivedDeliveryMode=PERSISTENT
expiration=null
priority=0
redelivered=false
receivedExchange=appExchange
receivedRoutingKey=messages.key
receivedDelay=null
deliveryTag=3
messageCount=0
consumerTag=amq.ctag-cZstJRR8omlFYGJDJ0zvcA
consumerQueue=appGenericQueue
```

### 3. 参考链接

1. [Sending and receiving JSON messages with Spring Boot AMQP and RabbitMQ]:(https://thepracticaldeveloper.com/2016/10/23/produce-and-consume-json-messages-with-spring-boot-amqp/#Sending_messages)

2. [Spring AMQP中文文档]:(http://liuxing.info/2017/06/30/Spring%20AMQP%E4%B8%AD%E6%96%87%E6%96%87%E6%A1%A3/)




































