# Flink

## 第一章 Flink简介

### 1.1 Flink 概述

- flink是一个开源的框架和**分布式**处理引擎，用于对**无界和有界**的数据流进行**有状态**计算。flink可在所有常见的集群环境中运行，以内存的速度执行任意规模的计算。



### 1.2 Flink 特点

1. 事件驱动型（Event-Driven）
   - 从一或多个事件流提取数据，并根据到来的**事件触发**计算、状态更新或其他外部动作。
2. 流处理与批处理共存
   - 批处理的数据特点为有界、持久、大量，需要访问全部记录才能完成的计算，延迟高
   - 流处理的数据特点为无界、实时、无需针对数据集整体、而是对每个数据项进行计算操作，延迟低
   - flink中，既可以批处理有界数据流（不擅长，不如spark），也可以流处理无界数据流（擅长）
3. 分层API
   - ProcessFunction（底层API）：对抽象的状态流操作
   - DataStream API（核心API，常用）：对数据流进行操作
   - SQL/Table API（上层API）：对表操作
4. Flink 主要模块
   - Flink Table & SQL
   - Flink Gelly（图计算）
   - Flink CEP（复杂事件处理）



## 第二章 Flink上手

### 2.1 相关依赖

1. 创建maven工程

2. 修改pom.xml文件，添加flink依赖及打包插件

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-scala_2.12</artifactId>
           <version>1.10.1</version>
       </dependency>
       <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-streaming-scala -->
       <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-streaming-scala_2.12</artifactId>
           <version>1.10.1</version>
       </dependency>
   </dependencies>
   
   <build>
       <plugins>
           <!-- 该插件用于将Scala代码编译成class文件 -->
           <plugin>
               <groupId>net.alchim31.maven</groupId>
               <artifactId>scala-maven-plugin</artifactId>
               <version>3.4.6</version>
               <executions>
                   <execution>
                       <!-- 声明绑定到maven的compile阶段 -->
                       <goals>
                           <goal>compile</goal>
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



### 2.2 批处理wordcount

```scala
object WordCount {
    def main(args: Array[String]): Unit = {

        //1.创建批处理执行环境
        val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
        
        //2.导入隐式转换需要的类
        import org.apache.flink.api.scala._

        //3.从本地文件获取数据流
        val inputPath =  "F:\\atguigu\\flink\\flink\\src\\main\\resources\\hello.txt"
        val inputDataSet: DataSet[String] = env.readTextFile(inputPath)

        //4.批处理逻辑
        val resultDataSet: AggregateDataSet[(String, Int)] = inputDataSet
        .flatMap(_.split(" "))
        .map((_,1))
        .groupBy(0)
        .sum(1)

        //5.输出结果到控制台
        resultDataSet.print()
    }
}
```



### 2.3 流处理wordcount

```scala
object StreamWordCount {
    def main(args: Array[String]): Unit = {

        //1.创建流处理执行环境,配置环境的全局并行度(默认与cpu核数一致)
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
        env.setParallelism(4)
        //并行度配置优先级说明
        //算子并行度(.setParallelism) > 全局并行度(env.setParallelism) > 提交job并行度 > 配置文件并行度

        //2.导入隐式转换需要的类
        import org.apache.flink.api.scala._

        //3.获取参数,创建socket文本流
        val paramTool: ParameterTool = ParameterTool.fromArgs(args)
        val host: String = paramTool.get("host")
        val port: Int = paramTool.getInt("port")
        val inputDataStream: DataStream[String] = env.socketTextStream(host, port)

        //4.流处理数据
        val resultDataStream: DataStream[(String, Int)] = inputDataStream
        .flatMap(_.split(" "))
        .filter(_.nonEmpty).map((_,1))
        .keyBy(0)
        .sum(1)

        //5.配置数据流输出到控制台的并行度
        resultDataStream.print().setParallelism(1)

        //6.启动流处理job
        env.execute("stream word count")
    }
}
```



## 第三章 Flink部署

### 3.1 Standalone 模式

#### 3.1.1 部署flink集群

1. 上传并解压安装到指定目录

   ```shell
   tar -zxvf flink-1.10.1-bin-scala_2.12.tgz -C /opt/module
   ```

2. 进入conf目录，修改配置文件，配置jobmanager节点

   ```shell
   cd /opt/module/flink-1.10.1/conf
   vi flink-conf.yaml
   ```

   ```yaml
   # 配置jobmanager节点
   jobmanager.rpc.address: hadoop201
   ```

3. 修改配置文件，配置taskmanager节点

   ```shell
   vi slaves
   ```

   ```shell
   hadoop202
   hadoop203
   ```

4. 分发flink到所有节点

   ```shell
   xsync flink-1.10.1
   ```

5. 启动standalone模式flink集群

   ```shell
   bin/start-cluster.sh
   ```



#### 3.1.2  WebUI运行job

1. 打包jar包（此处使用wordcount）

2. 启动flink，访问web界面

   ```
   http://hadoop201:8081/
   ```

3. 上传jar包到flink
   ![image-20200804203609349](Flink.assets/image-20200804203609349.png)

4. hadoop201启动nc

   ```shell
   nc -lk 9526
   ```

5. 提交job，开始执行

   ![image-20200804204053759](Flink.assets/image-20200804204053759.png)

6. hadoop201控制台输入数据

7. 查看TaskManagers控制台输出
   ![image-20200804205333367](Flink.assets/image-20200804205333367.png)

8. 关闭job
   ![image-20200804205421632](Flink.assets/image-20200804205421632.png)

9. （可选）默认web端上传的jar包会在flink重启时删除，可以通过配置存储路径永久保存web端上传的jar包

   ```shell
   vi conf/flink-conf.yaml
   ```

   ```yaml
   #配置自定义jar包存放的目录,flink会自动在该目录下创建flink-web-upload目录,用于存储web端上传的jar包
   web.upload.dir: /opt/module/flink-1.10.1
   ```



#### 3.1.3 命令行运行job

1. 打包并上传jar包到linux

2. 启动flink

   ```shell
   bin/start-cluster.sh
   ```

3. hadoop201启动nc

   ```shell
   nc -lk 9526
   ```

4. 执行job

   ```shell
   bin/flink run -c com.atguigu.wc.StreamWordCount wordcount.jar --host hadoop201 --port 9526
   ```

5. hadoop201控制台输入数据，查看hadoop202和hadoop203（2个TaskManager）的控制台输出

6. 停止job

   ```shell
   #查询job_id
   bin/flink list
   ```

   ```shell
   #停止job
   bin/flink cancel b7d35687a81afaf9991c67cf56aca071
   ```



### 3.2 Yarn 模式

#### 3.2.1 Flink on Yarn

Flink提供了2种在Yarn上的运行模式

- Session-Cluster模式
  - 在yarn中初始化1个常驻的flink集群，开辟固定的资源，所有的job都提交到该集群中处理，共享这一资源
  - 适用于小规模、执行时间短的作业
- Per-Job-Cluster模式
  - 每次提交job都创建1个新的flink集群，job之间相互独立，每个flink集群随着job的结束而消除
  - 适用于大规模、执行时间长的作业



#### 3.2.2 Session-Cluster模式

1. 启动hadoop集群

   ```shell
   start-dfs.sh
   start-yarn.sh
   ```

2. 启动yarn-session

   ```shell
   bin/yarn-session.sh -s 2 -jm 1024 -tm 1024 -nm test -d
   #参数说明
   # -s(--slots)  每个TaskManager的slot数量,默认为1
   # -jm  JobManager的内存(MB)
   # -tm  TaskManger的内存(MB)
   # -nm  自定义appName
   # -d  后台执行
   ```
   
3. 启动job

   ```shell
   #与Standalone模式命令完全一致
   bin/flink run -c com.atguigu.wc.StreamWordCount wordcount.jar --host hadoop201 --port 9526
   ```

4. 结束job

   ```shell
   #查看appID
   yarn application -list
   
   #结束app进程
   yarn application -kill application_1596550537820_0001
   ```



#### 3.2.3 Per-Job-Cluster模式

1. 启动hadoop集群

   ```shell
   start-dfs.sh
   start-yarn.sh
   ```

2. 启动job

   ```shell
   #添加配置项 -m yarn-cluster
   bin/flink run -m yarn-cluster -c com.atguigu.wc.StreamWordCount wordcount.jar --host hadoop201 --port 9526
   ```

3. 结束job

   ```shell
   #查看appID
   yarn application -list
   
   #查看进程日志
   yarn logs -applicationId application_1596550537820_0001
   
   #结束app进程
   yarn application -kill application_1596550537820_0001
   ```



## 第四章 Flink运行架构

### 4.1 Flink 运行组件

- 作业管理器 JobManager
  - 接收并控制应用程序（jar包）执行的主进程
  - 向ResourceManager申请执行作业所需的资源，即TaskManager中的slot
  - 将任务发送给TaskManager执行
  - 协调检查点（checkpoint）
- 任务管理器 TaskManager
  - 一个作业可以在多个TaskManager中执行，即多个工作进程
  - 每个TaskManager各自具有一定数量的插槽（slot），插槽数决定了1个TaskManager可以并行执行的任务数量，默认为1
  - TaskManager在启动后会向ResourceManager注册所有插槽，然后接收JobManager发送的Task并执行
  - 运行同一个应用程序的多个TaskManager间可以进行数据的交换
- 资源管理器 ResourceManager
  - 管理TaskManager的插槽，即资源的调度
  - 接收JobManager的资源申请，并返回空闲的TaskManager的插槽信息
  - yarn模式下，可启动更多的TaskManager以满足应用的执行需要
  - 释放完成任务的TaskManager所占用的计算资源
- 分发器 Dispatcher（Standalone模式特有）
  - 负责将应用程序提交给JobManager的REST接口
  - 可作为Flink集群的http接入点，提供WebUI



### 4.2 任务提交流程

- Flink的yarn模式应用提交流程与spark高度相似，JobManager对应spark的Driver，而TaskManager对应spark的Executor。

  ![img](Flink.assets/clip_image002.png)

- Spark的yarn模式
  ![image-20200617234251867](Flink.assets/image-20200617234251867.png)



### 4.3 任务调度原理

#### 4.3.1 TaskManager与Slots

<img src="Flink.assets/clip_image001-1596639312732.png" alt="img" style="zoom:80%;" />

- Flink中的每一个TaskManager都是一个独立的JVM进程，一个TaskManager可接受一或多个task，task数取决于配置的Slot数
- TaskManager将其管理的资源分配给各个slot，实现了多个task的内存隔离
- 一个TaskManager的多个slot执行的task共享1个JVM进程和TCP连接，实现多路复用，减轻了每个task的负载
- slot是静态的概念，指1个TaskManager的最大并发执行能力，若未指定parallelism足够的并行度，则会出现一些slot空闲的情况，因此要进行合适的配置



#### 4.3.2 程序与数据流（dataflow）

- 一个Flink应用程序又3个部分组成
  1. Source 读取数据源
  2. Transformation 利用算子对数据进行处理
  3. Sink 输出数据
- 应用程序可以被映射为逻辑数据流（dataflow），类似于有向无环图（DAG）



#### 4.3.3 执行图（ExecutionGraph）

<img src="Flink.assets/image-20200806105344371.png" alt="image-20200806105344371" style="zoom:80%;" />

- StreamGraph：根据StreamAPI代码直接映射的执行图，表示程序的拓扑结构
- JobGraph：将StreamGraph中可以合并的节点进行连接后的执行图
- ExecutionGraph：将JobGraph中需要并行的算子加以区分得到的执行图
- 物理执行图：在ExecutionGraph基础上，划分具体的Task后，得到的最终执行图



#### 4.3.4 并行度（parallelism）

- Flink程序中，1个特定的算子的子任务个数，称为该算子的并行度，而一个数据流的所有算子中，并行度最大的算子决定了这个数据流的并行度。
- 不同的算子具有不同的并行度
  - One-To-One：如map，filter等算子，算子计算前后，数据的分区不改变，类似于spark中的窄依赖
  - Redistributing：如KeyBy，window等算子，计算后会改变数据的分区，类似于spark中的宽依赖



#### 4.3.5 任务链（oprator chains）

- 并行度相同的One-To-One算子可以进行连接，最终形成1个Task，减少了数据的交换和延迟，提高吞吐量



## 第五章 Flink流处理API

### 5.1 Environment

#### 5.1.1 getExecutionEnvironment

```scala
//最常用的执行环境创建方式,自动根据调用的位置返回相应的运行环境
import org.apache.flink.streaming.api.scala._
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
```



#### 5.1.2 createLocalEnvironment

```scala
//创建本地运行环境,需要指定程序的全局默认并行度
import org.apache.flink.streaming.api.scala._
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.createLocalEnvironment(1)
```



#### 5.1.3 createRemoteEnvironment

```scala
//创建集群运行环境,需要制定集群的JobManager地址及jar包位置
import org.apache.flink.streaming.api.scala._
val env = ExecutionEnvironment.createRemoteEnvironment("hadoop201", 6123,"YOURPATH//wordcount.jar")
```



### 5.2 Source

#### 5.2.1 从集合读取数据

```scala
object SourceTest {

    //定义样例类
    case class SensorReading(id: String, timestamp: Long, temperature: Double)

    def main(args: Array[String]): Unit = {

        import org.apache.flink.streaming.api.scala._
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

        val datas = List(
            SensorReading("sensor_1", 1547718199, 35.8),
            SensorReading("sensor_6", 1547718201, 15.4),
            SensorReading("sensor_7", 1547718202, 6.7),
            SensorReading("sensor_10", 1547718205, 38.1)
        )

        val inputStream: DataStream[SensorReading] = env.fromCollection(datas)
        
        env.execute("source test")

    }
}
```



#### 5.2.2 从文件读取数据

```scala
object SourceTest {
    def main(args: Array[String]): Unit = {

        import org.apache.flink.streaming.api.scala._
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

        val inputStream: DataStream[String] = env.readTextFile("sensor.txt")
        
        env.execute("source test")

    }
}
```



#### 5.2.3 从Kafka主题读取数据

```xml
<!-- 导入flink-kafka依赖 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
    <version>1.10.1</version>
</dependency>
```

```scala
object SourceTest {
    def main(args: Array[String]): Unit = {

        import org.apache.flink.streaming.api.scala._
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

        //配置kafka集群和消费者组
        val properties = new Properties()
        properties.setProperty("bootStrap.servers","hadoop201:9092")
        properties.setProperty("group.id","consumer_group")
        properties.setProperty("auto.offset.reset","latest")

        //使用配置项创建kafka数据源
        val inputStream: DataStream[String] = env.addSource(new FlinkKafkaConsumer011[String]("topic_name", new SimpleStringSchema(), properties))
        
        env.execute("source test")

    }
}
```



#### 5.2.4 自定义Source

```scala
object SourceTest {

    def main(args: Array[String]): Unit = {

        import org.apache.flink.streaming.api.scala._
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

        val inputStream: DataStream[SensorReading] = env.addSource(new MySourceFunction)
        inputStream.print()

        env.execute()

    }

    //自定义样例类
    case class SensorReading(id: String, timestamp: Long, temperature: Double)

    //自定义类,继承SourceFunction类,泛型为数据流的每个单元数据的类型
    class MySourceFunction extends SourceFunction[SensorReading] {

        //自定义flag变量,用于结束配合cancel命令停止run方法
        var flag = true

        //数据流创建时被调用,不断产生数据
        override def run(sourceContext: SourceFunction.SourceContext[SensorReading]): Unit = {

            //创建1个随机数发生器
            val random = new Random()

            //模拟初始温度
            val curTemp: immutable.IndexedSeq[(String, Double)] = 1.to(10).map(i => ("sensor_" + i, random.nextDouble() * 100))

            //循环模拟温度变化
            while (flag) {
                curTemp.map{ data =>
                    val time: Long = System.currentTimeMillis()
                    val nowTemp: Double = data._2 + random.nextGaussian()
                    sourceContext.collect(SensorReading(data._1, time, nowTemp))
                    Thread.sleep(100)
                }
            }
        }

        //数据流接收cancel命令时被调用,用于结束run方法
        override def cancel(): Unit = { flag = false }
    }

}
```



### 5.3 Transform

#### 5.3.1 map

```scala
//映射
val stream1 = stream.map(x => x * 2)
```



#### 5.3.2 flatMap

```scala
//扁平映射,先进行1次映射返回集合型结果,再对集合进行扁平化
val stream1 = stream.flatMap(x => x.split(" "))
```



#### 5.3.3 filter

```scala
//过滤,返回boolean型结果,为true时保留,为false时丢弃
val Stream1 = stream.filter(x => x == 1)
```



#### 5.3.4 keyBy

```scala
//从逻辑上对数据流进行分组,得到多个支流,内部使用hash实现
//方式1: 按照样例类的指定属性划分
val aggStream: DataStream[SensorReading] = dataStream.keyBy("id")

//方式2: 按照指定索引的数据划分,数据流类型可以是样例类,也可以是元组
val aggStream: DataStream[SensorReading] = dataStream.keyBy(0)

//方式3: 自定义函数,根据返回值分组
val aggStream: DataStream[SensorReading] = dataStream.keyBy(_.id)



//单独的keyBy操作只有逻辑分组功能,没有实际意义,通常需要配合滚动聚合算子使用
//1.sum 对同一支流的数据的指定部分求和
val sumStream = dataStream.keyBy("id").sum(2)

//2.min 求同一支流的数据的指定部分的最小值
//注意在更新最小值数据时,仅会更新用于keyBy和比较最小值的属性,其他属性会始终保留最初状态
val minStream = dataStream.keyBy("id").min(2)

//3.minBy 作用于min基本相同,但minBy在更新数据时,会将整条数据一起更新
val minStream = dataStream.keyBy("id").minBy(2)

//4.max 求同一支流的数据的指定部分的最大值
//注意在更新最大值数据时,仅会更新用于keyBy和比较最大值的属性,其他属性会始终保留最初状态
val maxStream = dataStream.keyBy("id").max(2)

//5.maxBy 作用于max基本相同,但maxBy在更新数据时,会将整条数据一起更新
val maxStream = dataStream.keyBy("id").maxBy(2)
```



#### 5.3.5 reduce

```scala
//配合keyBy使用,先对数据流分组,再对组内数据按照指定规则进行聚合计算,返回每一次聚合的结果
val reduceStream = dataStream.keyBy(0).reduce((data1, data2) => (data1._1, data1._2 + data2._2))
//data1为前一次的聚合结果,data2为新的1条数据,注意函数返回值数据类型需要与原数据流一致
```



#### 5.3.6 split + select

```scala
//split和keyBy类似,可以对数据流的数据进行分组,不同点是经过split算子处理的数据流类型会转换为SplitStream
//split算子的执行类似于map算子,将流中的每一条数据分别分入不同的集合中
val splitStream: SplitStream[SensorReading] = dataStream.split { data =>
    if (data.temperature > 30.0) Seq("high") else Seq("low")
}

//SplitStream具有独有的select方法,可以挑选流中的指定的一或多个分组的数据,返回新的DataStream
val highStream: DataStream[SensorReading] = splitStream.select("high")
val lowStream: DataStream[SensorReading] = splitStream.select("low")
val allStream: DataStream[SensorReading] = splitStream.select("high","low")
```



#### 5.3.7 connect

```scala
//合并两个DataStream,两个数据流的数据类型可以不同,返回ConnectedStreams类型的数据流
val connectStream: ConnectedStreams[SensorReading, SensorReading] = highStream.connect(lowStream)

//ConnectedStream类同样具有map方法,但与DataStream类的map方法不同的是,需要传入2个函数,分别处理connect前的两种数据流
val coStream: DataStream[Product] = connectStream.map(
    highData => (highData.id, "warning"),
    lowData => (lowData.id, "healthy")
)
```



#### 5.3.8 union

```scala
//合并多个DataStream,多个数据流的数据类型必须相同,返回一个新的DataStream
val allStream: DataStream[SensorReading] = highStream.union(midStream,lowStream)
```



### 5.4 Flink支持的数据类型

Flink支持Java和Scala中所有的数据类型，常用的数据类型如下

1. 基本数据类型

   ```scala
   val stream: DataStream[Double] = env.fromElements(1, 2L, 3.0, 4.0f)
   ```

2. Java和Scala元组（Tuple）

   ```scala
   val stream: DataStream[Double] = env.fromElements(
       ("0001",1),
       ("0002",2)
   )
   ```

3. Scala样例类（case class）

   ```scala
   case class SensorReading(id: String, timestamp: Long, temperature: Double)
   
   val stream: DataStream[SensorReading] = env.fromElements(
       SensorReading("sensor_1", 1547718199, 35.8),
       SensorReading("sensor_6", 1547718201, 15.4),
       SensorReading("sensor_7", 1547718202, 6.7),
       SensorReading("sensor_10", 1547718205, 38.1)
   )
   ```

4. Java简单对象（POJO）

   ```java
   public class Person {
       public String name;
       public int age;
       public Person() {}
       public Person(String name, int age) { 
           this.name = name;      
           this.age = age;  
       }
   }
   
   DataStream<Person> stream = env.fromElements(   
       new Person("Alex", 42),   
       new Person("Wendy", 23)
   );
   ```

5. Array，List，Maps，Enum等等



### 5.5 UDF函数

#### 5.5.1 函数类

- 在Spark中，UDF算子通常是使用匿名函数的方式调用，而Flink提供了所有UDF函数（即算子）的接口，可通过实现类自定义函数的具体功能，再将实现类的对象作为UDF算子的参数使用

- 方式1：继承函数接口

  ```scala
  //自定义类继承FilterFunction类,泛型为调用函数的DataStream的类型(不能定义在main方法中)
  class FilterFunc extends FilterFunction[String] {
      //重写filter方法,实现自定义的过滤规则
      override def filter(t: String): Boolean = {
          t.contains("start")
      }
  }
  
  //以自定义类对象为参数,传入filter算子调用
  val filterStream: DataStream[String] = dataStream.filter(new FilterFunc)
  
  //扩展:自定义类还可以传入其他参数
  class FilterFunc(keyword: String) extends FilterFunction[String] {
      //重写filter方法,实现自定义的过滤规则
      override def filter(t: String): Boolean = {
          t.contains(keyword)
      }
  }
  ```

- 方式2：使用函数匿名实现类

  ```scala
  val filterStream: DataStream[String] = dataStream.filter(
      new FilterFunction[String] {
          override def filter(t: String): Boolean = {
              t.contains("start")
          }
      }
  )
  ```



#### 5.5.2 匿名函数

```scala
//匿名函数的使用与scala完全相同
val filterStream: DataStream[String] = dataStream.filter(_.contains("start"))
```



#### 5.5.3 富函数

- 对于所有的UDF函数接口，除常规版本外还具有1个Rich版本接口，即富函数，可以额外获取环境的上下文对象，和一些生命周期方法

- 常用方法有

  1. open()：初始化方法，算子被调用前调用1次
  2. close()：算子结束计算后最后调用的方法，常用于执行清理工作
  3. getRuntimeContext()：提供函数的RuntimeContext信息，包括函数的并行度、人物名、状态

- 示例

  ```scala
  class FilterFunc extends RichFilterFunction[String] {
  
      var indexOfThisSubtask: Int = _
  
      override def open(parameters: Configuration): Unit = {
          //算子执行前调用
          indexOfThisSubtask = getRuntimeContext.getIndexOfThisSubtask
      }
  
      override def filter(t: String): Boolean = {
          t.contains("start")
      }
  
      override def close(): Unit ={
          //算子执行结束后调用
      }
  
  }
  ```



### 5.6 Sink

#### 5.6.1 FileSink

```scala
//输出数据流到本地文件(根据并行度创建多个分区的文件)
dataStream.addSink(
    StreamingFileSink.forRowFormat(
        new Path("F:\\atguigu\\flink\\flink\\src\\main\\resources\\out.txt"),
        new SimpleStringEncoder[SensorReading]()
    ).build()
)
```



#### 5.6.2 KafkaSink

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
    <version>1.10.1</version>
</dependency>
```

```scala
inputStream.addSink(
    new FlinkKafkaProducer011[String](
        "hadoop201:9092",
        "TEST_TOPIC",
        new SimpleStringSchema()
    )
)
```



#### 5.6.3 RedisSink

```xml
<dependency>
    <groupId>org.apache.bahir</groupId>
    <artifactId>flink-connector-redis_2.11</artifactId>
    <version>1.0</version>
</dependency>
```

```scala
case class SensorReading(id: String, timestamp: Long, temperature: Double)

//1.自定义类继承RedisMapper接口,泛型与数据流类型一致
class MyRedisMapper extends RedisMapper[SensorReading] {

    //定义redis命令,用于写入数据到redis
    override def getCommandDescription: RedisCommandDescription = {
        new RedisCommandDescription(RedisCommand.HSET, "sensor_test")
    }

    //声明hset的field
    override def getKeyFromData(t: SensorReading): String = {
        t.id
    }

    //声明hset的value
    override def getValueFromData(t: SensorReading): String = {
        t.temperature.toString
    }
}

//2.Flink-Redis连接配置
val conf: FlinkJedisPoolConfig = new FlinkJedisPoolConfig.Builder().setHost("hadoop201").setPort(6379).build()

//3.使用连接配置和自定义类发送数据到Redis
dataStream.addSink(
    new RedisSink[SensorReading](conf, new MyRedisMapper)
)
```



#### 5.6.4 ElasticSearchSink

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-elasticsearch6_2.12</artifactId>
    <version>1.10.1</version>
</dependency>
```

```scala
case class SensorReading(id: String, timestamp: Long, temperature: Double)

//1.自定义类实现ElasticsearchSinkFunction接口,泛型与数据流类型一致
class MyEsSinkFunc extends ElasticsearchSinkFunction[SensorReading] {
    //重写process方法,每收到1条数据调用一次
    override def process(t: SensorReading, runtimeContext: RuntimeContext, requestIndexer: RequestIndexer): Unit = {
        //封装数据
        val json = new util.HashMap[String, String]()
        json.put("data", t.toString)
        //发送到es
        val request: IndexRequest = Requests.indexRequest()
        .index("sensor")
        .`type`("readingData")
        .source(json)
        requestIndexer.add(request)
    }
}

//2.配置es地址端口
val httpHosts = new util.ArrayList[HttpHost]()
httpHosts.add(new HttpHost("hadoop201", 9200))

//3.使用ElasticsearchSinkFunction的匿名实现类创建es的sink
val esSink = new ElasticsearchSink.Builder[SensorReading](httpHosts, new MyEsSinkFunc)

//4.使用创建的sink输出数据到es
dataStream.addSink( esSink.build() )
```



#### 5.6.5 JDBC自定义Sink

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.44</version>
</dependency>
```

```scala
case class SensorReading(id: String, timestamp: Long, temperature: Double)

//1.自定义类继承RichSinkFunction接口,泛型与数据流类型一致
class MyJdbcSinkFunc extends RichSinkFunction[SensorReading] {

    //定义连接,预编译语句
    var conn: Connection = _
    var insertStmt: PreparedStatement =_
    var updateStmt: PreparedStatement =_

    //创建jdbc连接,并进行初始化配置
    override def open(parameters: Configuration): Unit = {
        conn = DriverManager.getConnection("jdbc:mysql://hadoop201:3306/test","root","123456")
        insertStmt = conn.prepareStatement("insert into sernsor_temp(id, temp) values (?,?)")
        updateStmt = conn.prepareStatement("update sernsor_temp set temp = ? where id = ?")
    }

    //重写invoke方法,每条数据调用一次,将数据发送到mysql
    override def invoke(value: SensorReading, context: SinkFunction.Context[_]): Unit = {

        //更新数据
        updateStmt.setDouble(1,value.temperature)
        updateStmt.setString(2,value.id)
        updateStmt.execute()

        //若未更新成功,则改为添加数据
        if (updateStmt.getUpdateCount == 0){
            insertStmt.setString(1, value.id)
            insertStmt.setDouble(2, value.temperature)
            insertStmt.execute()
        }
    }
}

//2.使用自定义Sink发送数据
dataStream.addSink(new MyJdbcSinkFunc)
```



## 第六章 Flink中的Window

### 6.1 Window

#### 6.1.1 Window概述

- Streaming流式计算是一种用于处理无限数据集的数据处理引擎，无限数据集的数据量不断增加没有上限，而window则**将无限数据集划分为有限数据块**进行处理计算的手段。
- 与Spark的window函数对原数据流按微批次开窗聚合的方式不同，Flink的window是将每一条数据存入所属的window桶（buckets）中，再对每个桶分别做分析计算。
- **本质：Flink中window函数的本质是对流的每条数据添加了一个属性，用于描述该条数据所属的一或多个窗口，而开窗后的聚合运算会以这条属性作为groupBy的字段进行分组聚合**



#### 6.1.2 Window类型

- 根据Window的划分依据，可分为两类
  - CountWindow：每n条数据生成1个Window，与时间无关
  - TimeWindow：每隔一定时间生成1个Window，与数据量无关，可进一步分为3类
    1. 滚动窗口（Tumbling Window）：窗口长度一定，不同窗口的数据没有重叠
    2. 滑动窗口（Sliding Window）：窗口长度一定，不同窗口的数据有重叠
    3. 会话窗口（Session Window）：窗口长度不一定，窗口一定时间不接收数据自动关闭，不同窗口的数据没有重叠
- **window函数的调用对象通常为KeyedStream，即经过KeyBy算子分组后的流，这样不同key值的数据会进入不同的window分别计算互不影响**
- **数据流转换图**
  <img src="Flink.assets/image-20200809173150900.png" alt="image-20200809173150900" style="zoom:80%;" />



### 6.2 Window API

#### 6.2.1 TimeWindow

```scala
//1.滚动窗口,获取每连续15秒的最低温度
val reduceStream: DataStream[(String, Double, Long)] = dataStream
    .map(data => (data.id, data.temperature, data.timestamp))
    .keyBy(_._1)
    .timeWindow(Time.seconds(15)) //滚动窗口 长度15s
//      .window( TumblingEventTimeWindows.of(Time.seconds(15)) )  //同上
    .reduce((oldData, newData) =>
            (oldData._1, oldData._2.min(newData._2), newData._3))
reduceStream.print()


//2.滑动窗口
val reduceStream2: DataStream[(String, Double, Long)] = dataStream
    .map(data => (data.id, data.temperature, data.timestamp))
    .keyBy(_._1)
    .timeWindow(Time.seconds(15), Time.seconds(5)) //滑动窗口 长度15s 步长5s
//      .window( SlidingEventTimeWindows.of(Time.seconds(15), Time.seconds(5))) //同上
    .reduce((oldData, newData) =>
            (oldData._1, oldData._2.min(newData._2), newData._3))
reduceStream2.print()


//3.会话窗口
val reduceStream3: DataStream[(String, Double, Long)] = dataStream
    .map(data => (data.id, data.temperature, data.timestamp))
    .keyBy(_._1)
    .window(EventTimeSessionWindows.withGap(Time.seconds(10))) //会话窗口 10s无新数据则关闭窗口
    .reduce((oldData, newData) =>
            (oldData._1, oldData._2.min(newData._2), newData._3))
reduceStream3.print()
```



#### 6.2.2 CountWindow

```scala
val reduceStream: DataStream[(String, Double, Long)] = dataStream
    .map(data => (data.id, data.temperature, data.timestamp))
    .keyBy(_._1)
    .countWindow(2)
    .reduce((oldData, newData) =>
            (oldData._1, oldData._2.min(newData._2), newData._3))
reduceStream.print()
```



#### 6.2.3 Window Function

- 经过window算子的数据流类型为WindowedStream，对于窗口数据流的计算操作通常分为两类
  1. 增量聚合函数：窗口中每加入一条数据触发1次计算，如ReduceFunction，AggregateFunction
  2. 全窗口函数：窗口关闭后对数据集整体进行计算，如ProcessWindwFunction
- **在实际生产环境下，增量聚合函数和全窗口函数经常需要配置实用，增量聚合函数对每一条进入窗口的函数进行实时的聚合计算，全窗口函数在窗口关闭时对增量聚合的结果进行处理后输出**



#### 6.2.4 其他API （触发器）

```scala
//1.触发器
.trigger()
//触发器具有4个重写方法,分别在窗口的不同阶段被调用
onElement()  //每条数据进入窗口时调用
onProcessingTime()  //ProcessTime定时器触发时调用
onEventTime()  //EventTIme定时器触发时调用
clear()  //窗口清除时调用
//对于每个重写的方法,可进行4种枚举配置,决定窗口函数的执行策略
TriggerResult.CONTINUE //窗口不执行任何操作
TriggerResult.FIRE  //触发窗口函数的计算
TriggerResult.PURGE  //将窗口中的数据和状态都清除,并关闭窗口
TriggerResult.FIRE_AND_PURGE  //触发窗口函数的计算,并清除窗口


//2.移除器
.evitor()


//3.允许处理迟到的数据,即设置窗口延迟一定时间关闭,延迟期间仍然会触发针对窗口数据的算子计算
.allowedLateness()


//4.对于应进入的窗口已经关闭的迟到数据,存到侧输出流,不参与窗口数据的计算
.sideOutputLateDate()


//5.获取侧输出流
.getSideOutput()
```



## 第七章 时间语义与Watermark

### 7.1 Flink中的时间语义

- 流式处理框架中，时间概念的定义非常关键，Flink提供了3种不同的时间标记方式，用于标记每一条数据的时间
  1. Event Time：事件时间，事件自带的某个记录时间的属性，通过Flink处理后获取
  2. Ingestion Time：数据进入Flink Source的时间，由Flink通过系统时间定义
  3. Processing Time（默认）：数据执行具体的算子计算的时间，由Flink通过时间定义
- 在生产环境中，需要根据具体的需求选择相应的时间语义



### 7.2 EventTime API

- 在3种时间语义中，EventTime是业务中最常用的一种，使用EventTime需要2个主要步骤

  1. **设置Flink环境的时间语义采用EventTime**
  2. **处理数据流，获取每条数据的EventTime，设置Watermark**

- API操作

  ```scala
  //1.获取FlinkStream环境,设置时间语义为EventTime
  val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
  env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
  
  //2.配置Watermark
  ```



### 7.3 Watermark

#### 7.3.1 Watermark概念（重点）

- 在设置flink环境的**时间语义为EventTime**后，环境中的时间概念就不同于客观世界的时间概念，而是使用Watermark作为时间概念的替代。
  - 现实世界的时间概念是一分一秒进行的，是连续不断流动推进的
  - **Watermark是从数据流的每一条数据的EventTime一个个获取的，当一条数据的时间戳大于上一条数据时，就将新的时间戳处理后赋给Watermark，因此时间是由数据驱动、跳跃式推进的**
- 当使用Watermark作为环境时间时，需要面对一个问题，就是数据传输到算子时的物理时间顺序，不一定与数据本身的事件时间顺序一致（EventTime乱序），这就导致在使用以时间划分的窗口函数时，无法确认窗口关闭的时机。
  为了解决这一问题，通常不直接采用数据的事件时间对Watermark赋值，而是将该**事件时间添加一定的延迟作为这一时刻的Watermark**
- 举例说明
  - 一条数据的事件时间为08:10:10，环境设置Watermark延迟10s，则在接收该条数据后，更新Watermark为08:10:00，即当前环境时间推进到08:10:00，所有应该在此时关闭的窗口会关闭并执行各自的计算任务
  - 换个角度理解，就是当我收到一条事件时间为08:10:10的数据时，我就认为08:10:00之前的数据已经全部收集进入了所属的窗口，即结束时间为08:10:00之前的窗口都可以关闭了

#### 7.3.2 Watermark API

```scala
//1.为乱序数据流配置Watermark
dataStream.assignTimestampsAndWatermarks(
    //新建BoundedOutOfOrdernessTimestampExtractor对象,参数为Watermark的延迟时间
    new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(5)) {
        //定义方法从每条数据中获取单位为毫秒的13位时间戳
        override def extractTimestamp(t: SensorReading): Long = {
            t.timestamp * 1000
        }
    }
)


//2.为有序数据流配置Watermark
//直接从数据中抽取时间戳,底层会设置1毫秒延迟的Watermark
dataStream.assignAscendingTimestamps(_.timestamp * 1000)


//3.自定义类实现AssignerWithPeriodicWatermarks接口,从数据中周期性获取Watermark(常用)
class PeriodicAssigner extends AssignerWithPeriodicWatermarks[SensorReading] {
    
    val bound: Long = 60 * 1000 //设置延时为1分钟
    var maxTs: Long = Long.MinValue //记录一个周期内数据的最大时间戳

    //每个周期调用一次,返回最新的Watermark
    override def getCurrentWatermark: Watermark = {
        new Watermark(maxTs - bound)
    }

    //周期内的每条数据执行一次,更新所在周期的最大时间戳,同时返回该条数据的时间戳(13位)
    override def extractTimestamp(r: SensorReading, previousTS: Long) = {
        maxTs = maxTs.max(r.timestamp * 1000)
        r.timestamp * 1000
    }
}


//4.自定义类实现AssignerWithPunctuatedWatermarks接口,从每条数据中获取Watermark
class PunctuatedAssigner extends AssignerWithPunctuatedWatermarks[SensorReading] {
    
    val bound: Long = 60 * 1000 //设置延时为1分钟

    override def checkAndGetNextWatermark(r: SensorReading, extractedTS: Long): Watermark = {
        if (r.id == "sensor_1") {
            new Watermark(extractedTS - bound)
        } else {
            null
        }
    }
    
    //获取每条数据的时间戳
    override def extractTimestamp(r: SensorReading, previousTS: Long): Long = {
        r.timestamp
    }
}
```



### 7.4 EventTime在window中的使用

```scala
object WatermarkTest {
    def main(args: Array[String]): Unit = {

        //创建使用EventTime的flink环境
        val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
        env.setParallelism(1)

        val inputStream: DataStream[String] = env.socketTextStream("hadoop201",9526)

        val dataStream: DataStream[SensorReading] = inputStream.map { line =>
            val data: Array[String] = line.split(",")
            SensorReading(data(0), data(1).toLong, data(2).toDouble)
        }

        //添加带有延迟的Watermark到数据流中
        val dataWithWatermarkStream: DataStream[SensorReading] = dataStream.assignTimestampsAndWatermarks(
            //新建BoundedOutOfOrdernessTimestampExtractor对象,参数为Watermark的延迟时间
            new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(5)) {
                //定义方法从每条数据中获取单位为毫秒的13位时间戳
                override def extractTimestamp(t: SensorReading): Long = {
                    t.timestamp * 1000
                }
            }
        )

        //窗口处理数据流(3选1),窗口自动使用Watermark判定关闭时机
        val resultStream: DataStream[(String, Double, Long)] = dataWithWatermarkStream
        .map(data => (data.id, data.temperature, data.timestamp))
        .keyBy(0)
        //      .window( TumblingEventTimeWindows.of(Time.seconds(10)) ) //滚动窗口 长度10s
        //      .window( SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)) ) //滑动窗口 长度10s 步长2s
        .window( EventTimeSessionWindows.withGap(Time.seconds(10)) )  //会话窗口 数据间断时间10s
        .reduce((oldData, newData) =>
                (oldData._1, oldData._2.min(newData._2), newData._3))

        resultStream.print()

        env.execute()

    }
}
```



## 第八章 ProcessFunction API（底层）

- 将环境时间设定为EventTime后，常用的一些转换算子（如map）在使用上会有一定的局限性，因为这些转换算子无法直接访问事件的时间戳和水位线，显然这样不能满足一些特定场景的需求。
- 为了解决常用算子在EventTime环境下的局限性，Flink提供了8种更底层的ProcessFunction接口，这些接口可以访问事件的所有信息，用于实现更广泛的需求，接口如下
  - ProcessFunction
  - KeyedProcessFunction
  - CoProcessFunction
  - ProcessJoinFunction
  - BroadcastProcessFunction
  - KeyedBroadcastProcessFunction
  - ProcessWindowFunction
  - ProcessAllWindowFunction
- 所有的ProcessFunction都继承自RichFunction接口，因此都具有open()，close()，getRuntimeContext()等方法



### 8.1 KeyedProcessFunction

- KeyedProcessFunction的操作对象为KeyedStream，处理流的每一个数据，输出0到任意个结果数据

- KeyedProcessFunction的主要方法
  1. open，close，getRuntimeContext
  2. processElement
     每一条数据都会调用的方法
  3. onTimer
     回调函数，当定时器触发时被调用
  
- 代码示例

  ```scala
  //需求:自定义定时器,若温度在连续10s内连续上升,则输出信息并报警
  
  //自定义类继承KeyedProcessFunction类,参数为温度连续上升的报警阈值(毫秒)
  //泛型依次为[输入流的key值, 输入流数据类型, 返回流数据类型]
  class TempIncreWarning(interval: Long) extends KeyedProcessFunction[String, SensorReading, String] {
  
      //声明状态用于记录上一条温度数据
      lazy val lastTemp: ValueState[Double] = getRuntimeContext.getState(new ValueStateDescriptor[Double]("last_temp", classOf[Double]))
      //声明状态保存注册的定时器的时间戳
      lazy val timer: ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("timer", classOf[Long]))
  
      //每条数据调用1次
      override def processElement(i: SensorReading, context: KeyedProcessFunction[String, SensorReading, String]#Context, collector: Collector[String]): Unit = {
          //从状态取出上一条数据的温度并将本次数据的温度更新到状态中
          val lastTemperature: Double = lastTemp.value()
          lastTemp.update(i.temperature)
          //从状态取出注册的定时器的时间戳
          val warnTimer: Long = timer.value()
  
          if (i.temperature > lastTemperature && warnTimer == 0) { //温度上升且未注册定时器
              //注册1个10s的定时器
              val time: Long = context.timerService().currentProcessingTime() + interval
              context.timerService().registerProcessingTimeTimer(time)
              //更新注册定时器的时间戳到状态中
              timer.update(time)
          } else if (i.temperature < lastTemperature) { //温度下降
              //删除注册的定时器
              context.timerService().deleteProcessingTimeTimer(warnTimer)
              //清除定时器的时间戳状态
              timer.clear()
          }
      }
  
      //定时器触发后调用
      override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[String, SensorReading, String]#OnTimerContext, out: Collector[String]): Unit = {
          //输出报警信息
          out.collect("警告:传感器" + ctx.getCurrentKey + "的监测温度连续" + interval / 1000 + "秒上升")
          //清除定时器的时间戳状态
          timer.clear()
      }
  }
  ```

  ```scala
  //调用自定义的函数处理数据流
  val warnStream: DataStream[String] = dataStream
      .keyBy(_.id)
      .process(new TempIncreWarning(10000))
  warnStream.print()
  ```



### 8.2 侧输出流（SideOutput）

- 在使用自定义的ProcessFunction处理数据流时，有时不仅需要单一类型的输出结果流，而是需要额外输出一些如封装异常数据的流，或是输出报警信息的流，这时就需要使用侧输出流实现。

- 侧输出流是捆绑在数据流上的附加流，对数据流本身的操作或输出并不会作用到侧输出流，但是可以使用特定的API从数据流中提取侧输出流。

- 代码示例

  ```scala
  //自定义ProcessFunction
  class FreezingMonitor extends ProcessFunction[SensorReading, SensorReading] {
      
      //1.声明一个侧输出流标签,用于定义侧输出流的数据类型,和侧输出流的名字
      lazy val freezingAlarmOutput: OutputTag[String] = new OutputTag[String]("freezing-alarms")
  
      override def processElement(r: SensorReading,
                                  ctx: ProcessFunction[SensorReading, SensorReading]#Context,
                                  out: Collector[SensorReading]): Unit = {
          //报警信息输出
          if (r.temperature < 32.0) {
              //2.使用上下文对象调用output方法,传入指定的侧输出流
              ctx.output(freezingAlarmOutput, s"Freezing Alarm for ${r.id}")
          }
          //正常数据输出
          out.collect(r)
      }
  }
  
  
  //主方法中获取侧输出流
  val monitoredReadings: DataStream[SensorReading] = readings.process(new FreezingMonitor)
  //3.使用getSideOutput方法从数据流中提取侧输出流,需要传入目标侧输出流的标签和名称
  monitoredReadings.getSideOutput(new OutputTag[String]("freezing-alarms")).print()
  ```

  



## 第九章 状态编程和容错机制

### 9.1 有状态的算子和应用程序

- Flink内置的多种算子，source，sink都是含有状态的，这里的状态实际上就是数据流在处理的过程中，维护在本地的一或多个变量
  - 在DataStream中声明状态时，1个状态与算子的1个Task绑定，Task中的每一条数据共享这个状态
  - 在KeyedStream中声明状态时，1个状态与算子的1个Task的1个key绑定，Task中同一key值的数据共享这个状态

#### 9.1.1 算子状态（operator state）

![img](Flink.assets/clip_image002-1597060986308.jpg)

- 算子状态的**作用范围为算子的1个Task**
- 算子状态具有3种基本数据结构
  1. 列表状态（List State）
  2. 联合列表状态（Union List State）
  3. 广播状态（Broadcast State）



#### 9.1.2 键控状态（keyed state）

![img](Flink.assets/clip_image002-1597061000172.png)

- 键控状态的**作用范围为算子的1个Task中所有具有相同键值的数据**

- 键控状态的数据类型

  ```scala
  //1.ValueState[T]  数据类型为T的单个值
  valueState.value()  //get方法
  valueState.update(v: T)  //set方法
  
  //2.ListState[T]  数据类型为T的列表
  listState.add(v: T)  //添加元素到List
  listState.addAll(v: List[T])  //添加所有元素到List
  listState.get()  //get方法
  listState.update(v: List[T])  //set方法
  
  //3.MapState[K, V] 数据类型为K-V对
  mapState.get(k: K) //get方法
  mapState.put(k: K, v: V)  //set方法
  mapState.contains(k: K)
  mapState.remove(k: K)
  
  //4.ReducingState[T]
  
  //5.AggregationState[I, O]
  
  //6.通用api
  state.clear()  //清空状态
  ```

- API操作

  ```scala
  //方式1 自定义类继承接口
  
  val resultStream: DataStream[(String, Double, Double)] = dataStream
      .keyBy(0)
      .flatMap(new MyFlatMap(10.0))
  resultStream.print()
  
  //自定义有状态的flatmap,监控温度变化,当连续的两个数据温差超过指定值时报警
  class MyFlatMap(threshold: Double) extends RichFlatMapFunction[SensorReading, (String, Double, Double)] {
      //声明1个单值状态,用于记录上一条数据的温度
      //注意使用lazy关键字声明,类在初始化阶段无法获取环境变量,需要延迟到flatMap方法调用时获取
      lazy val lastTempState: ValueState[Double] = getRuntimeContext.getState( new ValueStateDescriptor[Double]("last_temp",classOf[Double]))
  
      override def flatMap(in: SensorReading, collector: Collector[(String, Double, Double)]): Unit = {
          val lastTemp: Double = lastTempState.value()
          //计算本次温度与上一次温度的差值的绝对值
          val diffTemp: Double = (in.temperature - lastTemp).abs
          if (diffTemp > threshold){ //触发报警,输出集合
              collector.collect((in.id, lastTemp, in.temperature))
          }
          //更新本次温度到状态
          lastTempState.update(in.temperature)
      }
  }
  ```

  ```scala
  //方式2 直接调用有状态的算子
  
  val resultStream: DataStream[(String, Double, Double)] = dataStream
      .keyBy(0)
      .flatMapWithState[(String, Double, Double), Double] { //泛型参数分别为返回值类型和状态类型
          case (data: SensorReading, None) => (List.empty, Some(data.temperature))
          case (data: SensorReading, lastTemp: Some[Double]) => {
              val diffTemp = (data.temperature - lastTemp.get).abs
              if (diffTemp > 10) {
                  (List((data.id, lastTemp.get, data.temperature)), Some(data.temperature))
              } else {
                  (List.empty, Some(data.temperature))
              }
          }
      }
  resultStream.print()
  ```



### 9.2  exactly once

- Flink在整个流处理计算中，通过不同的策略，实现了全流程数据的精确一次消费处理和输出，具体包括
  - source端：使用kafka source，保存每次消费数据的偏移量，宕机后可从指定位置重新消费
  - 内部流处理：使用checkpoint机制，将状态存盘，宕机后可恢复
  - sink端：外部连接器必须支持事务，采用两阶段提交机制



#### 9.2.1 一致性检查点（checkpoint）

- FLink的检查点机制是一种轻量级的分布式快照机制
- **某一个时间点下，整个工作流通道恰好处理完一条输入数据，且没有新的输入数据处于中间任务的整个工作流状态即为1个快照（检查点）**
- 宕机恢复时，仅需恢复checkpoint保存的数据通道状态，再通过保存的kafka偏移量从下一条数据继续消费，就能实现flink内部的精确一次消费



#### 9.2.2 两阶段提交

1. 一条数据经过处理到达sink阶段后，kafka为该条数据开启事务，预提交到kafka分区日志中，此时的数据带有未提交标志，不能被消费
2. 等待JobManager完成checkpoint的保存工作
3. checkpoint保存完成，表示该条数据从数据源到整个工作流的精确一次消费已经实现
4. kafka将数据正式提交，并关闭事务，提交后的数据可正常被消费



### 9.3 状态后端（state backend）

- Flink中的本地状态数据和checkpoint数据都需要进行存储，存储的方式即为状态后端

- Flink提供了3种不同的状态后端

  1. MemoryStateBackend（默认）

     ```scala
     //本地状态存储在TaskManager的JVM堆中
     //checkpoint存储在JobManager内存中
     ```

  2. FsStateBackend

     ```scala
     //本地状态存储在TaskManager的JVM堆中
     //checkpoint存储在文件系统(localhost/hdfs)中
     
     //配置方式
     env.setStateBackend(new FsStateBackend("file:///tmp/checkpoints"))
     env.enableCheckpointing(1000)
     // 配置重启策略
     env.setRestartStrategy(RestartStrategies.fixedDelayRestart(60, Time.of(10, TimeUnit.SECONDS)))
     ```

  3. RocksDBStateBackend

     ```scala
     //所有状态均存储到RocksDB中
     //需要引入依赖
     ```

     ```xml
     <dependency>
         <groupId>org.apache.flink</groupId>
         <artifactId>flink-statebackend-rocksdb_2.12</artifactId>
         <version>1.10.1</version>
     </dependency>
     ```



## 第十章 TableAPI与SQL

### 10.1 概述

- Flink的Table API和SQL是一种**批流统一**处理的上层API，目前仍处于不断的开发完善中
  - Table API类似于Spark SQL的DSL语法，将sql查询中的常用关键字功能封装为API中的方法，实现对表的操作
  - Flink SQL通过直接使用SQL语法实现对表的操作



### 10.2 API调用

- TableAPI和SQL的使用遵循以下主要步骤
  1. 创建执行环境（env+tableEnv）
  2. 从数据源创建表，或将数据流转换为表
  3. 使用API处理表数据
  4. 将结果表直接写出，或转换为数据流写出



#### 10.2.1 引入依赖

```xml        &lt;dependency&gt;
<!-- 旧版本planner -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

<!-- blink版本planner -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner-blink_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-api-scala-bridge_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-csv</artifactId>
    <version>1.10.1</version>
</dependency>
```



#### 10.2.2 创建表环境

```scala
//根据api的新旧和批流处理的不同,表环境共有4种不同的创建方式
import org.apache.flink.table.api.scala._
import org.apache.flink.streaming.api.scala._

//环境1  旧版本planner + 流处理
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
val settings: EnvironmentSettings = EnvironmentSettings
    .newInstance()
    .useOldPlanner()
    .inStreamingMode()
    .build()
val oldTableEnv: StreamTableEnvironment = StreamTableEnvironment.create(env, settings)

//环境2  旧版本planner + 批处理
val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
val oldBatchTableEnv: BatchTableEnvironment = BatchTableEnvironment.create(env)

//环境3  blink版本planner + 流处理(常用)
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
val blinkSettings: EnvironmentSettings = EnvironmentSettings
    .newInstance()
    .useBlinkPlanner()
    .inStreamingMode()
    .build()
val blinkStreamTableEnv: StreamTableEnvironment = StreamTableEnvironment.create(env, blinkSettings)

//环境4  blink版本planner + 批处理
val blinkBatchSettings: EnvironmentSettings = EnvironmentSettings
    .newInstance()
    .useBlinkPlanner()
    .inBatchMode()
    .build()
val blinkBatchTableEnv: TableEnvironment = TableEnvironment.create(blinkBatchSettings)

//注意 不使用setting创建流处理环境, flink1.10会默认创建环境1, flink1.11会默认创建环境3
```



#### 10.2.3 表的注册（创建）

- Flink中，表的定义由3部分组成：Catalog注册表，Database数据库，Table表名，Catalog和Database可使用默认值
- **Flink中的表本质上在flink环境中注册的引用表，这个引用可以来自本地文件，也可以来自kafka，mysql或其他外部数据库。当需要对引用的表数据进行操作时，可以使用flink环境的相关api对注册在环境中的表进行实例化后操作，同样的，实例化的表对象也可以写入环境的引用表，实现被引用表数据的追加和更新。**
- 表具有2中类型：Table和View，在API调用上二者没有任何区别，只是使用的场景略有区别
  - Table主要用于描述文件、数据库表、消息队列等外部数据（客观存在的）
  - View主要用于描述从DataStream转换的、对Table操作的结果数据（临时，虚拟的）

##### 从文件注册表

```scala
//配置文件路径
val filePath = "F:\\atguigu\\flink\\flink\\src\\main\\resources\\sensor.txt"
//注册文件表到环境中
tableEnv.connect( new FileSystem().path(filePath) )
    .withFormat( new Csv )  //配置文件的格式解析方式,csv为使用","分隔每个字段的数据
    .withSchema( new Schema()  //配置数据字段映射,按照文件中数据顺序依次配置字段名和数据类型
            .field("id", DataTypes.STRING())
            .field("timestamp", DataTypes.BIGINT())
            .field("temperature", DataTypes.DOUBLE())
           )
	.createTemporaryTable("inputTable")  //声明表名
//从环境中获取指定表
val inputTable: Table = tableEnv.from("inputTable")
```



##### 从Kafka注册表

```scala
//注册kafkaSource表到环境中
tableEnv.connect( 
        new Kafka()
        .version("0.11")
        .topic("sensor")
        .property("zookeeper.connect", "hadoop201:2181")
        .property("bootstrap.servers", "hadoop201:9092")
    )
    .withFormat( new Csv )
    .withSchema( 
        new Schema()
        .field("id", DataTypes.STRING())
        .field("timestamp", DataTypes.BIGINT())
        .field("temperature", DataTypes.DOUBLE())
    )
.createTemporaryTable("kafkaTable")
//从环境中获取指定表
val kafkaTable: Table = tableEnv.from("kafkaTable")
```



##### 从DataStream注册表

```scala
//从DataStream[T]创建表,T为已经定义的样例类,flink根据传入的参数名,自动匹配样例类的属性名,默认字段名与属性名一致,可以使用as为修改字段名
val dataTable: Table = tableEnv.fromDataStream(dataStream, 'id, 'timestamp as 'ts)
```



#### 10.2.4 表的查询

```scala
//1.Table API查询
val resultTable: Table = inputTable.select("id, temperature").filter("id = sensor_1")

//2.SQL查询
val sql =
  """
    |select id, temperature from inputTable where id = 'sensor_1'
    |""".stripMargin
val resultTable1: Table = tableEnv.sqlQuery(sql)
```



#### 10.2.5 视图的注册（创建）

```scala
//从DataStream创建视图
tableEnv.createTemporaryView("sensorView", dataStream, 'id, 'temperature, 'timestamp as 'ts)
val dataView: Table = tableEnv.from("sensorView")

//从Table创建视图
tableEnv.createTemporaryView("sensorView", sensorTable)
val dataView: Table = tableEnv.from("sensorView")
```



#### 10.2.6 输出表

- 使用Flink流处理输出表数据到外部连接器时，有3种模式可以选择
  1. 追加模式（Append Mode）：仅追加数据到外部连接，即仅支持insert操作
  2. 撤回模式（Retract Mode）：支持增删改操作，update操作的具体实现为先删除再追加
  3. 更新模式（Upsert）：支持增删改操作，update操作的
- 不同的外部连接器支持的模式各不相同，需要根据需求进行选用，在需求允许的情况下Append Mode是通用的



##### 输出到文件

```scala
//注册用于输出的表到flink环境中(映射文件)
tableEnv
.connect(new FileSystem().path("F:\\atguigu\\flink\\flink\\src\\main\\resources\\output.txt"))
.withFormat(new Csv)
.withSchema(
    new Schema()
    .field("id", DataTypes.STRING())
    .field("temperature", DataTypes.DOUBLE())
)
.createTemporaryTable("outputTable")

//输出目标表数据到上一步注册的表中,文件系统仅支持append模式
resultTable.insertInto("outputTable")
```



##### 输出到Kafka

```scala
//注册用于输出的表到flink环境(映射kafka主题)
tableEnv
.connect(
    new Kafka()
    .version("0.11")
    .topic("sensor_output")
    .property("zookeeper.connect", "hadoop201:2181")
    .property("bootstrap.servers", "hadoop201:9092")
)
.withFormat(new Csv)
.withSchema(
    new Schema()
    .field("id", DataTypes.STRING())
    .field("timestamp", DataTypes.BIGINT())
    .field("temperature", DataTypes.DOUBLE())
)
.createTemporaryTable("kafkaOutputTable")

//输出目标表数据到上一步注册的表中,kafka仅支持append模式
resultTable.insertInto("kafkaOutputTable")
```



##### 输出到ElasticSearch

```scala
//注册用于输出的表到flink环境(映射es的index)
tableEnv
.connect(
    new Elasticsearch()
    .version("6")
    .host("hadoop201", 9200, "http")
    .index("sensor")
    .documentType("temperature")
)
.inUpsertMode() //支持inAppendMode和inUpsertMode
.withFormat(new Json)
.withSchema(
    new Schema()
    .field("id", DataTypes.STRING())
    .field("count", DataTypes.BIGINT())
)
.createTemporaryTable("esOutputTable")

//表输出数据到es时,若表经过了groupby处理,则会自动按照groupby的字段值作为index的主键id,可以根据这一机制实现upsert模式的输出效果
//若表未经过groupby处理,则在向es输出数据时,es会自动生成主键id,此时只能实现append模式的输出效果
aggTable.insertInto("esOutputTable")
```



##### 输出到Mysql

```xml
<!-- flink的jdbc支持 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-jdbc_2.12</artifactId>
    <version>1.10.1</version>
</dependency>
```

```scala
//注册用于输出的表到flink环境(映射mysql表)
val sql =
	"""
        |create table jdbcOutputTable (
        |  id varchar(20) not null,
        |  cnt bigint not null
        |) with (
        |  'connector.type' = 'jdbc',
        |  'connector.url' = 'jdbc:mysql://localhost:3306/test',
        |  'connector.table' = 'sensor_count',
        |  'connector.driver' = 'com.mysql.jdbc.Driver',
        |  'connector.username' = 'root',
        |  'connector.password' = '123456'
        |)
        |""".stripMargin
//jdbcOutputTable为flink环境表名,sensor_count为mysql数据库中的实际表名
tableEnv.sqlUpdate(sql)  
//sqlUpdata相当于.inUpsertMode(),即启用Update模式输出表数据到外部连接
//注意这里仅是创建了flink环境中的表,需要手动到mysql创建相应表

//输出目标表数据到上一步注册的表中
aggTable.insertInto("jdbcOutputTable")
```



##### 转换为DataStream

- 将表转换为DataStream时，可转换为2种数据格式，一种是根据字段映射的元组，一种是统一转换为Row型（常用）
- 由表转换为数据流有2种模式（方法）
  1. 追加模式（Append Mode）：表中的数据仅向数据流进行追加写入
  2. 撤回模式（Retract Mode）：表数据在转换为数据流时，会转换为二维元组 DataStream[(Boolean,...)]
     第一个Boolean型数据用于标记该条数据是否过期失效（新数据true，老数据false）
- 根据表数据是否经过groupBy操作选择表的转换模式
  - 无groupBy -- Append Mode
  - 有groupBy -- Retract Mode

```scala
//1.追加转换
val dataAppendStream: DataStream[Row] = tableEnv.toAppendStream[Row](dataTable)

//2.撤回转换
val dataRetractStream: DataStream[(Boolean, Row)] = tableEnv.toRetractStream[Row](dataTable)
```



#### 10.2.7 Query的解释执行

- 类似于mysql的explain关键字，flink也提供了对表的explain方法，用于解释表的查询计划

  ```scala
  val explain: String = tableEnv.explain(dataTable)
  println(explain)
  ```

  

### 10.3 流处理中的特殊概念

#### 10.3.1 流处理表的特性

|                | 静态表         | 流处理表                           |
| -------------- | -------------- | ---------------------------------- |
| 处理的数据对象 | 有界集合       | 无限序列                           |
| 查询的数据集   | 完整的全部数据 | 等待流式输入的一段段数据           |
| 查询终止的条件 | 查询完所有结果 | 不终止，不断接收新数据返回新的结果 |



#### 10.3.2 动态表（Dynatic Table）

- 在Flink Table API中操作的表实例，称为动态表。
- 动态表通常由内部数据流或外部连接器转换而来，可以描述为一种数据的容器，动态表中的数据随着新数据的添加不断更新



#### 10.3.3 流式持续查询

- Flink的流式数据处理通常可全部使用DataStream API或更底层的Process API完成，但动态表的引入实现了一种新的流式数据处理方式，即数据流→ 动态表 → Table / SQL API → 动态表 → 数据流，这一过程称为流式持续查询，属于Flink的上层API，可以通过简单易用的SQL语句实现流式数据的处理
- 具体过程
  1. 流转表
     数据流的每一条数据，都被解释为对动态表的insert修改，即不断追加数据到动态表中
  2. 持续查询
     对动态表进行连续不断的查询计算，生成新的动态表，每一次查询结果对应新的动态表这一时刻的全部数据，类似于快照
  3. 表转流
     - Append模式：新动态表中新增或更新的数据均以追加的方式写入数据流
     - Retract模式：新动态表中新增的数据追加写入数据流，而更新的数据先将带有失效标记的老数据写入数据流，再将新数据写入数据流



### 10.4 窗口

#### 10.4.1 时间特性

- 窗口的划分依赖于数据的时间属性，动态表中的每条数据和数据流的每条数据一样，都可以赋予时间属性，具体实现的方式为使用表中的一个字段记录数据时间
- Table API 和 SQL 的时间语义一般有2种选择
  1. Processing Time 处理时间
  2. Event Time 事件时间
- **直到Flink1.10版本，当需要为动态表赋予时间戳属性时，尽可能先在数据流中处理，再转换为表，这是因为使用TableAPI或DDL声明时间字段存在一定Bug，仍在解决中**



##### Processing Time

1. DataStream转换为Table时添加时间字段（推荐使用）

   ```scala
   val sensorTable: Table = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'pt.proctime)
   //pt为自定义字段,.proctime方法会将数据处理所在时刻的时间赋值到该字段
   sensorTable.toAppendStream[Row].print()
   ```

2. 使用DDL创建表时添加时间字段

   ```scala
   //创建引用表时使用as proctime()声明时间字段
   val sql =
   	"""
           |create table sensorTable (
           |  id varchar(20) not null,
           |  ts bigint,
           |  temperature double,
           |  pt as proctime()
           |) with (
           |  'connector.type' = 'filesystem',
           |  'connector.path' = 'F:\\atguigu\\flink\\flink\\src\\main\\resources\\sensor.txt',
           |  'format.type' = 'csv'
           |)
           |""".stripMargin
   tableEnv.sqlUpdate(sql)
   //从文件读取数据到table示例时,会自动对数据源中没有的时间字段进行赋值
   val sensorTable: Table = tableEnv.from("sensorTable")
   sensorTable.toAppendStream[Row].print()
   ```



##### Event Time

1. DataStream转换为Table时添加时间字段（推荐使用）

   ```scala
   //1.环境时间语义设置为EventTime
   env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
   
   //2.为每条数据添加时间戳和watermark
   val dataWithWatermarkStream: DataStream[SensorReading] = dataStream.assignTimestampsAndWatermarks(
       //新建BoundedOutOfOrdernessTimestampExtractor对象,参数为Watermark的延迟时间
       new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(5)) {
           //定义方法从每条数据中获取单位为毫秒的13位时间戳
           override def extractTimestamp(t: SensorReading): Long = {
               t.timestamp * 1000
           }
       }
   )
   
   //3.数据流转换为表,直接添加rowtime字段,该字段会在转换时自动赋值
   val sensorTable: Table = tableEnv.fromDataStream(dataWithWatermarkStream, 'id, 'temperature, 'rt.rowtime)
   sensorTable.toAppendStream[Row].print()
   ```

2. 使用DDL创建表时添加时间字段（后续处理存在bug）

   ```scala
   //创建表时声明时间字段和watermark
   val sql =
   	"""
           |create table sensorTable(
           |  id varchar(20) not null,
           |  ts bigint,
           |  temperature double,
           |  rt as to_timestamp(from_unixtime(ts)),
           |  watermark for rt as rt - interval '1' second
           |) with (
           |  'connector.type' = 'filesystem',
           |  'connector.path' = 'F:\atguigu\flink\flink\src\main\resources\sensor.txt',
           |  'format.type' = 'csv'
           |)
           |""".stripMargin
   tableEnv.sqlUpdate(sql)
   val sensorTable: Table = tableEnv.from("sensorTable")
   sensorTable.printSchema()
   sensorTable.toAppendStream[Row].print()
   ```



#### 10.4.2 Group Windows

- Group Window是将动态表的数据，根据指定的时间区间或一定数量，对数据聚合到所属Group中，对每个Group中的数据分别进行一次聚合运算，类似于微批次数据的处理方式

- Table API

  ```scala
  //1.滚动窗口
  val resultTable: Table = sensorTable
      .window(Tumble over 10.seconds on 'rt as 'w) //使用表的时间戳字段(rt)创建指定长度(10s)的窗口(w)
      .groupBy('w) //分组聚合时必须带有窗口字段
      .select('w.start, 'w.end, 'id.count)
  
  //2.滑动窗口
  val resultTable: Table = sensorTable
      .window(Slide over 10.seconds every 5.seconds on 'rt as 'w)
      .groupBy('w)
      .select('w.start, 'w.end, 'id.count)
  
  //3.会话窗口
  val resultTable: Table = sensorTable
      .window(Session withGap 10.seconds on 'rt as 'w)
      .groupBy('w)
      .select('w.start, 'w.end, 'id.count)
  ```

- SQL

  ```scala
  //1.滚动窗口
  tableEnv.createTemporaryView("sensorTable", sensorTable)
  val sql =
  	"""
          |select
          |  count(id),
          |  tumble_start(rt, interval '10' second),
          |  tumble_end(rt, interval '10' second)
          |from sensorTable
          |group by
          |  tumble(rt, interval '10' second)
          |""".stripMargin
  val resultTable: Table = tableEnv.sqlQuery(sql)
  
  //2.滑动窗口(hop_*函数的参数2为窗口步长,参数3为窗口长度)
  tableEnv.createTemporaryView("sensorTable", sensorTable)
  val sql =
  	"""
          |select
          |  count(id),
          |  hop_start(rt, interval '5' second, interval '10' second),
          |  hop_end(rt, interval '5' second, interval '10' second)
          |from sensorTable
          |group by
          |  hop(rt, interval '5' second, interval '10' second)
          |""".stripMargin
  val resultTable: Table = tableEnv.sqlQuery(sql)
  
  //3.会话窗口
  tableEnv.createTemporaryView("sensorTable", sensorTable)
  val sql =
  	"""
          |select
          |  count(id),
          |  session_start(rt, interval '10' second),
          |  session_end(rt, interval '10' second)
          |from sensorTable
          |group by
          |  session(rt, interval '10' second)
          |""".stripMargin
  val resultTable: Table = tableEnv.sqlQuery(sql)
  ```



#### 10.4.3 Over Windows

- Over Window是SQL中既有的开窗概念，将全表数据根据partition by的字段分组后开窗

- 由于流处理的表数据是动态变化的，因此flink中的Over Window又分为2种类型

  1. 无界窗口：动态表每接收一条数据，就对**分组内的全部数据**执行一次开窗聚合
  2. 有界窗口：动态表每接收一条数据，就**该条数据向前一定时间或一定数量的同组数据**进行开窗聚合

- Table API

  ```scala
  //无界窗口
  val resultTable: Table = sensorTable
      .window(Over partitionBy 'id orderBy 'rt preceding UNBOUNDED_RANGE as 'w)
      .select('id, 'id.count over 'w)  //注意聚合计算后必须添加over子句配置窗口
  
  //有界窗口
  val resultTable: Table = sensorTable
      .window(Over partitionBy 'id orderBy 'rt preceding preceding 10.seconds as 'w)
      .select('id, 'id.count over 'w)  //注意聚合计算后必须添加over子句配置窗口
  ```

- SQL

  ```scala
  //聚合同一id的近10条数据
  tableEnv.createTemporaryView("sensorTable", sensorTable)
  val sql =
  	"""
          |select
          |  id,
          |  count(id) over ow
          |from sensorTable
          |window ow as (
          |  partition by id
          |  order by rt
          |  rows between 10 preceding and current row
          |)
          |""".stripMargin
  val resultTable: Table = tableEnv.sqlQuery(sql)
  ```



### 10.5 函数





## 第十一章 Flink CEP

### 11.1 概述

- 复杂事件处理（Complex Event Processing，CEP），是Flink处理数据流时，对于一些无法通过状态编程实现、具有复杂规则的业务需求，提供的一套专用的API解决方案
- CEP的具体实现方式
  1. 从简单事件流中通过**一定的规则**筛选提取数据
  2. 将满足规则的多个简单事件组封装为一个个的复杂事件
  3. 对由复杂数据组成的复杂数据流进行操作
- **CEP本质上是对数据流进行一种高级的filter处理，封装匹配自定义规则的一或多条数据，并将这个整体作为新数据流的一条数据输出**



### 11.2 Pattern API

- Pattern API是用于定义CEP检索规则的专用API，通过API语句描述复杂的事件过程，通过实例化的Pattern对象到数据流中检索符合规则的数据（类似于正则表达式匹配字符串，Pattern匹配数据流中的数据）

- 模式序列的声明

  ```scala
  //1.(必须)声明初始模式,作为模式序列的开始(即从简单数据流中寻找匹配的复杂事件的起始点)
  Pattern.begin[MyClass]("start").where(_.eventType == "fail")
  
  //2.声明量词(匹配多条数据的整体作为一个复杂事件)
  .times(3)  //匹配出现3次
  .times(3).optional  //匹配出现3次或0次
  .times(3,5)  //匹配出现3,4,5次
  .times(3,5).greedy  //匹配出现3,4,5次,并尽可能重复匹配多次
  .times.oneOrMore  //匹配出现1或多次
  .times.timesOrMore(3).optional.greedy  //匹配出现0,3或更多次,尽可能重复匹配多次
  
  //3.(必须)声明条件
  .where(_.eventType == "fail")  //筛选条件
  .where(xxx).or(_.eventType == "fail")  //连接多个筛选条件,逻辑为或
  .where(xxx).where(_.eventType == "fail")  //连接多个筛选条件,逻辑为与
  .until(_.eventType == "fail")  //通常配合oneOrMore使用,作为1个复杂事件的终止条件
  
  //4.数据流上下文联系
  .next("next").where(_.eventType == "fail")  //严格近邻,要求下一个事件匹配规则
  .followedBy("folow").where(_.eventType == "fail")  //宽松近邻,允许中间出现不匹配数据,匹配后续数据
  .followedByAny("folow").where(_.eventType == "fail")  //非确定性宽松近邻,前一条数据允许被重复匹配
  
  //5.排除上下文联系
  .notNext("notNext").where(_.eventType == "fail")  //匹配不满足指定规则的下一个事件
  
  //6.限制上下文匹配的等待时间
  .next("next").where(_.eventType == "fail").within(Time.seconds(5))  //匹配下一条数据的最大等待时间,若超时则重新从begin匹配
  ```



### 11.3 CEP API

```scala
//1.自定义模式序列
val loginFailPattern: Pattern[LoginEvent, LoginEvent] = Pattern
    .begin[LoginEvent]("loginFail").where(_.eventType == "fail").times(3).consecutive()

//2.套用模式序列到数据流,获得PatternStream
val patternStream: PatternStream[LoginEvent] = CEP.pattern(dataStream.keyBy(_.userId), loginFailPattern)


//3.自定义PatternSelectFunction实现类,提取PatternStream数据
  class LoginFailEventMatch() extends PatternSelectFunction[LoginEvent, LoginFailWarning] {
      override def select(map: util.Map[String, util.List[LoginEvent]]): LoginFailWarning = {
          val iter: util.Iterator[LoginEvent] = map.get("loginFail").iterator()
          val firstEvent: LoginEvent = iter.next()
          val secondEvent: LoginEvent = iter.next()
          val thirdEvent: LoginEvent = iter.next()
          LoginFailWarning(firstEvent.userId, firstEvent.timestamp, thirdEvent.timestamp, "login fail")
      }
  }

//4.使用自定义类从PatternStream中提取数据
val resultStream: DataStream[LoginFailWarning] = patternStream.select(new LoginFailEventMatch())
```

