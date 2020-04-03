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
        ActiveMQMessageProducer activeMQMessageProducer = session.createProducer(destination);
        for (int i = 0; i < 10; i++) {
            TextMessage tms = session.createTextMessage("textMessage:" + i);
            tms.setJMSMessageID(UUID.randomUUID().toString());
            String msgID = tms.getJMSMessageID();
            activeMQMessageProducer.send(tms,new AsyncCallback(){
                @Override
                public void onSuccess(){
                    //成功操作
                }
                @Override
                public void onException(JMSException exception){
                    //失败操作
                }
            });
        }

        // 9.提交事务
        session.commit();

        // 10.关闭connection
        session.close();
        activeMQMessageProducer.close();
        connection.close();
    }

}
```


