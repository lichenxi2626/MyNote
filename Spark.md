# Spark

## 第一章 Spark概述

### 1.1 概述

- Spark是一种基于内存的快速、通用、可扩展的大数据分析计算引擎

### 1.2 Spark与Hadoop

1. MapReduce在循环迭代式的数据处理中存在严重的计算效率问题，Spark对计算过程进行了大幅优化；
2. **MapReduce的多作业通信基于磁盘，而Spark基于内存；**
3. MapReduce采用创建新线程的方式启动Map Task，而Spark采用fork线程更快地启动Spark Task；
4. Spark的缓存机制比HDFS更高效



### 1.3 Spark核心模块

- Spark Core
  提供Spark最基础最核心的功能，Spark的其他模块都是基于此模块进行扩展的
- Spark SQL
  提供用于操作结构化数据的组件，可以直接使用SQL或HQL查询数据
- Spark Streaming
  提供对实时数据进行流式计算的组件，提供了丰富的数据流处理API
- Spark MLlib
  提供机器学习算法库
- Spark GraphX
  提供图计算的框架与算法库



## 第二章 Spark上手

### 2.1 创建Maven工程

1. 在IDEA中安装Scala插件

2. 创建Maven工程

3. 修改pom文件，添加依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.spark</groupId>
           <artifactId>spark-core_2.12</artifactId>
           <version>2.4.5</version>
       </dependency>
   </dependencies>
   <build>
       <plugins>
           <!-- 该插件用于将Scala代码编译成class文件 -->
           <plugin>
               <groupId>net.alchim31.maven</groupId>
               <artifactId>scala-maven-plugin</artifactId>
               <version>3.2.2</version>
               <executions>
                   <execution>
                       <!-- 声明绑定到maven的compile阶段 -->
                       <goals>
                           <goal>testCompile</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-assembly-plugin</artifactId>
               <version>3.0.0</version>
               <configuration>
                   <descriptorRefs>
                       <descriptorRef>jar-with-dependencies</descriptorRef>
                   </descriptorRefs>
               </configuration>
               <executions>
                   <execution>
                       <id>make-assembly</id>
                       <phase>package</phase>
                       <goals>
                           <goal>single</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
       </plugins>
   </build>
   ```

4. （可选）在 main/resources 目录下添加 log4j.properties 文件，配置日志的输出级别

   ```properties
   log4j.rootCategory=ERROR, console
   log4j.appender.console=org.apache.log4j.ConsoleAppender
   log4j.appender.console.target=System.err
   log4j.appender.console.layout=org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
   
   # Set the default spark-shell log level to ERROR. When running the spark-shell, the
   # log level for this class is used to overwrite the root logger's log level, so that
   # the user can have different defaults for the shell and regular Spark apps.
   log4j.logger.org.apache.spark.repl.Main=ERROR
   
   # Settings to quiet third party logs that are too verbose
   log4j.logger.org.spark_project.jetty=ERROR
   log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR
   log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR
   log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR
   log4j.logger.org.apache.parquet=ERROR
   log4j.logger.parquet=ERROR
   
   # SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support
   log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
   log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
   ```




### 2.2 编写WordCount

```scala
object WordCount {
    def main(args: Array[String]): Unit = {

        //1.准备spark环境
        val sparkConf: SparkConf = new SparkConf().setMaster("local").setAppName("WordCount")

        //2.建立spark连接
        val sparkContext = new SparkContext(sparkConf)

        //3.实现业务操作
        //3.1 读取数据文件
        val fileRDD: RDD[String] = sparkContext.textFile("input/word*")
        //3.2 切分单词
        val words: RDD[String] = fileRDD.flatMap(_.split(" "))
        //3.3 转换集合元素类型为键值对
        val mapWord: RDD[(String, Int)] = words.map((_,1))
        //3.4 根据key值分组reduce计算求和
        val wordCount: RDD[(String, Int)] = mapWord.reduceByKey(_+_)
        //3.5 收集计算结果,使用数组接收并输出到控制台
        val result: Array[(String, Int)] = wordCount.collect()
        println(result.mkString(","))

        //4.释放连接
        sparkContext.stop()

    }
}
```



## 第三章 Spark运行环境

### 3.1 Local模式

#### 3.1.1 概述

- Local模式仅需在一台主机节点部署Spark，且无需启动额外的进程
- 一般用于本地测试

#### 3.1.2 Local模式部署

1. 上传压缩包到linux，并解压安装到指定目录

   ```shell
   tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
   #由于spark2.4.5自带hadoop2.7相关jar包,因此默认不支持hadoop3,这次上传的是不包含hadoop的jar包的压缩包
   ```

2. 重命名安装目录

   ```shell
   mv /opt/module spark-2.4.5-bin-without-hadoop-scala-2.12 /opt/module/spark-local
   ```

3. 修改配置文件，或直接上传 hadoop3 相关的 jar 包到安装目录的 jars 目录下

   ```shell
   cd /opt/module/spark-local
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下内容
   SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
   #配置hadoop安装目录下的hadoop命令路径
   ```

4. 启动 Local 模式

   ```shell
   bin/spark-shell --master local[*]
   # []中配置local环境分配的cpu数量,*表示全部使用
   ```

5. web端访问

   ```
   http://hadoop100:4040
   ```

6. 退出 Local 模式

   ```scala
   :quit
   ```

7. 提交应用

   ```shell
   bin/spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master local[2] \
   ./examples/jars/spark-examples_2.12-2.4.5.jar \
   10
   # --class 执行程序的主类
   # --master 部署的模式
   # spark-examples_2.12-2.4.5.jar 主类所在jar包的路径
   # 10 设定应用的任务数量
   ```



### 3.2 Standalone模式

#### 3.2.1 概述

- 在hadoop1.x版本，mapreduce同时负责资源和任务的调度，替换mr需要spark自带一套资源调度系统，于是就诞生了spark的独立部署（Standalone）模式
- 类似于Yarn的ResourceManeger-NodeManager架构，Spark使用了Master-Slave的集群架构，实现资源调度

#### 3.2.2 Standalone模式部署

1. 上传spark压缩包到linux，解压安装到指定目录

   ```shell
   tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
   ```

2. 重命名安装目录

   ```shell
   mv /opt/module spark-2.4.5-bin-without-hadoop-scala-2.12 /opt/module/spark-standalone
   ```

3. 修改配置文件，或直接上传 hadoop3 相关的 jar 包到安装目录的 jars 目录下

   ```shell
   cd /opt/module/spark-local
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下内容
   SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
   #配置hadoop安装目录下的hadoop命令路径
   ```

4. 重命名修改配置文件，添加主机节点

   ```shell
   cd /opt/modulespark-standalone
   mv conf/slaves.template conf/slaves
   vi conf/slaves
   ```

   ```shell
   #添加spark集群节点
   hadoop100
   hadoop101
   hadoop102
   ```

5. 重命名并修改配置文件，添加Java环境变量和集群的Master节点

   ```shell
   mv conf/spark-env.sh.template conf/spark-env.sh
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下配置
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   SPARK_MASTER_HOST=hadoop100
   SPARK_MASTER_PORT=7077
   #7077为spark master用于内部通信的端口号
   ```

6. 分发Spark到所有节点

   ```shell
   xsync spark-standalone
   ```

7. 在hadoop100节点启动spark集群

   ```shell
   sbin/start-all.sh
   #启动后Master节点进程为 Master Worker
   #其余节点进程为 Worker
   ```

8. web端资源监控

   ```
   hadoop100:8080
   ```

9. 提交应用

   ```shell
   bin/spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master spark://hadoop100:7077 \
   ./examples/jars/spark-examples_2.12-2.4.5.jar \
   10
   #应用提交到master节点
   ```

10. 关闭spark集群

    ```shell
    sbin/stop-all.sh
    ```

#### 3.2.3 配置历史服务

1. 重命名并修改配置文件，配置日志的存储路径

   ```shell
   mv conf/spark-defaults.conf.template conf/spark-defaults.conf
   vi conf/spark-defaults.conf
   ```

   ```powershell
   #修改以下配置
   spark.eventLog.enabled           true
   spark.eventLog.dir               hdfs://hadoop100:9820/directory
   #历史服务数据会存储到hdfs中,因此需要在hdfs中创建该目录
   ```

2. 修改配置文件，添加日志的相关配置

   ```shell
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下配置
   export SPARK_HISTORY_OPTS="
   -Dspark.history.ui.port=18080 
   -Dspark.history.fs.logDirectory=hdfs://hadoop100:9820/directory 
   -Dspark.history.retainedApplications=30"
   # 18080为web端查看历史服务的端口号
   # 日志存储路径与spark-defaults.conf中配置一致
   # retainedApplications为历史服务保留的最大应用数,超过时旧日志会被删除
   ```

3. 分发配置文件

   ```shell
   xsync conf/
   ```

4. 先后启动hdfs、spark集群和历史服务

   ```shell
   start-dfs.sh
   bin/start-all.sh
   bin/start-history-server.sh
   ```
   
5. web端查看历史服务

   ```
   hadoop100:18080
   ```

6. 关闭历史服务

   ```
   bin/stop-history-server.sh
   ```

   

#### 3.2.4 配置高可用（HA）

1. 修改配置文件

   ```shell
   vi conf/spark-env.sh
   ```

   ```shell
   #修改以下内容
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   #SPARK_MASTER_HOST=hadoop100
   #SPARK_MASTER_PORT=7077
   SPARK_MASTER_WEBUI_PORT=8989
   #修改web端资源监控端口号防止冲突(默认8080)
   
   export SPARK_DAEMON_JAVA_OPTS="
   -Dspark.deploy.recoveryMode=ZOOKEEPER
   -Dspark.deploy.zookeeper.url=hadoop100,hadoop101,hadoop102
   -Dspark.deploy.zookeeper.dir=/spark"
   #配置spark到zookeeper实现高可用
   ```

2. 分发配置文件

   ```shell
   xsync conf/
   ```

3. 启动zookeeper

4. 在hadoop100节点启动spark集群

   ```shell
   sbin/start-all.sh
   #启动后hadoop100仍为Master节点
   ```

5. 在hadoop101节点单独启动Master进程实现热备

   ```shell
   sbin/start-master.sh
   #关闭集群时hadoop101的master进程同样需要单独关闭
   ```

6. web端查看节点状态

   ```shell
   http://hadoop100:8989
   http://hadoop101:8989
   # hadoop100状态为ALIVE,hadoop101状态为STANDBY 
   ```



### 3.3 Yarn模式

#### 3.3.1 概述

- 在hadoop2.x推出了资源框架Yarn后，对资源分配与计算任务进行了解耦，使spark可以仅作为计算框架依赖于Yarn调度执行。
- yarn模式不需要部署spark集群，因此只需要在一台节点安装spark并进行相关配置即可使用。

#### 3.3.2 Yarn模式部署

1. 上传spark压缩包到linux，解压安装到指定目录

   ```shell
   tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
   ```

2. 重命名安装目录

   ```shell
   mv /opt/module spark-2.4.5-bin-without-hadoop-scala-2.12 /opt/module/spark-standalone
   ```

3. 修改配置文件，或直接上传 hadoop3 相关的 jar 包到安装目录的 jars 目录下

   ```shell
   cd /opt/module/spark-local
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下内容
   SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
   #配置hadoop安装目录下的hadoop命令路径
   ```

4. 重命名并修改配置文件，添加 Java 和 Yarn 的相关配置

   ```shell
   mv conf/spark-env.sh.template conf/spark-env.sh
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下内容
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   YARN_CONF_DIR=/opt/module/hadoop-3.1.3/etc/hadoop
   ```

5. 启动 hdfs 和 yarn 集群

   ```shell
   # hadoop100节点启动hdfs
   start-dfs.sh
   # hadoop101节点启动yarn
   start-yarn.sh
   ```

6. 提交应用

   ```shell
   bin/spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master yarn \
   ./examples/jars/spark-examples_2.12-2.4.5.jar \
   10
   ```

7. web端查看yarn

   ```
   http://hadoop101:8088
   ```



#### 3.3.3 配置历史服务

1. 重命名并修改配置文件，配置日志的存储路径

   ```shell
   mv conf/spark-defaults.conf.template conf/spark-defaults.conf
   vi conf/spark-defaults.conf
   ```

   ```powershell
   #修改以下配置
   spark.eventLog.enabled           true
   spark.eventLog.dir               hdfs://hadoop100:9820/directory
   #历史服务数据会存储到hdfs中,因此需要在hdfs中创建该目录
   ```

2. 修改配置文件，添加日志的相关配置

   ```shell
   vi conf/spark-env.sh
   ```

   ```shell
   #添加以下配置
   export SPARK_HISTORY_OPTS="
   -Dspark.history.ui.port=18080 
   -Dspark.history.fs.logDirectory=hdfs://hadoop100:9820/directory 
   -Dspark.history.retainedApplications=30"
   # 18080为web端查看历史服务的端口号
   # 日志存储路径与spark-defaults.conf中配置一致
   # retainedApplications为历史服务保留的最大应用数,超过时旧日志会被删除
   ```

3. 分发配置文件

   ```shell
   xsync conf/
   ```

4. 先后启动hdfs、yarn和历史服务

   ```shell
   start-dfs.sh
   start-yarn.sh
   bin/start-history-server.sh
   ```

5. web端查看历史服务

   ```
   hadoop100:18080
   ```

6. 关闭历史服务

   ```
   bin/stop-history-server.sh
   ```



#### 3.3.4 提交应用相关参数

| 参数                     | 说明                                                         | 可选值举例                             |
| ------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| --class                  | Spark程序中包含主函数的类                                    |                                        |
| --master                 | Spark程序运行的模式                                          | local[*]  spark://hadoop100:7077  Yarn |
| --executor-memory 1G     | 指定每个executor可用内存为1G                                 | 符合集群内存配置即可，具体情况具体分析 |
| --total-executor-cores 2 | 指定所有executor使用的cpu核数为2个                           |                                        |
| --executor-cores         | 指定每个executor使用的cpu核数                                |                                        |
| application-jar          | 打包好的应用jar，包含依赖。<br />如hdfs://共享存储系统<br />若使用file://path，则所有的节点都需要提供该jar包 |                                        |
| application-arguments    | 传给main()方法的参数                                         |                                        |



### 3.4 K8S和Mesos模式

- Kubernetes（K8S）是一个容器管理工具让用户更加方便的对应用进行管理和运维
- Mesos是一个开源的分布式管理框架，被称为分布式系统的内核



### 3.5 Windows模式

1. 解压spark压缩包到无中文路径，并将hadoop依赖添加到 jars 目录中
2. 执行bin目录下的spark-shell.cmd文件，启动spark



### 3.6 常用端口号

| 说明                                                       | 端口号       |
| ---------------------------------------------------------- | ------------ |
| web查看 spark-shell 运行任务的端口                         | 4040（计算） |
| Spark Master内部通信服务端口，standalone模式提交应用的端口 | 7077         |
| web端对Spark Master资源监控的端口                          | 8080（资源） |
| web端查看Spark历史服务器端口                               | 18080        |
| web端查看Yarn任务端口                                      | 8088         |



## 第四章 Spark运行架构

### 4.1 运行架构

![image-20200603231644649](Spark.assets/image-20200603231644649.png)

- Spark框架的核心是一个计算引擎，采用了master-slave结构，Driver作为master负责作业的调度，Executor作为slave负责任务的执行。



### 4.2 核心组件

- Driver：spark的驱动器节点，具有以下职能
  1. 将用户程序转化为作业
  2. 在Executor间调度任务
  3. 监控Executor任务的执行情况
  4. 通过WebUI展示运行状况
- Executor：spark的执行节点，具有以下职能
  1. 运行任务，并将结果返回给驱动器进程
  2. 缓存任务中的RDD到内存中，加速运算
- Master & Worker：spark独立模式下的资源调度框架
  1. Master：资源的调度与分配，监控集群
  2. Worker：执行任务，处理计算数据
- ApplicationMaster：Yarn中用于向RM申请资源的进程
  - 在Yarn运行模式下，ApplicationMaster用于连接Yarn的ResourceManager和Spark的Driver



### 4.3 核心概念

- Executor与Core
  - Executor是集群中运行在工作节点上的一个JVM进程，通过配置参数可以设置一个作业的Executor个数，以及每个Executor可使用的资源。资源包括内存和CPU核数（Core）
- 并行度
  - Spark中的应用程序提交后通常由多节点多任务并行完成，整个集群并行执行的任务数量就称为并行度
- 有向无环图（DAG）
  - DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环
  - DAG框架将MR计算引擎的Map和Reduce阶段算法进行了拆分和细化，将多个Job串联完成一个完整的算法



### 4.4 提交流程

- Spark应用程序提交到Yarn环境执行时有两种部署执行的方式
  1. Yarn Client：Driver在提交应用的本地机器上运行（用于测试）
  
     ```shell
     bin/spark-submit \
     --class org.apache.spark.examples.SparkPi \
     #提交任务到yarn时默认使用Client模式
     --master yarn \
     ./examples/jars/spark-examples_2.12-2.4.5.jar \
     10
     ```
  
  2. Yarn Cluster：Driver在Yarn集群中的节点上运行（用于生产）
  
     ```shell
     bin/spark-submit \
     --class org.apache.spark.examples.SparkPi \
     --master yarn \
     #使用yarn集群部署模式需要配置参数--deploy-mode
     --deploy-mode cluster \
     ./examples/jars/spark-examples_2.12-2.4.5.jar \
     10
     ```



## 第五章 Spark核心编程

### 5.1 RDD

#### 5.1.1 RDD概述

- RDD是一个弹性分布式数据集，是Spark中最基本的数据处理模型。在API中，RDD是一个抽象类，表示一个集合。
- RDD特性
  1. 弹性
     - 存储弹性：内存与磁盘存储自动切换
     - 容错弹性：数据丢失可以自动恢复
     - 计算弹性：计算错误时会重新计算
     - 分片弹性：可根据需求重新分片
  2. 分布式：数据存储在集群不同节点上
  3. 数据集：RDD实际封装的是计算逻辑，不保存具体的数据
  4. 不可变：RDD封装的计算逻辑不可改变，修改逻辑时需要使用新的RDD接收
  5. 可分区：多个分区并行计算
- RDD中封装的核心属性
  1. 分区列表
     RDD数据结构中存在分区列表，实现多分区并行计算
  2. 分区计算函数
     具体的计算逻辑
  3. 依赖关系
     由于RDD的不可变性，执行复杂的计算时需要使用多层RDD封装逻辑，因此需要上层RDD依赖
  4. 分区器（可选）
     数据为KV类型时，可以使用分区器的对数据按照自定义规则分区
  5. 首选位置（可选）
     根据计算节点状态选择不同的节点位置进行计算
- 执行过程
  1. 启动Yarn集群环境
  2. Spark申请启动Driver和Executor节点
  3. Spark将RDD封装的计算逻辑按照分区划分为一个个Task
  4. Driver将Task发送给不同的Executor进行计算，一个Executor可以执行多个Task



#### 5.1.2 RDD的创建

Spark中创建RDD共有四种方式

1. 从内存创建

   ```scala
   val sparkConf: SparkConf = new SparkConf().setMaster("local").setAppName("wordCount")
   val sc = new SparkContext(sparkConf)
   
   //方式1:parallelize方法
   val list = List(1,2,3,4)
   val rdd: RDD[Int] = sc.parallelize(list)
   
   sc.stop()
   ```

   ```scala
   val sparkConf: SparkConf = new SparkConf().setMaster("local").setAppName("wordCount")
   val sc = new SparkContext(sparkConf)
   
   //方式2:makeRDD方法(底层仍然调用了parallelize方法)
   val list = List(1,2,3,4)
   val rdd: RDD[Int] = sc.parallelize(list)
   
   sc.stop()
   ```

2. 从磁盘中的文件创建

   ```scala
   val sparkConf: SparkConf = new SparkConf().setMaster("local").setAppName("filerdd")
   
   val sc = new SparkContext(sparkConf)
   
   //从磁盘读取文件 默认按行读取 指向目录时会读取目录下所有文件
   val rdd: RDD[String] = sc.textFile("input/word*")
   
   sc.stop()
   ```

3. 从其他RDD转换创建

4. 直接创建（new）



#### 5.1.3 RDD的并行度与分区

##### 读取内存数据的分区规则

1. 创建RDD时未指定分片数

   ```scala
   val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("wordCount")
   val sc = new SparkContext(sparkConf)
   val rdd: RDD[Int] = sc.parallelize(List(1,2,3,4))
   sc.stop()
   //根据setMaster方法中设置的当前环境可用的最大CPU核数决定分区数
   // local	单核执行，分区数为1
   // loca[n]	n核执行，分区数为n，可设置大于CPU核数的值，依然生效
   // local[*]	当前主机的cpu核数，分区数与核数一致
   ```

2. 创建RDD时指定分片数

   ```scala
   val rdd: RDD[Int] = sc.parallelize(List(1,2,3,4),2)
   //根据 makeRDD 方法中第二个参数直接决定分区数
   ```

3. 数据分配规则：均分（集合元素数 / 分区数）

   ```scala
   //示例
   List(1,2,3,4,5)
   //指定分区数为3,分区后结果为
   p0(1)  p1(2,3)  p2(4,5)
   ```



##### 读取文件数据的分区规则

1. 创建RDD时未指定最小分区数

   ```scala
   //最小分区数计算公式如下(取较小值)
   min(defaultParallelism,2)
   //defaultParallelism表示setMaster中设置的当前环境的最大CPU核数
   ```

2. 创建RDD时指定最小分区数

   ```scala
   val rdd: RDD[String] = sc.textFile("input/word.txt",3)
   //按照参数值设置最小分区数
   ```

3. 实际分区数与数据分配规则

   ```scala
   1.计算文件大小,读取多文件时计算总和
   
   2.计算 总文件大小(字节)/最小分区数 得到分区大小(字节)
     (读取单个文件时,若不能整除且 余数>0.1*分区大小 ,则会增加1个额外的分区)
   
   3.按照分区大小,分别对每个文件进行切分,每个分区记录了数据在文件中的偏移量
   	(切分过程不跨文件,即一个分区中不会包含不同文件的数据)
   
   4.每个切片根据数据偏移量按行读取文件中的数据,按行读取,行不可拆分
   ```



##### 并行度

1. 当分区数≤作业环境的CPU核数时，分区数即为作业的并行度
2. 当分区数>作业环境的CPU核数时，CPU核数即为作业的并行度



#### 5.1.4 RDD转换算子

- 在RDD数据结构中，不存储数据本身，而是存储计算逻辑，这些计算逻辑就称为转换算子
- 转换算子不会立即执行，而是在调用行动算子（如collect）时才开始执行
- **多个分区之间转换算子并行互不影响(shuffle除外)，每个分区内的转换算子按逻辑顺序串行**

##### Value类型

1. map 映射

   ```scala
   //功能 与scala中的map方法相同,使用自定义函数映射RDD中的每一个元素进行逐条转换
   //自定义函数参数 从集合读取时为集合中的一个元素;从文件读取数据时为文件中的一行数据
   //自定义函数返回值 任意类型
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd1: RDD[Int] = rdd.map(_*2)
   ```

2. mapPartitions 分区映射

   ```scala
   //功能 一次传入整个分区的数据进行数据处理,需要保证内存空间充足
   //自定义函数参数 包含一个分区全部数据的迭代器对象
   //自定义函数返回值 迭代器类型的对象
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd1: RDD[Int] = rdd.mapPartitions(
       iterator => {
           List(iterator.max).iterator //获取每个分区数据的最大值
       }
   )
   ```

3. mapPartitionsWithIndex 带分区索引的分区映射

   ```scala
   //功能 同mapPartitions
   //自定义函数参数 (分区的索引,包含一个分区全部数据的迭代器对象)
   //自定义函数返回值 迭代器类型的对象
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd1: RDD[(Int, Int)] = rdd.mapPartitionsWithIndex(
       (index, it) => {
           List((it.max, index)).iterator //获取每个分区数据的最大值及分区号
       }
   )
   println(rdd1.collect().toList)
   ```

4. flatMap 扁平映射

   ```scala
   //功能 与scala一致,扁平映射,对数据进行一次map后再进行一次flatten扁平化处理
   //自定义函数参数 每一个元素或每一行数据
   //自定义函数返回值 可迭代的数据类型(集合)
   val rdd: RDD[List[Int]] = sc.makeRDD(List(List(1,2),List(3,4)),2)
   val rdd1: RDD[Int] = rdd.flatMap(list=>list)
   ```

5. glom 分区聚合

   ```scala
   //功能 将每个分区的数据分别转换为数组,返回这些数组组成的集合
   //自定义函数参数 无
   //自定义函数返回值 数组
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd1: RDD[Array[Int]] = rdd.glom()
   ```

6. groupBy 分组

   ```scala
   //功能 将数据按照自定义函数分组,得到kv对组成的集合,key为自定义函数的返回值,v为迭代器对象
   //自定义函数参数 每一个元素或每一行数据
   //自定义函数返回值 任意类型,作为分组后的key值
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6),3)
   val rdd1: RDD[(Int, Iterable[Int])] = rdd.groupBy(_%2)
   //groupBy算子是对所有分区的统一操作,在执行时会对数据分区进行shuffle,但默认不会改变分区数
   //可以通过传参手动设定算子执行后的分区数量
   val rdd1: RDD[(Int, Iterable[Int])] = rdd.groupBy(_%2, 2) //修改分区数为2
   ```

7. filter 过滤

   ```scala
   //功能 将数据按照自定义函数进行过滤,丢弃不想保留的数据
   //自定义函数参数 每一个元素或每一行数据
   //自定义函数返回值 Boolean型变量,为true则保留数据,为false则丢弃数据
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6),3)
   val rdd1: RDD[Int] = rdd.filter(_%2 == 0)
   //不改变分区数
   ```

8. sample 抽取

   ```scala
   //功能1 不放回抽取,按照指定概率抽取每条数据
   //参数1 false(伯努利算法)
   //参数2 每条数据被抽取的概率[0,1]
   //参数3 (可选)随机数seed,seed相同时,抽取结果被唯一确定
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6))
   val rdd1: RDD[Int] = rdd.sample(false, 0.5)
   
   //功能2 放回抽取,按照指定期望抽取每条数据
   //参数1 true(泊松算法)
   //参数2 每条数据被抽取的期望次数
   //参数3 (可选)随机数seed,seed相同时,抽取结果被唯一确定
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6))
   val rdd1: RDD[Int] = rdd.sample(true, 2)
   ```

9. distinct 去重

   ```scala
   //功能 去除RDD中重复的数据,包含shuffle过程
   //参数 (可选)去重后的新分区数(可增加或缩减分区),默认和去重前分区数一致
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6),2)
   val rdd1: RDD[Int] = rdd.distinct()
   val rdd2: RDD[Int] = rdd.distinct(3)
   ```

10. coalesce 合并分区

    ```scala
    //功能 对RDD中的数据重新分区,根据参数2决定是否包含shuffle过程
    //参数1 新的分区数
    //参数2 (可选,默认为false)是否启动shuffle,为true时可以增加或缩减分区,为false时不能增加分区
    val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6),2)
    val rdd1: RDD[Int] = rdd1.coalesce(1)
    val rdd2: RDD[Int] = rdd3.coalesce(3,true)
    ```

11. repartiton 重分区

    ```scala
    //功能 数据重分区,支持增加或缩减,包含shuffle过程,底层实现为coalesce算子
    //参数 新的分区数
    val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5,6),2)
    val rdd1: RDD[Int] = rdd3.repartition(3)
    ```

12. sortBy 排序

    ```scala
    //功能 对RDD中的数据排序,默认为升序,包含shuffle过程
    //参数1 自定义函数,参数为每条数据,返回值作为排序的参数可以是任意可排序类型
    //参数2 (可选,默认为true)是否升序,为false时降序排序
    //参数3 (可选,默认与原分区数一致)排序后的分区数
    val rdd: RDD[Int] = sc.makeRDD(List(1,5,2,4,3))
    val rdd1: RDD[Int] = rdd.sortBy(num=>num)
    val rdd2: RDD[Int] = rdd.sortBy(num=>num,false,3)
    ```

13. pipe

    ```scala
    
    ```

    



##### 双Value类型

1. intersection 交集

   ```scala
   //功能 取两个RDD的交集,新RDD的分区数与分区数较大的RDD一致,含有shuffle过程
   //参数 与调用算子的RDD数据类型相同的RDD对象
   val rdd1: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd2: RDD[Int] = sc.makeRDD(List(3,4,5,6),2)
   val rdd3: RDD[Int] = rdd1.intersection(rdd2)
   ```

2. union 并集

   ```scala
   //功能 取两个RDD的并集,新RDD的分区数为两个RDD的分区数之和,无shuffle过程
   //参数 与调用算子的RDD数据类型相同的RDD对象
   val rdd1: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd2: RDD[Int] = sc.makeRDD(List(3,4,5,6),2)
   val rdd3: RDD[Int] = rdd1.union(rdd2)
   ```

3. subtract 差集

   ```scala
   //功能 取两个RDD的差集,新RDD的分区数与调用算子的RDD分区数一致
   //参数 与调用算子的RDD数据类型相同的RDD对象
   val rdd1: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd2: RDD[Int] = sc.makeRDD(List(3,4,5,6),2)
   val rdd3: RDD[Int] = rdd1.subtract(rdd2)
   ```

4. zip 拉链

   ```scala
   //功能 将两个RDD中的数据一一配对,返回键值对组成的集合
   //参数 与调用算子的RDD分区数、元素个数都相同的RDD
   val rdd1: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val rdd2: RDD[Int] = sc.makeRDD(List(3,4,5,6),2)
   val rdd3: RDD[(Int, Int)] = rdd1.zip(rdd2)
   ```



##### Key-Value类型

**Spark通过隐式转换对数据类型为键值对的RDD扩充了大量额外的转换算子**

1. partitionBy 重分区

   ```scala
   //功能 只用分区器对数据重新分区
   //参数 分区器对象
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3)))
   
   //1.HashPartitoner 默认的分区器,将数据key值的HashCode对分区数取模后确定分区
   //HashPartitoner对象参数 分区数
   val rdd1: RDD[(String, Int)] = rdd.partitionBy(new HashPartitioner(2))
   
   //2.RangePartitioner sortBy算子中自动调用的分区器,对排序后的数据分区
   
   //3.自定义分区器 继承Partitioner类,重写1个属性和1个方法
   class MyPartitioner(num: Int) extends Partitioner {
       //numPartitions表示分区数
       override def numPartitions: Int = num
       //getPartition方法参数为key值,返回分配的分区号(从0开始)
       override def getPartition(key: Any): Int = {
           key match {
               case "b" => 0
               case _ => 1
           }
       }
   }
   val rdd2: RDD[(String, Int)] = rdd.partitionBy(new MyPartitioner(2))
   ```

2. groupByKey 分组

   ```scala
   //功能 根据key对数据分组,将同组的所有value值转换为集合,包含shuffle过程
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3),("a",4),("b",5),("c",6)))
   val rdd1: RDD[(String, Iterable[Int])] = rdd.groupByKey()
   ```

3. reduceByKey 分组聚合

   ```scala
   //功能 根据key对数据分组,对同组数据的value聚合两两计算,包含shuffle过程
   //参数 自定义函数,参数为两条数据的value值,返回值类型与value一致 (Int,Int)=>Int
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3),("a",4),("b",5),("c",6)))
   val rdd1: RDD[(String, Int)] = rdd.reduceByKey(_+_)
   ```

4. foldByKey 分组折叠（含有初始值的reduceByKey）

   ```scala
   //功能 根据key对数据分组,从初始值开始,对同组数据的value聚合两两计算,包含shuffle过程
   //参数列表1 参与两两运算的首个初始值,数据类型与value一致
   //参数列表2 自定义函数,参数为两条数据的value值,返回值类型与value一致 (Int,Int)=>Int
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3),("a",4),("b",5),("c",6)))
   val rdd2: RDD[(String, Int)] = rdd.foldByKey(0)(_+_)
   ```

5. aggregateByKey 分组分区折叠

   ```scala
   //功能 根据key对数据分组,先在分区内从初始值开始,对同组数据的value聚合两两计算,然后对多个分区的同组数据的value聚合两两计算,两次聚合逻辑可以不同
   //参数列表1 参与两两运算的首个初始值,数据类型任意(决定最终的value类型)
   //参数列表2:
   	//参数1 分区内聚合的自定义函数,参数为聚合结果和下条数据的value值,返回值类型与初始值一致 (U,Int)=>U
   	//参数2 分区间聚合的自定义函数,参数为两条数据的value值,返回值类型与初始值一致 (U,U)=>U
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3),("a",4),("b",5),("c",6)),2)
   val rdd1: RDD[(String, Int)] = rdd.aggregateByKey(0)(
       (x, y) => math.max(x, y),
       (x, y) => x + y
   )
   //说明: 先计算每个分区中每个key的最大值,再计算每个分区的key的最大值之和
   ```

6. combineByKey 改变数据类型的分组分区聚合

   ```scala
   //功能 根据key对数据分组,先在分区内转换同组中首条数据的value的结构,再对分区内的同组数据的value聚合两两计算,最后对多个分区的同组数据聚合两两计算
   //参数1 自定义函数,参数为首条数据的value,返回值为任意类型的数据(决定最终的value类型) Int=>C
   //参数2 自定义函数,参数为聚合结果和下条数据的value值,返回值类型与参数1返回值一致 (C,Int)=>C
   //参数3 自定义函数,参数为两条数据的value值,返回值类型与参数1返回值一致 (C,C)=>C
   val rdd: RDD[String] = sc.makeRDD(List("hello","world","hello","scala","spark","hello","hadoop","spark","scala"))
   val rdd6: RDD[(String, Int)] = rdd.map((_, 1)).combineByKey(
       cnt => cnt,
       (cnt1: Int, cnt2) => cnt1 + cnt2,
       (cnt1: Int, cnt2: Int) => cnt1 + cnt2
   )
   //注意:分区内及分区间的聚合函数的参数类型无法确定,需要显式声明
   ```

7. sortByKey

   ```scala
   //功能 对key排序,与value无关,有shuffle过程
   //参数1 是否使用升序 true升序(默认) false降序
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("a",5),("b",4),("a",3),("b",6)))
   val rdd1: RDD[(String, Int)] = rdd.sortByKey()
   val rdd2: RDD[(String, Int)] = rdd.sortByKey(false)
   
   //key值为自定义类时,可通过实现Ordered特质实现可排序
   class User extends Ordered[User]{
       override def compare(that: User): Int = {
           1
       }
   }
   ```

8. join

   ```scala
   //功能 根据key关联两个RDD,新的key与join前相同,新value为两个RDD的value组成的对偶元组
   val rdd1: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3)))
   val rdd2: RDD[(String, Int)] = sc.makeRDD(List(("a",4),("b",5),("a",6)))
   val rdd3: RDD[(String, (Int, Int))] = rdd1.join(rdd2)
   //rdd3的元素 (a,(1,4)),(a,(1,6)),(b,(2,5))
   ```

9. leftOuterJoin

   ```scala
   //功能 与join相似,不同在于当调用算子的RDD中的key在参数RDD中不存在时也使用None返回
   val rdd1: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3)))
   val rdd2: RDD[(String, Int)] = sc.makeRDD(List(("a",4),("b",5),("a",6)))
   val rdd3: RDD[(String, (Int, Option[Int]))] = rdd1.leftOuterJoin(rdd2)
   val rdd4: RDD[(String, (Int, Any))] = rdd4.mapValues(
       kv => {
           (kv._1, kv._2.getOrElse(None))
       }
   )
   //rdd4的元素 (a,(1,4)),(a,(1,6)),(b,(2,5)),(c,(3,None))
   ```

10. cogroup

    ```scala
    //功能 根据key关联两个RDD,新key的与cogroup前相同,新value为RDD1和RDD2中该key的所有value组成的两个迭代器对象,没有shuffle过程
    val rdd1: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("b",3)))
    val rdd2: RDD[(String, Int)] = sc.makeRDD(List(("a",4),("b",5),("c",6)))
    val rdd3: RDD[(String, (Iterable[Int], Iterable[Int]))] = rdd1.cogroup(rdd2)
    //rdd3元素如下
    // (a,(CompactBuffer(1),CompactBuffer(4)))
    // (b,(CompactBuffer(2, 3),CompactBuffer(5)))
    // (c,(CompactBuffer(),CompactBuffer(6)))
    ```



#### 5.1.5 RDD行动算子

- 行动算子在在执行时会提交Job，不产生新的RDD

1. reduce 聚合

   ```scala
   //功能 RDD中元素两两聚合计算,返回计算结果
   //自定义函数参数 RDD中的两个元素
   //自定义函数返回值 类型与RDD元素类型一致
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
   val rdd1: Int = rdd.reduce(_+_)
   ```

2. collect 采集

   ```scala
   //功能 将RDD转换为数组后返回
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
   val arr: Array[Int] = rdd.collect()
   ```

3. count 计数

   ```scala
   //功能 返回RDD中元素个数
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
   val cnt: Long = rdd.count()
   ```

4. first

   ```scala
   //功能 返回RDD中的第一个元素
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
   val first: Int = rdd.first()
   ```

5. take

   ```scala
   //功能 返回RDD中的前n个元素
   //参数 元素个数
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
   val take: Array[Int] = rdd.take(3)
   ```

6. takeOrdered

   ```scala
   //功能 返回RDD排序后的前n个元素
   //参数1 元素个数
   //参数2 排序方式
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
   val take: Array[Int] = rdd.takeOrdered(3)(Ordering.Int.reverse)
   ```

7. aggregate

   ```scala
   //功能 首先在分区内从初始值开始两两聚合计算,然后将各分区结果从初始值开始两两聚合计算
   //参数1 初始值,决定聚合结果的数据类型
   //参数2 分区内的聚合函数 (U,Int)=>U
   //参数3 分区间的聚合函数 (U,U)=>U
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val agr: Int = rdd.aggregate(10)(_+_,_+_)  //40
   ```

8. fold

   ```scala
   //功能 同aggregate,分区内和分区间的聚合规则相同
   //参数1 初始值,数据类型与RDD中的元素类型一致
   //参数2 聚合函数 (Int,Int)=>Int
   val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4),2)
   val fold: Int = rdd.fold(10)(_+_) //40
   ```

9. countByKey

   ```scala
   //功能 根据key分组,返回key与原RDD相同,value为key的个数的键值对型RDD
   val rdd: RDD[(Int, String)] = sc.makeRDD(List((1, "a"), (1, "a"), (1, "a"), (2, "b"), (3, "c")))
   val rdd1: collection.Map[Int, Long] = rdd.countByKey()
   ```

10. countByValue

    ```scala
    //功能1 根据键值对分组,返回key为键值对,value为键值对个数的键值对型RDD
    //功能2 根据元素分组,返回key为元素,vlaue为元素个数的键值对型RDD
    val rdd: RDD[(Int, String)] = sc.makeRDD(List((1, "a"), (1, "a"), (1, "a"), (2, "b"), (3, "c")))
    val rdd1: collection.Map[(Int, String), Long] = rdd.countByValue()
    ```

11. save

    ```scala
    //功能 保存RDD中的数据到不同格式的文件中
    val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
    rdd.saveAsTextFile("output1")
    rdd.saveAsObjectFile("output2")
    rdd.map((_,1)).saveAsSequenceFile("output3")
    ```

12. foreach

    ```scala
    //功能 循环遍历RDD中的每一个元素,每一次循环会发送到不同的excutor中执行,因此返回结果是无序的
    val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
    rdd.foreach(println)
    ```



#### 5.1.6 RDD序列化

- 闭包检测

  - 在Spark中，算子内的代码会发送到Executor中执行，当算子中使用到算子外的内容时，就形成了闭包效果。
  - 在执行计算任务前，需要确认所有闭包对象是否支持序列化，这一过程称为闭包检测。

- 序列化类的方法和属性

  ```scala
  //1.混入Serializable特质
  class User extends Serializable(){
      val age = 20
  }
  
  //2.声明为样例类
  case class User(){
      val age = 20
  }
  ```

- Kryo序列化框架

  - Spark2.0起支持的轻型序列化框架

  - 在Spark的shuffle过程中，值类型和字符串类型的数据默认使用了kryo进行数据的序列化

  - 相比于Java的序列化机制速度更快，数据量更小，但使用前需要进行配置
  
    ```scala
    val conf: SparkConf = new SparkConf()
    .setAppName("SerDemo")
    .setMaster("local[*]")
    //1.替换默认的序列化机制
    .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    //2.注册需要使用 kryo 序列化的自定义类
    .registerKryoClasses(Array(classOf[User]))
    
    val sc = new SparkContext(conf)
    
    //3.声明自定义类
    case class User(){
        val age = 20
    }
    ```



#### 5.1.7 RDD依赖关系

1. RDD血缘关系

   ```scala
   //RDD的Lineage会记录RDD的元数据信息和转换行为，当RDD中部分分区数据丢失时，可以通过这些信息重新计算并恢复丢失的数据
   //查看RDD的血缘
   println(rdd.toDebugString)
   ```

2. RDD依赖关系

   ```scala
   //窄依赖(NarrowDependency)		父RDD中的一个分区被子RDD中的一个分区依赖,如map
   //宽依赖(ShuffleDependency)	父RDD中的一个分区被子RDD中的多个分区依赖,如reduceBy
   ```

3. stage阶段划分

   1. spark中一个行动算子的执行往往由一或多个阶段（stage）组成
   2. 阶段划分的根本依据为算子是否会产生shuffle过程，即是否含有宽依赖
   3. 一个计算过程的 **stage总数 = 宽依赖数 + 1**

4. RDD任务划分

   1. Application：初始化一个SparkContext就生成了一个Application
   2. Job：Application中的1个行动算子（包括个别转换算子）生成1个Job
   3. Stage：Job中的宽依赖数+1即为Job的Stage个数
   4. Task：每个Stage的最终分区数决定了Task个数



#### 5.1.8 RDD持久化

- 在一个Application中，每个行动算子的执行都需要按照依赖关系从头执行算子，而某些中间RDD可能被多个Job重复使用，若每次都从头执行则效率过低，因此可以将这种被重复使用的RDD的数据持久化，提高效率。

##### Cache 缓存

1. **保留RDD的血缘关系，将RDD中的数据缓存到内存中**
2. 底层实现调用了persist方法
3. 内存不足导致缓存数据丢失时，系统会自动判断缓存失效，从头执行算子
4. 缓存命令不会立刻执行，而是在RDD数据首次计算完成后缓存
5. **含有shuffle过程的算子默认会将计算结果缓存**

```scala
val sc = new SparkContext(new SparkConf().setMaster("local[*]").setAppName(""))
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
val rdd1: RDD[Any] = rdd.map(
    num => {
        println("map")
        (num, 1)
    }
)
//1.使用cache方法缓存RDD中的数据
rdd1.cache()
//执行2个行动算子,map中的打印操作只会执行一次
println(rdd1.collect().mkString(","))
println(rdd1.collect().mkString(";"))
```



##### CheckPoint 检查点

1. 切断RDD的血缘关系，将RDD中的数据存储到磁盘（生产环境为HDFS）中
2. CheckPoint为了确保计算结果的正确，默认会在首次执行目标RDD的计算后创建新Job再次计算后再写入磁盘
3. 配合Cache使用可在RDD执行后直接写入磁盘（省去一次计算）

```scala
val sc = new SparkContext(new SparkConf().setMaster("local[*]").setAppName("checkpoint"))
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
val rdd1: RDD[Any] = rdd.map(
    num => {
        println("map")
        (num, 1)
    }
)
//1.配置检查点文件的存储目录
sc.setCheckpointDir("cp")
//2.使用cache缓存RDD中的数据
rdd1.cache()
//3.建立checkpoint
rdd1.checkpoint()
//执行2个行动算子,map中的打印操作只会执行一次,相对路径cp文件下会存储检查点文件
println(rdd1.collect().mkString(","))
println(rdd1.collect().mkString(";"))
```



#### 5.1.9 RDD文件读取与保存

- text文件

  ```scala
  //读取文件
  val inputRDD: RDD[String] = sc.textFile("input/1.txt")
  
  //保存文件
  inputRDD.saveAsTextFile("output")
  ```

- sequence文件

  ```scala
  //读取文件
  sc.sequenceFile[Int,Int]("input").collect().foreach(println)
  
  //保存文件
  seqRDD.saveAsSequenceFile("output")
  ```

- object文件

  ```scala
  //读取文件
  sc.objectFile[Int]("input").collect().foreach(println)
  
  //保存文件
  objRDD.saveAsObjectFile("output")
  ```



### 5.2 累加器

- 功能：累加器用于实现多计算节点上数值或数据的叠加（分布式共享只写变量）
- 原理：Driver将累加器对象发送到Executor中执行计算（调用add方法），Executor计算结束后返回累加器对象，Driver对返回的累加器对象做聚合计算（自动调用merge方法），得到最终结果（调用value方法）



#### 5.2.1 系统累加器

- 系统默认提供了三种累加器：
  - LongAccumulator：累加整形数据，返回Long型结果
  - DoubleAccumulator：累加浮点型数据，返回Double型结果
  - CollectionAccumulator：累加集合元素，返回List()型结果

```scala
//示例:累加List中的数据
val sc = new SparkContext(new SparkConf().setMaster("local[*]").setAppName("acc"))
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
//1.获取累加器对象
val sum: LongAccumulator = sc.longAccumulator("sum")
//2.发送计算任务到Executor节点
rdd.foreach(
    num => {
        //3.在Excutor节点中调用累加器对象的add方法
        sum.add(num)
    }
)
//4.调用累加器对象的value方法获取聚合计算的结果
val value: lang.Long = sum.value
```



#### 5.2.2 自定义累加器

```scala
//使用自定义累加器实现WordCount
object Acc {
    def main(args: Array[String]): Unit = {

        val sc = new SparkContext(new SparkConf().setMaster("local[*]").setAppName(""))
        val rdd: RDD[String] = sc.makeRDD(List("hello","world","hello","scala","spark"))

        //1.新建并注册自定义累加器对象
        val acc = new MyWordCountAccumulator
        sc.register(acc)

        //2.分发计算任务到Executor
        rdd.foreach(
            acc.add(_)
        )

        //3.获取累加器中的数据
        println(acc.value)

        sc.stop()

    }

    //4.自定义累加器 继承AccumulatorV2类,重写6个方法
    //4.1 定义泛型 [累加器输入值类型,累加器返回值类型]
    class MyWordCountAccumulator extends AccumulatorV2[String,mutable.Map[String,Int]]{

        //4.2 声明存储WordCount结果的集合
        var map = mutable.Map[String,Int]()

        //4.3 判断累加器是否初始化的方法
        override def isZero: Boolean = {
            map.isEmpty
        }

        //4.4 复制初始化的累加器的方法
        override def copy(): AccumulatorV2[String, mutable.Map[String, Int]] = {
            new MyWordCountAccumulator
        }

        //4.5 重置累加器的方法
        override def reset(): Unit = {
            map.clear()
        }

        //4.6 Executor中向累加器添加或更新数据的方法
        override def add(v: String): Unit = {
            map.update(v, map.getOrElse(v,0) + 1)
        }

        //4.7 多个Executor返回的累加器对象聚合的方法
        override def merge(other: AccumulatorV2[String, mutable.Map[String, Int]]): Unit = {
            map = map.foldLeft(other.value)(
                (map, kv) => {
                    map.update(kv._1, map.getOrElse(kv._1,0) + kv._2)
                    map
                }
            )
        }

        //4.8 返回累加器中数据的方法
        override def value: mutable.Map[String, Int] = {
            map
        }
    }
}
```



### 5.3 广播变量

- 当一个Excutor中的多个Task都需要序列化发送一个较大的只读对象时，会出现大量的数据冗余，这是可以将该数据定义为广播变量发送到Executor节点，该节点下的所有Task可以共享该数据

```scala
//示例 使用广播变量实现类join功能
val sc = new SparkContext(new SparkConf().setMaster("local[*]").setAppName(""))

val rdd: RDD[(String, Int)] = sc.makeRDD(List(("a",1),("b",2),("c",3)))
val list = List(("a",4),("b",5),("c",6))
//1.将list封装为广播变量
val bc: Broadcast[List[(String, Int)]] = sc.broadcast(list)

val rdd1: RDD[(String, (Int, Int))] = rdd.map(
    kv => {
        var num: Int = 0
        //2.调用value方法获取广播变量
        for (kv2 <- bc.value) {
            if (kv._1 == kv2._1) {
                num = kv2._2
            }
        }
        (kv._1, (kv._2, num))
    }
)
rdd1.collect()
sc.stop()
```



## 第六章 Saprk实战

### 6.1 Spark应用框架编写

- Spark应用框架使用三层结构模型：Controller、Service、DAO

- Application特质的编写

  ```scala
  trait TApplication {
  
      var envData: Any = null
  
      //声明启动方法,柯里化后两个参数分别传入环境及业务逻辑
      def start (t: String = "jdbc")(op: => Unit): Unit = {
  
          //1.初始化环境
          if (t == "spark"){
              envData = EnvUtil.getEnv()
          }
  
          //2.业务逻辑
          try {
              op
          } catch {
              case ex: Exception => println("业务执行失败" + ex.getMessage)
          }
  
          //3.关闭环境
          if (t == "spark"){
              EnvUtil.clear()
          }
  
      }
  }
  ```

- Controller特质的编写

  ```scala
  trait TController {
  
      //声明抽象方法
      def execute(): Unit
  
  }
  ```

- Service特质的编写

  ```scala
  trait TService {
  
      //声明2个方法用于在实现类中继承重写具体的业务逻辑
      def analysis(): Any = {}
      def analysis(data: Any) = {}
  
  }
  ```

- DAO特质的编写

  ```scala
  trait TDAO {
  
      //声明获取数据源的方法
      def readFile(path: String) = {
          EnvUtil.getEnv().textFile(path)
      }
  
  }
  ```

- EnvUtil类的编写

  ```scala
  object EnvUtil {
  
      //声明共享内存中的对象
      private val scLocal = new ThreadLocal[SparkContext]
  
      def getEnv()={
          //从当前线程的共享内存中获取对象
          var sc: SparkContext = scLocal.get()
          if (sc == null) {
              //若未获取到,创建新的对象
              val conf: SparkConf = 
              	new SparkConf().setMaster("local").setAppName("sparkApplication")
              sc = new SparkContext(conf)
              //写入共享内存对象
              scLocal.set(sc)
          }
          sc
      }
  
      def clear(): Unit ={
          //获取共享内存对象并结束sc
          scLocal.get().stop()
          //从内存中清除共享对象
          scLocal.remove()
      }
  
  }
  ```

  

### 6.2  热门品类Top10

#### 6.2.1 使用shuffle算子实现

1. 编写Application

   ```scala
   object MyApplication extends App with TApplication{
   
       start("spark"){
           val controller = new MyController
           controller.execute()
       }
   
   }
   ```

2. 编写Controller

   ```scala
   class MyController extends TController{
   
       private val service = new MyService
   
       override def execute(): Unit = {
           val result = service.analysis()
           println(result.mkString("\n"))
       }
   
   }
   ```

3. 编写Service

   ```scala
   class MyService extends TService{
   
       private val dao = new MyDAO
   
       override def analysis() = {
   
           val actionRDD: RDD[bean.UserVisitAction] = dao.getActions("F:\\atguigu\\scala\\idea\\input\\user_visit_action.txt")
   
           val flatRDD: RDD[(String, (Int, Int, Int))] = actionRDD.flatMap(
               action => {
                   if (action.click_category_id != -1) {
                       List((action.click_category_id.toString, (1, 0, 0)))
                   } else if (action.order_category_ids != "null") {
                       val ids: Array[String] = action.order_category_ids.split(",")
                       ids.map((_, (0, 1, 0)))
                   } else if (action.pay_category_ids != "null") {
                       val ids: Array[String] = action.pay_category_ids.split(",")
                       ids.map((_, (0, 0, 1)))
                   } else {
                       Nil
                   }
               }
           )
   
           val reduceRDD: RDD[(String, (Int, Int, Int))] = flatRDD.reduceByKey {
               case ((a1, b1, c1), (a2, b2, c2)) => {
                   (a1 + a2, b1 + b2, c1 + c2)
               }
           }
   
           val result: Array[(String, (Int, Int, Int))] = reduceRDD.sortBy(_._2,false).take(10)
   
           result
   
       }
   
   }
   ```

4. 编写DAO

   ```scala
   class MyDAO extends TDAO{
   
       def getActions(path: String) = {
   
           val rdd: RDD[String] = EnvUtil.getEnv().textFile(path)
           val rdd1: RDD[UserVisitAction] = rdd.map(
               line => {
                   val strs: Array[String] = line.split("_")
                   val action = UserVisitAction(
                       strs(0),
                       strs(1).toLong,
                       strs(2),
                       strs(3).toLong,
                       strs(4),
                       strs(5),
                       strs(6).toLong,
                       strs(7).toLong,
                       strs(8),
                       strs(9),
                       strs(10),
                       strs(11),
                       strs(12).toLong
                   )
                   action
               }
           )
           rdd1
   
       }
   }
   ```

#### 6.2.2 使用累加器实现

1. 编写Accumulator

   ```scala
   class MyAcc extends AccumulatorV2[UserVisitAction,mutable.Map[String,(Long,Long,Long)]]{
   
       private var map: mutable.Map[String, (Long, Long, Long)] = mutable.Map[String, (Long, Long, Long)]()
   
       override def isZero: Boolean = {map.isEmpty}
   
       override def copy(): AccumulatorV2[UserVisitAction, mutable.Map[String, (Long, Long, Long)]] = {new MyAcc}
   
       override def reset(): Unit = {map.clear()}
   
       override def add(action: UserVisitAction): Unit = {
           if (action.click_category_id != -1){
               var count: (Long, Long, Long) = map.getOrElse(action.click_category_id.toString, (0,0,0))
               count = (count._1 + 1, count._2, count._3)
               map.update(action.click_category_id.toString, count)
           } else if (action.order_category_ids != "null") {
               val arr: Array[String] = action.order_category_ids.split(",")
               arr.foreach(
                   category => {
                       var count: (Long, Long, Long) = map.getOrElse(category, (0,0,0))
                       count = (count._1, count._2 + 1, count._3)
                       map.update(category, count)
                   }
               )
           } else if (action.pay_category_ids != "null") {
               val arr: Array[String] = action.pay_category_ids.split(",")
               arr.foreach(
                   category => {
                       var count: (Long, Long, Long) = map.getOrElse(category, (0,0,0))
                       count = (count._1, count._2, count._3 + 1)
                       map.update(category, count)
                   }
               )
           }
       }
   
       override def merge(other: AccumulatorV2[UserVisitAction, mutable.Map[String, (Long, Long, Long)]]): Unit = {
           map = map.foldLeft(other.value){
               (map, kv) => {
                   var count: (Long, Long, Long) = map.getOrElse(kv._1, (0,0,0))
                   count = (count._1 + kv._2._1, count._2 + kv._2._2, count._3 + kv._2._3)
                   map.update(kv._1, count)
                   map
               }
           }
       }
   
       override def value: mutable.Map[String, (Long, Long, Long)] = {map}
   }
   ```

2. 编写Service

   ```scala
   class MyService extends TService{
   
       private val dao = new MyDAO
   
       override def analysis() = {
   
           val actionRDD: RDD[bean.UserVisitAction] = dao.getActions("F:\\atguigu\\scala\\idea\\input\\user_visit_action.txt")
   
           val acc = new MyAcc
           EnvUtil.getEnv().register(acc)
   
           actionRDD.foreach(acc.add(_))
   
           val accList: List[(String, (Long, Long, Long))] = acc.value.toList
   
           val result = accList.sortBy(_._2).reverse.take(10)
   
           result
   
       }
   
   }
   ```



### 6.3  页面单跳转换率统计

```scala
class MyService extends TService{

    private val dao = new MyDAO

    override def analysis() = {

        val actionRDD: RDD[bean.UserVisitAction] = dao.getActions("F:\\atguigu\\scala\\idea\\input\\user_visit_action.txt")
        val list = List(1,2,3,4,5,6,7)
        val baseList: List[String] = list.zip(list.tail).map(kv=>kv._1+"-"+kv._2)

        //1.计算转换率分母 统计指定页面的访问次数 ("1",999)
        //1.1 根据指定页面id过滤数据
        val filterRDD: RDD[bean.UserVisitAction] = actionRDD.filter(action=>list.init.contains(action.page_id.toInt))
        //1.2 映射提取页面id字段并添加count计数 ("1",1)
        val mapRDD: RDD[(String, Int)] = filterRDD.map(action=>(action.page_id.toString, 1))
        //1.3 聚合计算count
        val resultRDD1: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)
        val result1: Map[String, Int] = resultRDD1.collect().toMap

        //2.计算转换率分子 统计从指定页面跳转到目标页面的次数 ("1-2",99)
        //2.1 按照session分组
        val sessionRDD: RDD[(String, Iterable[bean.UserVisitAction])] = actionRDD.groupBy(_.session_id)
        //2.2 组内映射,按照访问时间排序,映射提取页面id字段,zip得到pageFlow,过滤目标pageFlow,再次map加上次数 ("1-2",1)
        val pageFlowRDD: RDD[(String, List[(String, Int)])] = sessionRDD.mapValues(
            iter => {
                //按照访问时间排序
                val sortList: List[bean.UserVisitAction] = iter.toList.sortBy(_.action_time)
                //映射提取页面id字段
                val mapList: List[Long] = sortList.map(_.page_id)
                //拉链并映射后获取pageFlow信息,并添加count用于计数 ("1-2",99)
                val zipList: List[(String, Int)] = mapList.zip(mapList.tail).map(kv => (kv._1 + "-" + kv._2, 1))
                zipList
            }
        )
        //2.3 扁平映射保留values部分
        val flatMapRDD: RDD[(String, Int)] = pageFlowRDD.flatMap(_._2)
        //2.4 聚合计算count
        val resultRDD2: RDD[(String, Int)] = flatMapRDD.reduceByKey(_+_)
        val result2: Map[String, Int] = resultRDD2.collect().toMap

        //3.计算转换率 通过页面id关联后相除
        val result: Map[String, Double] = result2.map(
            kv => {
                (kv._1, kv._2.toDouble / result1(kv._1.substring(0,1)))
            }
        )
        result

    }

}
```



### 6.4 页面停留平均时长统计

```scala
class MyService extends TService{

    private val dao = new MyDAO

    override def analysis() = {

        val actionRDD: RDD[bean.UserVisitAction] = dao.getActions("input/user_visit_action.txt")
        val list = List(1,2,3,4,5,6,7)

        //1.按照sessionID分组
        val sessionRDD: RDD[(String, Iterable[bean.UserVisitAction])] = actionRDD.groupBy(_.session_id)

        //2.组内映射 取出页面id、动作时间字段,再使用zip map得到(1-2,staytime)型的结果(String,Long)
        val pageTimeStayRDD: RDD[(String, List[(String, Long)])] = sessionRDD.mapValues(
            iter => {

                val format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss")

                val pageTimeList: List[(Long, Long)] = iter.toList.map(
                    action => {
                        (action.page_id, format.parse(action.action_time).getTime)
                    }
                )

                val sortPageTimeRDD: List[(Long, Long)] = pageTimeList.sortBy(_._2)

                val zipPageTimeList: List[((Long, Long), (Long, Long))] = sortPageTimeRDD.zip(sortPageTimeRDD.tail)


                val pageTimeStayList: List[(String, Long)] = zipPageTimeList.map {
                    case ((page1, time1), (page2, time2)) => {
                        (page1.toString, time2 - time1)
                    }
                }
                pageTimeStayList
            }
        )

        //3.map保留value, 筛选目标pageflow的数据
        val fileterRDD: RDD[(String, Long)] = pageTimeStayRDD.flatMap(_._2)

        //4.聚合
        val reduceRDD: RDD[(String, (Long, Int))] = fileterRDD.map(kv => (kv._1, (kv._2, 1))).reduceByKey {
            case ((time1, count1), (time2, count2)) => {
                (time1 + time2, count1 + count2)
            }
        }

        //5.求平均值(秒)
        val result: RDD[(String, Double)] = reduceRDD.mapValues(kv => kv._1.toDouble/kv._2/1000)

        result.collect()

    }

}
```



# Spark内核

## 1. Spark应用提交

### 1.1 向Yarn提交应用流程 

![image-20200617234251867](Spark.assets/image-20200617234251867.png)

1. 使用spark-submit提交应用到Yarn

   ```shell
   bin/spark-submit \
   --class org.apache.spark.examples.SparkPi \
   --master yarn \
   #使用yarn集群部署模式需要配置参数--deploy-mode
   --deploy-mode cluster \
   ./examples/jars/spark-examples_2.12-2.4.5.jar \
   10
   ```

2. Yarn接收命令后在NM节点启动AM进程

3. AM创建线程启动Driver，执行main方法，创建SparkContext环境对象

4. AM向RM注册，申请资源

5. RM向AM返回可用资源列表，AM根据本地化级别对资源进行匹配后启动资源

6. AM访问启动资源的NM，启动ExecutorBackend进程

7. ExecutorBackend启动后向Driver进行注册（反向注册）

8. Driver向ExecutorBackend反馈注册完毕信息，Executor被正式创建，准备接收并执行Task



### 1.2 源码中的应用提交流程

1. 使用bin/spark-submit命令会生成JVM，**启动SparkSubmit进程**
   - 执行SparkSubmit的main()方法，new SparkSubmit构建SparkSubmit对象
   - 执行doSubmit()方法
     - 执行parseArguments()方法，该方法用于解析spark-submit命令中传入的所有参数；
     - 执行parse方法读取action参数（默认为SUBMIT），当参数值为SUBMIT时执行submit方法提交应用；
       - 执行runMain方法，将之前读取的参数作为形参传入 
         - 执行prepareSubmitEnvironment(args)方法准备提交的环境
           childMainClass在yarn-cluster部署模式下为org.apache.spark.deploy.yarn.YarnClusterApplication
           在yarn-client部署模式下为mainClass，即命令参数中传入的主类
         - mainClass = Utils.classForName(childMainClass)
           通过类名获取类
         - app = mainClass.newInstance().asInstanceOf[SparkApplication]
           反射获取类的对象
         - app.start()
           执行对象（YarnClusterApplication）的start方法
2. 执行YarnClusterApplication的start()方法
   - new Client创建Client对象
     - yarnClient = YarnClient.createYarnClient
       **创建Yarn客户端对象**，包括RM对象
   - 执行run()方法
     - submitApplication()方法提交应用，根据hadoop配置初始化yarn集群并启动客户端，建立连接
       - containerContext = createContainerLaunchContext(newAppResponse)
         封装启动AM的JVM参数
       - appContext = createApplicationSubmissionContext(newApp,containerContext)
         封装AM的启动指令
       - yarnClient.submitApplication(appContext)
         发送指令及相关资源到Yarn，**启动AM进程**
3. 执行ApplicationMaster的main()方法
   - new ApplicationMaster(amArgs)使用传入的参数创建AM对象
   - 执行run()方法
     - 执行runImpl()方法
       - 执行runDriver()方法
         - userClassThread = startUserApplication()
           创建新的线程启动用户的应用，即**启动Driver线程**，执行main方法，生成SparkContext环境对象
           **从此方法开始spark应用程序的执行分为了资源调度和任务执行两条线**
           **资源调度线负责Executor的注册和启动**
           **任务执行线负责Job的切分和Task的创建**
         - ThreadUtils.awaitResult
           阻塞线程，等待返回SparkContext环境对象
         - registerAM()
           AM向RM注册
         - 执行createAllocator()方法
           - client.createAllocator
             创建资源分配器对象
           - allocator.allocateResources()
             分配资源
             - allocateResponse.getAllocatedContainers()
               从RM获取可用的资源列表
             - handleAllocatedContainers()
               对可用资源进行匹配（根据本地化级别）
               - 执行runAllocatedContainers()方法
                 - startContainer
                   启动Container容器
                 - 执行run()方法
                   - prepareCommand()
                     封装Executor的启动参数
                   - nmClient.startContainer(container.get, ctx)
                     **启动CoarseGrainedExecutorBackend进程**
4. 执行CoarseGrainedExecutorBackend的main()方法
   - 执行run()方法
   - 执行onStart()方法，向Dirver注册Executor
5. Driver中的SparkContext环境对象执行receiveAndReply(RpcCallContext)方法
   - 执行executorRef.send(RegisteredExcutor)方法
     Driver反馈注册完成信息到Executor
6. 执行CoarseGrainedExecutorBackend的receive()方法
   - executor = new Executor
     真正**创建Executor对象**



### 1.3 本地化级别

- AM在获取到RM返回的可用资源列表后，需要选择NM节点的Executor执行计算，该选择遵循首选位置规则
- 数据的存储位置与Task的执行位置之间的关系称为本地化级别
  1. 进程本地化：数据与Task在同一Executor（同一CPU执行）中
  2. 节点本地化：数据与Task在同一节点中
  3. 机架本地化：数据与Task在同一机架中
- 首选位置的优先级别
  1. PROCESS_LOCAL（进程本地化）
  2. NODE_LOCAL（节点本地化）
  3. RACK_LOCAL（机架本地化）
  4. NO_PREF（）
  5. ANY



## 2.  Spark内部组件及通信

### 2.1 通信框架的发展

1. Spark早期版本中使用Akka框架作为内部通信通信
2. Spark1.3中引入了Netty通信框架，Netty框架基于AIO（异步通信）开发，但由于AIO与Linux兼容性较差，因此在Linux中还是使用NIO实现
3. Spark2.x起Netty框架完全取代了Akka，作为Spark的内部通信组件



### 2.2 通信过程

<img src="Spark.assets/image-20200618112410820.png" alt="image-20200618112410820" style="zoom:67%;" />

- 在Spark中，一个Client/Master/Worker节点被称为一个EndPoint终端，也称为RPC终端
- 每个EndPoint之间通过RpcEndpoint对象接收数据，通过RpcEndPointRef对象发送数据
- 每个EndPoint仅有1个Inbox，但可以有多个Outbox发送数据到其他终端



## 3. Spark任务调度

### 3.1 SparkContext

SparkContext中封装的核心属性

1. DAGScheduler：Job内部调度器
2. TaskScheduler：任务调度器
3. SchedulerBackend：RPC后台信息交互对象
4. SparkConf：配置信息
5. SparkEnv：封装NettyRpcEnv
6. heartbeatReceiver：接收Executor心跳



### 3.2 Job的执行

1. 行动算子会调用sc.runJob方法，触发作业的执行
2. 根据算子的依赖关系为Job划分Stage，从行动算子开始向前追溯宽依赖，**每有1个宽依赖，Stage数量+1**
   Stage分为**ResultStage**（行动算子产生的Stage）和**ShuffleMapStage**（shuffle算子产生的Stage）
3. Stage划分完成后，会在每个Stage内根据数据的分区数切分Task，**1个分区生成1个Task**
   yarn-cluster模式下，RDD的默认分区数为当前环境可用的最大CPU核数且最小为2
   在算子执行前后，数据的分区数默认不改变，但具有shuffle过程的算子可以通过参数改变计算后的分区数量
4. 1个Stage触发提交的前提条件是**父Stage执行完毕或无父Stage**
   提交时1个Stage中的所有Task会被封装进1个**TaskSet**，同时创建1个**TaskSetManager**对象用于管理该TaskSet
   TaskSetManager会被添加任务调度池（Pool）中等待调度
   任务调度池具有两种调度模式：FIFO（默认，先进先出）、FAIR（公平）
5. Task被取出时会根据Task的首选位置进行本地化级别的判断，进行**降级等待**（默认每个级别3s）处理
   选择好Executor后发送Task到该Executor中执行计算
6. Executor接受Task后将Task装入线程池，在新的线程中执行Task的run()方法，开始计算



## 4. Shuffle过程

- 对于两个Stage间的Shuffle过程，会经过上游Stage的所有Task将数据计算后落盘（Shuffle Map），下游Stage的所有Task读取落盘的数据再计算（Shuffle Reduce）两个阶段。这时，Shuffle Map的数据落盘策略就至关重要。
- 当前版本的shuffle策略为SortShuffle，对于不同的场景，SortShuffle有不同的实现方式
  1. **SortShuffle**（默认）
     按照数据key值排序，将数据分批写入磁盘临时文件，再统一对文件进行聚合，最终一个Executor的所有Task的临时文件merge为一个最终文件，最终文件的数据是有序的，同时生成一份索引文件用于记录下游每个Task的数据在最终文件中的索引位置。
  2. **Bypass SortShuffle**（非聚合类shuffle算子，且task数量<spark.s huffle.sort.bypassMergeThreshold参数值默认200）
     由于小文件数量可以接受，因此对于上游的每一个Task，都写出下游Task数对应的文件数到磁盘，即总文件数为上游Task数*下游Task数，由于整个shuffle阶段不需要进行数据的排序，因此可以节省大量性能



## 5. Spark内存管理

