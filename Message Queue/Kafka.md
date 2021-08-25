- [Kafka](#kafka)
  - [一、了解Kafka](#一了解kafka)
  - [二、Kafka核心概念](#二kafka核心概念)
  - [三、Linux环境中安装Kafka](#三linux环境中安装kafka)
    - [3.1 安装Zookeeper](#31-安装zookeeper)
    - [3.2 安装Kafka](#32-安装kafka)
  - [四、Kafka点对点-发布订阅](#四kafka点对点-发布订阅)
    - [4.1 命令行进行生产和消费](#41-命令行进行生产和消费)
    - [4.2 Kafka消费者组配置实现点对点消费模型](#42-kafka消费者组配置实现点对点消费模型)
    - [4.3 Kafka消费者组配置实现发布订阅消费模型](#43-kafka消费者组配置实现发布订阅消费模型)
    - [4.5 Kafka数据存储流程和原理概述和LEO+HW讲解](#45-kafka数据存储流程和原理概述和leohw讲解)
  - [五、Springboot整合kafka](#五springboot整合kafka)
    - [5.1 Admin相关Api](#51-admin相关api)
      - [5.1.1 配置客户端](#511-配置客户端)
      - [5.1.2 创建Topic](#512-创建topic)
      - [5.1.3 列举Topic](#513-列举topic)
      - [5.1.4 删除Topic](#514-删除topic)
      - [5.1.5 查看Topic详情](#515-查看topic详情)
      - [5.1.6 增加分区数量](#516-增加分区数量)
    - [5.2 生产者相关Api](#52-生产者相关api)
      - [5.2.1 发送分区策略和常见配置](#521-发送分区策略和常见配置)
      - [5.2.2 封装配置属性](#522-封装配置属性)
      - [5.2.3 生产者投递消息(同步发送)](#523-生产者投递消息同步发送)
      - [5.2.4 回调函数](#524-回调函数)
      - [5.2.5 生产者发送指定分区](#525-生产者发送指定分区)
      - [5.2.6 生产者自定义partition分区规则](#526-生产者自定义partition分区规则)
    - [5.3 消费者相关Api](#53-消费者相关api)
      - [5.3.1 消费者机制和分区策略](#531-消费者机制和分区策略)
      - [5.3.2 消费者重新分配策略和offset维护机制](#532-消费者重新分配策略和offset维护机制)
      - [5.3.3 Kafka调试日志配置](#533-kafka调试日志配置)
      - [5.3.4 Consumer配置和消费订阅](#534-consumer配置和消费订阅)
      - [5.3.5 手工提交offset](#535-手工提交offset)
  - [六、kafka可靠性保证](#六kafka可靠性保证)
    - [6.1 副本Replica+ACK](#61-副本replicaack)
    - [6.2 in-sync-replica-set机制](#62-in-sync-replica-set机制)
    - [6.3 HighWatermark的作用](#63-highwatermark的作用)
  - [七、kafka高可用和高性能](#七kafka高可用和高性能)
    - [7.1 搭建kafka集群](#71-搭建kafka集群)
    - [7.2 Springboot连接kafka集群](#72-springboot连接kafka集群)
    - [7.3 日志数据清理](#73-日志数据清理)
    - [7.4 ZeroCopy](#74-zerocopy)


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


## 五、Springboot整合kafka

### 5.1 Admin相关Api

#### 5.1.1 配置客户端
```java
public static AdminClient initAdminClient(){
    Properties properties = new Properties();
    // 设置连接地址
    properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    AdminClient adminClient = AdminClient.create(properties);
    return adminClient;
}
```

#### 5.1.2 创建Topic
```java
private static final String TOPIC_NAME = "demo-topic";

public void createTopic(){
    AdminClient adminClient = initAdminClient();

    // 指定分区数量，副本数量
    NewTopic newTopic = new NewTopic(TOPIC_NAME, 2, (short)1);
    CreateTopicsResult createTopicsResult = adminClient.createTopics(Arrays.asList(newTopic));

    try {
        //future等待创建，成功不会有任何报错，如果创建失败和超时会报错。
        createTopicsResult.all().get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```
#### 5.1.3 列举Topic
```java
public void listTopic(){
    AdminClient adminClient = initAdminClient();
    // 不带参数返回的是用户自己创建的相关Topic
    ListTopicsResult listTopicsResult = adminClient.listTopics();
    try {
        Set<String> topicSet = listTopicsResult.names().get();
        for (String item:topicSet) {
            System.err.println(item);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    
    // 如果想带参数进行查询
    ListTopicsOptions listTopicsOptions = new ListTopicsOptions();
    // 展示kafka内部创建的topic
    listTopicsOptions.listInternal(true);
    ListTopicsResult listTopicsResult2 = adminClient.listTopics(listTopicsOptions);
    try {
        Set<String> topicSet = listTopicsResult2.names().get();
        for (String item:topicSet) {
            System.err.println(item);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

#### 5.1.4 删除Topic
```java
public void delTopic(){
    AdminClient adminClient = initAdminClient();
    DeleteTopicsResult deleteTopicsResult = adminClient.deleteTopics(Arrays.asList("xudemo-topic"));
    try {
        deleteTopicsResult.all().get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```
#### 5.1.5 查看Topic详情
```java
public void detailTopic(){
    AdminClient adminClient = initAdminClient();
    DescribeTopicsResult describeTopicsResult = adminClient.describeTopics(Arrays.asList(TOPIC_NAME));
    try {
        Map<String, TopicDescription> stringTopicDescriptionMap = describeTopicsResult.all().get();
        Set<Map.Entry<String, TopicDescription>> entries = stringTopicDescriptionMap.entrySet();
        entries.stream().forEach((entry)-> System.out.println("name:" + entry.getKey() + " value:" + entry.getValue()));
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

#### 5.1.6 增加分区数量
- Kafka中的分区数只能增加不能减少，减少的话数据不知怎么处理
```java
public void  incrPartitions(){
    Map<String, NewPartitions> infoMap = new HashMap<>();
    // 分区增加到5个
    NewPartitions newPartitions = NewPartitions.increaseTo(5);
    AdminClient adminClient = initAdminClient();
    infoMap.put(TOPIC_NAME, newPartitions);
    // 创建分区
​    CreatePartitionsResult createPartitionsResult = adminClient.createPartitions(infoMap);
    try {
        createPartitionsResult.all().get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

### 5.2 生产者相关Api

#### 5.2.1 发送分区策略和常见配置
- 如果指定Partition ID,则PR(ProducerRecord)被发送至指定Partition
- 如果未指定Partition ID,但发送方的消息中指定了Key, PR会按照hash(key)发送至对应Partition
- 如果未指定Partition ID,也没指定Key，PR会按照默认 round-robin轮训模式发送到每个Partition
- 如果同时指定了Partition ID和Key, PR只会发送到指定的Partition 

![Partition](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/sendModel.png)
**生产者常见配置**
- 官方文档 http://kafka.apache.org/documentation/#producerconfigs


#### 5.2.2 封装配置属性
```java
public static Properties getProperties(){

    Properties props = new Properties();

    props.put("bootstrap.servers", "118.190.132.65:9092");

    // 当producer向leader发送数据时，可以通过request.required.acks参数来设置数据可靠性的级别,分别是0, 1，all。
    props.put("acks", "all");


    // 请求失败，生产者会自动重试，指定是0次，如果启用重试，则会有重复消息的可能性
    props.put("retries", 0);

    // 生产者缓存每个分区未发送的消息,缓存的大小是通过 batch.size 配置指定的，默认值是16KB
    props.put("batch.size", 16384);

    /**
        * 默认值就是0，消息是立刻发送的，即便batch.size缓冲空间还没有满
        * 如果想减少请求的数量，可以设置 linger.ms 大于0，即消息在缓冲区保留的时间，超过设置的值就会被提交到服务端
        * 通俗解释是，本该早就发出去的消息被迫至少等待了linger.ms时间，相对于这时间内积累了更多消息，批量发送减少请求
        * 如果batch被填满或者linger.ms达到上限，满足其中一个就会被发送
        */
    props.put("linger.ms", 1);

    /**
        * buffer.memory的用来约束Kafka Producer能够使用的内存缓冲的大小的，默认值32MB。
        * 如果buffer.memory设置的太小，可能导致消息快速的写入内存缓冲里，但Sender线程来不及把消息发送到Kafka服务器
        * 会造成内存缓冲很快就被写满，而一旦被写满，就会阻塞用户线程，不让继续往Kafka写消息了
        * buffer.memory要大于batch.size，否则会报申请内存不#足的错误，不要超过物理内存，根据实际情况调整
        * 需要结合实际业务情况压测进行配置
        */
    props.put("buffer.memory", 33554432);

    /**
        * key的序列化器，将用户提供的 key和value对象ProducerRecord 进行序列化处理，key.serializer必须被设置，
        * 即使消息中没有指定key，序列化器必须是一个实
        org.apache.kafka.common.serialization.Serializer接口的类，
        * 将key序列化成字节数组。
        */
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
    return props;
}
```
#### 5.2.3 生产者投递消息(同步发送)
```java
/**
    * send()方法是异步的，添加消息到缓冲区等待发送，并立即返回
    * 生产者将单个的消息批量在一起发送来提高效率,即 batch.size和linger.ms结合
    *
    * 实现同步发送：一条消息发送之后，会阻塞当前线程，直至返回 ack
    * 发送消息后返回的一个 Future 对象，调用get即可
    *
    * 消息发送主要是两个线程：一个是Main用户主线程，一个是Sender线程
    *  1)main线程发送消息到RecordAccumulator即返回
    *  2)sender线程从RecordAccumulator拉取信息发送到broker
    *  3) batch.size和linger.ms两个参数可以影响 sender 线程发送次数
    */
public void sender(){
    Properties properties = getProperties();
    Producer<String, String> producer = new KafkaProducer<String, String>(properties);
    for (int i = 0; i < 3 ; i++) {
        Future<RecordMetadata> future = producer.send(new ProducerRecord<>("demo-topic", "demo-key" + i, "demo-value" + i));
        try {
            // 如果不关心结果，以下代码无用
            // 同样这段代码实现了同步发送的功能
            RecordMetadata recordMetadata = future.get();
            System.out.println("发送状态码：" + recordMetadata.toString());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
    producer.close();
}
```
#### 5.2.4 回调函数
- 生产者发送消息是异步调用，怎么知道是否有异常？发送消息配置回调函数即可
```java
public void sendWithCallback(){
    Properties props = getProperties();
    Producer<String, String> producer = new KafkaProducer<String, String>(props);
    for (int i = 0; i < 3 ; i++) {
        producer.send(new ProducerRecord<>("demo-topic", "demo-key" + i, "demo-value" + i), new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                if (e == null){
                    System.err.println("发送状态:"+recordMetadata.toString());
                }else {
                    e.printStackTrace();
                }
            }
        });
    }
    producer.close();
}
```

#### 5.2.5 生产者发送指定分区
```java
public void sendAppointPartition(){
    Properties properties = getProperties();
    Producer<String, String> producer = new KafkaProducer<String, String>(properties);
    for (int i = 0; i < 3 ; i++) {
        // 在发送时第二个参数指定分区编号，分区编号从0开始
        // 如果发送到一个不存在的分区编号，则会报错
        producer.send(new ProducerRecord<>("demo-topic", 2, "demo-key" + i, "demo-value" + i), new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                if (e == null){
                    System.err.println("发送状态:"+recordMetadata.toString());
                }else {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

#### 5.2.6 生产者自定义partition分区规则
- 创建类，实现Partitioner接口，重写方法
```java
public class demoPartitioner implements Partitioner {

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        // 如果key为空则给出异常消息
        if (keyBytes == null){
            throw new IllegalArgumentException("Key 参数不能为空");
        }

        // 如果key等于demo，则放到分区编号为0的分区里面
        if ("demo".equals(key)){
            return 0;
        }

        // 否则根据默认hash取模对消息进行分布
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}
```
- 使用自定义的分区策略
```java
public void sendAppointPartition(){
    Properties properties = getProperties();
    // 指定分区策略
    props.put("partitioner.class", "com.blaster.kafkademo.config.demoPartitioner");
    Producer<String, String> producer = new KafkaProducer<String, String>(properties);
    for (int i = 0; i < 3 ; i++) {
        producer.send(new ProducerRecord<>("demo-topic", "demo-key" + i, "demo-value" + i), new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                if (e == null){
                    System.err.println("发送状态:"+recordMetadata.toString());
                }else {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

### 5.3 消费者相关Api

#### 5.3.1 消费者机制和分区策略
- **round-robin （RoundRobinAssignor非默认策略）轮训**

原理是将消费组内所有消费者以及消费者所订阅的所有topic的partition按照字典序排序，然后通过轮询方式逐个将分区以此分配给每个消费者。如果同一个消费组内所有的消费者的订阅信息都是相同的，那么RoundRobinAssignor策略的分区分配会是均匀的。

![Partition](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/RoundRobinAssignor.png)

它存在的弊端

![Partition](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/RoundRobinAssignor2.png)


- **range （RangeAssignor默认策略）范围**

按照主进行分配，如果不平均分配，则第一个消费者会分配比较多分区， 一个消费者监听不同主题也不影响。

![Partition](https://github.com/xujiangchen/Java-Study-Notes/blob/main/Message%20Queue/imgs/RangeAssignor.png)

它存在的弊端：

如果有 N 多个 topic，那么针对每个 topic，消费者 C-1 都将多消费 1 个分区，topic越多则消费的分区也越多，则性能有所下降

#### 5.3.2 消费者重新分配策略和offset维护机制

- **什么是Rebalance操作:** 
    - kafka 怎么均匀地分配某个 topic 下的所有 partition 到各个消费者，从而使得消息的消费速度达到最快，这就是平衡（balance），前面讲了 Range 范围分区 和 RoundRobin 轮询分区，也支持自定义分区策略。
    - 而 rebalance（重平衡）其实就是重新进行 partition 的分配，从而使得 partition 的分配重新达到平衡状态

> 例如70个分区，10个消费者，但是先启动一个消费者，后续再启动一个消费者，这个会怎么分配？
> 
> Kafka 会进行一次分区分配操作，即 Kafka 消费者端的 Rebalance 操作 ，下面都会发生rebalance操作:
>
> 1.当消费者组内的消费者数量发生变化（增加或者减少），就会产生重新分配patition
>
> 2.分区数量发生变化时(即 topic 的分区数量发生变化时)
>

- **当消费者在消费过程突然宕机了，重新恢复后是从哪里消费，会有什么问题？**
    - 消费者会记录offset，故障恢复后从这里继续消费
    - 记录在zk里面和本地，新版默认将offset保证在kafka的内置topic中，名称是 __consumer_offsets
    - 由 消费者组名+主题+分区，确定唯一的offset的key，从而获取对应的值
    - 三元组：group.id+topic+分区号，而 value 就是 offset 的值


#### 5.3.3 Kafka调试日志配置
```yml
logging:
  config: classpath:logback.xml
```
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!-- 格式化输出： %d表示日期， %thread表示线程名， %-5level: 级别从左显示5个字符宽度 %msg:日志消息, %n是换行符 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}[%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
​
    <root level="info">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

#### 5.3.4 Consumer配置和消费订阅
- **配置信息：**
```java
public static Properties getProperties() {
    Properties props = new Properties();
​
    //broker地址
    props.put("bootstrap.servers", "ip:9092");
​
    //消费者分组ID，分组内的消费者只能消费该消息一次，不同分组内的消费者可以重复消费该消息
    props.put("group.id", "");
​
    //开启自动提交offset
    props.put("enable.auto.commit", "true");
​
    //自动提交offset延迟时间
    props.put("auto.commit.interval.ms", "1000");
​
    //反序列化
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
​
    return props;
}
```
- **消费者订阅：**
```java
public void simpleConsumer(){
    // 获取配置信息
    Properties properties = getProperties();
    // 创建消费者
    KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<String, String>(properties);
    // 订阅他topic
    kafkaConsumer.subscribe(Arrays.asList("demo-topic"));
    while (true){
        //拉取时间控制，阻塞超时时间
        ConsumerRecords<String, String> poll = kafkaConsumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord record : poll){
            System.err.printf("topic = %s, offset = %d, key = %s, value = %s%n",record.topic(), record.offset(), record.key(), record.value());
        }
    }
}
```
- **重新从头消费：**
    - auto.offset.reset 配置策略即可 `props.put("auto.offset.reset","earliest")`
    - 消费者组名变更

#### 5.3.5 手工提交offset

- 自动提交offset问题:
```
在实际使用中，kafka只是一个中间件，后面可能关联各种各样的服务，消息被这些服务消费之后，进行后续操作时，例如保存数据库，如果失败了，自动提交offset就会出现问题。这个消息我们并没用成功处理，我们不应该让kafka确认消费。

适合不严谨的场景，比如日志收集发送
```

- 手工提交offset
	- 同步 commitSync 阻塞当前线程 
		- 优点：失败重试，（内部机制，不保证100%）
		- 缺点：同步请求，会阻塞线程，使用较少
	- 异步 commitAsync 不会阻塞当前线程 
		- 优点：不会阻塞当前线程，回调callback函数获取提交信息，记录日志
		- 缺点：没有失败重试


**更改配置：**：
```java
properties.put("enable.auto.commit", "false");
```

**测试代码：**
```java
public void simpleConsumer(){
    Properties properties = getProperties();
    KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<String, String>(properties);
    kafkaConsumer.subscribe(Arrays.asList("demo-topic"));
    while (true){
        ConsumerRecords<String, String> poll = kafkaConsumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord record : poll){
            System.err.printf("topic = %s, offset = %d, key = %s, value = %s%n",record.topic(), record.offset(), record.key(), record.value());
        }
        // 缺点：同步请求，会阻塞线程，使用较少
        // 优点：失败重试，（内部机制，不保证100%）
        // kafkaConsumer.commitSync();

        if (!poll.isEmpty()){
        	// 异步
            kafkaConsumer.commitAsync((map, e) -> {
                if (e == null){
                    System.out.println("提交成功！" + map.toString());
                }else {
                    e.printStackTrace();
                }
            });
        }
    }
}
```

## 六、kafka可靠性保证

### 6.1 副本Replica+ACK

- 生产者发送数据流程:
    - 1、producer 发送到指定的 topic
    - 2、topic 发送 ack 确认收到
    - 3、producer 收到 ack 进行下一轮的发送，否则重新发送数据

- 副本数据同步机制：
    - 当producer在向partition中写数据时，根据ack机制，默认ack=1，只会向leader中写入数据
    - 然后leader中的数据会复制到其他的replica中，follower会周期性的从leader中pull数据
    - 对于数据的读写操作都在leader replica中，follower副本只是当leader副本挂了后才重新选取leader

**问题：** follower并不向外提供服务，假如还没同步完成，leader副本就宕机了，怎么办？
```
首先，kafka通过LEO+HW 保证了消费者可见数据的同一性。其次，kafka对这个ack返回机制，提供了多种模式。
```
**要追求高吞吐量，那么就要放弃可靠性，两者不可兼得。**

- ack=0
    - producer发送一次就不再发送了，不管是否发送成功
    - 发送出去的消息还在半路，或者还没写入磁盘， Partition Leader所在Broker就直接挂了，客户端认为消息发送成功了，此时就会导致这条消息就丢失

- ack=1(默认)
    - 只要Partition Leader接收到消息而且写入【本地磁盘】，就认为成功了，不管他其他的Follower有没有同步过去这条消息了
    - 万一Partition Leader刚刚接收到消息，Follower还没来得及同步过去，结果Leader所在的broker宕机了

- ack= all（即-1）
    - producer只有收到分区内所有副本的成功写入全部落盘的通知才认为推送消息成功
    ```
    1、如果出现一个follower一直同步不成功，难道一直不返回ack吗？

        Leader会维护一个ISR（副本集合，包括leader自身），里面会记录所有的副本，当kafka任务某一个副本失去连接的时候，会在ISR中将该副本提出，ISR是动态变化的。
    
    2、acks=all 就可以代表数据一定不会丢失了吗？

        - Partition只有一个副本，也就是一个Leader，任何Follower都没有
        - 接收完消息后宕机，也会导致数据丢失，acks=all，必须跟ISR列表里至少有2个以上的副本配合使用
        - 在设置request.required.acks=-1的同时，也要min.insync.replicas这个参数设定 ISR中的最小副本数是多少，默认值为1，改为 >=2，如果ISR中的副本数少于min.insync.replicas配置的数量时，客户端会返回异常
    
    3、ack= all还有可能会导致数据重复

        数据发送到leader后 ，部分ISR的副本同步，leader此时挂掉，ack还没有返回。producer端会得到返回异常，producer端会重新发送数据，数据可能会重复。
    ```

### 6.2 in-sync-replica-set机制
- 什么是ISR (in-sync replica set )
    - leader会维持一个与其保持同步的replica集合，该集合就是ISR，**每一个leader partition都有一个ISR，leader动态维护**, 要保证kafka不丢失message，就要保证ISR这组集合存活（至少有一个存活），并且消息commit成功
    - Partition leader 保持同步的 Partition Follower 集合, 当 ISR 中的Partition Follower 完成数据的同步之后，就会给 leader 发送 ack
    - 如果Partition follower长时间(**replica.lag.time.max.ms**) 未向leader同步数据，则该Partition Follower将被踢出ISR
    - Partition Leader 发生故障之后，就**会从 ISR 中选举新的 Partition Leader**。

- OSR （out-of-sync-replica set）
    - 与leader副本分区 同步滞后过多的副本集合

- AR（Assign Replicas）
    - 分区中所有副本统称为AR, AR = ISR + OSR

### 6.3 HighWatermark的作用
保证消费数据的一致性和副本数据的一致性
```
假设没有HW,消费者消费leader到15，下面消费者应该消费16。
​
此时leader挂掉,选下面某个follower为leader，此时消费者找新leader消费数据，发现新Leader没有16数据，报错。
​
HW(High Watermark)是所有副本中最小的LEO。
```

- **Follower故障**
    - Follower发生故障后会被临时踢出ISR（动态变化），待该follower恢复后，follower会读取本地的磁盘记录的上次的HW，并将该log文件高于HW的部分截取掉，从HW开始向leader进行同步，等该follower的LEO大于等于该Partition的hw，即follower追上leader后，就可以重新加入ISR

- **Leader故障**
    - Leader发生故障后，会从ISR中选出一个新的leader，为了保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于hw的部分截掉（新leader自己不会截掉），然后从新的leader同步数据

## 七、kafka高可用和高性能

### 7.1 搭建kafka集群
```
# 解压
tar -zvxf kafka_2.13-2.8.0.tgz

# 对解压文件重命名，方便管理
mv kafka_2.13-2.8.0 kafka

# 编辑配置文件
1. broker.id                # 节点id,配置不同的id
2. listeners                # 局域网IP,内网部署 kafka 集群只需要用到 listeners
3. advertised.listeners     # 外网IP,内外网需要作区分时 才需要用到advertised.listeners
4. port                     # 添加端口属性，指定端口，每个节点都不一样
5. log.dirs                 # 日志存储位置
6. zookeeper.connect        # zk地址,集群的话用逗号分隔

# 启动kafka
./kafka-server-start.sh -daemon ../config/server.properties &   # 守护进程启动
​
./kafka-server-start.sh ../config/server.properties &       # 非守护进程启动
```
### 7.2 Springboot连接kafka集群
```java
public class Admin {

    private static final String TOPIC_NAME = "demo-cluster-topic";


    public static AdminClient initAdminClient(){
        Properties properties = new Properties();
        properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "118.191.132.65:9092,118.191.132.65:9093,118.191.132.65:9094");
        AdminClient adminClient = AdminClient.create(properties);
        return adminClient;
    }

    /**
     * 创建topic
     */
    @Test
    public void creatTopic(){
        AdminClient adminClient = initAdminClient();
        NewTopic newTopic = new NewTopic(TOPIC_NAME, 6, (short) 3);
        CreateTopicsResult topics = adminClient.createTopics(Arrays.asList(newTopic));
        try {
            topics.all().get();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 查看topic
     */
    @Test
    public void listTopic(){
        AdminClient adminClient = initAdminClient();
        ListTopicsResult listTopicsResult = adminClient.listTopics();

        try {
            Set<String> strings = listTopicsResult.names().get();
            for (String item : strings){
                System.err.println(item);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}

```

### 7.3 日志数据清理

Kafka将数据持久化到了硬盘上，为了控制磁盘容量，需要对过去的消息进行清理。kafka并不是为了做存储使用的。

- kafka的默认配置：
  - 内部有个定时任务检测删除日志，默认是5分钟 `log.retention.check.interval.ms`
  - 支持配置策略对数据清理
  - 根据segment单位进行定期清理


- 启用cleaner
  - log.cleaner.enable=true
  - log.cleaner.threads = 2 (清理线程数配置)

**在kafka中，清理日志给出了两大类的策略：日志删除 和 日志压缩**
- 日志删除 `log.cleanup.policy=delete`
```
#清理超过指定时间的消息,默认是168小时，7天，
#还有log.retention.ms, log.retention.minutes, log.retention.hours，优先级高到低
log.retention.hours=168
​
#超过指定大小后，删除旧的消息，下面是1G的字节数，-1就是没限制
log.retention.bytes=1073741824
​
还有基于日志起始位移（log start offset)，未来社区还有更多


问题一：配置了7天后删除，那7天如何确定呢？
​
每个日志段文件都维护一个最大时间戳字段，每次日志段写入新的消息时，都会更新该字段,一个日志段segment写满了被切分之后，就不再接收任何新的消息，最大时间戳字段的值也将保持不变,kafka通过将当前时间与该最大时间戳字段进行比较，从而来判定是否过期

log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除
```
- 日志压缩 `log.cleanup.policy=compact`
```
按照消息key进行整理，有相同key不同value值，只保留最后一个
```

### 7.4 ZeroCopy
Linux有两个上下文，内核态，用户态。

- 原始的文件发送：
  - 调用read,将文件拷贝到了kernel内核态
  - CPU控制 kernel态的数据copy到用户态
  - 调用write时，user态下的内容会copy到内核态的socket的buffer中
  - 最后将内核态socket buffer的数据copy到网卡设备中传送
  
- zero拷贝
  - 请求kernel直接把disk的data传输给socket，而不是通过应用程序传输。Zero copy大大提高了应用程序的性能，减少不必要的内核缓冲区跟用户缓冲区间的拷贝，从而减少CPU的开销和减少了kernel和user模式的上下文切换，达到性能的提升 