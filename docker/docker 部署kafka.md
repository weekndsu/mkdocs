# docker 部署kafka

docker 部署kafka

#### 1. 拉去kafka和zookeeper镜像

docker pull wurstmeister/zookeeper

docker pull wurstmeister/kafka

#### 2. 启动zookeeper

docker run -d --name kafka-zookeeper -p 2181:2181 --volume /etc/localtime:/etc/localtime wurstmeister/zookeeper:latest 

#### 3. 启动kafka

docker run -d --name kafka -p 9092:9092 --link kafka-zookeeper --env KAFKA_ZOOKEEPER_CONNECT=kafka-zookeeper:2181 --env KAFKA_ADVERTISED_HOST_NAME=localhost --env KAFKA_ADVERTISED_PORT=9092 --volume /etc/localtime:/etc/localtime wurstmeister/kafka:latest   

#### 4. 启动第一个终端

docker exec -it kafka /bin/bash 

#### 5. 查看topic

cd opt/kafka_2.11-2.0.0/ 

bin/kafka-topics.sh --list --zookeeper kafka-zookeeper:2181  

#### 6. 创建一个topic

bin/kafka-topics.sh --create --zookeeper kafka-zookeeper:2181 --replication-factor 1 --partitions 1 --topic mykafka

Created topic "mykafka".

#### 7. 创建一个生产者

bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka

\>

```
NOTE:waiting input message,once you input message in the producer terminal,you will see the same message on your consumer terminal.
```

#### 8. 打开另一个kafka终端

 docker exec -it kafka /bin/bash

#### 9. 再次检查topic

pwd

/opt/kafka_2.11-2.0.0

bash-4.4# bin/kafka-topics.sh --list --zookeeper kafka-zookeeper:2181

mykafka

#### 10. 创建一个consumer

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning

#### 11. 测试producer到consumer

##### 11.1 生产者终端输入消息

bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka

\>**i'm weeknd**

##### 11.2 检查消费者终端

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning

**i'm weeknd**



Finished！