# Clickhouse

## 第一章 Clickhouse概述

### 1.1 概念

- Clickhouse是一个开源的列式存储数据库（DBMS），使用C++语言编写，用于在线分析处理查询（OLAP），能够使用SQL查询实时生成的分析数据报告。
- 官网地址 
  - https://clickhouse.yandex/
  - http://repo.red-soft.biz/repos/clickhouse/stable/el6/



### 1.2 特点

1. clickhouse采用列式存储，具有如下优点
   - 列的聚合计算性能强
   - 易于压缩，压缩比大，节省磁盘空间
2. DBMS功能
   - 可使用大部分标准的SQL语法，包括DDL和DML，及大量自带函数
   - 可进行用户管理和权限管理
   - 可对数据进行备份与恢复
3. 引擎多样化
   - 与MySQL默认使用的InnoDB引擎不同，clickhouse可根据需求使用不同的存储引擎存储表数据，主要包括合并树、日志、接口等等
4. 写数据
   - clickhouse采用了类似hbase的类LSM Tree结构进行数据的写入。在接收写入数据请求后，先将数据**顺序写入**磁盘的临时文件中，再**异步周期性的**将临时文件数据合并到主数据文件中，写入性能优异
   - **不支持常规意义的修改和删除行操作**
   - 不支持事务
5. 读数据
   - clickhouse支持**单条SQL语句的多线程**执行（MySQL中单SQL仅支持单线程），因此查询效率快。但也因此CPU占用率高，不适合多条SQL的并行执行
   - clickhouse的索引使用了**稀疏索引**，默认颗粒度为8192行（即每隔8192行数据提取一行主键作为索引），这样加快了范围过滤的速度（节省对密集型索引逐行检索的时间），但缺点是不适合做点对点查询
   - clickhouse擅长单表查询（最快），多表关联查询性能相对较弱（较快）



## 第二章 Clickhouse安装

### 2.1 Linux环境准备

1. 修改配置文件，添加配置项（提高并发度）

   ```shell
   sudo vim /etc/security/limits.conf
   sudo vim /etc/security/limits.d/20-nproc.conf
   ```

   ```shell
   #两文件中均添加以下配置
   * soft nofile 65536 
   * hard nofile 65536 
   * soft nproc 131072 
   * hard nproc 131072
   ```

2. 取消selinux

   ```shell
   sudo vim /etc/selinux/config
   ```

   ```shell
   #修改以下配置
   SELINUX=disabled
   ```

3. 关闭防火墙

   ```shell
   systemctl stop firewalld #关闭防火墙服务
   systemctl disable firewalld #关闭防火墙自启动
   ```

4. 安装相关依赖

   ```shell
   sudo yum install -y libtool
   sudo yum install -y *unixODBC*
   ```

5. 重启主机

   ```shell
   reboot
   ```



### 2.2 安装clickhouse单机模式

1. 上传安装文件

   ```shell
   clickhouse-client-19.17.9.60-2.noarch.rpm
   clickhouse-common-static-19.17.9.60-2.x86_64.rpm
   clickhouse-common-static-dbg-19.17.9.60-2.x86_64.rpm
   clickhouse-server-19.17.9.60-2.noarch.rpm
   ```

2. 安装rpm

   ```shell
   sudo rpm -ivh *.rpm
   ```

3. 修改clickhouse配置文件，打开注释，解锁非本地的主机访问

   ```shell
   sudo vim /etc/clickhouse-server/config.xml
   ```

   ```xml
       <!-- Listen specified host. use :: (wildcard IPv6 address), if you want to accept connections both with IPv4 and IPv6 from everywhere. -->
       <listen_host>::</listen_host>
       <!-- Same for hosts with disabled ipv6: -->
       <!-- <listen_host>0.0.0.0</listen_host> -->
   ```

4. 关闭clickhouse开机自启

   ```shell
   sudo systemctl disable clickhouse-server
   ```

5. 启动clickhouse服务

   ```shell
   sudo systemctl start clickhouse-server
   ```

6. 连接clickhouse客户端

   ```shell
   clickhouse-client -m
   # -m 启动多行sql的支持
   ```



## 第三章 数据类型

### 3.1 整型

- 带符号整形
  - Int8  [-128 : 127]
  - Int16  [-32768 : 32767]
  - Int32  [-2147483648 : 2147483647]
  - Int64  [-9223372036854775808 : 9223372036854775807]
- 无符号整形
  - UInt8 - [0 : 255]
  - UInt16 - [0 : 65535]
  - UInt32 - [0 : 4294967295]
  - UInt64 - [0 : 18446744073709551615]



### 3.2 浮点型

- Float32 等价于java中的float
- Float64 等价于java中的double



### 3.3 布尔型

- clickhouse中没有专门的布尔型数据，一般使用UInt8存储，限制取值为0或1



### 3.4 小数型

在需要使用小数存储数据的场景中一般使用Decimal而非Float，因为Decimal可保证小数点精度

- Decimal32(n)  n位小数，整数+小数位数和为9
- Decimal64(n)  n位小数，整数+小数位数和为18
- Decimal128(n)  n位小数，整数+小数位数和为38



### 3.5 字符串型

- String 任意长度字符串
- FixedString(N)  定长字符串，N为正整数，当存储数据长度不足N时，会在数据末尾追加空字节补全（尽量不用）



### 3.6 枚举型

- 类似于字典表原理，在底层为某字段维护一个字典表，使用整型数据代替字符串数据存储，减少数据冗余。但当字段内容出现变化时需要维护字典表，操作不当还会引起数据丢失，需要谨慎使用

- 枚举类型

  - Enum8： 使用 Int8 映射 String 型数据
  - Enum16： 使用 Int16 映射 String 型数据

- 示例

  ```mysql
  #使用枚举型字段创建表
  CREATE TABLE t_enum
  (
      x Enum8('hello' = 1, 'world' = 2)
  )
  ENGINE = TinyLog;
  
  #向表中插入数据(仅能使用枚举中声明过的数据插入)
  INSERT INTO t_enum VALUES ('hello'), ('world'), ('hello');
  
  #查询数据
  SELECT * FROM t_enum
  ┌─x─────┐
  │ hello │
  │ world │
  │ hello │
  └───────┘
  
  #查询底层存储的数据
  SELECT CAST(x, 'Int8') FROM t_enum;
  ┌─CAST(x, 'Int8')─┐
  │            1    │
  │            2    │
  │            1    │
  └─────────────────┘
  ```



### 3.7 时间型

- Date  示例 '2020-01-01'
- Datetime  示例 '2020-01-01 12:00:00'
- Datetime64  示例 '2020-01-01 12:00:00.00'



### 3.8 数组

- Array[T]  可存储任意数据类型组成的数组集合
- 不推荐使用多维数组，不能在MergeTree表中使用多维数组



## 第四章 表引擎

### 4.1 表引擎的使用

- 在clickhouse中，建表语句中需要明确声明表的存储引擎，引擎直接决定了表的存储方式和策略，需要根据实际需求选择合适的引擎

- **引擎名称严格区分大小写**

- 语法

  ```mysql
  create table t_tinylog (id String, name String) engine = TinyLog;
  ```



### 4.2 TinyLog

- 数据列式存储在磁盘中
- 不支持索引，不支持并发访问
- 生产环境中一般不使用



### 4.3 Memory

- 数据以原始形式保存在内存中，不落盘，宕机会导致数据丢失
- 不支持索引
- 查询性能极高，一般用于测试



### 4.4 MergeTree（合并树）

- clickhouse中最强大、最常用的引擎，还具有多个分支引擎用于不同的业务场景

- 具体的存储策略为：在接收写入数据请求后，先将数据**顺序写入**磁盘的临时文件中，再**异步周期性的**将临时文件数据合并到主数据文件中

- 建表语句

  ```mysql
  create table t_order_mt(
      id UInt32,
      sku_id String,
      total_amount Decimal(16,2),
      create_time  Datetime
  ) engine = MergeTree
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id,sku_id);
  ```



#### 4.4.1 分区

- 与hive类似的分区存储数据，表数据会根据分区字段物理地进行切分，存储到磁盘的不同目录中

- 不声明分区字段时，**默认仅有1个分区**

- 数据写入后，不会立刻存储到相应分区的磁盘文件中，而是先存储到磁盘临时文件，再等待merge出发后合并到分区数据文件中

- merge（合并）的触发条件

  1. 数据写入后10-60分钟（无法进行相关配置），自动触发

  2. 执行命令手动触发merge

     ```mysql
     # merge的触发除了合并分区的临时数据文件外,还会执行其他事件(如清理声明周期结束的数据)
     optimize table t_order_mt;
     # 当判定某一分区数据不存在需要合并的临时文件时,optimize table命令会失效,也不会执行其他事件,此时可以添加final,强制执行其他的事件
     optimize table t_order_mt final;
     ```



#### 4.4.2 主键

- 不同于其他数据库，在clickhouse中**主键（primary key）没有唯一约束**
- 建表时会**根据主键字段做一级索引**（稀疏索引），若不声明主键，系统会直接根据order by的字段建立一级索引
- 主键可以是一或多个字段，但**必须取自order by的前n个字段**。
  如order by字段为 (id, name) ，则主键只能取 id 或(id, name)



#### 4.4.3 排序

- 建表时必须声明order by的字段，数据会在各自分区内按字典顺序对order by的字段进行升序排列（可设置降序）



#### 4.4.4 生命周期

- clickhouse可以对表的每行数据或每个单元的数据设置生命周期，当生命周期结束后会自动删除该数据

- **不能单独设置order by字段的声明周期**

- 列级TTL

  ```mysql
  #列级TTL:对具体的字段设置生命周期,生命周期结束后,该行数据的该字段内容会被置空(String)或清零(数值)
  create table t_order_mt(
      id UInt32,
      sku_id String,
      total_amount Decimal(16,2) TTL create_time + interval 10 SECOND,
      create_time  Datetime 
  ) engine = MergeTree
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id, sku_id);
  
  #说明 TTL用于判定的字段类型必须为日期类型
  ```

- 表级TTL

  ```mysql
  #表级TTL:对表中的每行数据设置生命周期,生命周期结束后整行数据被删除
  alter table t_order_mt MODIFY TTL create_time + INTERVAL 10 SECOND;
  ```

- 可使用的时间周期

  ```mysql
  - SECOND
  - MINUTE
  - HOUR
  - DAY
  - WEEK
  - MONTH
  - QUARTER
  - YEAR 
  ```



### 4.5 ReplacingMergeTree

- MergeTree的分支版本，存储特性与MergeTree完全相同，额外增加了**对Primary Key的去重**功能，可以借助这一特性实现数据写入的**延迟幂等性**

- **去重操作仅会在merge时执行，且不能跨分区去重**

- ReplacingMergeTree在使用时需要指定版本字段，当order by字段数据相同时触发数据去重，去重时根据版本字段数据保留最大即最新的一次数据

- 建表语句

  ```mysql
  create table t_order_rmt(
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2) ,
    create_time  Datetime 
  ) engine = ReplacingMergeTree(create_time)
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id, sku_id);
  ```



### 4.6 SummingMergeTree

- MergeTree的分支版本，存储特性与MergeTree完全相同，额外增加了**指定字段值的预求合**的功能，可以利用这一功能减少聚合查询时的性能开销

- **预聚合操作仅会在merge阶段发生，且不能跨分区预聚合**

- SummingMergeTree在使用时需要**指定聚合计算的字段（必须为数值型）**，**以order by的字段维度**进行聚合

- 分组维度字段和聚合字段**以外的字段会按排序保留组内第一行数据**

- 建表语句

  ```mysql
  create table t_order_smt(
      id UInt32,
      sku_id String,
      total_amount Decimal(16,2) ,
      create_time  Datetime 
  ) engine =SummingMergeTree(total_amount)
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id,sku_id);
  ```

- 注意进行聚合查询前，为保证临时文件已经合并到主文件，需要手动merge

  ```mysql
  optimize table t_order_mt final;
  ```



## 第五章 SQL操作

### 5.1 Insert

- clickhouse中的Inser语法与MySQL一致

- 语法

  ```mysql
  #1.标准插入
  insert into t_order values (...)
  #2.从表插入
  insert into t_order select ...
  ```



### 5.2 Update 和 Delete

- clickhouse中可以执行update和delete操作，但不同于MySQL仅对指定行进行更新或删除，clickhouse的实现方式需要重**新建立指定行所在的分区**

- clickhouse的数据修改之所以需要通过重建分区实现是因为对一行数据的修改或删除可能改变稀疏索引的粒度，进而需要重建并排序索引，因此不如直接新建分区再顺序写入磁盘

- update和delete操作具体分为2步执行，**不支持事务**

  1. 新建分区，对旧分区进行失效标记
  2. 触发merge，删除失效的旧分区数据

- 语法

  ```mysql
  #1.删除
  alter table t_order 
  delete where sku_id = 'sku_001';
  
  #2.修改
  alter table t_order 
  update tatal_amount = toDecimal32(2000.00, 2) where id = 101;
  ```



### 5.3 Select

- clickhouse的查询与MySQL基本一致，但存在一些限制

  1. clickhouse的join查询不支持缓存
  2. 不支持窗口函数
  3. 不支持自定义函数

- clickhouse对 group by 查询的功能进行了扩展，使其返回多个结果

  ```mysql
  #1.标准group by
  select id, sku_id, sum(total_amount)
  from t_order
  group by id, sku_id;
  #以id,sku_id维度聚合计算
  
  #2.group by with rollup
  select id, sku_id, sum(total_amount)
  from t_order
  group by id, sku_id with rollup;
  #依次以(id,sku_id),(id),()为维度(由右至左去除维度)聚合计算,返回3个查询结果
  
  #3.group by with cube(类似于kylin的cube预聚合计算)
  select id, sku_id, sum(total_amount)
  from t_order
  group by id, sku_id with cube;
  #依次以(id,sku_id),(id),(sku_id),()为维度聚合(所有可能的字段组合)计算,返回4个查询结果
  
  #4.group by with total
  select id, sku_id, sum(total_amount)
  from t_order
  group by id, sku_id with total;
  #依次以(id,sku_id),()为维度聚合计算,返回2个查询结果
  ```



### 5.4 Alter

- clickhouse中的alter操作与MySQL基本相同

- 语法

  ```mysql
  #1.添加字段
  alter table t_order
  add column spu_id String after sku_id;
  
  #2.修改字段类型
  alter table t_order
  modify column sku_id String;
  
  #3.删除字段
  alter table t_order
  drop column sku_id;
  ```



### 5.5 导出数据

- 使用命令行导出查询语句的结果

  ```shell
  #导出为csv文件
  clickhouse-client --query "select toHour(create_time) hr, count(*) from test.order_wide where dt='2020-06-23' group by hr" --format CSVWithNames> ~/rs1.csv
  ```



## 第六章 副本与分片

### 6.1 概述

- 对于clickhouse中的1个表，可以对数据进行分片存储和副本保存，分片存储实现了表的横向扩容，副本机制实现了数据的高可用
- 由于clickhouse服务本身需要消耗大量内存，因此对于一张数据表不论是不同分片，还是分片的不同副本，都不建议存储在同一节点中，因此对于一张分片数为3，副本数为2的表，至少需要3*2=6个部署了clickhouse的节点存储数据
- 在clickhouse集群模式中，**没有主从概念**，数据发送到任意一个节点，节点都会通知zookeeper，由zookeeper负责数据的分发。



### 6.2 Clickhouse集群部署（2主1从）

1. 分别在3台节点中部署单机模式的clickhouse

2. 编写配置文件（3台节点）

   ```shell
   sudo vim /etc/clickhouse-server/config.d/metrika.xml
   ```

   ```xml
   <yandex>
   
       <clickhouse_remote_servers>
           <gmall_cluster>   <!-- 标签中为自定义集群名称 --> 
               <shard>         <!-- 集群的第一个分片 -->
                   <internal_replication>true</internal_replication>
                   <replica>   <!-- 该分片的第一个副本 -->
                       <host>hadoop201</host>
                       <port>9000</port>
                   </replica>
                   <replica>   <!-- 该分片的第二个副本 -->
                       <host>hadoop202</host>
                       <port>9000</port>
                   </replica>
               </shard>
               <shard>         <!-- 集群的第二个分片 -->
                   <internal_replication>true</internal_replication>
                   <replica>   <!-- 该分片的第一个副本 -->
                       <host>hadoop203</host>
                       <port>9000</port>
                   </replica>
               </shard>
           </gmall_cluster>
       </clickhouse_remote_servers>
   
       <!-- zookeeper配置 --> 
       <zookeeper-servers>
           <node index="1">
               <host>hadoop201</host>
               <port>2181</port>
           </node>
           <node index="2">
               <host>hadoop202</host>
               <port>2181</port>
           </node>
           <node index="3">
               <host>hadoop203</host>
               <port>2181</port>
           </node>
       </zookeeper-servers>
   
       <!-- 分别配置每台节点存储的shard和replication --> 
       <macros>
           <shard>01</shard>   <!-- 不同机器放的分片数不一样 -->
           <replica>rep_1_1</replica>  <!-- 不同机器放的副本数不一样 -->
       </macros>
   
       <!-- hadoop202配置如下
   	<macros>
     		<shard>01</shard>   
    		<replica>rep_1_2</replica>  
   	</macros>
   	-->
   
       <!-- hadoop203配置如下
   	<macros>
     		<shard>02</shard>   
     		<replica>rep_2_1</replica>  
   	</macros>
   	-->
   
   </yandex> 
   ```
   
3. 追加上一步编写的配置文件到主配置文件（3台节点）

   ```shell
   sudo vim /etc/clickhouse-server/config.xml
   ```

   ```xml
   <!-- 追加配置 -->
   <zookeeper incl="zookeeper-servers" optional="true" />
   <include_from>/etc/clickhouse-server/config.d/metrika.xml</include_from>
   ```
   
4. 启动所有节点的zookeeper和clickhouse

   ```shell
   zkctl start
   sudo systemctl start clickhouse-server
   clickhouse-client
   ```



### 6.3 分布式表的创建和使用

1. 在集群任意节点创建各节点表

   ```mysql
   create table order_mt on cluster gmall_cluster (
   	id UInt32,
   	sku_id String,
   	total_amount Decimal(16,2),
   	create_time  Datetime
   ) engine = ReplicatedMergeTree('/clickhouse/tables/{shard}/order_mt','{replica}')
   partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id);
   # 说明
   # on cluster gmall_cluster 使用集群模式建表,指定metrica.xml中配置的集群
   # ReplicatedMergeTree() 副本模式专用的表引擎,参数分别为zookeeper中元数据信息的存储目录,节点存储的分片和副本
   # {shard} {replica} 分别读取metrica.xml配置文件中该节点的macros标签下的变量
   ```

2. 创建分布式表

   ```mysql
   create table order_mt_all on cluster gmall_cluster(
       id UInt32,
       sku_id String,
       total_amount Decimal(16,2),
       create_time  Datetime
   ) engine = Distributed(gmall_cluster, test, order_mt, hiveHash(sku_id));
   # 说明
   # 字段与集群中各节点表的字段完全一致
   # Distributed() 分布式表引擎,相当于各节点表的manager
   # 参数表示 (集群名, 库名, 节点表名, 分片键) 分片键必须为整型数据
   ```

3. 直接使用分布式表进行操作

   ```mysql
   # 直接查询各节点的表只能查到该节点存储的数据,因此一般直接使用分布式表进行查询
   select * from order_mt_all;
   ```





