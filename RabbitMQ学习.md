# 一、RabbitMQ是什么？

RabbitMQ是一个开源的消息代理和队列服务器，用来通过普通协议在完全不同的应用之间共享数据，或者简单的将作业排队以便让分布式服务器进行处理。

+ `AMQP`，即`Advanced Message Queuing Protocol`，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。
+ `AMQP`的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
+ `RabbitMQ`是一个开源的`AMQP`实现，服务器端用`Erlang`语言编写，支持多种客户端，如：`Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX`,**`最重要的是也支持OC和swift`**。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。



# 二、RabbitMQ解决了什么问题？

1、流量削峰

2、应用解耦

3、异步处理

| 标志 | 中文名     | 英文名   | 描述                                             |
| ---- | ---------- | -------- | ------------------------------------------------ |
| P    | 生产者     | Producer | 消息的发送者，可以将消息发送到交换机             |
| C    | 消费者     | Consumer | 消息的接收者，从队列中获取消息进行消费           |
| X    | 交换机     | Exchange | 接收生产者发送的消息，并根据路由键发送给指定队列 |
| Q    | 队列       | Queue    | 存储从交换机发来的消息                           |
| type | 交换机类型 | type     | direct表示直接根据路由键（orange/black）发送消息 |

# 三、RabbitMQ怎么用？

业务场景说明：

> 用于解决用户下单以后，订单超时如何取消订单的问题。

- 用户进行下单操作（会有锁定商品库存、使用优惠券、积分一系列的操作）；
- 生成订单，获取订单的id；
- 获取到设置的订单超时时间（假设设置的为60分钟不支付取消订单）；
- 按订单超时时间发送一个延迟消息给RabbitMQ，让它在订单超时后触发取消订单的操作；
- 如果用户没有支付，进行取消订单操作（释放锁定商品库存、返还优惠券、返回积分一系列操作）。

- mall.order.direct（取消订单消息队列所绑定的交换机）:绑定的队列为mall.order.cancel，一旦有消息以mall.order.cancel为路由键发过来，会发送到此队列。
- mall.order.direct.ttl（订单延迟消息队列所绑定的交换机）:绑定的队列为mall.order.cancel.ttl，一旦有消息以mall.order.cancel.ttl为路由键发送过来，会转发到此队列，并在此队列保存一定时间，等到超时后会自动将消息发送到mall.order.cancel（取消订单消息消费队列）。

1、在pom中添加MongoDB依赖

2、在application.yml中添加rabbitmq配置信息

3、添加消息队列的枚举配置类QueueEnum，用于延迟消息队列及处理取消订单消息队列的常量定义，包括交换机名称、队列名称、路由键名称

4、添加rabbitmq配置rabbitmq config，用于配置交换机、队列及队列与交换机的绑定关系

5、添加延迟消息的发送者CancelOrderSender，用于向订单延迟消息队列（mall.order.cancel.ttl）里发送消息

6、添加取消订单消息的接收者CancelOrderReceiver，用于从取消订单的消息队列（mall.order.cancel）里接收消息

7、添加service serviceImpl controller方法

# 四、与其他类似工具对比？

https://juejin.im/post/5b32044ef265da59654c3027

Todo

# 参考文章

https://blog.csdn.net/whoamiyang/article/details/54954780

