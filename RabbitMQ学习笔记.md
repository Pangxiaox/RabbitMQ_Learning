# RabbitMQ学习笔记

参考自

- https://www.cnblogs.com/ysocean/p/9251884.html
- https://blog.csdn.net/hellozpc/article/details/81436980



### 1. MQ介绍

消息队列（Message Queue、MQ），本质是个队列，队列中存放的是内容message。主要用途是进行不同进程或者不同线程之间的通信。

不同进程之间传递消息时，两个进程之间耦合程度过高，改动一个进程，引发必须修改另一个进程，为了隔离这两个进程，在两进程间抽离出一层（一个模块），所有两进程之间传递的消息，都必须通过消息队列来传递，单独修改某一个进程，不会影响另一个。

此外，为了实现标准化，将消息的格式规范化了，并且某一个进程接受的消息太多，一下子无法处理完，并且也有先后顺序，必须对收到的消息进行排队。

MQ框架很多，比较流行的有RabbitMQ、ActiveMQ、ZeroMQ、RocketMQ、Kafka等。



### 2. RabbitMQ介绍

MQ是应用程序和应用程序之间的通信方法。RabbitMQ是一个开源的，在AMQP（提供统一消息服务的应用层标准高级消息队列协议，为面向消息的中间件设计）基础上完整的可复用的企业消息系统。开发语言是Erlang，一种面向并发的编程语言。

安装配置完成后，在浏览器输入地址查看：http://127.0.0.1:15672/

账号和密码均为guest



### 3. 添加用户

##### 3.1 添加admin用户

![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ0.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ1.PNG)  



##### 3.2 用户角色

①超级管理员（administrator）

可登录管理控制台，可查看所有信息，并且可对用户，策略（policy）进行操作

②监控者（monitoring）

可登录管理控制台，同时可查看rabbitmq节点相关信息（进程数，内存使用情况，磁盘使用情况等）

③策略制定者（policymaker）

可登录管理控制台，同时可以对policy进行管理，但无法查看节点相关信息

④普通管理者（management）

仅可登录管理控制台，无法看到节点信息，也无法对策略进行管理

⑤其他

无法登录管理控制台，通常就是普通的生产者和消费者



##### 3.3 创建Virtual Hosts  

![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ2.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ4.PNG)  



选中admin用户，设置权限：   


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ3.PNG)  

验证已经成功设置权限：   


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ5.PNG)  



##### 3.4 管理界面功能  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/MQ6.PNG)  



### ⭐4. RabbitMQ的五种队列

pom文件：导入rabbitmq依赖包

```xml
    <dependency>
      <groupId>com.rabbitmq</groupId>
      <artifactId>amqp-client</artifactId>
      <version>3.4.1</version>
    </dependency>
```

工具类：ConnectUtil

```java
package com.ys.utils;

import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class ConnectionUtil {

public static Connection getConnection(String host,int port,String vHost,String userName,String passWord) throws Exception{
        //1、定义连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2、设置服务器地址
        factory.setHost(host);
        //3、设置端口
        factory.setPort(port);
        //4、设置虚拟主机、用户名、密码
        factory.setVirtualHost(vHost);
        factory.setUsername(userName);
        factory.setPassword(passWord);
        //5、通过连接工厂获取连接
        Connection connection = factory.newConnection();
        return connection;
    }
}
```

#### 4.1 简单队列

一个生产者对应一个消费者  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ1.PNG)  

生产者将消息发送到“hello”队列，消费者从该队列接收消息

- 生产者：Producer

```java
package com.ys.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.ys.utils.ConnectionUtil;

public class Producer {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明信道
        Channel channel = connection.createChannel();
        //3、声明(创建)队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、定义消息内容
        String message = "hello rabbitmq ";
        //5、发布消息
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
        System.out.println("[x] Sent'"+message+"'");
        //6、关闭通道
        channel.close();
        //7、关闭连接
        connection.close();
    }
}
```

- 消费者：Consumer

```java
package com.ys.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //5、监听队列
        /*
            true:表示自动确认，只要消息从队列中获取，无论消费者获取到消息后是否成功消费，都会认为消息已经成功消费
            false:表示手动确认，消费者获取消息后，服务器会将该消息标记为不可用状态，等待消费者的反馈，如果消费者一直没有反馈，那么该消息将一直处于不可用状态，并且服务器会认为该消费者已经挂掉，不会再给其发送消息，直到该消费者反馈。
         */

        channel.basicConsume(QUEUE_NAME,true,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" [x] Received '" + message + "'");
        }
    }
}
```

RabbitMQ效果：  

![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ1_1.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ1_2.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ1_3.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ1_4.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ1_5.PNG)  



#### 4.2 work模式

竞争消费者模式，一个生产者对应多个消费者，但是只能有一个消费者获得消息  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ2_2.PNG)  

- 生产者

```java
package com.ys.workqueue;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.ys.utils.ConnectionUtil;

public class Producer {
    private final static String QUEUE_NAME = "work_queue";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明信道
        Channel channel = connection.createChannel();
        //3、声明(创建)队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、定义消息内容(发布多条消息)
        for(int i = 0 ; i < 10 ; i++){
            String message = "hello rabbitmq "+i;
            //5、发布消息
            channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
            System.out.println("[x] Sent'"+message+"'");
            //模拟发送消息延时，便于演示多个消费者竞争接受消息
            Thread.sleep(i*10);
        }
        //6、关闭通道
        channel.close();
        //7、关闭连接
        connection.close();
    }
}
```

- 消费者1：每接收一条消息后休眠10ms

```java
package com.ys.workqueue;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer1 {

    private final static String QUEUE_NAME = "work_queue";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //同一时刻服务器只会发送一条消息给消费者
        //channel.basicQos(1);

        //4、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //5、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" [x] Received '" + message + "'");
            //消费者1接收一条消息后休眠10毫秒
            Thread.sleep(10);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }
}
```

- 消费者2：每接收一条消息后休眠1000ms

```java
package com.ys.workqueue;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer2 {

    private final static String QUEUE_NAME = "work_queue";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //同一时刻服务器只会发送一条消息给消费者
        //channel.basicQos(1);

        //4、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //5、监听队列，手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" [x] Received '" + message + "'");
            //消费者2接收一条消息后休眠1000毫秒
            Thread.sleep(1000);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }
}
```

- 测试结果  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ2_1.PNG)  

**消费者1：打印偶数条消息**  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ2_3.PNG)  

**消费者2：打印奇数条消息**  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ2_4.PNG)  

🔺同一个消息只能被一个消费者获取，两个消费者获取消息的条数一样。

⭐两个消费者获取消息的效率不一样，但获取消息条数一样，未构成竞争关系，下面展示怎样使得消费者1获取更多消息：

```java
channel.basicQos(1)
```

增加上面代码，表示同一时刻服务器只会发送一条消息给消费者

此时，消费者1和消费者2获取消息结果如图：  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ2_5.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ2_6.PNG)  

**⭐应用场景：效率高的消费者消费消息多，用来负载均衡**



#### 4.3 发布——订阅模式

一个消费者将消息首先发送到交换器，交换器绑定到多个队列，然后被监听该队列的消费者所接收并消费  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ3.PNG)  

X表示交换器，交换器主要有四种类型：direct、fanout、topic、headers，此处是fanout  

![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ3_4.PNG)  



- 生产者

```java
package com.ys.ps;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.ys.utils.ConnectionUtil;

public class Producer {
    private final static String EXCHANGE_NAME = "fanout_exchange";

    public static void main(String[] args) throws Exception {
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1", 5672, "testhost", "admin", "<YOUR_OWN_PWD>");
        //2、声明信道
        Channel channel = connection.createChannel();
        //3、声明交换器
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        //4、创建消息
        String message = "hello rabbitmq";
        //5、发布消息
        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
        System.out.println("[x] Sent'" + message + "'");
        //6、关闭通道
        channel.close();
        //7、关闭连接
        connection.close();
    }
}
```

- 消费者1

```java
package com.ys.ps;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer1 {

    private final static String QUEUE_NAME = "fanout_queue_1";

    private final static String EXCHANGE_NAME = "fanout_exchange";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、绑定队列到交换机
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
        //5、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //6、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" 消费者1：" + message + "'");
            //消费者1接收一条消息后休眠10毫秒
            Thread.sleep(10);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }

}
```

- 消费者2

```java
package com.ys.ps;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer2 {

    private final static String QUEUE_NAME = "fanout_queue_2";

    private final static String EXCHANGE_NAME = "fanout_exchange";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、绑定队列到交换机
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
        //5、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //6、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" 消费者2：" + message + "'");
            //消费者2接收一条消息后休眠10毫秒
            Thread.sleep(1000);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }
```

- 测试结果  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ3_1.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ3_2.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ3_3.PNG)  

🔺消费者1和消费者2都消费了该消息，因为消费者1和消费者2都监听了被同一个交换器绑定的队列。如果消息发送到没有队列绑定的交换器，消息将丢失，因为 **交换器没有存储消息能力，消息只能存储在队列中**

⭐ **应用场景：一个商城系统需要在管理员上传商品新的图片时，前台系统必须更新图片，日志系统必须记录相应的日志，那么就可以将两个队列绑定到图片上传交换器上，一个用于前台系统更新图片，另一个用于日志系统记录日志**



#### 4.4 路由模式  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ4.PNG)

生产者将消息发送到direct交换器，在绑定队列和交换器时有一个路由key，生产者发送的消息会指定一个路由key，那么消息只会发送到相应key相同的队列，接着监听该队列的消费者消费消息。 **让消费者有选择性地接收消息**

- 生产者

```java
package com.ys.routing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.ys.utils.ConnectionUtil;

public class Producer {
    private final static String EXCHANGE_NAME = "direct_exchange";

    public static void main(String[] args) throws Exception {
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1", 5672, "testhost", "admin", "<YOUR_OWN_PWD>");
        //2、声明信道
        Channel channel = connection.createChannel();
        //3、声明交换器，类型为direct
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        //4、创建消息
        String message = "hello rabbitmq";
        //5、发布消息
        channel.basicPublish(EXCHANGE_NAME, "update", null, message.getBytes());
        System.out.println("生产者发送" + message + "'");
        //6、关闭通道
        channel.close();
        //7、关闭连接
        connection.close();
    }
}
```

- 消费者1

```java
package com.ys.routing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer1 {

    private final static String QUEUE_NAME = "direct_queue_1";

    private final static String EXCHANGE_NAME = "direct_exchange";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、绑定队列到交换机，指定路由key为update
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"update");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"delete");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"add");
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
        //5、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //6、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" 消费者1：" + message + "'");
            //消费者1接收一条消息后休眠10毫秒
            Thread.sleep(10);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }

}
```

- 消费者2

```java
package com.ys.routing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer2 {

    private final static String QUEUE_NAME = "direct_queue_2";

    private final static String EXCHANGE_NAME = "direct_exchange";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、绑定队列到交换机，指定路由key为select
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"select");
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
        //5、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //6、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" 消费者1：" + message + "'");
            //消费者2接收一条消息后休眠10毫秒
            Thread.sleep(1000);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }
```

- 测试结果  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ4_1.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ4_2.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ4_3.PNG)  

⭐ **应用场景：商城系统的后台管理系统对于商品进行修改、删除、新增操作都需要更新前台系统的界面展示，而查询操作不需要，那么这两个队列分开接收消息比较好**



#### 4.5 主题模式

路由模式是根据路由key进行完整的匹配（完全相等才发送消息），这里通配符模式就是模糊匹配

符号#表示匹配一个或多个词，符号 * 表示匹配一个词     


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ5.PNG)  

- 生产者

```java
package com.ys.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.ys.utils.ConnectionUtil;

public class Producer {
    private final static String EXCHANGE_NAME = "topic_exchange";

    public static void main(String[] args) throws Exception {
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1", 5672, "testhost", "admin", "<YOUR_OWN_PWD>");
        //2、声明信道
        Channel channel = connection.createChannel();
        //3、声明交换器，类型为direct
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        //4、创建消息
        String message = "hello rabbitmq111";
        //5、发布消息
        channel.basicPublish(EXCHANGE_NAME, "update.Name", null, message.getBytes());
        System.out.println("生产者发送" + message + "'");
        //6、关闭通道
        channel.close();
        //7、关闭连接
        connection.close();
    }
}
```

- 消费者1

```java
package com.ys.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer1 {

    private final static String QUEUE_NAME = "topic_queue_1";

    private final static String EXCHANGE_NAME = "topic_exchange";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、绑定队列到交换机，指定路由key为update.#
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"update.#");
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
        //5、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //6、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //6、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" 消费者1：" + message + "'");
            //消费者1接收一条消息后休眠10毫秒
            Thread.sleep(10);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }
```

- 消费者2

```java
package com.ys.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.QueueingConsumer;
import com.ys.utils.ConnectionUtil;

public class Consumer2 {

    private final static String QUEUE_NAME = "topic_queue_2";

    private final static String EXCHANGE_NAME = "topic_exchange";

    public static void main(String[] args) throws Exception{
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection("127.0.0.1",5672,"testhost","admin","<YOUR_OWN_PWD>");
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、绑定队列到交换机，指定路由key为select.#
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"select.#");
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
        //5、定义队列的消费者
        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
        //6、监听队列,手动返回完成状态
        channel.basicConsume(QUEUE_NAME,false,queueingConsumer);
        //7、获取消息
        while (true){
            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" 消费者1：" + message + "'");
            //消费者2接收一条消息后休眠10毫秒
            Thread.sleep(1000);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        }
    }

}
```

- 测试结果  

![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ5_1.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ5_2.PNG)  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/RabbitMQ5_3.PNG)   

🔺生产者发送消息绑定的路由key未update.Name;消费者监听的队列和交换器绑定路由key为update.#；消费者2监听的队列和交换器绑定路由key为select.#。因而消费者1会接收到消息而消费者2接收不到



### 5. 四种交换器

有四种交换器：direct、fanout、topic和headers。

前面三种分别对应路由模式、发布订阅模式和通配符模式，headers交换器允许匹配AMQP消息的header而非路由键，除此之外，headers交换器和direct交换器完全一致，但是性能差很多，因此基本上不会用到headers交换器。

①direct

如果路由键完全匹配，消息才会被投放到相应队列  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/direct.PNG)  

②fanout

当发送一条消息到fanout交换器上，它会把消息投放到所有附加在此交换器上的队列  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/fanout.PNG)  

③topic

设置模糊的绑定方式，“*”操作符将“."视为分隔符，匹配单个字符；"#"操作符没有分块的概念，它将任意“.”均视为关键字的匹配部分，能够匹配多个字符


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/topic.PNG)  



### 6. 有交换器参与的队列中生产者和消费者的小结  


![Image text](https://github.com/Pangxiaox/RabbitMQ_Learning/blob/master/MQ-pic/Producer_Consumer.PNG)  





