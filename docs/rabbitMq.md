<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [四种Exchange模式](#四种exchange模式)
    - [Fanout Exchange](#fanout-exchange)
    - [Direct Exchange](#direct-exchange)
    - [Topic Exchange](#topic-exchange)
    - [Headers Exchange](#headers-exchange)

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