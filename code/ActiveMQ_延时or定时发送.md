要在activemq.xml中配置schedulerSupport属性为true

```xml
<!--
    The <broker> element is used to configure the ActiveMQ broker.
 -->
    <broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" schedulerSupport="true">
```





```java
package cn.qlq.activemq;

import javax.jms.*;

import org.apache.activemq.ActiveMQConnectionFactory;

/**
 * 生产消息
 * 
 * @author fyy
 * @time 
 */
public class MsgProducer {

    // 默认端口61616
    private static final String url = "tcp://localhost:61616?jms.useAsyncSend=true";
    private static final String queueName = "myQueue";
    private static Session session = null;

    public static void main(String[] args) throws JMSException {
        // 1.创建ConnectionFactory
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(url);
        // 2.由connectionFactory创建connection
        Connection connection = connectionFactory.createConnection();
        // 3.启动connection
        connection.start();
        // 4.创建Session===第一个参数是是否事务管理，第二个参数是应答模式/签收模式
        session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        // 5.创建Destination 获取消息目的地，消息发送给谁,主要是指Queue和Topic
        Destination destination = session.createQueue(queueName);
        // 6.创建生产者producer
        MessageProducer messageProducer = session.createProducer(destination);
        long delay = 3 * 1000;
        long period = 4 * 1000;
        long repeat = 5;
        for (int i = 0; i < 10; i++) {
            TextMessage tms = session.createTextMessage("textMessage:" + i);
            tms.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY.delay);
            tms.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD.period);
            tms.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT.repeat);
            messageProducer.send(tms);
        }

        // 9.提交事务
        session.commit();

        // 10.关闭connection
        session.close();
        messageProducer.close();
        connection.close();
    }

}
```

| Property name        | type   | description                                                  |
| -------------------- | ------ | ------------------------------------------------------------ |
| AMQ_SCHEDULED_DELAY  | long   | 消息在计划由代理传递之前等待的时间(以毫秒为单位)             |
| AMQ_SCHEDULED_PERIOD | long   | 在开始时间之后等待的时间(以毫秒为单位)，在再次调度消息之前等待 |
| AMQ_SCHEDULED_REPEAT | int    | 重复调度传递消息的次数                                       |
| AMQ_SCHEDULED_CRON   | String | 使用Cron条目来设置时间表                                     |

