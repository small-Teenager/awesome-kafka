## Kafka Consumer 概述
kafka 的Java 消费者API 主要对象是KafkaConsumer
## 什么时候创建TCP 连接
消费者的入口程序时KafkaConsumer。和生产者不同的是，构建KafkaConsumer 实例时是不会创建任何TCP 连接的。
TCP 连接是在调用KafkaConsumer.poll()方法时被创建的.在poll 方法中有3个时机是可以创建TCP 连接的。
### 1 发起FindCoordinator 请求时 确定协调者和获取集群元数据
消费者端有个组间协调者(Coordinator),它存在与broker 端的内存中，负责消费者组的组成员管理和各个消费者的位移提交管理
。当消费者程序首次调用poll 方法时，它需要向kafka 集群发送一个名为FindCoordinator 的请求，希望kafka 集群告诉它
那个broker 是管理它的协调者。
### 2 连接协调者时 连接协调者 令其执行组成员管理操作
broker 处理完上一步发送的FindCoordinator 请求之后，会返回对应的结果，显示的告诉消费者那个broker 时真正的协调者
，因此，消费者在知晓了真正的协调者之后，会创建连向broker 的TCP 连接，只有成功连入协调者，协调者才能开启正常的
组协调操作，比如加入组，等待组分配方案 心跳请求处理 位移获取 位移提交等。
### 3 消费消息时 执行实际的消息获取
消费者会为每个要消费的分区创建与领导者副本所在broker 连接的TCP. 假如消费者要消费5个分区的数据，这5个分区各自
的领导者副本分布在4台broker 上，那么该消费者在消费时会创建与这4台broker 的TCP 连接。
## 什么时候关闭TCP 连接
Consumer 端关闭TCP 连接方式和Producer 类似，一种是用户主动关闭TCP 连接，另一种是Kafka 自动关闭。
### 用户主动关闭TCP 连接
用户调用API KafkaConsumer.close()关闭，或者执行kill 命令。
### Kafka 自动关闭
kafka 自动关闭，这与consumer 端参数connection.max.idle.ms值有关。默认情况下该值是9分钟，即如果在9分钟内
没有任何请求流过某个TCP 连接，那么Kafka 会主动的帮你把TCP 连接关闭。
