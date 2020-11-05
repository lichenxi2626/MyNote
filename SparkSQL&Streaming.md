# SparkSQL

## 第一章 SparkSQL概述

### 1.1 SparkSQL概述

- SparkSQL是Spark用于结构化数据处理的Spark模块



### 1.2 Hive和SparkSQL

- MapReduce的计算过程会产生大量的磁盘IO，为了提高运行效率，出现了大量SQL-on-Hadoop组件，如Shark
- SparkSQL的前身是Shark，而Shark是基于Hive开发的工具，可以运行在Spark引擎上
- SparkSQL不同于Shark，经过重新开发后摆脱了对HIve的依赖
- SparkSQL兼容Hive，可以作为Hive的底层引擎使用



### 1.3 SparkSQL特点

1. 易整合
   整合了SQL查询和Spark编程
2. 统一的数据访问
   使用相同的连接方式连接不同的数据源（RDD、parquet、JSON）
3. 兼容Hive
   直接运行SQL或HQL
4. 标准数据连接
   通过JDBC或ODBC连接



### 1.4 DataFrame概述

- DataFrame是以RDD为基础的分布式数据集，类似于传统数据库中的二维表格，是一种数据抽象
- **DataFrame中带有schema元信息（每个字段的名称和类型）**
- DataFrame支持嵌套数据类型（struct、array、map）
- DataFrame与RDD相同是**懒执行**的，但性能高于RDD，这是由于底层自动优化了执行计划



### 1.5 DataSet概述

- DataSet是DataFrame的扩展，也是一种数据抽象
- DataSet通过样例类定义数据的结构信息，样例类中的每个属性映射DataSet中的字段
- DataSet是强类型的



## 第二章 SparkSQL核心编程

### 2.1 查询起点

- 在Spark Core中，执行应用需要构建环境对象SparkContext作为操作的起点
- 在SparkSQL中同样需要构建起始点，使用的是**SparkSession**，SparkSession内部封装了SparkContext
- 使用spark-shell命令时，Spark会自动创建一个SparkSession对象，命名为spark



### 2.2 DataFrame

#### 2.2.1 创建DataFrame

- SparkSQL支持从多种数据源创建DataFrame

  ```scala
  csv   format   jdbc   json   load   option   options   orc   parquet   schema   table   text   textFile
  ```

- 从JSON文件创建DataFrame

  ```scala
  //官方示例文件
  {"name":"Michael"}
  {"name":"Andy", "age":30}
  {"name":"Justin", "age":19}
  
  //读取json文件创建DataFrame对象
  val df = spark.read.json("/opt/module/spark-local/examples/src/main/resources/people.json")
  //返回DataFrame对象
  df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
  //从内存读取整形数据时默认使用Int类型接收,从文件读取整形数据时默认使用bigint接收,可转换为Long类型
  ```



#### 2.2.2 SQL语法

- **使用SQL语法需要通过DataFrame创建临时或全局视图后再操作**

```scala
//1.从json文件读取数据创建DataFrame
val df = spark.read.json("/opt/module/spark-local/examples/src/main/resources/people.json")

//2.创建视图并命名
df.createOrReplaceTempView("people") //临时视图只在当前会话窗口中生效
df.createGlobalTempView("people") //全局视图在使用时需要通过全路径global_temp.people访问

//3.调用sql方法,传入查询语句
//sql方法的底层实现其实还是将查询结果封装为了DataFrame,但不通过对象名而是通过视图名访问
spark.sql("select * from people").show()
//4.返回查询结果
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
```



#### 2.2.3 DSL语法

```scala
//1.从json文件读取数据创建DataFrame
val df = spark.read.json("/opt/module/spark-local/examples/src/main/resources/people.json")

//2.查看返回的DataFrame的元数据信息
df.printSchema
//返回元数据信息
root
 |-- age: long (nullable = true)
 |-- name: string (nullable = true)

//3.查询指定列的数据
df.select("name","age").show()
//返回查询结果
+-------+----+
|   name| age|
+-------+----+
|Michael|null|
|   Andy|  30|
| Justin|  19|
+-------+----+

//4.查询字段计算后的数据
//所有字段前添加$
df.select($"name", $"age"+1).show()
//所有字段去除双引号,改为一个单引号表示
df.select('name, 'age+1).show()
//返回计算后的结果
+-------+---------+
|   name|(age + 1)|
+-------+---------+
|Michael|     null|
|   Andy|       31|
| Justin|       20|
+-------+---------+

//5.筛选查询
df.filter($"age">20).show()

//6.分组查询与多行函数
df.groupBy("age").count.show()
//返回查询结果
+----+-----+                                                                    
| age|count|
+----+-----+
|  19|    1|
|null|    1|
|  30|    1|
+----+-----+
```



#### 2.2.4 RDD与DataFrame的转换

- 在IDEA中需要引入当前环境对象的隐式转换方法，实现RDD与DataFream的转换

  ```scala
  import spark.implicits._
  //spark-shell中自动引入,无需手动声明
  ```

- RDD转换为DataFrame

  ```scala
  //方式1:通过样例类转换
  //1.创建样例类,样例类的属性对应了DataFrame中的字段
  case class People(name:String, age:Int)
  
  //2.创建RDD
  val rdd = sc.makeRDD(List(People("zhangsan",20),People("lisi",30)))
  
  //3.使用toDF方法将RDD转换为DataFrame
  val df = rdd.toDF
  
  //方式2
  val rdd = sc.makeRDD(List(("zhangsan",20),("lisi",30)))
  rdd.toDF("name","age")
  ```

- DataFrame转换为RDD

  ```scala
  //直接使用rdd方法转换
  val rdd = df.rdd
  
  //此方法返回的RDD中元素的存储类型为Row
  org.apache.spark.rdd.RDD[org.apache.spark.sql.Row]
  //可使用getAs方法获取Row类型数据的值
  val array = rdd.collect
  array(0).getAs[String]("name")
  ```



### 2.3 DataSet

#### 2.3.1 创建DataSet

- 使用样例类创建DataSet

  ```scala
  //1.创建样例类,样例类的属性对应了DataFrame中的字段
  case class People(name:String, age:Int)
  
  //2.使用样例类对象创建List
  val list = List(People("zhangsan",20),People("lisi",30))
  
  //3.使用toDS方法使用List创建DataSet
  val ds = list.toDS
  ```

- 使用基本类型的序列创建DataSet

  ```scala
  //1.创建基本类型数据组成的序列
  val list = List(1,2,3,4)
  
  //2.使用toDF方法使用List创建DataSet
  val ds = list.toDS
  ```



#### 2.3.2 RDD与DataSet的转换

- 在IDEA中需要引入当前环境对象的隐式转换方法，实现RDD与DataSet的转换

  ```scala
  import spark.implicits._
  //spark-shell中自动引入,无需手动声明
  ```

- RDD转换为DataSet

  ```scala
  //1.创建样例类,样例类定义了表的结构,属性对应表的字段
  case class People(name:String, age:Int)
  
  //2.创建RDD
  val rdd = sc.makeRDD(List(People("zhangsan",20),People("lisi",30)))
  
  //3.使用toDS方法将RDD转换为DataSet
  val ds = rdd.toDS
  ```

- DataSet转换为RDD

  ```scala
  //直接使用rdd方法转换
  val rdd = ds.rdd
  
  //此方法返回的RDD中元素的存储类型为样例类
  org.apache.spark.rdd.RDD[People]
  ```



### 2.4 DataFrame与DataSet转换

- DataFrame在本质上是DataSet的特例，DataSet集合的数据类型可以是任意类型，而DataFrame集合的数据类型是封装了多个字段类型的Row类型

- DataFrame转换为DataSet

  ```scala
  //1.df到ds的转换需要依托于样例类
  case class People(name:String, age:Int)
  
  //2.使用as[]方法将df转换为ds
  val ds = df.as[People]
  //在转换时,系统会从df的元数据中依次寻找与People类中属性同名的列名,关联后映射数据
  //若同名的属性与列的数据类型不同,会首先尝试将列数据隐式转换为属性数据,若无法转换则会抛出异常
  //若数据成功转换,最终获得的ds会沿用df中列的数据类型,而不是样例类的属性类型
  ```

- DataSet转换为DataFrame

  ```scala
  //直接使用toDF方法转换,ds会将类的属性映射为df中的字段
  val df = ds.toDF
  ```



### 2.5 RDD、DataFrame、DataSet三者对比

- 共同点
  1. 都是分布式弹性数据集
  2. 都具有惰性机制，在调用行动算子时执行
  3. 自动根据内存情况进行缓存运算
  4. 都有partition
  5. DataFrame和DataSet都可以通过模式匹配获取字段的值和类型
- 不同点
  1. RDD一般与Spark MLlib同时使用，不支持SparkSQL操作
  2. DataFrame每行的数据类型固定为Row，列的值必须通过解析获取
  3. DataFrame和DataSet一般不与Spark MLlib同时使用，都支持SparkSQL的操作，同时支持sql语句操作
  4. DataFrame中每行的类型都是Row，不解析，必须通过getAs方法或模式匹配获取数据
  5. DataSet中每行的类型是根据字段的组合决定的



### 2.6 IDEA中创建SparkSQL环境

1. 添加依赖

   ```xml
   <dependency>
       <groupId>org.apache.spark</groupId>
       <artifactId>spark-sql_2.12</artifactId>
       <version>2.4.5</version>
   </dependency>
   ```

2. 环境代码

   ```scala
   object SparkSQL01 {
       def main(args: Array[String]): Unit = {
   
           //1.获取配置参数对象
           val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL")
           //2.获取环境对象
           val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()
           //3.使用环境对象导入隐式转换方法
           import spark.implicits._
   
           //4.代码逻辑
   
           //5.释放资源
           spark.stop()
           
       }
   }
   ```




### 2.7 用户自定义函数

#### 2.7.1 UDF函数

```scala
//1.获取查询所需的视图对象
val rdd = sc.makeRDD(List(People("zhangsan",20),People("lisi",30)))
val df: DataFrame = rdd.toDF()
df.createOrReplaceTempView("people")

//2.注册udf函数(单行函数)
//参数1 函数的名称,用于在查询语句中调用
//参数2 函数对象,参数为一行数据中的一或多个字段,返回值为任意类型的数据
spark.udf.register("addName", (s:String) => "name" + s)

//3.在sql语句中使用自定义的函数
spark.sql("select addName(name) from people").show()
```



#### 2.7.2 UDAF函数

- 需求：自定义UDAF函数实现平均值计算函数

- 弱类型UDAF（输入值为已知类型的一或多个字段）

  ```scala
  //1.自定义类继承UserDefinedAggregateFunction,重写类中的8个方法
  class MyAvgAge extends UserDefinedAggregateFunction {
  
      //1.1 输入数据的结构信息 这里的数据类型指自定义函数的参数,与字段的数据类型对应
      override def inputSchema: StructType = {
          StructType(Array(StructField("age",IntegerType)))
      }
  
      //1.2 缓冲区数据的结构信息 udaf函数在内存缓冲区中做两两聚合运算时,持久化的数据类型
      override def bufferSchema: StructType = {
          StructType(Array(StructField("totalAge",LongType),StructField("count",LongType)))
      }
  
      //1.3 聚合函数返回数据的结构信息
      override def dataType: DataType = DoubleType
  
      //1.4 函数稳定性 对于相同的输入是否始终返回相同的输出结果
      override def deterministic: Boolean = true
  
      //1.5 函数在缓冲区的初始化值 
      override def initialize(buffer: MutableAggregationBuffer): Unit = {
          buffer(0) = 0L
          buffer(1) = 0L
      }
  
      //1.6 缓冲区的数据聚合逻辑 缓冲区数据与一行输入数据的计算逻辑
      override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
          buffer(0) = buffer.getLong(0) + input.getInt(0)
          buffer(1) = buffer.getLong(1) + 1
      }
  
      //1.7 缓冲区间的数据聚合逻辑
      override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
          buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
          buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
      }
  
      //1.8 缓冲区数据聚合结果的处理逻辑
      override def evaluate(buffer: Row): Any = {
          buffer.getLong(0) / buffer.getLong(1).toDouble
      }
  }
  ```

  ```scala
  //2.自定义udaf函数的使用
  //2.1 准备用于查询的数据视图
  val rdd = sc.makeRDD(List(People("zhangsan",20),People("lisi",30)))
  val df: DataFrame = rdd.toDF()
  df.createOrReplaceTempView("people")
  
  //2.2 注册自定义聚合函数并命名
  spark.udf.register("avg_age", new MyAvgAge)
  
  //2.3 在sql语句中使用自定义的聚合函数
  spark.sql("select avg_age(age) from people").show()
  ```

- 强类型UDAF

  ```scala
  //1.自定义样例类,封装自定义UDAF函数的输入数据与缓存区数据的类型
  case class Person(name: String, age: Int){}
  case class AvgBuffer(var sum: Long, var count: Long){}
  ```

  ```scala
  //2.自定义类继承Aggregator类,泛型参数为[输入数据类型,缓冲区数据类型,输出数据类型]
  class MyAvgAgeClass extends Aggregator[Person, AvgBuffer, Double]{
  
      //2.1 函数在缓冲区的初始化值 返回缓冲区数据类型的对象
      override def zero: AvgBuffer = {
          AvgBuffer(0L,0L)
      }
  
      //2.2 缓冲区的数据聚合逻辑 缓冲区数据与一个输入数据的计算逻辑,返回缓冲区数据类型的对象
      override def reduce(b: AvgBuffer, a: Person): AvgBuffer = {
          b.sum += a.age
          b.count += 1
          b
      }
  
      //2.3 缓冲区数据聚合结果的处理逻辑 返回缓冲区数据类型的对象
      override def merge(b1: AvgBuffer, b2: AvgBuffer): AvgBuffer = {
          b1.sum += b2.sum
          b1.count += b2.count
          b1
      }
  
      //2.4 对缓冲区的聚合结果计算 返回输出数据类型的对象
      override def finish(reduction: AvgBuffer): Double = {
          reduction.sum / reduction.count.toDouble
      }
  
      //2.5 缓冲区数据的编码器 自定义类的编码器默认使用product方法获取
      override def bufferEncoder: Encoder[AvgBuffer] = Encoders.product
  
      //2.6 输出数据的编码器
      override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
  }
  ```

  ```scala
  //3.自定义udaf函数的使用
  //3.1 准备用于操作的Dataset对象
  val rdd = sc.makeRDD(List(Person("zhangsan",20),Person("lisi",30)))
  val ds: Dataset[Person] = rdd.toDS()
  
  //3.2 获取自定义函数的对象
  val udaf = new MyAvgAgeClass
  
  //3.3 调用聚合函数的toColumn方法将自定义函数转换为列,再获取计算结果
  ds.select(udaf.toColumn).show()
  //强类型udaf函数操作的基本单位对应表中的一整行数据而非具体字段,因此无法通过sql语句直接调用
  ```



### 2.8 数据的加载和保存

#### 2.8.1 通用加载保存方法

- 加载数据

  ```scala
  //1.使用spark.read.load加载数据,默认读取的文件格式为parquet(列式存储文件)
  val df = spark.read.load("input/user.parquet")
  
  //2.通过format方法可以指定读取的文件的格式(csv,jdbc,json,orc,parquet,textFile)
  val df = spark.read.format("json").load("input/user.json")
  ```

- 保存数据

  ```scala
  //1.使用df.write.save保存数据,默认保存的文件格式为parquet(列式存储文件)
  df.write.save("output/user")
  
  //2.通过format方法可以指定保存的文件的格式(csv,jdbc,json,orc,parquet,textFile)
  df.write.format("json").save("output/user")
  
  //3.通过mode方法可以选择保存的具体操作
  df.write.mode("error").save("output/user")		//若文件已存在则抛出异常(默认)
  df.write.mode("append").save("output/user")		//若文件已存在则追加保存
  df.write.mode("overwrite").save("output/user")	//若文件已存在则覆盖保存
  df.write.mode("ignore").save("output/user")		//若文件已存在则不保存
  ```



#### 2.8.2 Parquet

- Parquet是spark加载保存文件的默认格式，无需额外操作

- 加载数据

  ```scala
  val df = spark.read.load("input/user.parquet")
  ```

- 保存数据

  ```scala
  df.write.save("output/user")
  ```



#### 2.8.3 JSON

- Saprk加载和保存的JSON文件格式与普通JSON文件不同。由于Spark按行读取文件，因此要求JSON文件中的每一行满足普通JSON文件的格式

  ```scala
  {"name": "zhangsan", "age": 20}
  {"name": "lisi", "age": 30}
  {"name": "wangwu", "age": 40}
  ```

- 加载数据

  ```scala
  //除通用加载数据方法外,还可以直接调用read.json读取
  val df = spark.read.json("input/user.json")
  ```

- 保存数据

  ```scala
  df.write.json("output/people")
  ```



#### 2.8.4 CSV

- 加载数据

  ```scala
  //导入csv数据时需要进行一些常用配置项的配置
  spark.read.format("csv")
  	.option("sep", "_") //指定分隔符(默认为,)
  	.option("inferSchema", "true") //是否自动推断字段类型
  	.option("header", "true") //是否读取字段名(需要csv第一行为字段名,默认为flase)
  	.load("output/user")
  ```

- 保存数据

  ```scala
  df.write.format("csv")
  	.option("sep", ";") //指定分隔符(默认为,)
  	.option("inferSchema", "true") //是否自动推断字段类型
  	.option("header", "true") //是否读取字段名(需要csv第一行为字段名,默认为flase)
  	.save("output/user")
  ```



#### 2.8.5 MySQL

- 在IDEA中通过JDBC操作MySQL需要先导入依赖

  ```xml
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.27</version>
  </dependency>
  ```

- 加载数据

  ```scala
  val df: DataFrame = spark.read.format("jdbc")
  	.option("url", "jdbc:mysql://hadoop100:3306/spark_sql") //配置连接的数据库
  	.option("driver", "com.mysql.jdbc.Driver") //配置驱动类
  	.option("user", "root") //配置访问的用户
  	.option("password", "123456") //用户密码
  	.option("dbtable", "t_people") //加载的表
  	.load()
  //返回的df中Row的字段与读取的表的字段名相同且类型一致
  ```

- 保存数据

  ```scala
  ds.write
  	.format("jdbc")
  	.option("url", "jdbc:mysql://hadoop100:3306/spark_sql") //配置连接的数据库
  	.option("user", "root") //配置访问的用户
  	.option("password", "123456") //用户密码
  	.option("dbtable", "t_people") //保存数据的表
  	.mode(SaveMode.Append) //保存数据的模式
  	.save()
  //注意ds中类的属性名需要与mysql中表的字段名相同,且数据类型一致
  ```



#### 2.8.6 Hive

1. 导入hive依赖

   ```xml
   <dependency>
       <groupId>org.apache.spark</groupId>
       <artifactId>spark-hive_2.12</artifactId>
       <version>2.4.5</version>
   </dependency>
   <dependency>
       <groupId>org.apache.hive</groupId>
       <artifactId>hive-exec</artifactId>
       <version>3.1.2</version>
   </dependency>
   ```

2. 复制hive-site.xml文件到resource目录中（注意引擎使用MR）

3. 创建spark环境，操作hive

   ```scala
   //配置hadoop访问用户(省略可能导致权限问题),该用户为hadoop中core-site.xml文件中配置的hive代理用户
   System.setProperty("HADOOP_USER_NAME", "atguigu")
   
   val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL")
   //在创建SparkSession时需要添加enableHiveSupport()方法启动hive的支持
   val spark: SparkSession = SparkSession.builder().enableHiveSupport().config(conf).getOrCreate()
   
   //访问hive
   spark.sql("show databases").show
   ```



## 第三章 SparkSQL实战

### 3.1 数据准备

1. 启动hdfs和yarn集群

2. 启动hive

3. 创建3个表并从本地导入数据

   ```scala
   object DataPrepare {
       def main(args: Array[String]): Unit = {
   
           System.setProperty("HADOOP_USER_NAME", "atguigu")
   
           val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL")
           val spark: SparkSession = SparkSession.builder().enableHiveSupport().config(conf).getOrCreate()
   
           spark.sql(
               """
           |use spark_sql
           |""".stripMargin)
   
           spark.sql(
               """
           |CREATE TABLE `user_visit_action`(
           |  `date` string,
           |  `user_id` bigint,
           |  `session_id` string,
           |  `page_id` bigint,
           |  `action_time` string,
           |  `search_keyword` string,
           |  `click_category_id` bigint,
           |  `click_product_id` bigint,
           |  `order_category_ids` string,
           |  `order_product_ids` string,
           |  `pay_category_ids` string,
           |  `pay_product_ids` string,
           |  `city_id` bigint)
           |row format delimited fields terminated by '\t'
           |""".stripMargin)
   
           spark.sql(
               """
           |CREATE TABLE `product_info`(
           |  `product_id` bigint,
           |  `product_name` string,
           |  `extend_info` string)
           |row format delimited fields terminated by '\t'
           |""".stripMargin)
   
           spark.sql(
               """
           |CREATE TABLE `city_info`(
           |  `city_id` bigint,
           |  `city_name` string,
           |  `area` string)
           |row format delimited fields terminated by '\t'
           |""".stripMargin)
   
           spark.sql("load data local inpath 'input/user_visit_action.txt' into table user_visit_action")
           spark.sql("load data local inpath 'input/product_info.txt' into table product_info")
           spark.sql("load data local inpath 'input/city_info.txt' into table city_info")
   
           spark.stop()
   
       }
   }
   ```

### 3.2 统计各区域商品Top3

```scala
object SQLExer {
    def main(args: Array[String]): Unit = {

        System.setProperty("HADOOP_USER_NAME", "atguigu")
        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL")
        val spark: SparkSession = SparkSession.builder().enableHiveSupport().config(conf).getOrCreate()
        import spark.implicits._

        val udaf = new MyUDAF
        spark.udf.register("cityInfo", udaf)

        spark.sql("use spark_sql")

        spark.sql(
            """
        |select t3.*
        |from
        |(
        |select t2.*, rank() over(partition by t2.area order by t2.cnt desc) cnt_rank
        |from
        |(
        |select t1.area, t1.product_name, sum(t1.cnt) cnt, cityInfo(t1.city_name, t1.cnt) city_info
        |from
        |(
        |select c.area , u.city_id , c.city_name , p.product_name , count(1) cnt
        |from user_visit_action u
        |left join city_info c on u.city_id = c.city_id
        |left join product_info p on u.click_product_id = p.product_id
        |where u.click_product_id != -1
        |group by c.area , u.city_id , c.city_name , p.product_name
        |) t1
        |group by t1.area, t1.product_name
        |) t2
        |) t3
        |where t3.cnt_rank <= 3
        |""".stripMargin).show()

        spark.stop()

    }

    //自定义弱类型UDAF函数
    class MyUDAF extends UserDefinedAggregateFunction{
        override def inputSchema: StructType = {
            StructType(Array(
                StructField("city_name",StringType),
                StructField("cnt",IntegerType)
            ))
        }

        override def bufferSchema: StructType = {
            StructType(Array(
                StructField("cityMap",MapType(StringType,LongType)),
                StructField("sum",LongType)
            ))
        }

        override def dataType: DataType = StringType

        override def deterministic: Boolean = true

        override def initialize(buffer: MutableAggregationBuffer): Unit = {
            buffer(0) = Map[String, Long]()
            buffer(1) = 0L
        }

        override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
            val map: Map[String, Long] = buffer.getAs[Map[String,Long]](0)
            val sum: Long = buffer.getLong(1)
            val city: String = input.getString(0)
            val cnt: Int = input.getInt(1)

            buffer(0) = map.updated(city, map.getOrElse(city, 0L) + cnt.toLong)
            buffer(1) = sum + cnt.toLong
        }

        override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
            val map1: Map[String, Long] = buffer1.getAs[Map[String,Long]](0)
            val sum1: Long = buffer1.getLong(1)
            val map2: Map[String, Long] = buffer2.getAs[Map[String,Long]](0)
            val sum2: Long = buffer2.getLong(1)

            buffer1(0) = map1.foldLeft(map2){
                case(map, (city, cnt)) => {
                    val newMap: Map[String, Long] = map.updated(city, map.getOrElse(city, 0L) + cnt)
                    newMap
                }
            }
            buffer1(1) = sum1 + sum2
        }

        override def evaluate(buffer: Row): Any = {
            val cityMap: Map[String, Long] = buffer.getAs[Map[String,Long]](0)
            val sum: Long = buffer.getLong(1)

            var elseSum = sum
            var result = ""
            val top2City: List[(String, Long)] = cityMap.toList.sortBy(_._2)(Ordering.Long.reverse).take(2)
            top2City.foreach{
                case (city, cnt) => {
                    elseSum -= cnt
                    result = result + city + ":" + (cnt * 100 / sum) + "%,"
                }
            }
            result = result + "其他城市:" + (elseSum * 100 / sum) + "%"
            result
        }

    }

}
```



# SparkStreaming

## 第一章 SparkStreaming概述

### 1.1 SparkStreaming定义

- SparkStreaming用于流式数据的处理
- SparkStreaming并非真正的实现了数据的流式处理，而是通过**微批次**对数据进行处理，因此SparkStreaming是一个**准实时的数据处理引擎**
- SparkStreaming使用离散化流（DStream）作为数据的抽象，**DSteam内部封装了根据时间周期划分的多个RDD组成的序列**。

### 1.2 SparkStreaming特点

- 易用
- 容错
- 易整合到Spark

### 1.3 SparkStreaming架构

- SparkStreaming架构
  1. 在一个Executor中执行数据的接收器，接收器长期运行，将DStream发送给Driver节点。
  2. Driver节点接收后创建作业，并将任务分发给多个Executor执行
  3. Executor执行后直接输出或返回计算结果到Driver
- 背压机制
  - 定义：根据JobScheduler反馈作业的执行信息，动态的调整Receiver数据的接收速率
  - 启用方法：配置参数spark.streaming.backpressure.enabled=true，默认为false



## 第二章 DStream入门

### 2.1 WordCount案例

1. 添加SparkStreaming依赖

   ```xml
   <dependency>
       <groupId>org.apache.spark</groupId>
       <artifactId>spark-streaming_2.12</artifactId>
       <version>2.4.5</version>
   </dependency>
   ```

2. 编写WordCount程序

   ```scala
   object SSWordCount {
       def main(args: Array[String]): Unit = {
   
           //1.创建SparkStreaming环境,注意采集器与Driver各需要1个CPU内核启动
           val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
           val ssc = new StreamingContext(conf, Duration(3000))
           //参数2表示批处理周期 Duration(Long),默认单位毫秒,也可以使用Seconds(3)
   
           //2.获取离散化流
           val socketDS: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop100",9999)
   
           //3.DStream处理逻辑
           val wordDS: DStream[String] = socketDS.flatMap(_.split(" "))
           val result: DStream[(String, Int)] = wordDS.map((_,1)).reduceByKey(_+_)
           result.print()
   
           //4.启动采集器,注意采集器启动后不停止
           ssc.start()
           //5.等待采集器的结束
           ssc.awaitTermination()
   
       }
   }
   ```

3. 在连接的主机节点上使用netcat发送数据

   ```shell
   nc -lk 9999
   ```



### 2.2 WordCount执行过程

1. DStream是SparkStreaming的基础抽象，由一系列连续的RDD表示，每个RDD含有一段时间间隔内的数据。
2. DStream对数据的操作也以RDD为单位进行。
3. 具体的计算过程由Spark引擎完成。



## 第三章 DStream创建

### 3.1 RDD队列

```scala
object QueueRDD {
    def main(args: Array[String]): Unit = {

        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
        val ssc = new StreamingContext(conf, Duration(3000))

        //1.创建mutable.Queue队列对象,接收RDD
        val queue: mutable.Queue[RDD[Int]] = mutable.Queue[RDD[Int]]()
        //2.使用ssc.queueStream传入队列对象,获取DStream
        val queueDS: InputDStream[Int] = ssc.queueStream(queue)
        //3.打印queue中的RDD
        queueDS.print()
        //4.启动ssc
        ssc.start()
        //5.向队列中循环添加RDD,1个Streaming执行周期中只处理1个RDD
        for(i <- Range(1,100)){
            val rdd: RDD[Int] = ssc.sparkContext.makeRDD(List(i))
            queue.enqueue(rdd)
        }
        ssc.awaitTermination()

    }
}
```



### 3.2 自定义数据源



### 3.3 Kafka数据源

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.apache.spark</groupId>
       <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>
       <version>2.4.5</version>
   </dependency>
   ```

2. 编写程序

   ```scala
   object KafkaStreaming {
       def main(args: Array[String]): Unit = {
   
           val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
           val ssc = new StreamingContext(conf, Duration(3000))
   
           //1.配置kafka参数(集群地址,用户组...)
           val kafkaPara: Map[String, Object] = Map[String, Object](
               ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> "hadoop100:9092,hadoop101:9092,hadoop102:9092",
               ConsumerConfig.GROUP_ID_CONFIG -> "atguigu",
               "key.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer",
               "value.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer"
           )
   
   
           //2.使用SparkStreaming读取kafka中的数据,配置读取的Topic
           val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream(
               ssc,
               LocationStrategies.PreferConsistent,
               //将要消费的Topic名装入可迭代集合 Set("atguigu")
               ConsumerStrategies.Subscribe[String, String](Set("atguigu"), kafkaPara)
           )
   
           //3.取出每条消息的kv
           val valueDStream: DStream[String] = kafkaDStream.map(_.value())
   
           //4.计算逻辑
           valueDStream.flatMap(_.split(" "))
           .map((_,1))
           .reduceByKey(_+_)
           .print()
   
           ssc.start()
           ssc.awaitTermination()
   
       }
   }
   ```



## 第四章 DStream转换

### 4.1 无状态转化操作

- 无状态转化操作的作用对象为**DStream中的每一个单独的RDD**
- 注意使用时需要添加import StreamingContext._



#### 4.1.1 常用的无状态转换操作

1. map
2. flatMap
3. filter
4. repartition：修改DStream分区数
5. reduceByKey：对每个批次数据单独对key进行聚合计算
6. groupByKey：对每个批次数据单独根据key分组



#### 4.1.2 transform

- transform：参数为自定义函数，将DStream中的每个RDD作为函数的参数整体转入，返回一个任意类型的新RDD，相当于对每个批次数据的RDD整体进行转换

- 代码实现

  ```scala
  val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
  val ssc = new StreamingContext(conf, Duration(3000))
  import StreamingContext._
  
  val ds: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop100",9999)
  val newDS: DStream[String] = ds.transform(rdd=>rdd.map(_*2))
  newDS.print()
  
  ssc.start()
  ssc.awaitTermination()
  ```



#### 4.1.3 join

- join：与RDD的join函数效果相同，作用对象为两个DStream中的相同批次的两个RDD，因此**要求两个DSTream的批次周期相同**

- 代码实现

  ```scala
  val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
  val ssc = new StreamingContext(conf, Duration(3000))
  import StreamingContext._
  
  val dStream1: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop100", 9999)
  val dStream2: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop101", 8888)
  val wordDStream1: DStream[(String, String)] = dStream1.flatMap(_.split(" ")).map((_, 1))
  val wordDStream1: DStream[(String, String)] = dStream2.flatMap(_.split(" ")).map((_, 1))
  val joinDStream: DStream[(String, (String, String))] = wordDStream1.join(wordDStream2)
  joinDStream.print()
  
  ssc.start()
  ssc.awaitTermination()
  ```



### 4.2 有状态转化操作

- 有状态转化操作的作用对象为DStream一或多个RDD，即一或多个批次的数据。
- 有状态转化操作可以用于历史数据的累积统计，近期变化率等需求的实现。



#### 4.2.1 UpdateStateByKey

```scala
//功能 处理键值对类型的RDD,将一次采集周期数据的计算结果保存,参与到下一个采集周期的计算,以此类推
//自定义函数参数1 当前批次中,key相同的value组成的Seq集合
//自定义函数参数2 上批次数据计算的结果,与函数的返回值类型一致
//自定义函数返回值 两个参数计算后产生的新计算结果

//示例 wordcount累积计算
val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
val ssc = new StreamingContext(conf, Duration(3000))
import StreamingContext._

//注意 持久化每个采集周期的计算结果需要设置检查点目录
ssc.checkpoint("savepoint")
val ds: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop100",9999)

ds.flatMap(_.split(" "))
.map((_,1))
.updateStateByKey(
    (values: Seq[Int], state:Option[Long]) => {
        val newBuffer: Long = state.getOrElse(0L) + values.sum.toLong
        Option(newBuffer)
    }
).print()

ssc.start()
ssc.awaitTermination()
```



#### 4.2.2 WindowOperations

- 窗口函数：处理连续的多个批次的数据，即以DStream中的多个连续的RDD为单位进行数据的处理

  - 窗口时长：决定一次处理的批次数量（必须为批次周期的整数倍）
  - 滑动步长：决定两次窗口函数处理的批次间隔（必须为批次周期的整数倍）

- 代码实现

  ```scala
  val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
  val ssc = new StreamingContext(conf, Duration(3000))
  import StreamingContext._
  
  val dStream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop100",9999)
  val words = dStream.flatMap(_.split(" ")).map((_,1))
  //reduceByKeyAndWindow 对同一个窗口下的多个批次数据中,相同key的value进行聚合计算
  val wordCounts = words.reduceByKeyAndWindow((a:Int,b:Int) => (a + b),Seconds(12), Seconds(6))
  wordCounts.print()
  
  ssc.start()
  ssc.awaitTermination()
  ```

- 常用Windows函数

  1. window(windowLength, slideInterval)：划分窗口时长和滑动步长后返回新的DStream

  2. countByWindow(windowLength, slideInterval)：返回一个窗口中的元素个数

  3. reduceByWindow(func, windowLength, slideInterval)：聚合计算窗口中的数据

  4. reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks])：按key分组聚合计算窗口中的数据

  5. reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks])：按key分组增量聚合计算窗口中的函数，函数1为聚合逻辑，函数2为移除旧批次的逻辑

     ```scala
     //增量聚合用于相邻窗口中重复数据量大于新数据量的场合,可有效减少重复的计算
     val wordCount = DStream.reduceByKeyAndWindow(
         {(x, y) => x + y},
         {(x, y) => x - y},
         Seconds(30),
         Seconds(10))
     ```



## 第五章 DStream输出

- DStream与RDD相同**具有惰性执行机制**，**必须通过输出函数触发**DSream转换函数的执行。

- 常见的输出函数

  ```scala
  //1.print()
  //功能 在Driver节点上打印每一批次RDD中的前10个元素
  
  //2.saveAsTextFiles(prefix, [suffix])
  //功能 存储DStream中的数据到text文件,参数分别为文件全路径及后缀名
  
  //3.saveAsObjectFiles(prefix, [suffix])
  //功能 使用Java的对象序列化将DStream中的数据存储到序列化文件,参数分别为文件全路径及后缀名
  
  //4.saveAsHadoopFiles(prefix, [suffix])
  //功能 将DStream中的数据存储到HDFS,参数分别为文件全路径及后缀名
  
  //5.foreachRDD(func)
  //功能 将DStream中的每个RDD作为参数传入自定义函数,可用于将数据写出到外部数据库
  ```
  
- **在foreachRDD算子中，只有rdd.xxx的代码会在各个Executor中执行，其余代码均在Driver中执行**



## 第六章 优雅关闭

### 6.1 关闭SparkStreaming

- 流式任务需要24小时执行，当需要进行业务升级或需求更改时需要手动停止程序，为避免数据丢失，需要使用特定的方法关闭SparkStreaming

- 通过监控目录变化实现手动关闭

  ```scala
  class MonitorStop(ssc: StreamingContext) extends Runnable {
  
      override def run(): Unit = {
  
          val fs: FileSystem = FileSystem.get(new URI("hdfs://hadoop100:9000"), new Configuration(), "atguigu")
  
          //循环执行检查实现目录下文件的监控
          while (true) {
              try
              Thread.sleep(5000)
              catch {
                  case e: InterruptedException =>
                  e.printStackTrace()
              }
              val state: StreamingContextState = ssc.getState
  
              //通过上传stopSpark到监控目录实现手动关闭SparkStreaming程序
              val bool: Boolean = fs.exists(new Path("hdfs://hadoop100:9000/stopSpark"))
  
              if (bool) {
                  if (state == StreamingContextState.ACTIVE) {
                      ssc.stop(stopSparkContext = true, stopGracefully = true)
                      System.exit(0)
                  }
              }
          }
      }
  }
  ```

  

### 6.2 重启SparkStreaming

- 当需要再次启动SparkStreaming环境时，可以通过检查点恢复DStream中的历史数据

  ```scala
  object ContinueSS {
  
    def main(args: Array[String]): Unit = {
  
      //从检查点重启SparkStreaming,可以获得关闭前的历史数据
      val ssc: StreamingContext = StreamingContext.getActiveOrCreate("cp", getStreamingContext)
  
      //代码逻辑
      //...
  
      ssc.start()
      ssc.awaitTermination()
    }
  
    //自定义方法获取新的SparkStreaming环境对象
    def getStreamingContext(): StreamingContext = {
      val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
      val newssc = new StreamingContext(conf, Duration(3000))
      newssc.checkpoint("cp")
      newssc
    }
  
  }
  ```



## 第七章 SparkStreaming实战

