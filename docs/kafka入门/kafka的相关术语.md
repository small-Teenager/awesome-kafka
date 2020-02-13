# topic 
发布订阅的对象，可以为每个业务，每个应用，甚至是每类数据创建专属的主题。承载消息的逻辑容器，
# producer
向主题发送消息的客户端应用程序。生产者程序（producer）通常向一个或多个主题（topic）发送消息，而订阅这些主题消息的客户端
应用程序称为消费者（consumer），和生产者类似，消费者也能同时订阅一个或多个topic，我们把生产者和消费者统称为
客户端（clients）
生产者 从主题生产消息的应用程序
# consumer
消费者 从主题订阅消息的应用程序
# record
消息，kafka处理的主要对象
# partition （解决伸缩性）
分区，一个有序不变的消息序列，每一个topic可以有多个partition
# offset
消息位移，表示partition中每条消息的位置信息，是一个单调递增且不变的值。
# consumer offset
消费者位移 表明消费者消费进度 每个消费者都有自己的consumer offset
# consumer group
消费者组 多个消费者实例共同组成一个组，同时消费多个分区以实现高吞吐
# replica （实现高可用）
副本 kafka中的每条消息（record）能被copy到多个地方提供数据冗余，这些地方就是副本，副本分为leader 和follower
各自有不同的角色划分。leader 对外提供服务（与客户端程序进行交互），follower只是被动的追随leader，不对外提供服务
副本是在partition层级下的，即每个partition 可能配置多个replica 提供高可用。

## 副本的工作机制
producer 向leader写消息，consumer从leader读消息，follower只做向leader发送请求，请求leader的最新数据，保持
与leader的数据同步
# rebalance
重平衡 消费者组中某个消费者实例挂掉，其他消费者实例自动重新分配订阅主题分区的过程。rebalance 是kafka 消费端实现
高可用的重要手段

# kafka的消息架构

    1 第一层topic 每个topic 可以配置m个partition，每个partition可以配置n的replica
    2 第二层partition，每个partition的n个 replica只有一个能充当leader，其他n-1个皆为follower，提供数据冗余
    3 第三层消息层 分区中包含着若干条信息，每条消息的位移从0开始，依次递增。                                                                    