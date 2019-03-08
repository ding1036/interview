<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [四种Exchange模式](#四种exchange模式)
    - [Fanout Exchange](#fanout-exchange)
    - [Direct Exchange](#direct-exchange)
    - [Topic Exchange](#topic-exchange)
    - [Headers Exchange](#headers-exchange)
- [RabbitMQ如何解决各种情况下丢数据的问题](#rabbitmq如何解决各种情况下丢数据的问题)

<!-- /TOC -->

# 四种Exchange模式

代码参考：[demo](https://github.com/ding1036/seckillproject/tree/master/src/main/java/com/springboot/seckill/rabbitmq)

## Fanout Exchange

![](/img/fanout_exchange.png)
所有发送到Fanout Exchange的消息都会被转发到与该Exchange 绑定(Binding)的所有Queue上。

Fanout Exchange  不需要处理RouteKey 。只需要简单的将队列绑定到exchange 上。这样发送到exchange的消息都会被转发到与该交换机绑定的所有队列上。类似子网广播，每台子网内的主机都获得了一份复制的消息。

所以，Fanout Exchange 转发消息是最快的。

```java
　　　　 /// <summary>
        /// 生产者
        /// </summary>
        /// <param name="change"></param>
        private static void ProducerMessage(MyMessage msg)
        {
            var advancedBus = CreateAdvancedBus();

            if (advancedBus.IsConnected)
            {
                var exchange = advancedBus.ExchangeDeclare("user", ExchangeType.Fanout);

                advancedBus.Publish(exchange, "", false, new Message<MyMessage>(msg));
            }
            else
            {
                Console.WriteLine("Can't connect");
            }

        }

        /// <summary>
        /// 消费者
        /// </summary>
        private static void ConsumeMessage()
        {
            var advancedBus = CreateAdvancedBus();
            var exchange = advancedBus.ExchangeDeclare("user", ExchangeType.Fanout);

            var queue = advancedBus.QueueDeclare("user.notice.wangwu");
            advancedBus.Bind(exchange, queue, "user.notice.wangwu");
            advancedBus.Consume(queue, registration =>
            {
                registration.Add<MyMessage>((message, info) => { Console.WriteLine("Body: {0}", message.Body); });
            });
        }
```

## Direct Exchange

![](/img/direct_exchange.png)
所有发送到Direct Exchange的消息被转发到RouteKey中指定的Queue。

　　Direct模式,可以使用rabbitMQ自带的Exchange：default Exchange 。所以不需要将Exchange进行任何绑定(binding)操作 。消息传递时，RouteKey必须完全匹配，才会被队列接收，否则该消息会被抛弃。

```java
/// <summary>
        /// 生产者
        /// </summary>
        /// <param name="change"></param>
        private static void ProducerMessage(MyMessage msg)
        {
            var advancedBus = CreateAdvancedBus();

            if (advancedBus.IsConnected)
            {
                var queue = advancedBus.QueueDeclare("user.notice.zhangsan");

                advancedBus.Publish(Exchange.GetDefault(), queue.Name, false, new Message<MyMessage>(msg));
            }
            else
            {
                Console.WriteLine("Can't connect");
            }

        }

        /// <summary>
        /// 消费者
        /// </summary>
        private static void ConsumeMessage()
        {
            var advancedBus = CreateAdvancedBus();

            var exchange = advancedBus.ExchangeDeclare("user", ExchangeType.Direct);

            var queue = advancedBus.QueueDeclare("user.notice.lisi");

            advancedBus.Bind(exchange, queue, "user.notice.lisi");

            advancedBus.Consume(queue, registration =>
            {
                registration.Add<MyMessage>((message, info) =>
                {
                    Console.WriteLine("Body: {0}", message.Body);
                });
            });
        }
```

## Topic Exchange

![](/img/topic_exchange.png)
所有发送到Topic Exchange的消息被转发到所有关心RouteKey中指定Topic的Queue上，

　　Exchange 将RouteKey 和某Topic 进行模糊匹配。此时队列需要绑定一个Topic。可以使用通配符进行模糊匹配，符号“#”匹配一个或多个词，符号“*”匹配不多不少一个词。因此“log.#”能够匹配到“log.info.oa”，但是“log.*” 只会匹配到“log.error”。

　　所以，Topic Exchange 使用非常灵活。

```java
/// <summary>
        /// 生产者
        /// </summary>
        /// <param name="change"></param>
        private static void ProducerMessage(MyMessage msg)
        {
            //// 创建消息bus
            IBus bus = CreateBus();

            try
            {
                bus.Publish(msg, x => x.WithTopic(msg.MessageRouter));
            }
            catch (EasyNetQException ex)
            {
                //处理连接消息服务器异常 
            }

            bus.Dispose();//与数据库connection类似，使用后记得销毁bus对象
        }

        /// <summary>
        /// 消费者
        /// </summary>
        private static void ConsumeMessage(MyMessage msg)
        {
            //// 创建消息bus
            IBus bus = CreateBus();

            try
            {
                bus.Subscribe<MyMessage>(msg.MessageRouter, message => Console.WriteLine(msg.MessageBody), x => x.WithTopic("user.notice.#"));
            }
            catch (EasyNetQException ex)
            {
                //处理连接消息服务器异常 
            }
        }
```

## Headers Exchange

eaders类型的exchange使用的比较少，它也是忽略routingKey的一种路由方式。是使用Headers来匹配的。Headers是一个键值对，可以定义成Hashtable。发送者在发送的时候定义一些键值对，接收者也可以再绑定时候传入一些键值对，两者匹配的话，则对应的队列就可以收到消息。匹配有两种方式all和any。这两种方式是在接收端必须要用键值"x-mactch"来定义。all代表定义的多个键值对都要满足，而any则代码只要满足一个就可以了。

[toTop](#jump)

# RabbitMQ如何解决各种情况下丢数据的问题

1.**生产者丢数据**

RabbitMQ提供``transaction``和``confirm``模式来确保生产者不丢消息。
transaction机制:发送消息前，开启事物``channel.txSelect()``，然后发送消息，如果发送过程中出现什么异常，事物就会回滚``channel.txRollback()``，如果发送成功则提交事物``channel.txCommit()``。
然而缺点就是**吞吐量下降了**。
生产上用confirm模式的居多。一旦channel进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID(从1开始)，一旦
消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个``Ack``给生产者(包含消息的唯一ID)，这就使得生产者知道消息已经正确到达目的队列了.如果rabiitMQ没能处理该消息，则会发送一个``nack``消息给你，你可以进行重试操作。

confirm例子
```java
@Service
public class HelloSender1 implements RabbitTemplate.ConfirmCallback {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    public void send() {
    String context = "你好现在是 " + new Date() +"";
    this.rabbitTemplate.setConfirmCallback(this);
    //exchange,queue 都正确,confirm被回调, ack=true
    //this.rabbitTemplate.convertAndSend("exchange","topic.message", context);
    //exchange 错误,queue 正确,confirm被回调, ack=false
    //this.rabbitTemplate.convertAndSend("fasss","topic.message", context);
    //exchange 正确,queue 错误 ,confirm被回调, ack=true; return被回调 replyText:NO_ROUTE
    //this.rabbitTemplate.convertAndSend("exchange","", context);
    //exchange 错误,queue 错误,confirm被回调, ack=false
    this.rabbitTemplate.convertAndSend("fasss","fass", context);
    }
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {

    }
}
```

2.**消息队列丢数据**
处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

如何持久化
①将queue的持久化标识``durable``设置为``true``,则代表是一个持久的队列
②发送消息的时候将``deliveryMode=2``
这样设置以后，rabbitMQ就算挂了，重启后也能恢复数据。在消息还没有持久化到硬盘时，可能服务已经死掉，这种情况可以通过引入``mirrored-queue``即镜像队列，但也不能保证消息百分百不丢
失（整个集群都挂掉）

例子
```java
/**
 * 第二个参数：queue的持久化是通过durable=true来实现的。
 * 第三个参数：exclusive：排他队列，如果一个队列被声明为排他队列，该队列仅对首次申明它的连接可见，并在连接断开时自动删除。这里需要注意三点：
　　 1. 排他队列是基于连接可见的，同一连接的不同信道是可以同时访问同一连接创建的排他队列；
　　 2.“首次”，如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同；
　　 3.即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除的，这种队列适用于一个客户端发送读取消息的应用场景。
 * 第四个参数：自动删除，如果该队列没有任何订阅的消费者的话，该队列会被自动删除。这种队列适用于临时队列。
 * @param
 * @return
 */
 @Bean
 public Queue queue() {
    Map<String, Object> arguments = new HashMap<>();
    arguments.put("x-message-ttl", 25000);//25秒自动删除
    Queue queue = new Queue("topic.messages", true, false, true, arguments);
    return queue;
 }
    MessageProperties properties=new MessageProperties();
    
    properties.setContentType(MessageProperties.DEFAULT_CONTENT_TYPE);

    properties.setDeliveryMode(MessageProperties.DEFAULT_DELIVERY_MODE);//持久化设置
    
    properties.setExpiration("2018-12-15 23:23:23");//设置到期时间
    
    Message message=new Message("hello".getBytes(),properties);
    
    this.rabbitTemplate.sendAndReceive("exchange","topic.message",message);
```

3.**消费者丢数据**
启用手动确认模式可以解决这个问题
①自动确认模式，消费者挂掉，待ack的消息回归到队列中。消费者抛出异常，消息会不断的被重发，直到处理成功。不会丢失消息，即便服务挂掉，没有处理完成的消息会重回队列，但是异常会让消息不断重试。
②手动确认模式
③不确认模式，``acknowledge="none"`` 不使用确认机制，只要消息发送完成会立即在队列移除，无论客户端异常还是断开，只要发送完就移除，不会重发。

指定Acknowledge的模式：
``spring.rabbitmq.listener.direct.acknowledge-mode=manual
``
表示该监听器手动应答消息

针对手动确认模式，有以下特点：

1.使用手动应答消息，有一点需要特别注意，那就是不能忘记应答消息，因为对于RabbitMQ来说处理消息没有超时，只要不应答消息，他就会认为仍在正常处理消息，导致消息队列出现阻塞，影响业务执行。

2.如果消费者来不及处理就死掉时，没有响应ack时，会项目启动后会重复发送一条信息给其他消费者；

3.可以选择丢弃消息，这其实也是一种应答，如下，这样就不会再次收到这条消息。
```java
channel.basicNack(message.getMessageProperties().getDeliveryTag(), false,false);
```

4.如果消费者设置了手动应答模式，并且设置了重试，出现异常时无论是否捕获了异常，都是不会重试的

5.如果消费者没有设置手动应答模式，并且设置了重试，那么在出现异常时没有捕获异常会进行重试，如果捕获了异常不会重试。
重试机制：
```java
//最大重试次数
spring.rabbitmq.listener.simple.retry.max-attempts=5

//是否开启消费者重试（为false时关闭消费者重试，这时消费端代码异常会一直重复收到消息）
spring.rabbitmq.listener.simple.retry.enabled=true 

//重试间隔时间（单位毫秒）
spring.rabbitmq.listener.simple.retry.initial-interval=5000 

//重试次数超过上面的设置之后是否丢弃（false不丢弃时需要写相应代码将该消息加入死信队列）
spring.rabbitmq.listener.simple.default-requeue-rejected=false 

```

如果设置了重试模式，那么在出现异常时没有捕获异常会进行重试，如果捕获了异常不会重试。
当出现异常时，我们需要把这个消息回滚到消息队列，有两种方式：

```java
//ack返回false，并重新回到队列，api里面解释得很清楚
//ack返回false，并重新回到队列，api里面解释得很清楚
channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);

//拒绝消息
channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);

```

经过开发中的实际测试，当消息回滚到消息队列时，这条消息不会回到队列尾部，而是仍是在队列头部，这时消费者会立马又接收到这条消息进行处理，接着抛出异常，进行回滚，如此反复进行。这种情况会导致消息队列处理出现阻塞，消息堆积，导致正常消息也无法运行。对于消息回滚到消息队列，我们希望比较理想的方式时出现异常的消息到 达消息队列尾部，这样既保证消息不会丢失，又保证了正常业务的进行，因此我们采取的解决方案是，将消息进行应答，这时消息队列会删除该消息，同时我们再次发送该消息 到消息队列，这时就实现了错误消息进行消息队列尾部的方案。

```java
 //手动进行应答
 channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
 
 //重新发送消息到队尾
 channel.basicPublish(message.getMessageProperties().getReceivedExchange(),
 message.getMessageProperties().getReceivedRoutingKey(), MessageProperties.PERSISTENT_TEXT_PLAIN,
 JSON.toJSONBytes(new Object()));

 ```
 
如果**一个消息体本身有误，会导致该消息体，一直无法进行处理，而服务器中刷出大量无用日志**。解决这个问题可以采取两种方案：

1.一种是对于日常细致处理，分清哪些是可以恢复的异常，哪些是不可以恢复的异常。对于可以恢复的异常我们采取第三条中的解决方案，对于不可以处理的异常，我们采用记录日志，直接丢弃该消息方案。

2.另一种是我们对每条消息进行标记，记录每条消息的处理次数，当一条消息，多次处理仍不能成功时，处理次数到达我们设置的值时，我们就丢弃该消息，但需要记录详细的日志。
消息监听内的异常处理有两种方式：
1) 内部catch后直接处理，然后使用channel对消息进行确认

2) 配置``RepublishMessageRecoverer``将处理异常的消息发送到指定队列专门处理或记录。监听的方法内抛出异常貌似没有太大用处。因为抛出异常就算是重试也非常有可能会继续出现异常，当重试次数完了之后消息就只有重启应用才能接收到了，很有可能导致消息消费不及时。当然可以配置``RepublishMessageRecoverer``来解决，但是万一``RepublishMessageRecoverer``发送失败了呢。。那就可能造成消息消费不及时了。所以即使需要将处理出现异常的消息统一放到另外队列去处理，个人建议两种方式：
① catch异常后，手动发送到指定队列，然后使用channel给rabbitmq确认消息已消费
② 给Queue绑定死信队列，使用nack（requque为false）确认消息消费失败

[toTop](#jump)