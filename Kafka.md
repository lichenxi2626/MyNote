# Kafka

## 第一章 Kafka概述

### 1.1 消息队列（MQ）

- 点对点模式（队列中消息被拉取后删除）
  - 消息生产者将消息发送到队列（Message Queue）中，再由消费者从队列中提取并消费消息。
  - 消息被消费后将不再存储在队列中，因此1个消息只能被1个消费者消费。
- 发布/订阅模式（队列中消息被拉取后不删除）
  - 消息生产者将消息发送到Topic中，由多个消费者共同消费Topic中的消息。
  - Topic中的每个消息被消费者提取后不删除，因此每个消息可以被多个消费者依次消费。



### 1.2 Kafka定义

- Kafka是**基于发布/订阅模式的分布式消息队列**，应用于大数据的实时处理。
- 官网地址 http://kafka.apache.org



### 1.3 Kafka基础架构

<img src="Kafka.assets/wps2.png" alt="img" style="zoom:150%;" />

- Produce：消息生产者
- Consumer：消息消费者
- Topic：等价于1个消息队列，1个Topic分为多个Partition，存放在不同的Broker中
- Broker：物理上的主机节点，用于存放各个Topic的消息
- Partition：为了提高效率，每一个Topic中的消息又被分区存储在不同的Partition中
- Replica：Partition的副本，防止Partition所在节点故障引起的数据丢失
- Consumer Group：为了高效拉取Topic中每个Partition的数据，使用多个Consumer分别拉取Topic中每个Partition的消息，组成了Consumer Group



## 第二章 Kafka入门

### 2.1 Kafka 安装部署

1. kafka作为分布式框架，首先需要规划集群

   ```shell
   #在hadoop100,101,102三台主机节点上均安装部署zookeeper和kafka
   hadoop100	hadoop101	hadoop102
   zookeeper	zookeeper	zookeeper
   kafka		kafka		kafka
   ```

2. 官网下载kafka安装包，并上传到linux

   ```
   http://kafka.apache.org/downloads
   ```

3. 解压安装包到指定目录

   ```shell
   tar -zxvf kafka_2.11-2.4.1.tgz -C /opt/module/
   ```

4. 修改配置文件

   ```shell
   vim /opt/module/kafka_2.11-2.4.1/config/server.properties
   ```

   ```properties
   #broker的全局唯一编号，不能重复(必须修改)
   broker.id=0
   #删除topic功能使能
   delete.topic.enable=true
   #处理网络请求的线程数量
   num.network.threads=3
   #用来处理磁盘IO的现成数量
   num.io.threads=8
   #发送套接字的缓冲区大小
   socket.send.buffer.bytes=102400
   #接收套接字的缓冲区大小
   socket.receive.buffer.bytes=102400
   #请求套接字的缓冲区大小
   socket.request.max.bytes=104857600
   #kafka运行日志存放的路径(必须修改)
   log.dirs=/opt/module/kafka_2.11-2.4.1/logs
   #topic在当前broker上的分区个数
   num.partitions=1
   #用来恢复和清理data下数据的线程数量
   num.recovery.threads.per.data.dir=1
   #segment文件保留的最长时间，超时将被删除
   log.retention.hours=168
   #配置kafka节点数据在zk集群中的存储位置
   zookeeper.connect=hadoop100:2181,hadoop101:2181,hadoop102:2181/kafka
   ```

5. 在所有节点配置kafka环境变量

   ```shell
   #添加kafka家目录和bin目录到环境变量
   sudo vim /etc/profile.d/my_env.sh
   ```

   ```properties
   #JAVA_HOME
   JAVA_HOME=/opt/module/jdk1.8.0_212
   #HADOOP_HOME
   HADOOP_HOME=/opt/module/hadoop-3.1.3
   #HIVE_HOME
   HIVE_HOME=/opt/module/hive-3.1.2
   #FLUME_HOME
   FLUME_HOME=/opt/module/flume-1.9.0
   #KAFKA_HOME
   KAFKA_HOME=/opt/module/kafka_2.11-2.4.1
   PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin:$FLUME_HOME/bin:$KAFKA_HOME/bin
   export PATH JAVA_HOME HADOOP_HOME HIVE_HOME FLUME_HOME KAFKA_HOME
   ```

   ```shell
   source /etc/profile.d/my_env.sh
   ```

6. 分发kafka安装目录至所有节点

   ```shell
   xsync /opt/module/kafka_2.11-2.4.1
   ```

7. 修改各节点下配置文件中的broker.id

   ```shell
   vim /opt/module/kafka_2.11-2.4.1/config/server.properties
   ```

   ```properties
   #broker的全局唯一编号，不能重复(必须修改)
   broker.id=1
   ```

8. 在kafka集群所有节点启动kafka

   ```shell
   #先启动配置文件中配置的zookeeper集群
   #启动kafka
   kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
   #关闭kafka
   kafka-server-stop.sh
   ```

9. 编写kafka群起，群停脚本

   ```shell
   vim /home/atguigu/bin/kafkactl
   ```

   ```shell
   #!/bin/bash
   if [ $# -lt 1 ]
   	then
   		echo "No Args Input Error"
   	exit
   fi
   case $1 in
   "start")
   	for i in `cat $HADOOP_HOME/etc/hadoop/workers`
   	do
   		echo "==========start $i kafka=========="
   		ssh $i '$KAFKA_HOME/bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties'
   	done
   ;;
   "stop")
   	for i in `cat $HADOOP_HOME/etc/hadoop/workers`
   	do
   		echo "==========stop $i kafka=========="
   		ssh $i '$KAFKA_HOME/bin/kafka-server-stop.sh'
   	done
   ;;
   *)
   	echo "Input Args Error"
   ;;
   esac
   ```
   
   ```shell
   chmod 777 /home/atguigu/bin/kafkactl
   ```



### 2.2 Kafka 命令行操作

```shell
#1.查看指定服务器中的所有topic
kafka-topics.sh --list --bootstrap-server hadoop100:9092

#2.创建topic
kafka-topics.sh --create --bootstrap-server hadoop100:9092 --topic first --partitions 2 --replication-factor 2
#参数说明
#--topic 定义topic名
#--partitions 配置分区数
#--replication-factor 配置副本数(包括leader节点)

#3.删除topic
kafka-topics.sh --delete --zookeeper hadoop100:2181/kafka --topic first
#需要server.properties中设置delete.topic.enable=true否则只是标记删除

#4.查看topic详情
kafka-topics.sh --describe --bootstrap-server hadoop100:9092 --topic first

#5.修改分区数(仅支持向上修改,增加分区)
kafka-topics.sh --alter --zookeeper hadoop100:2181/kafka --topic first --partitions 3

#6.生产消息测试
kafka-console-producer.sh --broker-list hadoop100:9092 --topic first

#7.消费消息测试
#从命令执行后开始消费topic消息
kafka-console-consumer.sh --bootstrap-server hadoop100:9092 --topic first
kafka-console-consumer.sh --bootstrap-server hadoop100:9092 --topic first --from-beginning
```



## 第三章 Kafka深入

### 3.1 Kafka 工作流程及文件存储机制

- 工作流程

<img src="Kafka.assets/wps1.png" alt="img" style="zoom:150%;" />

- 存储机制
  - Topic
    - Topic是逻辑上的完整消息队列，创建后持久化的存储在kafka集群中，为了实现topic内消息的并行输入输出，将1个topic从物理上进行切分，在topic创建时通过参数配置，分为多个partiiton存储在不同kafka集群主机节点（broker）中。
    - Producer，Consumer和Topic是各自独立的模块。Producer和Consumer（Group）在启动时，通过配置的方式向指定的Topic发送或读取数据。
    - **数据默认在磁盘中保留7天，生产环境一般设置为3天。**
  - Partition
    - 1个Partition中包含3个文件
      - log文件：存储Producer生产的消息，新消息不断追加到文件末尾。当存储量达到配置值时会自动新建log文件，将消息存储到新的**分片**中。
      - index文件：索引，记录log文件中每条消息的位置（偏移地址）。
      - timeindex文件：时间戳索引，根据消息存入的时间作为每条消息的索引。
    - log，index，timeindex文件成组存在，即对于Partition中的每一个分片，都有1组**文件名相同**的log，index，timeindex文件。
    - 分片的文件名决定于log文件中第一条数据在整个Partition中的**索引偏移量**，因此消息在**每个Partition中是有序存储**的，而同一Topic下的不同Partition之间的消息则是无序的。



### 3.2 Kafka 生产者

#### 3.2.1 生产分区策略

- 分区目的
  1. 方便Topic在集群中扩展，适应任意大小的数据
  2. 实现Topic消息输入输出并发，提高效率
- **分区规则**
  1. Producer发送消息时，直接指定发送的分区号
  2. 未指定分区号时，对消息的key值hash后再对partition个数取模，结果为目标分区号
  3. 未指定分区且不含key值的消息在发送时，会装入1个batch（默认batch.size=16384），当batch装满时，随机生成1个整数，并对partition数量取模确定分区，将整个batch的消息发送到该分区，之后的batch轮询所有分区发送。



#### 3.2.2 数据可靠性

- 副本数据同步策略
  - 为减少数据冗余，kafka没有选择半数副本同步策略，而是选择了全副本同步策略。即当有新消息发送到partition时，leader节点收到所有副本节点的数据落盘响应后再向Producer发送ack应答确认。
- ack应答机制
  - acks是Producer Configs中的一项配置，他决定了服务端（Leader）在收到Producer发送的消息后的响应级别
    1. acks=0：Producer在发送数据后不向leader做确认，**直接视为数据成功发送**，提交offset。这种配置方式可能导致leader或其他副本节点的数据丢失。
    2. acks=1（默认）：Producer在发送数据后等待，**leader在数据落盘后会向Producer返回ack**，Producer收到后认为数据成功发送，提交offset。这种配置方式可能导致leader以外其他follwer节点的数据丢失。
    3. acks=-1或acks=all：Producer在发送数据后等待，**leader在会在所有副本节点数据落盘后再向Producer返回ack**，Producer收到后认为数据成功发送。这种配置方式完全避免了数据丢失问题，但是由于leader节点的数据在落盘完成后需要等待副本节点落盘和响应，这时如果leader节点故障，没有发出ack应答，Producer会认为数据没有成功发送，再次发送时就会造成数据重复的问题。
- ISR
  - ISR可以理解为存储同一个Partition的**健康的**Broker节点组。由于kafka采用了全副本同步策略，为了避免Partition的副本组中leader等待某一节点响应的时间过长，leader会动态的维护一个ISR，用于记录所有健康的副本节点，将长时间未响应的节点移出ISR。
  - ISR移除节点
    - 当leader接收到数据后，ISR中的所有节点会对leader接收的数据进行同步，若某一节点在规定时间内未发送同步请求，则ISR会将该节点直接移除，仅同步ISR中存在的节点。默认的最大相应时间可通过Broker Configs中的**replica.lag.time.max.ms**项配置，默认为10秒。
    - 同样的，当leader节点长时间无应答时，ISR也会移除leader并重新选举新的leader
  - 节点重新加入ISR
    - leader节点在每次向Producer返回ack的同时，会记录该ack响应的数据在log文件中的偏移量HW（High Watermark），HW所标识的位置即为所有副本节点都同步完成的可靠的数据。
    - 被移除ISR的副本节点在恢复工作后会继续同步leader中的数据，**当同步的数据偏移量大于HW时**，该节点会重新加入ISR。

![img](Kafka.assets/wps2-1588841100187.png)



#### 3.2.3 Exactly Once 数据去重（幂等性）

- 通过配置acks=-1可以解决数据的丢失问题，但会带来数据重复的可能。kafka在0.11版本引入了**幂等性**实现数据在发送阶段的去重。
- 原理：每条消息在发送时会附带一组 <PID,Partition,SeqNumber> 型的主键标识，用于唯一确定发送消息的Producer，目标Partition以及消息的Sequence Number（从0递增的数字序列）。当该主键标识相同时，Broker只会持久化1条。
- 配置方式：acks配置必须为-1或all，然后配置Producer Configs中的**enable.idempotence**项为true（默认false）。



### 3.3 Kafka 消费者

#### 3.3.1 消费方式

- **Consumer采用pull模式从broker中拉取数据，由Consumer决定消费信息的速率**。
- 若kafka中没有数据，Consumer就会陷入拉取空数据的循环，这个问题可以通过配置timeout参数解决。当Condumer拉取到空数据时，会等待timeout配置的时间后再进行下一次拉取。



#### 3.3.2 消费分区策略

- 当一个Group中的多个Consumer向一个Topic的多个Partition拉取数据时，Consumer和Partition的对应关系就涉及了分区的分配问题，kafka提供了三种不同的分配策略，**通过Consumer Configs中的partition.assignment.strategy配置项配置**。

  1. range（默认）

     ```shell
     #对于每一个Topic,根据Partition/Consumer的结果,将1个Topic的所有Partition依次分配到1个Group的所有Consumer
     #场景举例
     TopicA分区:A1 A2 A3 A4 A5
     TopicB分区:B1 B2 B3
     ConsumerGroup用户:C1 C2 C3
     #分配结果,容易出现数据倾斜
     C1: A1 A2 B1
     C2: A3 A4 B2
     C3: A5 B3
     ```

  2. roundrobin（轮询）

     ```shell
     #轮询分配:跨话题依次分配所有Topic的所有Partition到1个Group的所有Consumer
     #场景举例
     TopicA分区:A1 A2 A3 A4 A5
     TopicB分区:B1 B2 B3
     ConsumerGroup用户:C1 C2 C3
     #分配结果
     C1: A1 A4 B2
     C2: A2 A5 B3
     C3: A3 B1
     ```

  3. sticky（粘性）

     ```shell
     #轮询分配的优化版,区别为当topic分区因添加需要重新分配数据时,sticky的算法会最大程度的减少数据的迁移
     
     #场景举例:当roundrobin分配案例中TopicA和TopicB各增加1个Partition时
     #roundrobin的重新分配结果(全partition重新分配)
     C1: A1 A4 B1 B4
     C2: A2 A5 B2
     C3: A3 A6 B3
     #sticky的重新分配结果(在原有基础上继续轮询追加新partition)
     C1: A1 A4 B2 B4
     C2: A2 A5 B3
     C3: A3 B1 A6
     ```



#### 3.3.3 offset

- 为了实时记录每个Consumer消费的位置，**kafka内置了一个Topic：_consumer_offsets**，该Topic维护了每个Consumer消费数据的offset（位移）。
- **每次Consumer会提交当前消费位置的offset值+1**，以便下一次直接从提交值的offset位置拉取数据。



### 3.4 Kafka 读写策略

1. 顺序写入磁盘
   - kafka不断的将新的消息以追加的方式写入log文件存储，这种顺序写入效率远高于随机写入。
2. 使用Page Cache，将数据直接持久化到Page Cache中
   - 从内存中单独开辟一块区域，用于连续的临时存放小块文件，积攒一定数据量后再一并写入磁盘，减少磁盘头的移动时间。
   - 充分利用非JVM内存的空闲内存。
   - 当生产消费速度相当时，可直接在Page Cache中交换数据
   - Page Cache中的数据不会因进程结束而丢失，但会因为宕机丢失。
3. 零拷贝技术
   - 数据文件进入内核缓冲区后不经过用户态和Socket缓冲，直接传输进如网络协议传输阶段。



### 3.5 Zookeeper与Controller

- Kafka集群在启动时虽然不像hadoop集群或zookeeper集群存在中央节点，但依然会有一个broker节点被选举为Controller，**选举的方式为先到先得**。
- **Controller负责管理集群broker的上下线，所有Topic的partition副本分配以及leader的选举。**
- **Controller的运行时监听Zookeeper中维护的kafka节点信息**，因此kafka通过外置元数据实现了去中心化。



### 3.6 Kafka事务

- kafka从0.11版本开始引入事务支持，实现跨分区、跨会话的事务原子性。

#### 3.6.1 Producer事务

- Kafka维护了一个内部Topic，用于存储Transaction ID。
- Transaction ID与Producer的PID绑定，当Producer重启时可以通过Transaction ID获取原来的PID。
- Producer还可以通过Transaction ID向Transaction Coordinator获取上一次的任务状态，从而继续执行。

#### 3.6.2 Consumer事务

- Consumer在消费数据后会向kafka提交offset信息，用于记录消费的位置，但默认的提交的时机为间隔一定时间提交一次，与数据消费的生命周期不同。
- 为了实现Consumer的精准一次性消费，可以将Consumer消费数据的过程与提交offset做原子绑定。



## 第四章 Kafka API

### 4.1 Producer API

#### 4.1.1 Producer工作流程

- Producer发送消息到Topic的方式为**双线程异步发送**
  - **main线程**：将消息发送到RecordAccumulator
  - **sender线程**：将RecordAccumulator中的消息发送到Topic



#### 4.1.2 异步发送API

1. 启动IDEA，创建Maven工程，导入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.kafka</groupId>
           <artifactId>kafka-clients</artifactId>
           <version>2.4.1</version>
       </dependency>
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-slf4j-impl</artifactId>
           <version>2.12.0</version>
       </dependency>
   </dependencies>
   ```

2. 在/main/resources目录下新建文件log4j2.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Configuration status="error" strict="true" name="XMLConfig">
       <Appenders>
           <!-- 类型名为Console，名称为必须属性 -->
           <Appender type="Console" name="STDOUT">
               <!-- 布局为PatternLayout的方式，
               输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
               <Layout type="PatternLayout"
                       pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
           </Appender>
       </Appenders>
       <Loggers>
           <!-- 可加性为false -->
           <Logger name="test" level="info" additivity="false">
               <AppenderRef ref="STDOUT" />
           </Logger>
           <!-- root loggerConfig设置 -->
           <Root level="info">
               <AppenderRef ref="STDOUT" />
           </Root>
       </Loggers>
   </Configuration>
   ```

3. 编写代码

   ```java
   public class MyProducer {
       public static void main(String[] args) {
   
           //1.创建配置文件对象,添加配置项
           Properties properties = new Properties();
           properties.setProperty("key.serializer",
                   "org.apache.kafka.common.serialization.StringSerializer");
           properties.setProperty("value.serializer",
                   "org.apache.kafka.common.serialization.StringSerializer");
           properties.setProperty("acks", "all");//配置acks策略
           properties.setProperty("bootstrap.servers", "hadoop100:9092");
   
           //2.使用配置文件创建producer对象
           Producer<String, String> producer = new KafkaProducer<String, String>(properties);
   
           //3.发送数据
           for (int i = 0; i < 10; i++){
               //使用ProducerRecord类对象封装消息对象,再通过producer的send()发送消息
               producer.send(new ProducerRecord<String, String>(
                               "first", "message" + i, "这是第" + i + "条消息"),
                       new Callback() {
                           //回调函数,该方法在Producer收到ack相应后被调用
                           public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                               String topic = recordMetadata.topic();
                               int partition = recordMetadata.partition();
                               long offset = recordMetadata.offset();
                               System.out.println("话题" + topic + "的" + partition + "号分区的第" + offset + "条消息发送成功");
                           }
                       });
               System.out.println("第" + i + "条数据发送完成");
           }
           
           //4.关闭资源
           producer.close();
       }
   }
   ```

4. 主机节点启动zk和kafka，运行测试代码，查看日志信息



#### 4.1.3 同步发送API

- 对于异步发送API的代码，仅需**在循环中对send()返回的Future对象进行调用**，阻塞main线程就会因等待而阻塞，从而实现main线程发送一条消息，sender线程将该消息发送，main线程再继续发送的同步发送（串行）效果。

  ```java
  public class MyProducer {
      public static void main(String[] args) throws ExecutionException, InterruptedException {
  
          Properties properties = new Properties();
          properties.setProperty("key.serializer",
                  "org.apache.kafka.common.serialization.StringSerializer");
          properties.setProperty("value.serializer",
                  "org.apache.kafka.common.serialization.StringSerializer");
          properties.setProperty("acks", "all");
          properties.setProperty("bootstrap.servers", "hadoop100:9092");
  
          Producer<String, String> producer = new KafkaProducer<String, String>(properties);
  
          for (int i = 0; i < 10; i++){
              producer.send(new ProducerRecord<String, String>(
                              "first", "message" + i, "这是第" + i + "条消息"),
                      new Callback() {
                          public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                              String topic = recordMetadata.topic();
                              int partition = recordMetadata.partition();
                              long offset = recordMetadata.offset();
                              System.out.println("话题" + topic + "的" + partition + "号分区的第" + offset + "条消息发送成功");
                          }
                      }).get(); //对send方法返回的对象执行get()方法
              System.out.println("第" + i + "条数据发送完成");
          }
  
          producer.close();
      }
  }
  ```



### 4.2 Consumer API

- kafka针对不同需求提供了自动或手动提交offset的API配置，但不论采用哪种方式，都无法避免数据的**遗漏消费或重复消费**

#### 4.2.1 自动提交offset（默认）

1. 启动IDEA，创建Maven工程，导入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.kafka</groupId>
           <artifactId>kafka-clients</artifactId>
           <version>2.4.1</version>
       </dependency>
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-slf4j-impl</artifactId>
           <version>2.12.0</version>
       </dependency>
   </dependencies>
   ```

2. 在/main/resources目录下新建文件log4j2.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Configuration status="error" strict="true" name="XMLConfig">
       <Appenders>
           <!-- 类型名为Console，名称为必须属性 -->
           <Appender type="Console" name="STDOUT">
               <!-- 布局为PatternLayout的方式，
               输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
               <Layout type="PatternLayout"
                       pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
           </Appender>
       </Appenders>
       <Loggers>
           <!-- 可加性为false -->
           <Logger name="test" level="info" additivity="false">
               <AppenderRef ref="STDOUT" />
           </Logger>
           <!-- root loggerConfig设置 -->
           <Root level="info">
               <AppenderRef ref="STDOUT" />
           </Root>
       </Loggers>
   </Configuration>
   ```

3. 编写代码

   ```java
   public class MyConsumer {
       public static void main(String[] args) {
   
           //1.创建配置文件对象,添加配置项
           Properties properties = new Properties();
           properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
           properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer" );
           properties.setProperty("bootstrap.servers", "hadoop100:9092");
           properties.setProperty("group.id" , "test01");//指定所属group
           properties.setProperty("enable.auto.commit", "true");//自动提交offset(默认true)
           properties.setProperty("auto.commit.interval.ms", "1000");//自动提交offset的时间间隔(默认5000毫秒)
           properties.setProperty("auto.offset.reset", "earliest");//配置从头读取(默认latest)
   
           //2.使用配置文件创建consumer对象
           KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);
   
           //3.消费数据
           consumer.subscribe(Collections.singleton("second"));//指定订阅的所有topic的集合
           Duration duration = Duration.ofMillis(500);//配制拉取数据的间隔时间
           while (true){
               ConsumerRecords<String, String> poll = consumer.poll(duration);
               for (ConsumerRecord<String, String> record : poll) {
                   System.out.println(record);
               }
           }
   
       }
   }
   ```

4. 主机节点启动zk和kafka，运行测试代码，启动Producer，查看日志信息



#### 4.2.2 手动同步提交offset

- 通过调用commitSync()可以手动的**同步提交当次拉取的数据的最高偏移量（offset）**，同步提交会**阻塞线程**，直到提交成功再开始下一次数据的拉取。

  ```java
  public class MyConsumer {
      public static void main(String[] args) {
  
          Properties properties = new Properties();
          properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
          properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer" );
          properties.setProperty("bootstrap.servers", "hadoop100:9092");
          properties.setProperty("group.id" , "test01");
          properties.setProperty("enable.auto.commit", "false");//1.关闭自动提交offset(默认true)
          properties.setProperty("auto.offset.reset", "earliest");
  
          KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);
  
          consumer.subscribe(Collections.singleton("second"));
          Duration duration = Duration.ofMillis(500);
          while (true){
              ConsumerRecords<String, String> poll = consumer.poll(duration);
              for (ConsumerRecord<String, String> record : poll) {
                  System.out.println(record);
              }
              //2.同步提交offset
              consumer.commitSync();
          }
  
      }
  }
  ```



#### 4.2.3 手动异步提交offset

- 通过调用commitAsync()可以手动的**异步提交当次拉取的数据的最高偏移量（offset）**，异步提交**不阻塞线程**，不等待提交结果，直接开始下一次数据的拉取。

  ```java
  public class MyConsumer {
  
      public static void main(String[] args) {
  
          Properties properties = new Properties();
          properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
          properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer" );
          properties.setProperty("bootstrap.servers", "hadoop100:9092");
          properties.setProperty("group.id" , "test01");
          properties.setProperty("enable.auto.commit", "false");//关闭自动提交offset(默认true)
          properties.setProperty("auto.offset.reset", "earliest");
  
          KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);
  
          consumer.subscribe(Collections.singleton("second"));
          Duration duration = Duration.ofMillis(500);
          while (true){
              ConsumerRecords<String, String> poll = consumer.poll(duration);
              for (ConsumerRecord<String, String> record : poll) {
                  System.out.println(record);
              }
  
              //2.异步提交offset
              consumer.commitAsync(
                  new OffsetCommitCallback() {
                      @Override
                      public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                          offsets.forEach(
                              (t, o) -> {
                                  System.out.println("分区：" + t + "\nOffset：" + o);
                              }
                          );
                      }
                  }
              );
          }
  
      }
  }
  ```



## 第五章 Kafka Eagle（KE监控）

1. 下载并上传Kafka Eagle安装包到本地

2. 解压安装包，修改文件名

   ```shell
   tar -zxvf kafka-eagle-bin-1.4.5.tar.gz
   cd kafka-eagle-bin-1.4.5
   tar -zxvf kafka-eagle-web-1.4.5-bin.tar.gz -C /opt/module
   mv kafka-eagle-web-1.4.5 eagle-1.4.5
   ```

3. 配置环境变量

   ```shell
   sudo vi /etc/profile.d/my_env.sh
   ```

   ```properties
   #JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   #HADOOP_HOME
   export HADOOP_HOME=/opt/module/hadoop-3.1.3
   #HIVE_HOME
   export HIVE_HOME=/opt/module/hive-3.1.2
   #FLUME_HOME
   export FLUME_HOME=/opt/module/flume-1.9.0
   #KAFKA_HOME
   export KAFKA_HOME=/opt/module/kafka_2.11-2.4.1
   #KE_HOME
   export KE_HOME=/opt/module/eagle-1.4.5
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin:$FLUME_HOME/bin:$KAFKA_HOME/bin:$KE_HOME/bin
   ```

4. 赋予启动文件执行权限

   ```shell
   chmod 777 bin/ke.sh
   ```

5. 修改配置文件

   ```shell
   vim conf/system-config.properties
   ```

   ```properties
   #修改以下配置项
   #为KE监控的kafka集群命名,支持多个集群
   kafka.eagle.zk.cluster.alias=cluster1
   
   #配置集群cluster1的在zk中维护的数据位置
   cluster1.zk.list=hadoop100:2181,hadoop101:2181,hadoop102:2181/kafka
   
   #配置集群cluster1的offset的存储位置
   cluster1.kafka.eagle.offset.storage=kafka
   
   kafka.eagle.metrics.charts=true
   
   kafka.eagle.sql.fix.error=false
   
   #打开注释,配置mysql服务
   kafka.eagle.driver=com.mysql.jdbc.Driver
   kafka.eagle.url=jdbc:mysql://hadoop100:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
   #配置mysql的用户密码
   kafka.eagle.username=root
   kafka.eagle.password=123456
   ```

6. 修改kafka启动命令

   ```shell
   vi $KAFKA_HOME/bin/kafka-server-start.sh
   ```

   ```shell
   #修改以下内容
   if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
       export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
   fi
   
   #修改为
   if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
       export KAFKA_HEAP_OPTS="-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"
       export JMX_PORT="9999"
       #export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
   fi
   ```

   ```shell
   #同步到所有kafka集群节点
   xsync kafka-server-start.sh
   ```

7. 先后启动zk，kafka，kafka eagle

   ```shell
   #启动ke
   ke.sh start
   #关闭ke
   ke.sh stop
   ```

8. 使用控制台显示的用户密码访问web端 http://hadoop100:8048/ke