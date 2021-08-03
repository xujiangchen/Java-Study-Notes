- [一、了解Kafka](#一了解Kafka)
- [二、Kafka核心概念](#二Kafka核心概念)
- [三、Linux环境中安装Kafka](#三Linux环境中安装Kafka)
    - [3.1 安装Zookeeper](#31-安装Zookeeper)
    - [3.2 安装Kafka](#32-安装Kafka)

# Kafka

## 一、了解Kafka
kafka（open-source distributed event streaming platform），高吞吐量的分布式流处理平台

Kafka严格意义上是不属于队列产品，是一个**流处理平台**，它提供了类似于JMS的特性，但是在设计实现上完全不同，它并不是JMS规范的实现。类似MQ，功能较为简单，**主要支持常规的MQ功能**，但是它的运维难度大，文档比较少, 需要掌握Scala。

它可以处理消费者在网站中的所有动作流数据。

官网：http://kafka.apache.org/

## 二、Kafka核心概念

**Broker：**
- 可以类比于数据库
- Kafka的服务端程序，可以认为**一个mq节点就是一个broker**
- broker存储topic的数据

**Producer生产者：**
- 创建消息Message，然后发布到MQ中
- 该角色将消息发布到Kafka的topic中

**Consumer消费者：**
- 消费队列里面的消息

**ConsumerGroup消费者组**
- 同个topic, 广播发送给不同的group，一个group中只有一个consumer可以消费此消息

**Topic：**
- 可以类比于数据库中的表
- 每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic，主题的意思

**Partition分区：**
 - kafka数据存储的基本单元，topic中的数据分割为一个或多个partition，**每个topic至少有一个partition，是有序的**
 - 一个Topic的多个partitions, 被分布在kafka集群中的多个server上
 - **消费者数量 <=小于或者等于Partition数量**

**Replication 副本（备胎）：**
- 同个Partition会有多个副本replication ，多个副本的数据是一样的，当其他broker挂掉后，系统可以主动用副本提供服务
- 默认每个topic的副本都是1（默认是没有副本，节省资源），也可以在创建topic的时候指定

**ReplicationLeader、ReplicationFollower：**
- 每一个Partition**有且只有一个Replica可以作为Leader**
- 每个Partition中除了Leader以外的所有Replica均为follower
- Partition有多个副本，但只有一个replicationLeader负责该Partition和生产者消费者交互
- ReplicationFollower只是做一个备份，从replicationLeader进行同步

**ReplicationManager：**
- 负责Broker所有分区副本信息，Replication 副本状态切换

**offset：**
- 每个consumer实例需要为他消费的partition维护一个记录自己消费到哪里的偏移offset
- kafka把offset保存在消费端的消费者组里

## 三、Linux环境中安装Kafka

- kafka下载地址：http://kafka.apache.org/downloads
- zookeeper下载格式：https://zookeeper.apache.org/releases.html

### 3.1 安装Zookeeper
- 默认2181端口
```
# 解压
tar -zxvf apache-zookeeper-3.7.0-bin.tar.gz

# 对解压文件重命名，方便管理
mv apache-zookeeper-3.7.0-bin zookeeper

# 进入zookeeper配置文件夹
cd zookeeper/conf/

# 复制默认配置文件进行自定义
cp zoo_sample.cfg zoo.cfg

# 启动zookeeper，回到上级，进入bin目录
cd bin/
./zkServer.sh start

# 查看日志
cd logs/
tail -300f zookeeper-root-server-izm5e55czz7lmt0ny9zu62z.out

# 查看端口：
yum install -y lsof
lsof -i:2181
```


### 3.2 安装Kafka
- 默认端口9092
```
# 解压
tar -zvxf kafka_2.13-2.8.0.tgz

# 对解压文件重命名，方便管理
mv kafka_2.13-2.8.0 kafka
```
- config目录下 server.properties
```
#标识broker编号，集群中有多个broker，则每个broker的编号(id)需要设置不同
broker.id=0


#修改下面两个配置 ( listeners 配置的ip和advertised.listeners相同时启动kafka会报错)

listeners(内网Ip) =>  listeners=PLAINTEXT://内网Ip:9092
advertised.listeners(公网ip) => advertised.listeners=PLAINTEXT://公网ip:9092


#修改zk地址,默认地址(如果zk和kafka在一台机器上就不要修改)
zookeeper.connection=localhost:2181
```
- bin目录启动
```
# 启动，并指定配置文件
./kafka-server-start.sh  ../config/server.properties &

# 停止kafka
./kafka-server-stop.sh
```

- 创建topic
```
./kafka-topics.sh --create --zookeeper 外网ip:2181 --replication-factor 1 --partitions 1 --topic demo-topic

# 创建成功后可以到，这个目录下载查看
cd /tmp/kafka-logs/
```

- 查看当前的topic 
```
./kafka-topics.sh --list --zookeeper 外网ip:2181
```