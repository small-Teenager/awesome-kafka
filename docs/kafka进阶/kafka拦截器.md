## kafka Interceptor
kafka 拦截器由生产者拦截器和消费者拦截器，分别对应ProducerInterceptor和ConsumerInterceptor。
## 拦截器原理
对于producer来说，interceptor使得用户在消息发送前以及producer回调逻辑前有机会对消息做一些定制化需求，比如修改消息等。
对consumer来说，一个插件接口，允许你截取消费者收到的记录。主要用于第三方组件连接到消费者应用程序以进行自定义监视、日志记录等。
## ProducerInterceptor
```
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.apache.kafka.clients.producer;

import org.apache.kafka.common.Configurable;

/**
 * A plugin interface that allows you to intercept (and possibly mutate) the records received by the producer before
 * they are published to the Kafka cluster.
 * <p>
 * This class will get producer config properties via <code>configure()</code> method, including clientId assigned
 * by KafkaProducer if not specified in the producer config. The interceptor implementation needs to be aware that it will be
 * sharing producer config namespace with other interceptors and serializers, and ensure that there are no conflicts.
 * <p>
 * Exceptions thrown by ProducerInterceptor methods will be caught, logged, but not propagated further. As a result, if
 * the user configures the interceptor with the wrong key and value type parameters, the producer will not throw an exception,
 * just log the errors.
 * <p>
 * ProducerInterceptor callbacks may be called from multiple threads. Interceptor implementation must ensure thread-safety, if needed.
 * <p>
 * Implement {@link org.apache.kafka.common.ClusterResourceListener} to receive cluster metadata once it's available. Please see the class documentation for ClusterResourceListener for more information.
 */
public interface ProducerInterceptor<K, V> extends Configurable {
    /**
     * This is called from {@link org.apache.kafka.clients.producer.KafkaProducer#send(ProducerRecord)} and
     * {@link org.apache.kafka.clients.producer.KafkaProducer#send(ProducerRecord, Callback)} methods, before key and value
     * get serialized and partition is assigned (if partition is not specified in ProducerRecord).
     * <p>
     * This method is allowed to modify the record, in which case, the new record will be returned. The implication of modifying
     * key/value is that partition assignment (if not specified in ProducerRecord) will be done based on modified key/value,
     * not key/value from the client. Consequently, key and value transformation done in onSend() needs to be consistent:
     * same key and value should mutate to the same (modified) key and value. Otherwise, log compaction would not work
     * as expected.
     * <p>
     * Similarly, it is up to interceptor implementation to ensure that correct topic/partition is returned in ProducerRecord.
     * Most often, it should be the same topic/partition from 'record'.
     * <p>
     * Any exception thrown by this method will be caught by the caller and logged, but not propagated further.
     * <p>
     * Since the producer may run multiple interceptors, a particular interceptor's onSend() callback will be called in the order
     * specified by {@link org.apache.kafka.clients.producer.ProducerConfig#INTERCEPTOR_CLASSES_CONFIG}. The first interceptor
     * in the list gets the record passed from the client, the following interceptor will be passed the record returned by the
     * previous interceptor, and so on. Since interceptors are allowed to modify records, interceptors may potentially get
     * the record already modified by other interceptors. However, building a pipeline of mutable interceptors that depend on the output
     * of the previous interceptor is discouraged, because of potential side-effects caused by interceptors potentially failing to
     * modify the record and throwing an exception. If one of the interceptors in the list throws an exception from onSend(), the exception
     * is caught, logged, and the next interceptor is called with the record returned by the last successful interceptor in the list,
     * or otherwise the client.
     *
     * @param record the record from client or the record returned by the previous interceptor in the chain of interceptors.
     * @return producer record to send to topic/partition
     */
    public ProducerRecord<K, V> onSend(ProducerRecord<K, V> record);

    /**
     * This method is called when the record sent to the server has been acknowledged, or when sending the record fails before
     * it gets sent to the server.
     * <p>
     * This method is generally called just before the user callback is called, and in additional cases when <code>KafkaProducer.send()</code>
     * throws an exception.
     * <p>
     * Any exception thrown by this method will be ignored by the caller.
     * <p>
     * This method will generally execute in the background I/O thread, so the implementation should be reasonably fast.
     * Otherwise, sending of messages from other threads could be delayed.
     *
     * @param metadata The metadata for the record that was sent (i.e. the partition and offset).
     *                 If an error occurred, metadata will contain only valid topic and maybe
     *                 partition. If partition is not given in ProducerRecord and an error occurs
     *                 before partition gets assigned, then partition will be set to RecordMetadata.NO_PARTITION.
     *                 The metadata may be null if the client passed null record to
     *                 {@link org.apache.kafka.clients.producer.KafkaProducer#send(ProducerRecord)}.
     * @param exception The exception thrown during processing of this record. Null if no error occurred.
     */
    public void onAcknowledgement(RecordMetadata metadata, Exception exception);

    /**
     * This is called when interceptor is closed
     */
    public void close();
}
```
## ConsumerInterceptor
```
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.apache.kafka.clients.consumer;


import org.apache.kafka.common.Configurable;
import org.apache.kafka.common.TopicPartition;

import java.util.Map;

/**
 * A plugin interface that allows you to intercept (and possibly mutate) records received by the consumer. A primary use-case
 * is for third-party components to hook into the consumer applications for custom monitoring, logging, etc.
 *
 * <p>
 * This class will get consumer config properties via <code>configure()</code> method, including clientId assigned
 * by KafkaConsumer if not specified in the consumer config. The interceptor implementation needs to be aware that it will be
 * sharing consumer config namespace with other interceptors and serializers, and ensure that there are no conflicts.
 * <p>
 * Exceptions thrown by ConsumerInterceptor methods will be caught, logged, but not propagated further. As a result, if
 * the user configures the interceptor with the wrong key and value type parameters, the consumer will not throw an exception,
 * just log the errors.
 * <p>
 * ConsumerInterceptor callbacks are called from the same thread that invokes
 * {@link org.apache.kafka.clients.consumer.KafkaConsumer#poll(java.time.Duration)}.
 * <p>
 * Implement {@link org.apache.kafka.common.ClusterResourceListener} to receive cluster metadata once it's available. Please see the class documentation for ClusterResourceListener for more information.
 */
public interface ConsumerInterceptor<K, V> extends Configurable, AutoCloseable {

    /**
     * This is called just before the records are returned by
     * {@link org.apache.kafka.clients.consumer.KafkaConsumer#poll(java.time.Duration)}
     * <p>
     * This method is allowed to modify consumer records, in which case the new records will be
     * returned. There is no limitation on number of records that could be returned from this
     * method. I.e., the interceptor can filter the records or generate new records.
     * <p>
     * Any exception thrown by this method will be caught by the caller, logged, but not propagated to the client.
     * <p>
     * Since the consumer may run multiple interceptors, a particular interceptor's onConsume() callback will be called
     * in the order specified by {@link org.apache.kafka.clients.consumer.ConsumerConfig#INTERCEPTOR_CLASSES_CONFIG}.
     * The first interceptor in the list gets the consumed records, the following interceptor will be passed the records returned
     * by the previous interceptor, and so on. Since interceptors are allowed to modify records, interceptors may potentially get
     * the records already modified by other interceptors. However, building a pipeline of mutable interceptors that depend on the output
     * of the previous interceptor is discouraged, because of potential side-effects caused by interceptors potentially failing
     * to modify the record and throwing an exception. If one of the interceptors in the list throws an exception from onConsume(),
     * the exception is caught, logged, and the next interceptor is called with the records returned by the last successful interceptor
     * in the list, or otherwise the original consumed records.
     *
     * @param records records to be consumed by the client or records returned by the previous interceptors in the list.
     * @return records that are either modified by the interceptor or same as records passed to this method.
     */
    public ConsumerRecords<K, V> onConsume(ConsumerRecords<K, V> records);

    /**
     * This is called when offsets get committed.
     * <p>
     * Any exception thrown by this method will be ignored by the caller.
     *
     * @param offsets A map of offsets by partition with associated metadata
     */
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets);

    /**
     * This is called when interceptor is closed
     */
    public void close();
}
```