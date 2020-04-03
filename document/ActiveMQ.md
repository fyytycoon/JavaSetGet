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

为了避免意外宕机以后丢失信息，需要做到重启后可以恢复消息队列，消息系统一半都会采用持久化机制。<br>ActiveMQ的消息持久化机制有JDBC，AMQ，KahaDB和LevelDB，无论使用哪种持久化方式，消息的存储逻辑都是一致的。<br> <br>就是在发送者将消息发送出去后，消息中心首先将消息存储到本地数据文件、内存数据库或者远程数据库等。再试图将消息发给接收者，成功则将消息从存储中删除，失败则继续尝试尝试发送。<br><br>消息中心启动以后，要先检查指定的存储位置是否有未成功发送的消息，如果有，则会先把存储位置中的消息发出去。

**AMQ Mesage Store(了解）**

AMQ是一种文件存储形式，它具有写入速度快和容易恢复的特点。消息存储再一个个文件中文件的默认大小为32M，当一个文件中的消息已经全部被消费，那么这个文件将被标识为可删除，在下一个清除阶段，这个文件被删除。AMQ适用于ActiveMQ5.3之前的版本

**KahaDB消息存储(默认)**ActiveMQ5.4

KahaDB是目前默认的存储方式，可用于任何场景，提高了性能和恢复能力。<br>消息存储使用一个**事务日志**和仅仅用一个**索引文件**来存储它所有的地址。<br>KahaDB是一个专门针对消息持久化的解决方案，它对典型的消息使用模型进行了优化。<br>数据被追加到data logs中。当不再需要log文件中的数据的时候，log文件会被丢弃。

![](../img/KahaDB.png)



KahaDB在消息保存的目录中有4类文件和一个lock，跟ActiveMQ的其他几种文件存储引擎相比，这就非常简洁了。

1，db-<number>.log

KahaDB存储消息到预定大小的数据纪录文件中，文件名为db-<number>.log。当数据文件已满时，一个新的文件会随之创建，number数值也会随之递增，它随着消息数量的增多，如没32M一个文件，文件名按照数字进行编号，如db-1.log，db-2.log······。当不再有引用到数据文件中的任何消息时，文件会被删除或者归档。

2，db.data

该文件包含了持久化的BTree索引，索引了消息数据记录中的消息，是消息的索引文件，本质是BTree，指向了db-<number>.log 里的消息

3，db.free

当前db.data 文件里哪些页面是空闲的，文件具体内容是所有空页面的ID

4，db.redo

用来进行消息恢复，如果KahaDB消息存储在强制退出后启动，用于恢复BTree索引

5， lock 

文件锁，表示当前获得KahaDB读写权限的broker

**JDBC存储消息**

将数据持久化到数据库中。建一个名为activemq的数据库，有三张表ACTIVEMQ_MSGS、ACTIVEMQ_ACKS、ACTIVEMQ_LOCK，如果是queue，在没有消费者消费的情况下会将消息保存到ACTIVEMQ_MSGS表中，只要有任意一个消费者消费了，就会删除消费过的消息。如果是topic，一般是先启动消费订阅者然后再生产的情况下会将持久订阅者永久保存到ACTIVEMQ_ACKS，而消息则永久保存在ACTIVEMQ_MSGS，在acks表中的订阅者有一个last_ack_id对应了activemq_msgs中的id字段，这样就知道订阅者最后收到的消息是哪一条。



[JDBC存储消息配置](../code/ActiveMQ_JDBC持久化.md)

**JDBC Persistence without Journaling**

这种方式克服了JDBC Store的不足，JDBC每次消息过来，都需要去写库读库。<br>ActiveMQ Journal，使用高速缓存写入技术，大大提高了性能。<br><br>当消费者的速度能够及时跟上生产者消息的生产速度时，journal文件能够大大减少需要写入到DB中的消息。<br>举个例子：<br>生产者生产了1000条消息，这1000条消息会保存到journal文件，如果消费者的消费速度很快的情况下，在journal文件还没有同步到DB之前，消费者已经消费了90%的以上消息，那么这个时候只需要同步剩余的10%的消息到DB。如果消费者的速度很慢，这个时候journal文件可以使消息以批量方式写到DB。





### 二、可用性

**集群**

![](../img/zk+activemq集群.png)

基于zookeeper和LevelDB搭建ActiveMQ集群。集群仅提供主备方式的高可用集群功能，避免单点故障。

**原理：**

使用Zookeeper集群注册所有的ActiveMQ Broker但只有其中一个Broker可以提供服务，它将被视为Master,其他的Broker处于待机状态被视为Slave。<br>如果Master因故障而不能提供服务，Zookeeper会从Slave中选举出一个Broker充当Master。<br>Slave连接Master并同步他们的存储状态，Slave不接受客户端连接。所有的存储操作都将被复制到连接至Maste的Slaves。<br>如果Master宕机得到了最新更新的Slave会变成Master。故障节点在恢复后会重新加入到集群中并连接Master进入Slave模式。<br><br>所有需要同步的消息操作都将等待存储状态被复制到其他法定节点的操作完成才能完成。<br>所以，如给你配置了replicas=3，name法定大小是（3/2）+1 = 2。Master将会存储更新然后等待（2-1）=1个Slave存储和更新完成，才汇报success

容错性

### 三、延时/定时发送

要在activemq.xml中`<broker>`配置schedulerSupport属性为true

Message设置属性，ScheduledMessage类

`message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY.delay);`

| Property name        | type   | description                                                  |
| -------------------- | ------ | ------------------------------------------------------------ |
| AMQ_SCHEDULED_DELAY  | long   | 消息在计划由代理传递之前等待的时间(以毫秒为单位)             |
| AMQ_SCHEDULED_PERIOD | long   | 在开始时间之后等待的时间(以毫秒为单位)，在再次调度消息之前等待 |
| AMQ_SCHEDULED_REPEAT | int    | 重复调度传递消息的次数                                       |
| AMQ_SCHEDULED_CRON   | String | 使用Cron条目来设置时间表                                     |

[ActiveMQ_延时or定时发送](../code/ActiveMQ_延时or定时发送.md)



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

#### 异步发送，自写回调函数

ActiveMQ默认使用异步发送的模式，除非**明确指定使用同步发送的方式**或者在**未使用事务的前提下发送持久化的消息**，这两种情况都是同步发送的。<br>如果你没有使用事务且发送的是持久化的消息，每一次发送都是同步发送的且会阻塞producer知道broker返回一个确认，表示消息已经被安全的持久化到磁盘。确认机制提供了消息安全的保障，但同时会阻塞客户端带来了很大的延时。<br>

异步发送可能丢失消息，官网配置下：

**Configuring Async Send using a Connection URI**

You can use the Connection Configuration URsends as follows

```java
cf = new ActiveMQConnectionFactory("tcp://locahost:61616?jms.useAsyncSend=true");
```

**Configuring Async Send at the ConnectionFactory Level**

You can enable this feature on the ActiveMQConnectionFactory object using the property.

```java
((ActiveMQConnectionFactory)connectionFactory).setUseAsyncSend(true);
```

**Configuring Async Send at the Connection Level**

Configuring the dispatchAsync setting at this level overrides the settings at the connection factory level.

You can enable this feature on the ActiveMQConnection object using the property.

```java
((ActiveMQConnection)connection).setUseAsyncSend(true);
```



异步发送丢失消息的场景是：生产者设置userAsyncSend=true，使用producer.send(msg)持续发送消息。<br>如果消息不阻塞，生产者会认为所有send的消息均被成功发送至MQ。<br>如果MQ突然宕机，此时生产者端内存中尚未被发送至MQ的消息都会丢失。<br><br>所以，正确的异步发送方法是**需要接收回调的**。<br><br>同步发送和异步发送的区别就在此，<br>同步发送等send不阻塞了就表示一定发送成功了，<br>异步发送需要客户端回执并由客户端再判断一次是否发送成功<br>



[ActiveMQ_异步发送消息并回调代码演示](../code/ActiveMQ_异步发送消息并回调.md)



#### 消息重发机制

### 七、消息顺序消费

### 八、ActiveMQ的传输协议