点对点（Point-to-Point）

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
    private static final String url = "tcp://localhost:61616/";
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
        MessageProducer producer = session.createProducer(destination);
        // 持久模式和非持久模式
        producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
        for (int i = 0; i < 10; i++) {
            // 7.创建Message
            //	TxtMessage=>普通字符串消息，包含一个String
            //	MapMessage=>一个Map类型的消息，key为Strng类型，而值为Java基本类型
            //	BytesMessage=>二进制数组消息，包含一个byte[]
            //	StreamMessage=>Java数据流消息，用标准流操作来顺序填充和读取
            //	ObjectMessage=>对象消息，包含一个可序列化的Java对象
            TextMessage tms = session.createTextMessage("textMessage:" + i);
            
            //tms.setJMSDeliveryMode  持久模式和非持久模式
            //tms.setJMSExpiration    过期时间
            //tms.setJMSPriority      优先级 0-9十个级别，0-4是普通消息5-9是加急消息 默认是4级 
            //tms.setJMSMessageID     唯一标识每个消息的标识由MQ产生

            // 设置附加属性
            tms.setStringProperty("str", "stringProperties" + i);
            

            /*Student student = new Student();
            student.setId((long) i);
            student.setName("学生" + i);
            student.setBirthday(new Date());
            TextMessage message = session.createTextMessage(CommonUtils.serialize(student));
            // 发送消息到目的地方
            producer.send(message);
            System.out.println(String.format("发送消息:%d-%s-%s", student.getId(), student.getName(),
                    DateFormatUtils.format(student.getBirthday(), "yyyy-MM-dd")));*/
            // 8.生产者发送消息
            producer.send(tms);
        }

        // 9.提交事务
        session.commit();

        // 10.关闭connection
        session.close();
        connection.close();
    }

}
```



```java
package cn.qlq.activemq;

import java.util.Enumeration;

import javax.jms.*;

import org.apache.activemq.ActiveMQConnectionFactory;

/**
 * 消费消息
 * 
 * @author fyy
 * @time 
 */
public class MsgConsumer {

    // 默认端口61616
    private static final String url = "tcp://localhost:61616/";
    private static final String queueName = "myQueue";

    public static void main(String[] args) throws JMSException {
        // 1创建ConnectionFactory
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(url);
        // 2.由connectionFactory创建connection
        Connection connection = connectionFactory.createConnection();
        Enumeration jmsxPropertyNames = connection.getMetaData().getJMSXPropertyNames();
        while (jmsxPropertyNames.hasMoreElements()) {
            String nextElement = (String) jmsxPropertyNames.nextElement();
            System.out.println("JMSX name ===" + nextElement);
        }
        // 3.启动connection
        connection.start();
        // 4.创建Session===第一个参数是是否事务管理，第二个参数是应答模式
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        // 5.创建Destination(Queue继承Queue)
        Destination destination = session.createQueue(queueName);
        // 6.创建消费者consumer
        MessageConsumer consumer = session.createConsumer(destination);

        /*int i = 0;
        while (i < 10) {
            TextMessage textMessage = (TextMessage) consumer.receive();
            System.out.println("接收消息:" + textMessage.getText() + ";属性" + textMessage.getStringProperty("str"));
            i++;

        }*/
        
        consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                try {
                    if (null != message) {
                        TextMessage textMessage = (TextMessage) message;
                        System.out.println("接收消息:" + textMessage.getText() + ";属性" + textMessage.getStringProperty("str"));
                        
                        /*Student student = CommonUtils.deserialize(((TextMessage) message).getText());
                        System.out.println(
                                String.format("ActiveMQConsumer1-接受消息:%d-%s-%s", student.getId(), student.getName(),
                                        DateFormatUtils.format(student.getBirthday(), "yyyy-MM-dd")));*/
                    }
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });

        // 提交事务，进行确认收到消息
        session.commit();

        session.close();
        connection.close();
    }
}
```

