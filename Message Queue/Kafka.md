- [一、了解Kafka](#一了解Kafka)
- [二、Kafka核心概念](#二Kafka核心概念)
- [三、Linux环境中安装Kafka](#三Linux环境中安装Kafka)
    - [3.1 安装Zookeeper](#31-安装Zookeeper)
    - [3.2 安装Kafka](#32-安装Kafka)
- [四、Kafka点对点-发布订阅](#四Kafka点对点-发布订阅)
    - [4.1 命令行进行生产和消费](#41-命令进行生产和消费)
    - [4.2 Kafka消费者组配置实现点对点消费模型](#42-Kafka消费者组配置实现点对点消费模型)
    - [4.3 Kafka消费者组配置实现发布订阅消费模型](#43-Kafka消费者组配置实现发布订阅消费模型)
    - [4.5 Kafka数据存储流程和原理概述和LEO+HW讲解](#45-Kafka数据存储流程和原理概述和LEO-HW讲解)

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

- 守护进程启动kafka
```
./kafka-server-start.sh -daemon ../config/server.properties &
```

## 四、Kafka点对点-发布订阅

### 4.1 命令行进行生产和消费
- 创建topic
```
./kafka-topics.sh --create --zookeeper 外网ip:2181 --replication-factor 1 --partitions 2 --topic version1-topic
```

- 查看topic
```
./kafka-topics.sh --list --zookeeper 外网ip:2181
```

- 生产者发送消息
```
# --topic 后面为指定的topic，默认配置下如果没有，就会自动创建topic
# 真实开发时建议将这个设置给关闭

./kafka-console-producer.sh --broker-list 外网ip:9092  --topic version1-topic
```

- 消费者消费消息
```
#  --from-beginning：从头开始消费，并不会使用偏移量（上次消费到哪），会把主题中以往所有的数据都读取出来, 重启后会有这个重复消费

./kafka-console-consumer.sh --bootstrap-server 外网ip:9092 --from-beginning --topic version1-topic
```

- 删除topic
topic 删除的时候，并不是立即删除，而是先改名字，检查相关配置，再进行真实删除
```
./kafka-topics.sh --zookeeper 外网ip:2181 --delete --topic version1-topic
```

- 查看broker节点topic状态信息
```
./kafka-topics.sh --describe --zookeeper 外网ip:2181  --topic version1-topic
```

### 4.2 Kafka消费者组配置实现点对点消费模型

**一个分区，只能被消费者组下的某一个消费者进行消费**

**编辑消费者配置**

默认的消费者配置文件在，`config/consumer.properties` ,其中指定了相关的消费者配置项包括消费者组，**注意：不要认为默认启动消费者不指定配置文件时用的就时该文件**。在编辑该文件的时候，应该先拷贝一份，再进行修改

- 指定配置文件开启消费者
```
./kafka-console-consumer.sh --bootstrap-server 外网ip:9092 --from-beginning --topic version1-topic --consumer.config ../config/consumer.properties
```
### 4.3 Kafka消费者组配置实现发布订阅消费模型
**两个不同消费者组的节点，都可以消费到消息，实现发布订阅模型**
- 创建两个消费者配置（确保group.id 不一样）
- 创建topic, 2个分区
```
./kafka-topics.sh --create --zookeeper 外网ip:2181 --replication-factor 1 --partitions 2 --topic t2
```
- 指定配置文件启动 两个消费者
```
./kafka-console-consumer.sh --bootstrap-server 112.74.55.160:9092 --from-beginning --topic t1 --consumer.config ../config/consumer-1.properties
​
./kafka-console-consumer.sh --bootstrap-server 112.74.55.160:9092 --from-beginning --topic t1 --consumer.config ../config/consumer-2.properties
```

### 4.5 Kafka数据存储流程和原理概述和LEO+HW讲解

- **Partition：**
    - topic物理上的分组，一个topic可以分为多个partition，每个partition是一个**有序的队列**。
    - 是以文件夹的形式存储在具体Broker本机上

![Partition](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/Partition.png)

- **LEO（LogEndOffset）**
    - 表示每个partition的log最后一条Message的位置

- **HW（HighWatermark）**
    - 表示partition各个replicas数据间同步且一致的offset位置，即表示allreplicas已经commit的位置
    - HW之前的数据才是Commit后的，对消费者才可见
    - ISR集合里面最小leo

![LEO AND HW](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/HW%E5%92%8CLEO.png)

- **offset**
    - 每个partition都由一系列有序的、不可变的消息组成，这些消息被连续的追加到partition中
    - partition中的每个消息都有一个连续的序列号叫做offset，用于partition唯一标识一条消息
    - 可以认为offset是partition中Message的id

- **Segment**
    - segment file 由2部分组成，分别为index file和data file（log file）
    - 两个文件是一一对应的，后缀”.index”和”.log”分别表示索引文件和数据文件
    - 命名规则：partition的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset+1

![整体架构](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.png)

- **Kafka高效文件存储设计特点：**
    - Kafka把topic中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
    - 通过索引信息可以快速定位message
    - producer生产数据，要写入到log文件中，写的过程中一直追加到文件末尾，为顺序写，官网数据表明。同样的磁盘，顺序写能到600M/S，而随机写只有100K/S