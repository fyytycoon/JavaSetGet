## JMS

什么是Java消息服务

Java消息服务指的是两个应用程序之间进行异步通信的API，它为标准协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持Java应用程序开发。在JavaEE中，当两个应用程序使用JMS进行通信时，它们之间不是直接相连的，而是通过一个共同的消息收发服务组件关联起来以达到解耦/异步削峰的效果。

### JMS的组成结构

JMS提供者(JMS Provider)、JMS生产者(JMS Producer)、JMS消费者(JMS Consumer)、JMS消息(JSM Message)

[JMS百度百科](https://baike.baidu.com/item/JMS/2836691?fr=aladdin)



### 对象模型

JMS对象模型包含如下几个要素： 

1）连接工厂。连接工厂（ConnectionFactory）是由管理员创建，并绑定到[JNDI](https://baike.baidu.com/item/JNDI)树中。客户端使用JNDI查找连接工厂，然后利用连接工厂创建一个JMS连接。

2）JMS连接。JMS连接（Connection）表示JMS客户端和服务器端之间的一个活动的连接，是由客户端通过调用连接工厂的方法建立的。

3）JMS会话。JMS会话（Session）表示JMS客户与JMS服务器之间的会话状态。JMS会话建立在JMS连接上，表示客户与服务器之间的一个会话线程。

4）JMS目的。JMS目的（Destination），又称为消息队列，是实际的消息源。

5）JMS生产者和消费者。生产者（Message Producer）和消费者（Message Consumer）对象由Session对象创建，用于发送和接收消息。

6）JMS消息通常有两种类型：

① 点对点（Point-to-Point）。在点对点的消息系统中，消息分发给一个单独的使用者。点对点消息往往与队列（javax.jms.Queue）相关联。

点对点的消息发送方式主要建立在 Message Queue,Sender,reciever上，Message Queue 存贮消息，Sneder 发送消息，receive接收消息.具体点就是Sender Client发送Message Queue ,而 receiver Cliernt从Queue中接收消息和"发送消息已接受"到Quere,确认消息接收。消息发送客户端与接收客户端没有时间上的依赖，发送客户端可以在任何时刻发送信息到Queue，而不需要知道接收客户端是不是在运行

② 发布/订阅（Publish/Subscribe）。发布/订阅消息系统支持一个事件驱动模型，消息生产者和消费者都参与消息的传递。生产者发布事件，而使用者订阅感兴趣的事件，并使用事件。该类型消息一般与特定的主题（javax.jms.Topic）关联。

发布/订阅方式用于多接收客户端的方式.作为发布订阅的方式，可能存在多个接收客户端，并且接收端客户端与发送客户端存在时间上的依赖。一个接收端只能接收他创建以后发送客户端发送的信息。作为subscriber ,在接收消息时有两种方法，destination的receive方法，和实现message listener 接口的onMessage 方法。

![](../img/JMS.gif)



### JMS消息(JSM Message)属性

#### **消息格式**

· StreamMessage -- Java原始值的数据流

· MapMessage--一套名称-值对

· TextMessage--一个字符串对象

· ObjectMessage--一个序列化的 Java对象

· BytesMessage--一个未解释字节的数据流

#### **消息属性**

· JMSDestination -- 消息发送的目的地，主要是指Queue和Topic

· JMSDeliveryMode -- 持久模式和非持久模式

· JMSExpiration -- 过期时间

· JMSPriority -- 优先级 0-9十个级别，0-4是普通消息5-9是加急消息 默认是4级

· JMSMessageID -- 唯一标识每个消息的标识由MQ产生

· StringProperty -- 消息属性 Property



**发送消息的基本步骤：**

(1)、创建连接使用的工厂类JMS ConnectionFactory

(2)、使用管理对象JMS ConnectionFactory建立连接Connection，并启动

(3)、使用连接Connection 建立会话Session

(4)、使用会话Session和管理对象Destination创建消息生产者MessageSender

(5)、使用消息生产者MessageSender发送消息

 

**消息接收者从JMS接受消息的步骤**

(1)、创建连接使用的工厂类JMS ConnectionFactory

(2)、使用管理对象JMS ConnectionFactory建立连接Connection，并启动

(3)、使用连接Connection 建立会话Session

(4)、使用会话Session和管理对象Destination创建消息接收者MessageReceiver

(5)、使用消息接收者MessageReceiver接受消息，需要用setMessageListener将MessageListener接口绑定到MessageReceiver消息接收者必须实现了MessageListener接口，需要定义onMessage事件方法。

[java操作代码演示](../code/java_jms.md)

## 解决了什么问题

异步处理提高系统性能

解耦

削峰 把请求放到队列里面，然后至于每秒消费多少请求，就看自己的服务器处理能力

## 产生的问题

### 一、可靠性

#### PERSISTENT：持久性

DeliveryMode(配送模式持久化)

Queue

```java
//创建Destination 获取消息目的地 Queue
Destination destination = session.createQueue(queueName);

//创建生产者producer
MessageProducer messageProducer = session.createProducer(destination);

//非持久化：当服务器宕机，消息不存在。
//messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT)
//持久化：当服务器宕机，消息依然存在。
messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT)

/**
持久化消息
这是队列的默认传递模式，此模式保证这些消息只被传送一次和成功使用一次。对于这些消息，可靠性是优先考虑的因素。
可靠性的另一个重要方面是确保持久性消息传送至目标后，消息服务在向消费者传送它们之前不会丢失这些消息。
*/
```



Topic

`先启动定阅消费者再启动定阅生产者`

```java
/**持久的发布主题生产者*/

// 获取消息目的地，消息发送给谁
Destination destination = session.createTopic("test-topic");
// 获取消息生产者
MessageProducer producer = session.createProducer(destination);
//
producer.setDeliveryMode(DeliveryMode.PERSISTENT);
```



**Consumer 持久化 变 Subscriber**

```java
/**持久的定阅主题消费者*/

// 构造ConnectionFactory实例对象
ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
// 从工厂获取连接对象
Connection connection = connectionFactory.createConnection();
connection.setClientID("")

// 获取操作连接，一个发送或接收消息的线程
Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
// 获取消息目的地，消息发送给谁
Topic topic = session.createTopic("test-topic");
// 订阅者，消息接收者
//MessageConsumer consumer = session.createConsumer(destination);
TopicSubscriber topicSubscriber = session.createDurableSubscriber(topic,"remark...");

// 启动
connection.start();
```

![](../img/subscriber.png)



#### Transaction：事务

事务偏生产者/签收偏消费者

`false`

只要执行send，就进入到队列中

关闭事务，那第2个签收参数的设置需要有效

`true`

先执行send再执行commit，消息才被真正提交到队列中

消息需要需要批量提交，需要缓冲处理

```java
// 1.创建ConnectionFactory
ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(url);
// 2.由connectionFactory创建connection
Connection connection = connectionFactory.createConnection();
// 3.启动connection
connection.start();
// 4.创建Session===第一个参数是是否事务管理，第二个参数是应答模式/签收模式
Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);

....
    
// 8.生产者发送消息
producer.send(tms);
// 9.提交事务
session.commit();
```



> 消费者开启事务，不`commit`，会**重复消费**，切不是真正消费

#### Acknowledge：签收/答应

事务偏生产者/签收偏消费者

**非事务**

`Session.AUTO_ACKNOWLEDGE`自动签收(默认)

`Session.CLIENT_ACKNOWLEDGE`手动签收

`Session.DUPS_OK_ACKNOWLEDGE`允许重复消息

**事务**

生产事务开启，只有commit后才能将全部消息变为已消费

在事务中，当一个事务被成功提交，则消息会被自动签收
由于消费者开启了事务,没有提交事务(就算手动签收也没用),服务器认为,消费者没有收到消息



> 点对点模型是基于队列的，生产者发送消息到队列，消费者从队列接收消息，队列的存在使得消息的异步传输成为可能。和我们平时给朋友发送短信类似。<br><br>1：如果在Session关闭时有部分消息被收到但还没有被签收（acknowledge），那当消费者下次连接到相同的队列时，这些消息还会被再次接收<br><br>2：队列可以长久的保存消息直到消费者收到消息。消费者不需要因为担心消息会丢失而时刻和队列保持激活的链接状态，充分体现了异步传输模式的优势

> 非持久订阅只有当客户端处于激活状态，也就是和MQ保持连接状态才能收发到某个主题的消息。<br><br>如果消费者处于离线状态，生产者发送的主题消息将会丢失作废，消费者永远不会收到。<br><br>先订阅注册才能接受到发布，只给订阅者发布消息。
>
> 持久订阅客户端首先向MQ注册一个自己的身份ID识别号，当这个客户端处于离线时，生产者会为这个ID保存所有发送到主题的消息，当客户再次连接到MQ的时候，会根据消费者的ID得到所有当自己处于离线时发送到主题的消息<br><br>当持久订阅状态下，不能恢复或重新派送一个未签收的消息。<br><br>持久订阅才能恢复或重新派送一个未签收的消息。

#### [物理]消息存储和持久化

### 二、可用性

集群

容错性

### 三、延时/定时发送

### 四、数据一致性

#### 分布式事务

环境情况：

两个数据库更新操作的事务，如，下单扣款成功后加积分

思路：

1. 下单扣款操作数据库成功，向MQ中投递消息，投递MQ消息成功，加积分操作消费消息

问题：

1. 下单扣款操作数据库成功，向MQ中投递消息；若操作数据库失败，不会向MQ中投递消息了。

2. 下单扣款操作数据库成功，但是向MQ中投递消息时失败，向外抛出了异常，刚刚执行的更新数据库的操作将被回滚。

3. 下单扣款操作数据库成功，投递MQ消息成功，加积分消费异常，数据未更新，通过扫描**消息表**再次把数据取出进行消费。

   `消息表，在下单扣款操作数据库成功，并且MQ消息投递也成功，将消息内容保存一份到消息表，防止消息被消费者已消费，但消费者业务异常的情况`

### 五、消息重复消费

幂等性，强校验，流水号

### 六、消息丢失

可靠性

异步发送，自写回调函数
消息重发机制

### 七、消息顺序消费

### 八、ActiveMQ的传输协议