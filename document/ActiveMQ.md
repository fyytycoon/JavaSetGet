## JMS

什么是Java消息服务

Java消息服务指的是两个应用程序之间进行异步通信的API，它为标准协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持Java应用程序开发。在JavaEE中，当两个应用程序使用JMS进行通信时，它们之间不是直接相连的，而是通过一个共同的消息收发服务组件关联起来以达到解耦/异步削峰的效果。

### JMS的组成结构

JMS提供者(JMS Provider)、JMS生产者(JMS Producer)、JMS消费者(JMS Consumer)、JMS消息(JSM Message)

[JMS百度百科](https://baike.baidu.com/item/JMS/2836691?fr=aladdin)

[java操作代码](../code/java_jms.md)

### 对象模型

JMS对象模型包含如下几个要素： 

1）连接工厂。连接工厂（ConnectionFactory）是由管理员创建，并绑定到[JNDI](https://baike.baidu.com/item/JNDI)树中。客户端使用JNDI查找连接工厂，然后利用连接工厂创建一个JMS连接。

2）JMS连接。JMS连接（Connection）表示JMS客户端和服务器端之间的一个活动的连接，是由客户端通过调用连接工厂的方法建立的。

3）JMS会话。JMS会话（Session）表示JMS客户与JMS服务器之间的会话状态。JMS会话建立在JMS连接上，表示客户与服务器之间的一个会话线程。

4）JMS目的。JMS目的（Destination），又称为消息队列，是实际的消息源。

5）JMS生产者和消费者。生产者（Message Producer）和消费者（Message Consumer）对象由Session对象创建，用于发送和接收消息。

6）JMS消息通常有两种类型：

① 点对点（Point-to-Point）。在点对点的消息系统中，消息分发给一个单独的使用者。点对点消息往往与队列（javax.jms.Queue）相关联。

② 发布/订阅（Publish/Subscribe）。发布/订阅消息系统支持一个事件驱动模型，消息生产者和消费者都参与消息的传递。生产者发布事件，而使用者订阅感兴趣的事件，并使用事件。该类型消息一般与特定的主题（javax.jms.Topic）关联。



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



## 解决了什么问题

异步处理提高系统性能

解耦

削峰 把请求放到队列里面，然后至于每秒消费多少请求，就看自己的服务器处理能力

## 产生的问题

### 一、可靠性

#### PERSISTENT：持久性

#### Transaction：事务

#### Acknowledge：签收/答应

#### [物理]消息存储和持久化

### 二、可用性

集群

容错性

### 三、延时/定时发送

### 四、数据一致性

#### 分布式事务

### 五、消息重复消费

### 六、消息丢失

### 七、消息顺序消费