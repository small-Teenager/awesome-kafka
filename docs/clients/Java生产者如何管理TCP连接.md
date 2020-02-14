## 为什么采用TCP
apache kafka 的所有通信都是基于TCP 的，而是不是HTTP 和其他协议，无论是producer，broker，consumer都是如此
。
## Kafka producer 概述
kafka 的Java 生产者API 主要对象是KafkaProducer。
通常开发一个producer的步骤有4步
    
    1. 构造生产者对象所需的参数对象
    2. 利用第一步的参数对象，创建KafkaProducer 的对象实例
    3. 调用KafkaProducer 的send方法发送消息
    4. 调用KafkaProducer 的close方法关闭producer 并释放各种系统资源
```
     Properties props = new Properties();
     props.put("bootstrap.servers", "localhost:9092");
     props.put("acks", "all");
     props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
     props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    
     Producer<String, String> producer = new KafkaProducer<>(props);
     for (int i = 0; i < 100; i++)
         producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)),callback);
    
     producer.close();
```
当我们开发一个producer应用程序时，生产者会向kafka集群中指定的topic 发送消息，这必然会涉及到与kafka broker
创建TCP 连接。那么，kafka 的producer 客户端是如何管理这些TCP 连接的呢？ 
## 什么时候创建TCP 连接
就上面的代码 创建TCP 连接的地方可能有两处   Producer<String, String> producer = new KafkaProducer<>(props);
和producer.send(msg,callback);

首先，在创建KafkaProducer 实例时，生产者应用会在后台创建并启动一个Sender 线程，该Sender线程开始运行时会创建
与Broker 的连接。
## 什么时候关闭TCP 连接
Producer 端关闭TCP 连接的方式有两种，一种时用户主动关闭TCP 连接，另一种是Kafka 自动关闭。
### 用户主动关闭TCP 连接
用户调用API producer.close()关闭
### Kafka 自动关闭
kafka 自动关闭，这与producer 端参数connection.max.idle.ms值有关。默认情况下该值是9分钟，即如果在9分钟内
没有任何请求流过某个TCP 连接，那么Kafka 会主动的帮你把TCP 连接关闭。用户可以在producer 端设置
connection.max.idle.ms=-1 禁用这种机制。一旦该值为-1，TCP 连接将成为永久长连接。
kafka 自动关闭TCP 连接，是在Broker 端被关闭的。但是这个TCP 连接的发起者是客户端，
因此在TCP 看来，这属于被动关闭场景，即passive close。被动关闭的后果是产生大量的CLOSE_WAIT连接，因此
Producer 端或Client 端没有机会显示的观测到连接已被断开。

## 可以参阅 kafka-example 中代码
    