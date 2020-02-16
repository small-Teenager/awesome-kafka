## kafka 面试题
    1.Kafka中的ISR(InSyncRepli)、OSR(OutSyncRepli)、AR(AllRepli)代表什么？
    2.Kafka中的HW、LEO等分别代表什么？
    3.Kafka中是怎么体现消息顺序性的？
    4.Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？
    5.Kafka生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？
    6.“消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？
    7.消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1？
    8.有哪些情形会造成重复消费？
    9.那些情景会造成消息漏消费？
    10.当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？
        1）会在zookeeper中的/brokers/topics节点下创建一个新的topic节点，如：/brokers/topics/first
        2）触发Controller的监听程序
        3）kafka Controller 负责topic的创建工作，并更新metadata cache
    11.topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？
    12.topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？
    13.Kafka有内部的topic吗？如果有是什么？有什么所用？
    14.Kafka分区分配的概念？
    15.简述Kafka的日志目录结构？
    16.如果我指定了一个offset，Kafka Controller怎么查找到对应的消息？
    17.聊一聊Kafka Controller的作用？
    18.Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？
    19.失效副本是指什么？有那些应对措施？
    20.Kafka的哪些设计让它有如此高的性能？