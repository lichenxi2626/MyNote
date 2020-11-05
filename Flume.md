# Flume

## 第一章 Flume概述

### 1.1 Flume定义

- FLume是Cloudera提供的一个高可用、高可靠的，分布式海量日志采集、集合、传输系统。
- Flume基于流式框架。
- 官网：http://flume.apache.org/

### 1.2 Flume基础架构

![image-20200504182238350](flume.assets/image-20200504182238350.png)

- Agent
  - 1个agent即为1个JVM进程，该进程以事件形式将数据从数据源送至目标位置
  - agent由source，channel，slnk三部分组成
- Source
  - source负责接收各种类型，各种格式的数据到flume组件
- Sink
  - sink不断将channel中的事件批量写入到目标位置，并移除channel中已写出的事件
- Channel
  - channel是source和sink间的缓冲区
  - channel是线程安全的，可同时处理多个source的写入和sink的读取操作
  - Flume自带的两种channel：**memory channel（数据存入内存），file channel（数据存入磁盘）**
- Event
  - event（事件）是flume数据传输的基本单元。
  - 每个event由Header（kv对）和Body（byte array）两部分组成



## 第二章 Flume入门

### 2.1 Flume的安装部署

1. 官网下载Flume

   ```shell
   #下载地址
   archive.apache.org/dist/flume/
   #查看文档
   flume.apache.org/FlumeUserGuide.html
   ```

2. 上传安装包到linux

3. 解压到安装目录

   ```shell
   tar -zxvf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/
   ```

4. 修改安装目录名

   ```shell
   mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume-1.9.0
   ```

5. 删除与hadoop冲突的依赖

   ```shell
   rm /opt/module/flume-1.9.0/lib/guava-11.0.2.jar
   ```



### 2.2 Flume入门案例

#### 2.2.1 监听端口数据，输出到控制台（netcat source）

1. 安装 netcat 包

   ```shell
   sudo yum install -y nc
   ```

2. 查看监听端口是否被占用

   ```shell
   sudo netstat -tunlp | grep 44444
   ```

3. 编写配置文件

   ```shell
   mkdir -p /opt/module/flume-1.9.0/job
   vi /opt/module/flume-1.9.0/job/flume-netcat-logger.conf
   ```

   ```properties
   #命名agent,整个配置文件使用1个agent名,这里为a1
   #为agent(a1)的source,channel,sink分别命名
   a1.sources = r1
   a1.sinks = k1
   a1.channels = c1
   
   #配置source(r1),输入源类型为netcat,绑定r1需要监听的IP及端口号
   a1.sources.r1.type = netcat
   a1.sources.r1.bind = localhost
   a1.sources.r1.port = 44444
   
   #配置sink(k1),写出类型为logger(控制台日志)
   a1.sinks.k1.type = logger
   
   #配置channel(c1),类型为memory(内存型),c1的容量为1000个event,每收集100个event提交1次事务
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   #配置channel连接,r1连接c1,k1连接c1
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
   ```

4. 在 flume 家目录下启动 flume

   ```shell
   bin/flume-ng agent -c conf/ -n a1 -f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
   
   #配置参数说明
   # -c 默认配置文件存储目录
   # -n 本次使用的agent名
   # -f 本次使用的配置文件
   # -Dflume.root.logger=INFO,console 本次运行时修改flume.root.logger参数值为INFO(控制台查看信息)
   ```

5. 使用 netcat 连接端口并发送内容

   ```shell
   nc localhost 44444
   helloworld
   ```

6. 在flume监听端查看信息变化



#### 2.2.2 监控文件追加内容，上传到HDFS（exec source）

1. flume 输出数据到 hdfs 需要依赖 hadoop 中的相关 jar 包，因此需要检查环境变量

   ```shell
   echo $HADOOP_HOME
   echo $JAVA_HOME
   echo $PATH
   ```

2. 编写配置文件

   ```shell
   mkdir -p /opt/module/flume-1.9.0/job
   vi /opt/module/flume-1.9.0/job/flume-exer-hdfs.conf
   ```

   ```properties
   #命名agent,整个配置文件使用1个agent名,这里为a2
   #为agent(a2)的source,channel,sink分别命名
   a2.sources = r2
   a2.sinks = k2
   a2.channels = c2
   
   #配置source(r2),输入源类型为exer(执行linux命令读取数据),此处命令为监听hive日志文件数据
   a2.sources.r2.type = exec
   a2.sources.r2.command = tail -F /opt/module/hive-3.1.2/logs/hive.log
   a2.sources.r2.shell = /bin/bash -c
   
   #配置sink(k2),写出类型为hdfs(上传到hdfs集群),并配置集群上的存储路径
   a2.sinks.k2.type = hdfs
   a2.sinks.k2.hdfs.path = hdfs://hadoop100:9820/flume/%Y%m%d/%H
   #上传文件的前缀
   a2.sinks.k2.hdfs.filePrefix = logs-
   #是否对时间戳取整
   a2.sinks.k2.hdfs.round = true
   #多少时间单位创建一个新的文件夹
   a2.sinks.k2.hdfs.roundValue = 1
   #重新定义时间单位
   a2.sinks.k2.hdfs.roundUnit = hour
   #是否使用本地时间戳
   a2.sinks.k2.hdfs.useLocalTimeStamp = true
   #积攒多少个Event才flush到HDFS一次
   a2.sinks.k2.hdfs.batchSize = 100
   #设置文件类型，可支持压缩
   a2.sinks.k2.hdfs.fileType = DataStream
   #多久生成一个新的文件(秒)
   a2.sinks.k2.hdfs.rollInterval = 60
   #设置每个文件的滚动大小
   a2.sinks.k2.hdfs.rollSize = 134217700
   #文件的滚动与Event数量无关
   a2.sinks.k2.hdfs.rollCount = 0
   
   #配置channel(c2),类型为memory(内存型),c2的容量为1000个event,每收集100个event提交1次事务
   a2.channels.c2.type = memory
   a2.channels.c2.capacity = 1000
   a2.channels.c2.transactionCapacity = 100
   
   #配置channel连接,r2连接c2,k2连接c2
   a2.sources.r2.channels = c2
   a2.sinks.k2.channel = c2
   ```

3. 在 flume 家目录下启动 flume

   ```shell
   bin/flume-ng agent -c conf/ -n a2 -f job/flume-exer-hdfs.conf
   ```

4. 启动 hdfs 和 hive，查看 hdfs 中的文件变化



#### 2.2.3 监控目录下新文件，上传到HDFS（spooldir source）

1. 编写配置文件

   ```shell
   mkdir -p /opt/module/flume-1.9.0/job
   vi /opt/module/flume-1.9.0/job/flume-spooldir-hdfs.conf
   ```

   ```properties
   #命名agent,整个配置文件使用1个agent名,这里为a3
   #为agent(a3)的source,channel,sink分别命名
   a3.sources = r3
   a3.sinks = k3
   a3.channels = c3
   
   #配置source(r3),输入源类型为spooldir,具体功能为监听目录下新增文件后会进行读取
   a3.sources.r3.type = spooldir
   a3.sources.r3.spoolDir = /opt/module/flume-1.9.0/datas
   #监听目录下的新增文件在读取完成后追加的后缀
   a3.sources.r3.fileSuffix = .COMPLETED
   a3.sources.r3.fileHeader = true
   #忽略所有以.tmp结尾的文件,不上传(使用正则表达式)
   a3.sources.r3.ignorePattern = ([^ ]*\.tmp)
   
   #配置sink(k3),写出类型为hdfs(上传到hdfs集群),并配置集群上的存储路径
   a3.sinks.k3.type = hdfs
   a3.sinks.k3.hdfs.path = hdfs://hadoop100:9820/flume/datas/%Y%m%d/%H
   #上传文件的前缀
   a3.sinks.k3.hdfs.filePrefix = datas-
   #是否对时间戳取整
   a3.sinks.k3.hdfs.round = true
   #多少时间单位创建一个新的文件夹
   a3.sinks.k3.hdfs.roundValue = 1
   #重新定义时间单位
   a3.sinks.k3.hdfs.roundUnit = hour
   #是否使用本地时间戳
   a3.sinks.k3.hdfs.useLocalTimeStamp = true
   #积攒多少个Event才flush到HDFS一次
   a3.sinks.k3.hdfs.batchSize = 100
   #设置文件类型，可支持压缩
   a3.sinks.k3.hdfs.fileType = DataStream
   #多久生成一个新的文件(秒)
   a3.sinks.k3.hdfs.rollInterval = 60
   #设置每个文件的滚动大小大概是128M
   a3.sinks.k3.hdfs.rollSize = 134217700
   #文件的滚动与Event数量无关
   a3.sinks.k3.hdfs.rollCount = 0
   
   #配置channel(c3),类型为memory(内存型),c3的容量为1000个event,每收集100个event提交1次事务
   a3.channels.c3.type = memory
   a3.channels.c3.capacity = 1000
   a3.channels.c3.transactionCapacity = 100
   
   #配置channel连接,r3连接c3,k3连接c3
   a3.sources.r3.channels = c3
   a3.sinks.k3.channel = c3
   ```

2. 在 flume 家目录下启动 flume

   ```shell
   bin/flume-ng agent -c conf/ -n a3 -f job/flume-spooldir-hdfs.conf
   ```

3. 启动 hdfs 集群

4. 创建监听的目录并向目录中添加文件

   ```shell
   mkdir -p /opt/module/flume-1.9.0/datas
   touch /opt/module/flume-1.9.0/datas/test.txt
   touch /opt/module/flume-1.9.0/datas/test.tmp
   ```

5. 查看 hdfs 集群中的文件变化



#### 2.2.4 监控目录下追加文件，上传到HDFS（taildir source）

1. 编写配置文件

   ```shell
   mkdir -p /opt/module/flume-1.9.0/job
   vi /opt/module/flume-1.9.0/job/flume-taildir-hdfs.conf
   ```

   ```properties
   #命名agent,整个配置文件使用1个agent名,这里为a4
   #为agent(a4)的source,channel,sink分别命名
   a4.sources = r4
   a4.sinks = k4
   a4.channels = c4
   
   #配置source(r4),输入源类型为taildir,具体功能为监听目录下文件有更新时读取更新后的数据
   a4.sources.r4.type = taildir
   #配置json文件存储目录,json文件记录目录中所有文件的最近更新位置,实现断点续传
   a4.sources.r4.positionFile = /opt/module/flume-1.9.0/datas/tail_dir.json
   #配置filegroup,实现对多个目录、多种文件的共同监听
   a4.sources.r4.filegroups = f1 f2
   a4.sources.r4.filegroups.f1 = /opt/module/flume-1.9.0/datas/.*file.*
   a4.sources.r4.filegroups.f2 = /opt/module/flume-1.9.0/datas/.*log.*
   
   #配置sink(k4),写出类型为hdfs(上传到hdfs集群),并配置集群上的存储路径
   a4.sinks.k4.type = hdfs
   a4.sinks.k4.hdfs.path = hdfs://hadoop100:9820/flume/datas2/%Y%m%d/%H
   #上传文件的前缀
   a4.sinks.k4.hdfs.filePrefix = datas2-
   #是否对时间戳取整
   a4.sinks.k4.hdfs.round = true
   #多少时间单位创建一个新的文件夹
   a4.sinks.k4.hdfs.roundValue = 1
   #重新定义时间单位
   a4.sinks.k4.hdfs.roundUnit = hour
   #是否使用本地时间戳
   a4.sinks.k4.hdfs.useLocalTimeStamp = true
   #积攒多少个Event才flush到HDFS一次
   a4.sinks.k4.hdfs.batchSize = 100
   #设置文件类型，可支持压缩
   a4.sinks.k4.hdfs.fileType = DataStream
   #多久生成一个新的文件(秒)
   a4.sinks.k4.hdfs.rollInterval = 60
   #设置每个文件的滚动大小大概是128M
   a4.sinks.k4.hdfs.rollSize = 134217700
   #文件的滚动与Event数量无关
   a4.sinks.k4.hdfs.rollCount = 0
   
   #配置channel(c4),类型为memory(内存型),c4的容量为1000个event,每收集100个event提交1次事务
   a4.channels.c4.type = memory
   a4.channels.c4.capacity = 1000
   a4.channels.c4.transactionCapacity = 100
   
   #配置channel连接,r4连接c4,k4连接c4
   a4.sources.r4.channels = c4
   a4.sinks.k4.channel = c4
   ```

2. 在 flume 家目录下启动 flume

   ```shell
   bin/flume-ng agent -c /conf -n a4 -f job/flume-taildir-hdfs.conf
   ```

3. 启动 hdfs 集群

4. 创建监听的目录，并向目录中添加文件，再向文件中追加数据

   ```shell
   mkdir -p /opt/module/flume-1.9.0/datas
   touch /opt/module/flume-1.9.0/datas/file.txt
   ls >> /opt/module/flume-1.9.0/datas/file.txt
   ```

5. 查看 hdfs 集群中的文件变化



## 第三章 Flume进阶

### 3.1 Flume事务

![image-20200504220105824](flume.assets/image-20200504220105824.png)

- Flume事务流程包括Put和Take两个事务流程
- Put事务
  1. doPut：将批数据先写入临时缓冲区putList
  2. doCommit：检查channel内存队列的剩余空间是否足够合并putList中的数据，若足够则写入channel
  3. doRollback：若channel内存队列的剩余空间不足，则回滚数据，间隔一段时间后再次执行doCommit
- Take事务
  1. doTake：将channel内存队列的数据的引用提取到缓冲区takeList
  2. doCommit：写出takeList中的数据，若成功则清空takeList并清除channel中的相应数据
  3. doRollback：若数据写出失败，则回滚数据清空takeList，间隔一段时间后再次执行doTake



### 3.2 Flume Agent内部原理

<img src="flume.assets/wps2.png" alt="img" style="zoom: 150%;" />

- Channel Selector的两种类型
  - Replicating：将source获取的event发往所有Channel
  - Mutiplexing：将source获取的event根据配置文件发往指定Channel
- SinkProcessor的三种类型
  - DefaultSinkProcessor：默认类型，对单个Sink使用
  - LoadBalancingSinkProcessor：对SinkGroup使用，实现负载均衡
  - FailoverSinkProcessor：对SinkGroup使用，实现故障转移
- **Source与Channel，Channel与Sink的对应关系**
  - 1个Souce可以对应多个Channel，分发多个相同的event
  - 1个Channel可以对应多个Sink（SinkGroup），但Channle中的1个event只能被1个Sink（根据配置的优先级）写出。



### 3.3 Flume拓扑结构及案例

#### 3.3.1 串联

<img src="flume.assets/image-20200504222716583.png" alt="image-20200504222716583" style="zoom:150%;" />

- 将多个flume串联运行，当flume数量过多时，传输效率较低，同时单个节点的故障会直接影响整个系统，健壮性差。



#### 3.3.2 复制和多路复用

- flume支持1个事务流向多个目的地，方式是将相同的数据复制发送到多个channel中，再使用不同的sink将数据发送到不同的目标位置。

- 案例

  1. 需求

     ```shell
     flume1 监控目标文件
     |--flume2 接收flume1的数据存储到hdfs
     |--flume3 接收flume1的数据输出到Local FileSystem
     
      #flume1
      source ------> channel1 ------> sink1
      |--> channel2 ------> sink2
      #flume2
      source ---> channel ---> sink
      #flume3
      source ---> channel ---> sink
     ```
  
  
  2. 编写flume1的配置文件
  
     ```shell
     mkdir -p /opt/module/flume-1.9.0/job/g1
     vim /opt/module/flume-1.9.0/job/g1/a1.conf
     ```
     
     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a1
     #为agent(a1)的source,channel,sink分别命名
     a1.sources = r1
     a1.sinks = k1 k2
     a1.channels = c1 c2
     
     #配置channel selector的类型为replicating,复制数据到所有Channel
     a1.sources.r1.selector.type = replicating
     #配置source(r1),输入源类型为exec(执行linux命令读取数据),此处命令为监听hive日志文件数据
     a1.sources.r1.type = exec
     a1.sources.r1.command = tail -F /opt/module/hive-3.1.2/logs/hive.log
     a1.sources.r1.shell = /bin/bash -c
     
     #配置sink(k1,k2),写出类型为avro(flume提供的序列化方式),并配置数据交互的IP及端口号
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop100
     a1.sinks.k1.port = 4141
     a1.sinks.k2.type = avro
     a1.sinks.k2.hostname = hadoop100
     a1.sinks.k2.port = 4142
     
     #配置channel(c1,c2),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 1000
     a1.channels.c1.transactionCapacity = 100
     a1.channels.c2.type = memory
     a1.channels.c2.capacity = 1000
     a1.channels.c2.transactionCapacity = 100
     
     #配置channel连接,r1连接c1、c2,k1连接c1,k2连接c2
     a1.sources.r1.channels = c1 c2
     a1.sinks.k1.channel = c1
     a1.sinks.k2.channel = c2
     ```
  
  3. 编写flume2的配置文件
  
     ```shell
     vim /opt/module/flume-1.9.0/job/g1/a2.conf
     ```
  
     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a2
     #为agent(a2)的source,channel,sink分别命名
     a2.sources = r1
     a2.sinks = k1
     a2.channels = c1
     
     #配置source(r1),输入源类型为avro(flume提供的序列化方式),配置内容需要与flime1中sink的配置一致
     a2.sources.r1.type = avro
     a2.sources.r1.bind = hadoop100
     a2.sources.r1.port = 4141
     
     #配置sink(k4),写出类型为hdfs(上传到hdfs集群),并配置集群上的存储路径
     a2.sinks.k1.type = hdfs
     a2.sinks.k1.hdfs.path = hdfs://hadoop100:9820/flume/g1/%Y%m%d/%H
     #上传文件的前缀
     a2.sinks.k1.hdfs.filePrefix = g1-
     #是否对时间戳取整
     a2.sinks.k1.hdfs.round = true
     #多少时间单位创建一个新的文件夹
     a2.sinks.k1.hdfs.roundValue = 1
     #重新定义时间单位
     a2.sinks.k1.hdfs.roundUnit = hour
     #是否使用本地时间戳
     a2.sinks.k1.hdfs.useLocalTimeStamp = true
     #积攒多少个Event才flush到HDFS一次
     a2.sinks.k1.hdfs.batchSize = 100
     #设置文件类型，可支持压缩
     a2.sinks.k1.hdfs.fileType = DataStream
     #多久生成一个新的文件
     a2.sinks.k1.hdfs.rollInterval = 600
     #设置每个文件的滚动大小大概是128M
     a2.sinks.k1.hdfs.rollSize = 134217700
     #文件的滚动与Event数量无关
     a2.sinks.k1.hdfs.rollCount = 0
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a2.channels.c1.type = memory
     a2.channels.c1.capacity = 1000
  a2.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a2.sources.r1.channels = c1
     a2.sinks.k1.channel = c1
     ```
  
  4. 编写flume3的配置文件
  
     ```shell
     vim /opt/module/flume-1.9.0/job/g1/a3.conf
     ```
     
     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a3
     #为agent(a3)的source,channel,sink分别命名
     a3.sources = r1
     a3.sinks = k1
     a3.channels = c1
     
     #配置source(r1),输入源类型为avro(flume提供的序列化方式),配置内容需要与flime1中sink的配置一致
     a3.sources.r1.type = avro
     a3.sources.r1.bind = hadoop100
     a3.sources.r1.port = 4142
     
     #配置sink(k4),写出类型为file_roll(写到本地),并配置本地的存储路径
     a3.sinks.k1.type = file_roll
     a3.sinks.k1.sink.directory = /opt/module/flume1.9.0/datas/g1
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a3.channels.c1.type = memory
     a3.channels.c1.capacity = 1000
     a3.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a3.sources.r1.channels = c1
     a3.sinks.k1.channel = c1
     ```
  
  5. 依次启动3个flume
  
     ```shell
  #先启动flume2和flume3
     bin/flume-ng agent -c conf/ -n a2 -f job/g1/a2.conf
  bin/flume-ng agent -c conf/ -n a3 -f job/g1/a3.conf
     #最后启动flume1
     bin/flume-ng agent -c conf/ -n a1 -f job/g1/a1.conf
     ```
  
  6. 启动 hdfs 和 hive
  
  7. 查看集群和本地的文件变化



#### 3.3.3 负载均衡和故障转移

- flume支持多个sink组成的sink组，sink组根据配置的SinkProcessor实现**负载均衡**或**错误恢复**的功能

- 案例

  1. 需求

     ```shell
     flume1读取netcat输入的数据
     |--flume2优先接收flume1的数据
     |--flume3当flume2故障时,取代flume2接收flume1的数据
     
     #flume1
     source ---> channel ------> sink1
                            |--> sink2
     #flume2
     source ---> channel ---> sink
     #flume3
     source ---> channel ---> sink
     ```

  2. 编写flume1的配置文件

     ```shell
     mkdir -p /opt/module/flume-1.9.0/job/g2
     vim /opt/module/flume-1.9.0/job/g2/a1.conf
     ```

     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a1
     #为agent(a1)的source,channel,sink分别命名
     a1.sources = r1
     a1.channels = c1
     a1.sinks = k1 k2
     #故障转移的功能实现需要创建sink组,并为组中分配多个sink
     a1.sinkgroups = g1
     a1.sinkgroups.g1.sinks = k1 k2
     
     #配置source(r1),输入源类型为netcat,绑定r1需要监听的IP及端口号
     a1.sources.r1.type = netcat
     a1.sources.r1.bind = localhost
     a1.sources.r1.port = 44444
     
     #配置sink(k1,k2),写出类型为avro(flume提供的序列化方式),并配置数据交互的IP及端口号
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop100
     a1.sinks.k1.port = 4141
     a1.sinks.k2.type = avro
     a1.sinks.k2.hostname = hadoop100
     a1.sinks.k2.port = 4142
     #配置sink组的processor类型为failover(故障转移),并配置优先级(10>5)
     a1.sinkgroups.g1.processor.type = failover
     a1.sinkgroups.g1.processor.priority.k1 = 10
     a1.sinkgroups.g1.processor.priority.k2 = 5
     a1.sinkgroups.g1.processor.maxpenalty = 10000
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 1000
     a1.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1、k2连接c1
     a1.sources.r1.channels = c1
     a1.sinks.k1.channel = c1
     a1.sinks.k2.channel = c1
     ```

  3. 编写flume2的配置文件

     ```shell
     vim /opt/module/flume-1.9.0/job/g2/a2.conf
     ```

     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a2
     #为agent(a2)的source,channel,sink分别命名
     a2.sources = r1
     a2.sinks = k1
     a2.channels = c1
     
     #配置source(r1),输入源类型为avro(flume提供的序列化方式),配置内容需要与flime1中sink的配置一致
     a2.sources.r1.type = avro
     a2.sources.r1.bind = hadoop100
     a2.sources.r1.port = 4141
     
     #配置sink(k1),写出类型为logger(控制台日志)
     a2.sinks.k1.type = logger
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a2.channels.c1.type = memory
     a2.channels.c1.capacity = 1000
     a2.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a2.sources.r1.channels = c1
     a2.sinks.k1.channel = c1
     ```

  4. 编写flume3的配置文件

     ```shell
     vim /opt/module/flume-1.9.0/job/g2/a3.conf
     ```

     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a3
     #为agent(a2)的source,channel,sink分别命名
     a2.sources = r1
     a2.sinks = k1
     a2.channels = c1
     
     #配置source(r1),输入源类型为avro(flume提供的序列化方式),配置内容需要与flime1中sink的配置一致
     a2.sources.r1.type = avro
     a2.sources.r1.bind = hadoop100
     a2.sources.r1.port = 4142
     
     #配置sink(k1),写出类型为logger(控制台日志)
     a2.sinks.k1.type = logger
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a2.channels.c1.type = memory
     a2.channels.c1.capacity = 1000
     a2.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a2.sources.r1.channels = c1
     a2.sinks.k1.channel = c1
     ```

  5. 依次启动3个flume

     ```shell
     #先启动flume2和flume3
     bin/flume-ng agent -c conf/ -n a2 -f job/g2/a2.conf -Dflume.root.logger=INFO,console
     bin/flume-ng agent -c conf/ -n a3 -f job/g2/a3.conf -Dflume.root.logger=INFO,console
     #最后启动flume1
     bin/flume-ng agent -c conf/ -n a1 -f job/g2/a1.conf -Dflume.root.logger=INFO,console
     ```

  6. 使用netcat连接本地，并输入数据

     ```shell
     nc localhost 44444
     hello
     world
     ```

  7. 查看flume2和flume3窗口的信息（全部数据由flume2窗口输出）

  8. 结束flume2程序，继续在netcat输入数据

  9. 查看flume3窗口的信息（全部数据由flume3窗口输出）



#### 3.3.4 聚合

- 在多个服务器节点分别部署flume采集日志，再集中收集到1个flume节点，最后上传到hdfs、hive、habase等终端。

- 案例

  1. 需求

     ```shell
     flume1读取hadoop100节点的hive日志文件数据
     flume2读取hadoop101节点的netcat输入数据
     |--flume3收集flume1和flume2读取的数据
     
     #flume1
     source ---> channel ---> sink
     #flume2
     source ---> channel ---> sink
     #flume3
     source ---> channel ---> sink
     ```

  2. 编写flume1的配置文件

     ```shell
     mkdir -p /opt/module/flume-1.9.0/job/g3
     vim /opt/module/flume-1.9.0/job/g3/a1.conf
     ```

     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a1
     #为agent(a1)的source,channel,sink分别命名
     a1.sources = r1
     a1.sinks = k1
     a1.channels = c1
     
     #配置source(r2),输入源类型为exer(执行linux命令读取数据),此处命令为监听hive日志文件数据
     a1.sources.r1.type = exec
     a1.sources.r1.command = tail -F /opt/module/hive-3.1.2/logs/hive.log
     a1.sources.r1.shell = /bin/bash -c
     
     #配置sink(k1),写出类型为avro(flume提供的序列化方式),并配置数据交互的IP及端口号,配置内容需要与flime3中sink的配置一致
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop102
     a1.sinks.k1.port = 4141
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 1000
     a1.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a1.sources.r1.channels = c1
     a1.sinks.k1.channel = c1
     ```

  3. 编写flume2的配置文件

     ```shell
     vim /opt/module/flume-1.9.0/job/g3/a2.conf
     ```

     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a2
     #为agent(a2)的source,channel,sink分别命名
     a2.sources = r1
     a2.sinks = k1
     a2.channels = c1
     
     #配置source(r1),输入源类型为netcat,绑定r1需要监听的IP及端口号
     a2.sources.r1.type = netcat
     a2.sources.r1.bind = hadoop101
     a2.sources.r1.port = 44444
     
     #配置sink(k1),写出类型为avro(flume提供的序列化方式),并配置数据交互的IP及端口号,配置内容需要与flime3中sink的配置一致
     a2.sinks.k1.type = avro
     a2.sinks.k1.hostname = hadoop102
     a2.sinks.k1.port = 4141
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a2.channels.c1.type = memory
     a2.channels.c1.capacity = 1000
     a2.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a2.sources.r1.channels = c1
     a2.sinks.k1.channel = c1
     ```

  4. 编写flume3的配置文件

     ```shell
     vim /opt/module/flume-1.9.0/job/g3/a3.conf
     ```

     ```properties
     #命名agent,整个配置文件使用1个agent名,这里为a3
     #为agent(a3)的source,channel,sink分别命名
     a3.sources = r1
     a3.sinks = k1
     a3.channels = c1
     
     #配置source(r1),输入源类型为avro(flume提供的序列化方式)
     a3.sources.r1.type = avro
     a3.sources.r1.bind = hadoop102
     a3.sources.r1.port = 4141
     
     #配置sink(k1),写出类型为logger(控制台日志)
     a3.sinks.k1.type = logger
     
     #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
     a3.channels.c1.type = memory
     a3.channels.c1.capacity = 1000
     a3.channels.c1.transactionCapacity = 100
     
     #配置channel连接,r1连接c1,k1连接c1
     a3.sources.r1.channels = c1
     a3.sinks.k1.channel = c1
     ```

  5. 将flume（未安装flume）或配置文件（已安装flume）分发到其他节点

     ```shell
     xsync /opt/module/flume-1.9.0
     ```

  6. 先后启动3个flume

     ```shell
     #首先在hadoop102启动flume3
     bin/flume-ng agent -c conf/ -n a2 -f job/g3/a3.conf -Dflume.root.logger=INFO,console
     #然后在hadoop100和hadoop101分别启动flume1和flume2
     bin/flume-ng agent -c conf/ -n a2 -f job/g3/a1.conf -Dflume.root.logger=INFO,console
     bin/flume-ng agent -c conf/ -n a2 -f job/g3/a2.conf -Dflume.root.logger=INFO,console
     ```

  7. 在hadoop100启动 hdfs 和 hive，查看hadoop103中的控制台信息变化

  8. 在hadoop101使用 netcat 输入数据，查看hadoop103中的控制台信息变化

     ```shell
     nc hadoop101 44444
     helloworld
     ```



### 3.4 自定义Interceptor

- 需求：对于一个flume程序，读取的数据可能包含多种类型，为了将不同类型的数据发送到不同的目标位置，就需要使用Multiplexing类型的Channel Selector，将不同类型的数据发给不同的Channel，最终发送到不同的位置。

- 原理：在source收集数据的过程中，通过自定义Interceptor（拦截器），对每个event的header添加一个自定义的kv对用于Channel Selector识别并分配该event的去向

- 案例

  1. 启动idea，创建Maven工程

  2. 在 pom.xml 中添加 flume 依赖

     ```xml
     <dependencies>
         <dependency>
             <groupId>org.apache.flume</groupId>
             <artifactId>flume-ng-core</artifactId>
             <version>1.9.0</version>
             <scope>provided</scope>
         </dependency>
     </dependencies>
     ```
3. 自定义类实现Interceptor接口
  
     ```java
     //需求:根据输入数据的首字符,将不同的event分配给不同的channel
     public class MyInterceptor implements Interceptor {
     
         /**
          * 初始化方法
          */
         public void initialize() {}
     
         /**
          * 处理event
          * @param event 传入的数据
          * @return 处理完成的数据
          */
         public Event intercept(Event event) {
             //1.获取event的header和body
             Map<String, String> headers = event.getHeaders();
             byte[] body = event.getBody();
             //2.处理body数据,使用key创建配置项startWith,并使用value区分数据去向
             String s = new String(body);
             char c = s.charAt(0);
             if ((c >= 'a' && c <= 'z')||(c >= 'A' && c <= 'Z')){
                 headers.put("startWith", "alphabet");
             }else {
                 headers.put("startWith", "not_alphabet");
             }
             //3.返回修改过header的event
             return event;
         }
     
         /**
          * 批量处理event
          * @param list 传入的数据
          * @return 处理完成的数据
          */
         public List<Event> intercept(List<Event> list) {
             for (Event event : list) {
                 intercept(event);
             }
             return list;
         }
     
         /**
          * 关闭资源
          */
         public void close() {}
     
         /**
          * 内部类实现Interceptor.Builder接口
          * 用于创建Interceptor对象
          */
         public static class MyBuilder implements Interceptor.Builder{
     
             /**
              * 获取Interceptor实例
              * @return
              */
             public Interceptor build() {
                 return new MyInterceptor();
             }
     
             /**
              * 读取配置文件中的自定义配置项参数
              * @param context
              */
             public void configure(Context context) {}
             
         }
         
     }
     ```
     
4. 打包后将jar包上传到flume家目录的lib目录下
  
5. 分别在hadoop100，hadoop101，hadoop102节点上编写flume配置文件
  
     ```properties
   #在hadoop100节点上编写配置文件a1.conf
   #为agent(a1)的source,channel,sink分别命名
   a1.sources = r1
   a1.sinks = k1 k2
   a1.channels = c1 c2
   
   #配置source(r1),输入源类型为netcat,绑定r1监听的ip地址和端口号
   a1.sources.r1.type = netcat
   a1.sources.r1.bind = localhost
   a1.sources.r1.port = 44444
   #配置interceptor(拦截器),可配置多个
   a1.sources.r1.interceptors = i1
   #配置拦截器对象的build()方法的路径,此处为MyInterceptor类的内部类MyBuilder
   a1.sources.r1.interceptors.i1.type = com.atguigu.interceptor.MyInterceptor$MyBuilder
   #配置channel selector类型为multiplexing
   a1.sources.r1.selector.type = multiplexing
   #配置channel selector用于区分数据去向的header中的配置项(key)
   a1.sources.r1.selector.header = startWith
   #对于配置项的不同value,配置不同的channel去向
   a1.sources.r1.selector.mapping.alphabet = c1
   a1.sources.r1.selector.mapping.not_alphabet = c2
   
   #配置sink(k1,k2),写出类型为avro(flume提供的序列化方式),并配置数据交互的IP及端口号
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop101
   a1.sinks.k1.port = 4141
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop102
   a1.sinks.k2.port = 4242
   
   #配置channel(c1,c2),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 1000
   a1.channels.c2.transactionCapacity = 100
   
   #配置channel连接,r1连接c1、c2,k1连接c1,k2连接c2
   a1.sources.r1.channels = c1 c2
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   ```
  
     ```properties
   #在hadoop101节点上编写配置文件a2.conf
   #为agent(a2)的source,channel,sink分别命名
   a2.sources = r1
   a2.sinks = k1
   a2.channels = c1
   
   #配置source(r1),输入源类型为avro(flume提供的序列化方式),配置内容需要与flime1中sink的配置一致
   a2.sources.r1.type = avro
   a2.sources.r1.bind = hadoop101
   a2.sources.r1.port = 4141
   
   #配置sink(k1),写出类型为logger(控制台日志)
   a2.sinks.k1.type = logger
   
   #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
   a2.channels.c1.type = memory
   a2.channels.c1.capacity = 1000
   a2.channels.c1.transactionCapacity = 100
   
   #配置channel连接,r1连接c1,k1连接c1
   a2.sinks.k1.channel = c1
   a2.sources.r1.channels = c1
     ```
  
     ```properties
   #在hadoop102节点上编写配置文件a3.conf
   #为agent(a3)的source,channel,sink分别命名
   a3.sources = r1
   a3.sinks = k1
   a3.channels = c1
   
   #配置source(r1),输入源类型为avro(flume提供的序列化方式),配置内容需要与flime1中sink的配置一致
   a3.sources.r1.type = avro
   a3.sources.r1.bind = hadoop102
   a3.sources.r1.port = 4242
   
   #配置sink(k1),写出类型为logger(控制台日志)
   a3.sinks.k1.type = logger
   
   #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
   a3.channels.c1.type = memory
   a3.channels.c1.capacity = 1000
   a3.channels.c1.transactionCapacity = 100
   
   #配置channel连接,r1连接c1,k1连接c1
   a3.sinks.k1.channel = c1
   a3.sources.r1.channels = c1
     ```
  
6. 分别在hadoop100，hadoop101，hadoop102节点上启动flume
  
     ```shell
   #先启动flume2和flume3
   flume-ng agent -c conf/ -n a2 -f job/interceptor/a2.conf -Dflume.root.logger=INFO,console
   flume-ng agent -c conf/ -n a3 -f job/interceptor/a3.conf -Dflume.root.logger=INFO,console
   #最后启动flume1
   flume-ng agent -c conf/ -n a1 -f job/interceptor/a1.conf -Dflume.root.logger=INFO,console
   ```
  
7. 在hadoop100上使用netcat发送数据
  
     ```shell
   nc localhost 44444
   hello
   123456
   ```
  
  8. 查看hadoop101和hadoop102控制台的输出信息



### 3.5 自定义Source

1. 创建Maven工程，添加flume依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.flume</groupId>
           <artifactId>flume-ng-core</artifactId>
           <version>1.9.0</version>
           <scope>provided</scope>
       </dependency>
   </dependencies>
   ```

2. 自定义Source类

   ```java
   //需求:为flume接收的每条数据添加前缀后输出
   public class MySource extends AbstractSource implements Configurable, PollableSource {
   
       //自定义的配置项参数
       private String prefix;
       private Long interval;
   
       /**
        * 循环调用的拉取数据方法
        * @return 返回拉取状态
        * @throws EventDeliveryException
        */
       public Status process() throws EventDeliveryException {
           Status status = null;
           try {
               //使用自定义的方法拉取数据
               Event e = getSomeData();
               getChannelProcessor().processEvent(e);
               status = Status.READY;
           } catch (Throwable t) {
               status = Status.BACKOFF;
               if (t instanceof Error) {
                   throw (Error)t;
               }
           }
           return status;
       }
   
       /**
        * 自定义数据的拉取包装方法
        * @return
        */
       private Event getSomeData() throws InterruptedException {
           //生成随机数据
           int i = (int) (Math.random() * 10000);
           String s = prefix + i;
           Thread.sleep(interval);
           //创建event对象并写入数据
           Event event = new SimpleEvent();
           event.setBody(s.getBytes());
           //返回event
           return event;
       }
   
       /**
        * 拉取数据失败后等待间隔时间的增长速度(毫秒)
        * @return
        */
       public long getBackOffSleepIncrement() {
           return 1000;
       }
   
       /**
        * 最大的拉取间隔时间(毫秒)
        * @return
        */
       public long getMaxBackOffSleepInterval() {
           return 100000;
       }
   
       /**
        * 读取配置文件中自定义的配置项参数
        * @param context
        */
       public void configure(Context context) {
           //形参列表为配置文件中的配置项和默认值
           prefix = context.getString("prefix", "xxx");
           interval = context.getLong("interval", 500L);
       }
   }
   ```

3. 打包后将jar包上传到flume家目录的lib目录下

4. 编写配置文件

   ```properties
   a1.sources = r1
   a1.sinks = k1
   a1.channels = c1
   
   #使用自定义source的全类名配置r1的类型
   a1.sources.r1.type = com.atguigu.source.MySource
   a1.sources.r1.interval = 1000
   a1.sources.r1.prefix = MySource-
   
   #配置sink(k1),写出类型为logger(控制台日志)
   a1.sinks.k1.type = logger
   
   #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   #配置channel连接,r1连接c1,k1连接c1
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
   ```

5. 启动flume测试

   ```shell
   flume-ng agent -c conf/ -n a1 -f job/source/a1.conf -Dflume.root.logger=INFO,console
   ```



### 3.6 自定义Sink

1. 创建Maven工程，添加flume依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.flume</groupId>
           <artifactId>flume-ng-core</artifactId>
           <version>1.9.0</version>
           <scope>provided</scope>
       </dependency>
   </dependencies>
   ```

2. 自定义Sink类

   ```java
   public class MySink extends AbstractSink implements Configurable {
   
       /**
        * 从Channel中拉取数据并处理
        * @return 处理状态
        * @throws EventDeliveryException
        */
       public Status process() throws EventDeliveryException {
           Status status = null;
   
           Channel ch = getChannel();
           Transaction txn = ch.getTransaction();
           txn.begin();
           try {
               Event event = ch.take();
               while ((event = ch.take()) == null){
                   Thread.sleep(100);
               }
               //使用自定义方法处理event
               storeSomeData(event);
               txn.commit();
               status = Status.READY;
           } catch (Throwable t) {
               txn.rollback();
               status = Status.BACKOFF;
               if (t instanceof Error) {
                   throw (Error)t;
               }
           } finally {
               //关闭txn资源
               txn.close();
           }
           return status;
       }
   
       /**
        * 自定义拉取到数据后的输出方式
        * @param event
        */
       private void storeSomeData(Event event) {
           //输出数据到控制台
           byte[] body = event.getBody();
           System.out.println(body.toString());
       }
   
       /**
        * 读取配置文件中自定义配置项的参数
        * @param context
        */
       public void configure(Context context) {}
   }
   ```

3. 打包后将jar包上传到flume家目录的lib目录下

4. 编写配置文件

   ```properties
   a1.sources = r1
   a1.sinks = k1
   a1.channels = c1
   
   #配置source(r1),输入源类型为netcat,绑定r1监听的ip地址和端口号
   a1.sources.r1.type = netcat
   a1.sources.r1.bind = localhost
   a1.sources.r1.port = 44444
   
   #使用自定义Sink类的全类名配置k1的type
   a1.sinks.k1.type = com.atguigu.sink.MySink
   
   #配置channel(c1),类型为memory(内存型),容量为1000个event,每收集100个event提交1次事务
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   #配置channel连接,r1连接c1,k1连接c1
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
   ```

5. 启动flume测试

   ```shell
   flume-ng agent -c conf/ -n a1 -f job/sink/a1.conf -Dflume.root.logger=INFO,console
   ```



### 3.7 Flume数据流监控

#### 3.7.1 Ganglia的安装与部署

- Ganglia 是用于实时监控系统指标数据的第三方框架，包括 gmond，gmetad，gweb三部分

  - gmond（Ganglia Monitoring Daemon）用于收集多种系统指标数据（CPU、内存、磁盘、网络、Flume等），安装在每台需要收集指标数据的主机节点上。
  - gmetad（Ganglia Meta Daemon）整合所有gmond节点的信息，并以RRD格式存储至磁盘。
  - gweb（Ganglia Web）是Ganglia的可视化工具，将监控的指标数据以图表形式呈现在浏览器中。

- 安装与部署过程

  1. 在所有节点安装Ganglia

     ```shell
     sudo yum install -y epel-release
     ```

  2. 在hadoop100节点安装gmond，gmetad，gweb

     ```shell
     sudo yum -y install ganglia-gmetad ganglia-web ganglia-gmond
     ```

  3. 在hadoop101、hadoop102节点安装gmond

     ```shell
     sudo yum -y install ganglia-gmond
     ```

  4. 在安装gweb的节点上修改配置文件

     ```shell
     sudo vim /etc/httpd/conf.d/ganglia.conf
     ```

     ```shell
     #
     # Ganglia monitoring system php web frontend
     #
     
     Alias /ganglia /usr/share/ganglia
     
     <Location /ganglia>
       # 下一行Require local默认为非注释状态,这里需要进行注释
       # Require local
       # 下一行默认为注释状态,打开注释,并将ip设置为所有需要配置ganglia节点所在的网段(支持配置多行多个网段)
       Require ip 192.168.145.1
       # Require all granted 若网段不识别 可改为授予所有ip访问(不安全)
       # Require host example.org
     </Location>
     ```
     
  5. 在安装gmetad的节点上修改配置文件
  
     ```shell
     sudo vim /etc/ganglia/gmetad.conf
     ```
  
     ```shell
     #找到data_source行,配置集群名和gmetad节点的主机ip
     #该集群名会用在所有gmond节点配置中,所有gmond节点会将收集到的数据信息发送到该节点
     data_source "mycluster" hadoop100
     ```
  
  6. 在安装gmond的**所有**节点上修改配置文件
  
     ```shell
     sudo vim /etc/ganglia/gmond.conf
     ```
  
     ```properties
     #1.修改ganglia集群配置
     cluster {
     #此处的name项为gmetad节点中自定义的集群名
     name = "mycluster"
     owner = "unspecified"
     latlong = "unspecified"
     url = "unspecified"
     }
     
     #2.修改udp_send_channel配置
     udp_send_channel {
     #bind_hostname = yes # Highly recommended, soon to be default.
     		 # This option tells gmond to use a source address
     		 # that resolves to the machine's hostname.  Without
     		 # this, the metrics may appear to come from any
     		 # interface and the DNS names associated with
     		 # those IPs will be used to create the RRDs.
     # mcast_join = 239.2.11.71
     #host地址需设置为gmetad节点的ip地址
     host = hadoop100
     port = 8649
     ttl = 1
     }
     
     #3.修改udp_recv_channel配置
     udp_recv_channel {
     # mcast_join = 239.2.11.71
     port = 8649
     #bind地址设置为0.0.0.0,接收所有地址发送的数据
     bind = 0.0.0.0
     retry_bind = true
     # Size of the UDP buffer. If you are handling lots of metrics you really
     # should bump it up to e.g. 10MB or even higher.
     # buffer = 10485760
     }
     ```
  
  7. 在安装gmond的**所有**节点上修改配置文件
  
     ```shell
     sudo vim /etc/selinux/config
     ```
  
     ```properties
     # This file controls the state of SELinux on the system.
     # SELINUX= can take one of these three values:
     #     enforcing - SELinux security policy is enforced.
     #     permissive - SELinux prints warnings instead of enforcing.
     #     disabled - No SELinux policy is loaded.
     #SElinux(最小权限原则):最大程度的减少服务进程访问的资源,为避免访问受限,这里需要关闭
     SELINUX=disabled
     # SELINUXTYPE= can take one of these two values:
     #     targeted - Targeted processes are protected,
     #     mls - Multi Level Security protection.
     SELINUXTYPE=targeted
     ```
  
     ```shell
     #配置文件修改完成后需要重启linux系统生效
     #也可以使用临时生效命令
     sudo setenforce 0
     ```
  
  8. 编写ganglia集群群起脚本（可选）
  
     ```shell
     vim /home/atguigu/bin/gangliactl
     ```
  
     ```shell
     #!/bin/bash
     if [ $# -lt 1 ]
     	then
     		echo "Input args is null"
     	exit
     fi
     case $1 in
     "start")
     	echo "---------- ganglia start -----------"
     	ssh hadoop100 sudo systemctl start httpd
     	sudo systemctl start gmetad
     	sudo systemctl start gmond
     	ssh hadoop101 sudo systemctl start gmond
     	ssh hadoop102 sudo systemctl start gmond
     ;;
     "stop")
     	echo "---------- ganglia stop -----------"
     	ssh hadoop100 sudo systemctl stop httpd
     	sudo systemctl stop gmetad
     	sudo systemctl stop gmond
     	ssh hadoop101 sudo systemctl stop gmond
     	ssh hadoop102 sudo systemctl stop gmond
     ;;
     *)
     	echo "Input args not found"
     ;;
     esac
     ```
  
     ```shell
     chmod 777 /home/atguigu/bin/gangliactl
     ```
  
  9. 在hadoop100启动gweb，gmetad，gmond，在hadoop102，hadoop103启动gmond
  
     ```shell
     sudo systemctl start httpd
     sudo systemctl start gmetad
     sudo systemctl start gmond
     ```
  
  10. 打开浏览器查看ganglia主页 http://hadoop100/ganglia



#### 3.7.2 操作Flume测试监控

1. 完成3.7.1中ganglia的安装部署

2. 启动任意flume任务

   ```shell
   #启动命令如下,需在启动时配置ganglia及监听的ip端口
   bin/flume-ng agent \
   --conf conf/ \
   --name a1 \
   --conf-file job/flume-netcat-logger.conf \
   -Dflume.root.logger=INFO,console \
   -Dflume.monitoring.type=ganglia \
   -Dflume.monitoring.hosts=hadoop100:8649
   ```

3. 使用netcat命令传输数据

   ```shell
   nc localhost 44444
   ```

4. 查看web端监控图表变化

   ```
   http://hadoop100/ganglia
   -->选择ganglia集群(hadoop100)
   -->选择集群中的具体节点(hadoop100/hadoop101/hadoop102)
   ```

| 字段（图表名称）      | 字段含义                            |
| --------------------- | ----------------------------------- |
| EventPutAttemptCount  | source尝试写入channel的事件总数量   |
| EventPutSuccessCount  | 成功写入channel且提交的事件总数量   |
| EventTakeAttemptCount | sink尝试从channel拉取事件的总数量。 |
| EventTakeSuccessCount | sink成功读取的事件的总数量          |
| StartTime             | channel启动的时间（毫秒）           |
| StopTime              | channel停止的时间（毫秒）           |
| ChannelSize           | 目前channel中事件的总数量           |
| ChannelFillPercentage | channel占用百分比                   |
| ChannelCapacity       | channel的容量                       |