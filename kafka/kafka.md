# kafka
- 消息队列的作用
  - 应用解耦
  - 异步处理
  - 并发限流
  - 消息通信
- Kafka的组成
  - 消息处理
    - Broker
    - Producer
    - Consumer
    - Comsumer Group
  - 消息对象
    - Topic
    - Partition(offset)
    - Replication
    - Record(key,value,timestamp)

- 集群配置项
  - server.properties
    - broker.id=0
    - listeners=PLAINTEXT://kafka0:9092
    - log.dirs=/usr/local/application/kafka/log
    - zookeeper.connect=kafka0:2181,kafka1:2181,kafka2:2181

- 启动命令
  - ./kafka-server-start.sh ../config/server.properties

- zkCli.sh
  - ls /brokers/ids/ 查看zk中是否注册成功
  - get /brokers/ids/0 查看注册的broker详情

- 常用参数
  ```
  # 创建topic，1副本，4分区
  kafka-topics.sh --create  --zookeeper 192.168.1.16:2181 192.168.1.17:2181 192.168.1.18:2181 --replication-factor 1 --partitions 4 --topic zoo
  
  # topic 列表
  kafka-topics.sh --zookeeper 192.168.1.16:2181,192.168.1.17:2181,192.168.1.18:2181 --list
  
  # 查看topic详情
  kafka-topics.sh --describe --zookeeper 192.168.1.16:2181,192.168.1.17:2181,192.168.1.18:2181
  
  # 生产者
  kafka-console-producer.sh --broker-list 192.168.1.16:9092,192.168.1.17:9092,192.168.1.18:9092,192.168.1.19:9092 --topic zoo
  
  # 消费者
  kafka-console-consumer.sh --bootstrap-server 192.168.1.16:9092,192.168.1.17:9092,192.168.1.18:9092,192.168.1.19:9092 --topic zoo
  
  # 消费者，指定offset从最开始消费，指定offset后必须指定partition
  kafka-console-consumer.sh --bootstrap-server 192.168.1.16:9092,192.168.1.17:9092,192.168.1.18:9092,192.168.1.19:9092 --topic zoo --offset earliest --partition 0
  
  # 创建自定义group属性的消费者
  kafka-console-consumer.sh --bootstrap-server kafka0:9092,kafka1:9092,kafka2:9092 --topic test --consumer-property group.id=test1
  
  # 自带压力测试
  kafka-producer-perf-test.sh --topic test --num-records 100 --record-size 1 --throughput 100  --producer-props bootstrap.servers=localhost:9092
  
 
  ```

- broker 重要参数
```
# 消息保留时长
log.retention.hours=168
# 消息保留大小
log.retention.bytes=1073741824

```
  
- kafka-eagle
  - 修改kafka启动参数：vi kafka-server-start.sh
    ```
    if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
        export KAFKA_HEAP_OPTS="-server -Xmx1G -Xms1G -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"
        export JMX_PORT="9999"

        #export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
    fi
    ```
  - vi conf/system-config.properties
    ```
    kafka.eagle.zk.cluster.alias=cluster1
    cluster1.zk.list=kafka0:2181,kafka1:2181,kafka2:2181
    cluster1.kafka.eagle.offset.storage=kafka
    kafka.eagle.metrics.charts=true
    kafka.eagle.sql.fix.error=true
    ```

- 0.10.1.x相关配置解释
http://kafka.apache.org/0101/documentation.html#producerconfigs


- KafkaOffsetMonitor
```
https://github.com/Morningstar/kafka-offset-monitor
java.security.krb5.conf

java -cp KafkaOffsetMonitor-assembly-0.4.6-SNAPSHOT.jar \
       com.quantifind.kafka.offsetapp.OffsetGetterWeb \
     --offsetStorage kafka \
     --kafkaBrokers 192.168.1.16:9092,192.168.1.17:9092,192.168.1.18:9092,192.168.1.19:9092 \
     --kafkaSecurityProtocol PLAINTEXT \
     --zk 192.168.1.16:2181,192.168.1.17:2181,192.168.1.18:2181 \
     --port 15125 \
     --refresh 10.seconds \
     --retain 1.days \
     --dbName offsetapp_kafka
     
     
java -Xms800m -Xmx4096m -Xmn1024m -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection \
     -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:+CMSClassUnloadingEnabled \
     -XX:SurvivorRatio=8 -XX:+DisableExplicitGC -Xss1024K -XX:PermSize=256m -XX:MaxPermSize=1024m \
     -cp kafka-offset-monitor-assembly-0.4.7.jar \
     com.quantifind.kafka.offsetapp.OffsetGetterWeb \
     --offsetStorage kafka \
     --kafkaBrokers 192.168.1.16:9092,192.168.1.17:9092,192.168.1.18:9092,192.168.1.19:9092 \
     --kafkaSecurityProtocol PLAINTEXT \
     --zk 192.168.1.16:2181,192.168.1.17:2181,192.168.1.18:2181 \
     --port 8086 \
     --refresh 10.seconds \
     --retain 1.days \
     --dbName offsetapp_kafka

java -cp KafkaOffsetMonitor-assembly-0.4.6-SNAPSHOT.jar \
       com.quantifind.kafka.offsetapp.OffsetGetterWeb \
     --offsetStorage kafka \
     --kafkaBrokers 192.168.1.16:9092,192.168.1.17:9092,192.168.1.18:9092,192.168.1.19:9092 \
     --kafkaSecurityProtocol SASL_PLAINTEXT \
     --zk 192.168.1.16:2181,192.168.1.17:2181,192.168.1.18:2181 \
     --port 15125 \
     --refresh 10.seconds \
     --retain 1.days \
     --dbName offsetapp_kafka


-Djava.security.krb5.conf=/etc/krb5.conf -Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf
java -cp KafkaOffsetMonitor-assembly-0.4.6-SNAPSHOT.jar com.quantifind.kafka.offsetapp.OffsetGetterWeb --zk 192.168.1.16:2181,192.168.1.17:2181,192.168.1.18:2181 --port 8088 --refresh 5.seconds --retain1.days
```
