Hbase

## 第一章 HBase简介

### 1.1 HBase定义

- HBase是一种分布式、可扩展、支持海量数据存储的NoSQL数据库
- NoSQL：不具有ACID特性（原子性、一致性、隔离性、持久性）的非关系型数据库
- Hbase中仅有字符串1种数据类型



### 1.2 HBase数据模型

- HBase在逻辑上的数据模型与关系型数据库类似，将数据存储在一张具有行列的表中。但HBase的底层物理存储结构实际是以KV键值对存储，类似于multi-dimensional map（多维度map），key值由多个维度共同组成

#### 1.2.1 HBase逻辑结构

![image-20200619182957858](Hbase.assets/image-20200619182957858.png)

1. 列族（column family）：由多个字段共同组成，是HBase表中列的基本结构
2. 列（colume）：对应关系型数据库中的字段
3. Row key：类似于关系型数据库的主键，唯一标识每一行数据
4. Region：分区，一个HBase表的数据会被横切为多个Region，分别存储到HDFS中
5. Store：一个分区下的不同列族会被竖切为多个Store，分别存储为多个StoreFile（HFile）文件



#### 1.2.2 HBase物理存储结构

![image-20200619183626177](Hbase.assets/image-20200619183626177.png)

- 对于HBase表中的通过行列确定的一个具体的数据，会在底层以多个维度包装后存储数据的值
- HBase底层存储结构为LSMT（log struct merge tree），意为内存+有序文件+合并



#### 1.2.3 数据模型

1. Name Space
   类似于关系型数据库中的Database，每个命名空间下含有多个表，Hbase默认自带2个命名空间
   - hbase：存放HBase内置的表
   - default：用户不指定命名空间时，创建的表默认选择的命名空间
2. Table
   类似于关系型数据库中的Table，HBase中Table的创建仅需声明列族而不需要具体的列
3. Row
   行，HBase中的每行数据都以RowKey和多个字段的值组成，数据根据RowKey的字典顺序存储
4. Column
   列，HBase中的列由列族+列共同组成
5. TimeStamp
   时间戳，记录操作的时间，HBase中的数据在修改和删除时不会直接修改数据所在行的数据值，而是直接追加数据，追加的内容还包括了时间戳和操作类型，用于在后续查询中定位最新的版本
6. Cell
   由RowKey，列族，列，时间戳四个维度唯一确定的数据单元，以字节码形式存储



### 1.3 HBase基本架构

- Master
  - 定义：HBase集群中的一个节点
  - 功能：表的创建、修改、删除，分配Region到RegionServer，监控RegionServer状态
  - 执行HBase群起命令的节点直接成为Master节点，可通过配置文件配置备用Master实现高可用
- RegionServer
  - 定义：HBase集群中的每个节点
  - 功能：数据的增删改查，Region数据的切分与合并
- Zookeeper
  - 功能：监控Master和RegionServer状态，维护HBase集群
- HDFS
  - 功能：存储HBase的数据



## 第二章 HBase入门

### 2.1 HBase的安装部署

1. 上传并解压HBase安装文件到指定目录

   ```shell
   tar -zxvf hbase-2.0.5-bin.tar.gz -C /opt/module
   ```

2. 配置环境变量

   ```shell
   sudo vi /etc/profile.d/my_env.sh
   ```

   ```shell
   #HBASE_HOME
   export HBASE_HOME=/opt/module/hbase-2.0.5
   
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$KAFKA_HOME/bin:$KE_HOME/bin:$HIVE_HOME/bin:$HBASE_HOME/bin
   ```

   ```shell
   #分发到所有节点
   sudo xsync /etc/profile.d/my_env.sh
   source /etc/profile.d/my_env.sh
   ```

3. 修改配置文件hbase-env.sh

   ```shell
   vi conf/hbase-env.sh
   ```

   ```shell
   #解除注释并设置为false,关闭HBase自带的zookeeper服务
   export HBASE_MANAGES_ZK=false
   ```

4. 修改配置文件hbase-site.xml

   ```shell
   vi conf/hbase-site.xml
   ```

   ```xml
   <property>
       <name>hbase.rootdir</name>
       <value>hdfs://hadoop201:8020/hbase</value>
   </property>
   
   <property>
       <name>hbase.cluster.distributed</name>
       <value>true</value>
   </property>
   
   <property>
       <name>hbase.zookeeper.quorum</name>
       <value>hadoop201,hadoop202,hadoop203</value>
   </property>
   
   <!-- 以下两项为使用hadoop2.x版本时的扩展功能配置,hadoop3.x版本无需配置
   <property>
       <name>hbase.unsafe.stream.capability.enforce</name>
       <value>false</value>
   </property>
   
   <property>
       <name>hbase.wal.provider</name>
       <value>filesystem</value>
   </property>
   -->
   ```

5. 配置RegionServer节点

   ```shell
   vi conf/regionservers
   ```

   ```shell
   hadoop201
   hadoop202
   hadoop203
   ```

6. （可选）配置Master高可用

   ```shell
   vi conf/backup-masters
   ```

   ```shell
   #直接添加备用Master节点到文件中
   hadoop202
   ```

7. 分发HBase到其他节点

   ```shell
   xsync /opt/module/hbase-2.0.5
   ```

8. 启动hdfs和zookeeper

   ```shell
   start-dfs.sh
   zkctl start #自定义启动脚本
   ```

9. 群起hbase，命令执行节点即为Master节点

   ```shell
   start-hbase.sh
   #群停hbase
   stop-hbase.sh
   ```

10. Web端查看

    ```shell
    #Master
    http://hadoop201:16010
    #ReigonServer
    http://hadoop201:16030
    ```



### 2.2 HBase Shell操作

1. 启动hbase shell

   ```shell
   bin/hbase shell
   ```

2. 查看帮助命令

   ```shell
   #查看所有命令
   help
   #查看具体命令的使用方式
   help 'put'
   ```

3. 操作表（DDL）

   ```shell
   #1.查看当前namespace下的表
   list
   
   #2.创建表
   create 'nsl:t1', {NAME => 'f1', VERSIONS => 5}
   #命名空间为nsl,表名为t1,列族为f1,f1的保留版本数为5个
   create 'student', 'info', 'class_info'
   #命名空间为default,表名为student,两个列族分别为info、class_info
   
   #3.查看表的结构
   desc 'student'
   
   #4.修改表(不支持列族名的修改)
   alter 'student', {NAME => 'info', VERSIONS => 3}
   #修改列族info的保留版本数,其他列族不变
   
   #5.删除表
   disable 'student'
   #先设置表不可用后再执行drop删除
   drop 'student'
   ```

4. 操作数据（DML）

   ```shell
   #1.添加或更新数据单元
   put 'student','1001','info:name','zhangsan'
   #以 表名,RowKey,列族:列 定位数据单元
   
   #2.查询数据单元
   get 'student','1001','info:name'
   
   #3.查询一或多行数据
   scan 'student', {STARTROW => '1001', STOPROW  => '1001'}
   #查询指定RowKey的数据
   scan 'student', {STARTROW => '1001', STOPROW  => '1003'}
   #查询多行RowKey的数据,范围左闭右开
   
   #4.删除数据单元的最后一次数据
   delete 'student','1001','info:name'
   #注意:只删除最后更新的一次更新记录,历史数据不变更
   
   #5.删除数据单元的所有历史数据
   deleteall 'student', '1001'
   #根据RowKey删除一整行的数据,类似于关系型数据库的删除操作
   deleteall 'student', '1001', 'info:age'
   
   #6.清空表数据
   truncate 'student'
   ```



## 第三章 HBase进阶

### 3.1 ResionServer架构

<img src="Hbase.assets/image-20200620165326013.png" alt="image-20200620165326013" style="zoom:80%;" />

1. StoreFile（HFile）
   保存实际数据的物理文件，存储在HDFS上，每个Store会生成一或多个HFile文件，在每个HFile文件中数据会按照RowKey的字典顺序排序。
2. MemStore
   Store的缓存数据，每个Store各自拥有1个MemStore，在对Store的数据进行操作时，会将操作结果缓存在MemStore中，在达到刷写条件时再一次性写入HFile文件。
3. WAL（Write-Ahead logFile）
   在数据写入MemStore前，会先将操作写入WAL日志文件，避免内存数据的丢失，1个RegionServer的所有Store共享1个WAL文件，WAL文件存储在HDFS中。
4. BlockCache
   在执行查询操作时，查询结果会被缓存到BlockCache中，便于下次查询，1个RegionServer共用1个BlockCache。



### 3.2 写入数据流程

1. Client访问Zookeeper，获取hbase:meta表所属的RegionServer
2. Client访问查到的RegionServer节点，根据put请求查找目标数据匹配的Region所在的RegionServer节点，同时缓存该表的所有region信息到Client的meta cache中，便于下一次访问
3. Client与目标数据的RegionServer节点通信，将数据追加写入WAL
4. WAL的数据写入对应Store的MemStore中，MemStore中的数据会按照字典顺序排序，并等待刷写
5. RegionServer向Client返回ack确认信息
6. MemStore触发刷写后，将其中的数据一并刷写到1个新的HFile文件中



### 3.3 MemStore Flush

#### 3.3.1 刷写时机

- MemStore虽然存储的是所属Store中的数据，但刷写的执行却以Region整体为最小单位，即Region下的1个MemStore触发刷写时，该Region下的所有MemStore都会执行刷写操作。

- 自动刷写的触发时机

  1. MemStore中的数据量达到指定大小
     **hbase.hregion.memstore.flush.size**（默认128M）
     即默认1个MemStore中的数据达到128M时，Region下的所有MemStore都会进行刷写。

     当MemStore中的数据量达到
     **hbase.hregion.memstore.flush.size**（默认128M）* 
     **hbase.hregion.memstore.block.multiplier** （默认4）
     即默认1个MemStore中的数据达到512M时，系统会阻止数据继续向MemStore写入。

  2. RegionServer节点下所有MemStore的数据量即将达到内存分配的上限
     **java_heapsize** *
     **hbase.regionserver.global.memstore.size**（默认0.4）*
     **hbase.regionserver.global.memstore.size.lower.limit**（默认0.95）
     按照从大到小的顺序对多个Region中的MemStore依次进行刷写，直到MemStore的数据总和低于上述计算值。

     RegionServer节点下所有MemStore的数据量达到分配上限
     **java_heapsize** *
     **hbase.regionserver.global.memstore.size**（默认0.4）
     阻止向RegionServer下的所有MemStore写入数据

  3. 距离上一次刷写达到一定时间间隔
     **hbase.regionserver.optionalcacheflushinterval**（默认1hour）
     会自动进行一次刷写

  4. 当WAL中的文件数量超过32时，系统会按照时间顺序对多个Region中的MemStore依次进行刷写，直到WAL中的文件数不足32时结束刷写



### 3.4 读取数据流程

1. Client访问Zookeeper，获取hbase:meta表所属的RegionServer

2. Client访问查到的RegionServer节点，根据put请求查找目标数据匹配的Region所在的RegionServer节点，同时缓存该表的所有region信息到Client的meta cache中，便于下一次访问
3. Client与目标数据的RegionServer节点通信
4. 分别在Block Cache，MemStore和HFile中查找目标数据，并对查找结果进行合并（取得最新版本的数据）
5. 将查找到的数据所在的数据块（HFile中的存储单元，64KB）缓存到Block Cache中，便于后续查询
6. RegionServer将合并的结果数据返回Client



### 3.5 StoreFile Compaction

- MemStore中的数据每进行1次刷写，就会生成1个新的HFile文件，文件量越积越多，Stroe的Compaction机制会对同一Store所属的HFile文件进行合并
- 两种合并方式
  1. Minor Compaction
     - 作用：合并若干个HFile文件，并对合并后的文件数据进行归并排序
     - 触发时机：MemStore Flush、定期检测等操作都可能触发Minor Compaction
  2. Major Compaction
     - 作用：将1个Store下的所有HFile文件合并为1个HFile文件，并对合并后的文件数据进行归并排序
     - 触发时机：默认定期执行（1周1次），生产环境下通常设置为手动，在集群负载较小时手动执行



### 3.6 Region Split

- 默认情况下，Table在创建时只会产生1个Region，随着数据的不断增加，Region会根据拆分策略自动进行拆分。
- 拆分结束后的两个Region都位于当前的RegionServer，HMaster会在根据负载均衡调整Region所属的RegionServer。
- HBase提供了多种Region的拆分策略



#### ConstantSizeRegionSplitPolicy

- 0.94版本前的默认拆分策略
- 说明：当1个Region中的数据总量超过 hbase.hregion.max.filesize 的设置值（默认10G）后会进行拆分



#### IncreasingToUpperBoundRegionSplitPolicy

- 0.94-2.0版本前的默认拆分策略
- 说明：当1个Region中的数据总量超过 Min(initialSize*R^3 ,hbase.hregion.max.filesize") 后会进行拆分
  - initialSize = 2 * hbase.hregion.memstore.flush.size （默认2 * 128M）
  - R = 当前RegionServer中该Table的Region个数
  - hbase.hregion.max.filesize （默认10G）



#### SteppingSplitPolicy

- 2.0版本起的默认拆分策略
- 说明：
  - 若当前RegionServer中该Table的Region个数为1，则Region达到 2 * hbase.hregion.memstore.flush.size （默认256M）时拆分
  - 若当前RegionServer中该Table的Region个数为2或以上，则Region达到 hbase.hregion.max.filesize（默认10G）时拆分



## 第四章 HBase API

### 4.1 HBase依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-server</artifactId>
        <version>2.0.5</version>
    </dependency>

    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>2.0.5</version>
    </dependency>
</dependencies>
```



### 4.2 DDL API

- HBase的DDL操作核心流程

  1. 通过Zookeeper地址获取HBase连接对象
  2. 通过连接对象获取Admin对象进行DDL方法的调用

  ```java
  public class HBaseUtil {
  
      //1.声明Connection对象
      public static Connection connection;
  
      //2.通过配置Zookeeper集群创建Connection对象
      static {
          try {
              Configuration conf = HBaseConfiguration.create();
              conf.set("hbase.zookeeper.quorum","hadoop201,hadoop202,hadoop203");
              ConnectionFactory.createConnection(conf);
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      
      //3.定义后续的功能方法,实现HBase的DDL操作
  
  }
  ```

- 创建命名空间

  ```java
  public static void createNameSpace(String nameSpace) throws IOException {
      Admin admin = connection.getAdmin();
      try {
          NamespaceDescriptor name = NamespaceDescriptor.create(nameSpace).build();
          admin.createNamespace(name);
      } catch (NamespaceExistException e){
          System.err.println(nameSpace + "已经存在");
      } finally {
          admin.close();
      }
  }
  ```

- 创建表

  ```java
  public static void createTable(String nameSpace, String tableName, String... families) throws IOException {
      if (families.length < 1){
          System.out.println("至少传入1个列族");
          return;
      }
      Admin admin = connection.getAdmin();
      try {
          if (admin.tableExists(valueOf(nameSpace,tableName))){
              System.out.println(tableName + "表已存在");
              return;
          }
          TableDescriptorBuilder tableDescriptor
              = TableDescriptorBuilder.newBuilder(valueOf(nameSpace, tableName));
          for (String family : families) {
              ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder
                  = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(family));
              tableDescriptor.setColumnFamily(columnFamilyDescriptorBuilder.build());
          }
          admin.createTable(tableDescriptor.build());
      } finally {
          admin.close();
      }
  }
  ```

- 删除表

  ```java
  public static void deleteTable(String nameSpace, String tableName) throws IOException {
      Admin admin = connection.getAdmin();
      try {
          if (!admin.tableExists(valueOf(nameSpace,tableName))){
              System.out.println(tableName + "表不存在");
              return;
          }
          TableName table = valueOf(Bytes.toBytes(nameSpace), Bytes.toBytes(tableName));
          admin.deleteTable(table);
      } finally {
          admin.close();
      }
  }
  ```



### 4.3 DML API

- HBase的DML操作核心流程

  1. 通过Zookeeper地址获取HBase连接对象
  2. 通过连接对象获取Table对象进行DML方法的调用

  ```java
  public class HBaseUtil {
  
      //1.声明Connection对象
      public static Connection connection;
  
      //2.通过配置Zookeeper集群创建Connection对象
      static {
          try {
              Configuration conf = HBaseConfiguration.create();
              conf.set("hbase.zookeeper.quorum","hadoop201,hadoop202,hadoop203");
              ConnectionFactory.createConnection(conf);
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      
      //3.定义后续的功能方法,实现HBase的DDL操作
  
  }
  ```

- 插入数据单元

  ```java
  public static void putCell(String nameSpace, String tableName, String rowKey, String family, String column, String value) throws IOException {
      Table table = connection.getTable(valueOf(nameSpace, tableName));
      try {
          Put put = new Put(Bytes.toBytes(rowKey));
          put.addColumn(Bytes.toBytes(family), Bytes.toBytes(column), Bytes.toBytes(value));
          table.put(put);
      } finally {
          table.close();
      }
  }
  ```

- 查询数据单元

  ```java
  public static void getCell(String nameSpace, String tableName, String rowKey, String family, String column) throws IOException {
      Table table = connection.getTable(valueOf(nameSpace, tableName));
      try {
          Get get = new Get(Bytes.toBytes(rowKey));
          get.addColumn(Bytes.toBytes(family),Bytes.toBytes(column));
          Result result = table.get(get);
          Cell[] cells = result.rawCells();
          for (Cell cell : cells) {
              System.out.println(
                  "columnFamily = " + Bytes.toString(CellUtil.cloneFamily(cell)) +
                  ", column = " + Bytes.toString(CellUtil.cloneQualifier(cell)) +
                  ", value = " + Bytes.toString(CellUtil.cloneValue(cell))
              );
          }
      } finally {
          table.close();
      }
  }
  ```

- 扫描查询数据

  ```java
  public static void scanRows(String nameSpace, String tableName, String startRow, String stopRow) throws IOException {
      Table table = connection.getTable(TableName.valueOf(nameSpace, tableName));
      Scan scan = new Scan();
      Scan scan1 = scan.withStartRow(Bytes.toBytes(startRow)).withStopRow(Bytes.toBytes(stopRow));
      ResultScanner scanner = table.getScanner(scan1);
      try {
          for (Result result : scanner) {
              Cell[] cells = result.rawCells();
              for (Cell cell : cells) {
                  System.out.println(
                      "columnFamily = " + Bytes.toString(CellUtil.cloneFamily(cell)) +
                      ", column = " + Bytes.toString(CellUtil.cloneQualifier(cell)) +
                      ", value = " + Bytes.toString(CellUtil.cloneValue(cell))
                  );
              }
          }
      }finally {
          scanner.close();
          table.close();
      }
  }
  ```

- 删除数据单元（包括所有历史版本）

  ```java
  public static void deleteCell(String nameSpace, String tableName, String rowKey, String family, String column) throws IOException {
      Table table = connection.getTable(valueOf(nameSpace, tableName));
      try {
          Delete delete = new Delete(Bytes.toBytes(rowKey));
          delete.addColumns(Bytes.toBytes(family), Bytes.toBytes(column));
          table.delete(delete);
      } finally {
          table.close();
      }
  }
  ```



## 第五章 HBase优化

### 5.1 预分区

- 一个表的多个Region中分别维护了startRowKey和endRowKey，在向表中插入数据时，会根据rowKey的范围将数据交给指定Region维护，为避免多个Region的数据量不均导致数据倾斜，可以使用预分区控制数据的Region分配，同时提高作业的并行度

- 在生产环境下，1个表在1个RegionServer中的Region数为5-10个为宜

- 方式1：手动对表设定预分区

  ```shell
  create 'student', 'info', SPLITS => ['1000','2000','3000','4000']
  #产生5个Region (-∞,100) (1000,2000] (2000,3000] (3000,4000] (4000, +∞)
  #根据RowKey的字典顺序将数据插入对应的Region
  ```

- 方式2：生成16进制序列预分区

  ```shell
  create 'student', 'info', {NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
  #产生15个Region
  #在RowKey值也为16进制序列,且均匀分布的场景下使用
  ```

- 方式3：读取文件的预分区规则

  ```shell
  #1.在文件中设定分区值,效果同方式1
  aaa
  bbb
  ccc
  ddd
  ```

  ```shell
  #使用分区文件建表,注意相对路径起点为hbase shell命令执行时所在路径
  create 'student', 'info', SPLITS_FILE => 'splits.txt'
  ```

- 方式4：使用JavaAPI创建预分区

  ```java
  //1.自定义函数获取散列值
  byte[][] splitKeys
  
  //2.建表时传入第二个参数
  admin.createTable(tableDescriptor.build(), splitKeys);
  ```



### 5.2 RowKey设计

- RowKey是一条数据的唯一标识，同时直接决定了数据所属的Region，因此在设计RowKey时，要集中考虑两点
  1. 在不断插入数据到表中时，尽量让数据均匀的写入不同分区，**避免数据倾斜**
  2. 根据需求尽量将需要做关联、聚合计算的数据送往1个Region，**减少作业的磁盘IO**
- 常见的RowKey设计方案有
  1. 生成随机数、hashCode、散列值
  2. 反转字符串（如时间相关字段）
  3. 字符串拼接（如user_id + timestamp）



### 5.3 内存优化

- HBase中数据的读写需要大量的内存，但不建议分配过大的堆内存，这样会使GC过程持续过长导致RegionServer长时间不可用
- 一般情况下，**每个RegionServer节点分配16-36G堆内存**为宜



### 5.4 基础优化

通过修改hbase-site.xml文件可以对hbase进行一系列的基础优化

1. Zookeeper会话超时时间

   ```xml
   <!-- 当超过指定时间(毫秒)未响应后,Zookeeper判定RegionServer节点故障,生产环境可配置为60000 -->
   <property>
       <name>zookeeper.session.timeout</name>
       <value>90000</value>
   </property>
   ```

2. 监听的RPC数量

   ```xml
   <!-- 指定RPC监听的数量,读写请求较多时可增加此参数 -->
   <property>
       <name>hbase.regionserver.handler.count</name>
       <value>30</value>
   </property>
   ```

3. 手动进行Major Compaction

   ```xml
   <!-- 间隔指定时间(秒)后自动进行1次Major Compaction,默认7天,关闭此功能可设置为0 -->
   <property>
       <name>hbase.regionserver.handler.count</name>
       <value>604800000</value>
   </property>
   ```

4. 优化HFile文件大小

   ```xml
   <!-- RS下Region数达到2个以后的Region自动拆分临界值(字节),默认10GB,可适当下调,减少单个map任务的作业量 -->
   <property>
       <name>hbase.hregion.max.filesize</name>
       <value>10737418240</value>
   </property>
   ```

5. 优化HBase客户端缓存

   ```xml
   <!-- HBase客户端的缓存大小(字节),默认2M,适当增大可减少RPC次数 -->
   <property>
       <name>hbase.client.write.buffer</name>
       <value>2097152</value>
   </property>
   ```

6. 指定scan.next每次扫描HBase的行数

   ```xml
   <!-- scan.next方法获取的默认行数,增大可增加效率,内存消耗也会增大 -->
   <property>
       <name>hbase.client.scanner.caching</name>
       <value>1</value>
   </property>
   ```

7. BlockCache占用RegionServer堆内存的比例

   ```xml
   <!-- 默认0.4,该配置与global.memstore.size总和应配置为0.8,读请求较多时应增大 -->
   <property>
       <name>hfile.block.cache.size</name>
       <value>0.4</value>
   </property>
   ```

8. MemStore占用RegionServer堆内存的比例

   ```xml
   <!-- 默认0.4,该配置与block.cache.size总和应配置为0.8,写请求较多时应增大 -->
   <property>
       <name>hbase.regionserver.global.memstore.size</name>
       <value>0.4</value>
   </property>
   ```



## 第六章 整合Phoenix

### 6.1 Phoenix概述

- 定义：Phoenix是HBase的开源SQL皮肤，可以使用标准的JDBC API代替HBase客户端API进行DDL，DML操作
- 特点
  1. 易集成：与Spark，Hive，Pig，Flume，MR均可集成
  2. 操作简单：直接通过DML和DDL命令操作数据库
  3. 支持HBase二级索引创建，加快查询速度
- 架构：Phoenix支持两种客户端访问模式
  1. thin client：与RegionServer节点中的Phoenix Query Server交互，建立JDBC连接
  2. thick client：与Zookeeper交互，建立JDBC连接
- Phoenix支持数据库的多种数据类型（如int），但这些数据类型时Phoenix专有而非hbase的数据类型，通过原生hbase是无法获取这些专有数据类型的数据的（乱码），因此**建议Phoenix建表字段全部使用varchar类型**



### 6.2 Phoenix入门

#### 6.2.1 安装Phoenix

1. 上传并解压安装phoenix

   ```shell
   tar -zxvf apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz -C /opt/module/
   #注意:官方的phoenix仅支持HBase2.0版本,不支持2.0.5版本,2.0.5版本的HBase需要修改源码重编译后才能使用
   mv /opt/module/apache-phoenix-5.0.0-HBase-2.0-bin /opt/module/phoenix-5.0.0
   ```

2. 复制jar包到HBase

   ```shell
   cp /opt/module/phoenix-5.0.0/phoenix-5.0.0-HBase-2.0-server.jar /opt/module/hbase-2.0.5/lib/
   ```

3. 分发复制的jar包

   ```shell
   xsync /opt/module/hbase-2.0.5/lib
   ```

4. 配置环境变量

   ```shell
   sudo vi /etc/profile.d/my_env.sh
   ```

   ```shell
   #PHONIX_HOME
   export PHOENIX_HOME=/opt/module/phoenix-5.0.0
   export PHOENIX_CLASSPATH=$PHOENIX_HOME
   export PATH=$PATH:$PHOENIX_HOME/bin
   ```

   ```
   source /etc/profile.d/my_env.sh
   ```

5. 修改HBase的conf/hbase-site.xml文件

   ```shell
   vi /opt/module/hbase-2.0.5/conf/hbase-site.xml
   ```

   ```xml
   <!-- 添加如下配置 -->
   <property>
       <name>phoenix.schema.isNamespaceMappingEnabled</name>
       <value>true</value>
   </property>
   ```

6. 修改Phoenix的bin/hbase-site.xml文件

   ```xml
   <!-- 同样添加如下配置 -->
   <property>
       <name>phoenix.schema.isNamespaceMappingEnabled</name>
       <value>true</value>
   </property>
   ```

7. 依次启动hdfs，zookeeper，hbase，phoenix

   ```shell
   start-dfs.sh
   zkctl start
   start-hbase.sh
   #使用厚客户端模式启动phoenix
   sqlline.py hadoop201,hadoop202,hadoop203:2181
   ```



#### 6.2.2 Phoenix Shell

- SQL操作

  ```mysql
  #注意 phoenix中所有命名与查询的表名或字段名在hbase中均会转换为大写存储,若要使用小写需要加""
  
  #1.显示所有表
  !table
  
  #2.创建表(phoenix中创建表必须指定主键,主键字段会作为hbase中表的rowkey存储)
  #方式1 将1个字段作为主键
  #在hbase的底层存储中,RowKey会作为1个数据单元额外存储1行,这行数据用于该RowKey的占位
  create table if not exists "student"(
  "id" varchar primary key,
  "name" varchar
  "age" varchar);
  #方式2 使用多个字段作为联合主键
  #作为主键的多个字段在habase底层存储中作为1个数据单元只占用1行,且不再额外单独存储
  create table if not exists "population" (
  "state" char(2) NOT NULL,
  "city" varchar NOT NULL,
  "population" bigint
  constraint "my_pk" primary key ("state", "city"));
  
  #3.插入数据(phoenix中将insert与update关键字功能合并为upsert使用)
  upsert into "student" values('1001', 'lisi', '20');
  
  #4.查询数据
  select * from "student";
  
  #5.删除数据
  delete from "student" where "id" = '1001';
  
  #6.删除表
  drop table "student";
  
  #7.退出命令行模式
  !quit
  ```
  
- 映射HBase中已有的表

  ```mysql
  #1.视图映射(将hbase中的表映射为phoenix中的视图,视图只读不可增删改)
  create view "student"(
      id varchar primary key,
      "info"."name" varchar,
      "info"."age" varchar);
  #查看视图
  select * from "student";
  #删除视图
  drop view "student";
  
      
  #2.表映射(将hbase中的表映射到phoenix中,可修改表的内容)
  create table "student"(
      id varchar primary key,
      "info"."name" varchar, 
      "info"."age" varchar)
  column_encoded_bytes=0;
  #查看表
  select * from "student";
  #删除表(会将hbase中的表一并删除)
  drop table "student";
  ```



#### 6.2.3 Phoenix JDBC

- thin客户端

  ```xml
  <!-- 导入依赖 -->
  <dependency>
      <groupId>org.apache.phoenix</groupId>
      <artifactId>phoenix-queryserver-client</artifactId>
      <version>5.0.0-HBase-2.0</version>
  </dependency>
  ```

  ```java
  public class ThinClient {
  
      public static void main(String[] args) throws SQLException {
          //通过queryserver所在节点及端口号获取url连接
          String url = ThinClientUtil.getConnectionUrl("hadoop201", 8765);
          Connection connection = DriverManager.getConnection(url);
          PreparedStatement preparedStatement = connection.prepareStatement("select * from \"student\"");
          ResultSet resultSet = preparedStatement.executeQuery();
          while (resultSet.next()){
              System.out.println(resultSet.getString(1) + ":" + resultSet.getString(2));
          }
      }
  
  }
  ```

- thick客户端

  ```xml
  <!-- 导入依赖 -->
  <dependency>
      <groupId>org.apache.phoenix</groupId>
      <artifactId>phoenix-core</artifactId>
      <version>5.0.0-HBase-2.0</version>
  </dependency>
  ```

  ```java
  public class ThickClient {
  
      public static void main(String[] args) throws SQLException {
          //通过zookeeper集群获取url连接
          String url = "jdbc:phoenix:hadoop201,hadoop202,hadoop203,2181";
          Properties props = new Properties();
          //配置参数与hbase-site.xml中一致
          props.setProperty("phoenix.schema.isNamespaceMappingEnabled", "true");
          Connection connection = DriverManager.getConnection(url,props);
          PreparedStatement preparedStatement = connection.prepareStatement("select * from \"student\"");
          ResultSet resultSet = preparedStatement.executeQuery();
          while (resultSet.next()){
              System.out.println(resultSet.getString(1) + ":" 
                      + resultSet.getString(2) + ":"
                      + resultSet.getString(3)
              );
          }
      }
  
  }
  ```



### 6.2 Phoenix二级索引

1. 修改HBase下的hbase-site.xml配置文件

   ```xml
   <!-- 添加如下配置-->
   <property>
       <name>hbase.regionserver.wal.codec</name>
       <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
   </property>
   ```

2. 分发到所有RegionServer节点

   ```shell
   xsync /opt/module/hbase-2.0.5/conf/hbase-site.xml
   ```



#### 6.2.1 全局二级索引

- 全局二级索引会在hbase中创建新的索引表，新表存储了二级索引字段到原表RowKey的映射

- 索引创建语句

  ```mysql
  #创建指定字段的二级索引表
  create index "my_index" on "student"("info"."name");
  #查询字段必须为索引字段,若查询其他字段则索引失效
  select "ID","name" from "student" where "name" = 'zhangsan'; #索引有效
  select "ID","name","age" from "student" where "name" = 'zhangsan'; #索引无效
  
  #创建覆盖多字段的二级索引
  create index "my_index1" on "student"("info"."name") include ("info"."age");
  #查询被覆盖的字段时,索引依然生效
  select "ID","name","age" from "student" where "name" = 'zhangsan'; #索引有效
  
  #删除索引
  drop index "my_index" on "student";
  ```



#### 6.2.2 本地二级索引

- 本地二级索引会直接在原HFile文件中追加索引，不会创建新的索引表

- 索引创建语句

  ```mysql
  #创建二级索引
  create local index "my_index" on "student"("info"."name");
  #本地二级索引自动包含了所有字段的覆盖索引效果
  select "ID","name","age" from "student" where "name" = 'zhangsan'; #索引有效
  
  #删除索引
  drop index "my_index" on "student";
  ```



## 第七章 集成Hive

### 7.1 对比

- Hive
  1. Hive是数据仓库工具，将HDFS中存储的数据文件在MySQL中进行映射，再使用HQL语言对数据进行操作
  2. 用于数据的离线分析和清理，延迟高
  3. 基于HDFS和MapReduce（可替换）
- HBase
  1. HBase是**面向列族存储的非关系型数据库**
  2. 用于存储结构化或非结构化数据，不适合多表关联查询（join操作），延迟低，提供了高速的数据访问
  3. 基于HDFS



### 7.2 Hbase集成Hive

- 在hive-site.xml配置文件中添加zookeeper配置

  ```shell
  vi /opt/module/hive-3.1.2/conf/hive-site.xml
  ```

  ```xml
  <property>
      <name>hive.zookeeper.quorum</name>
      <value>hadoop201,hadoop202,hadoop203</value>
  </property>
  <property>
      <name>hive.zookeeper.client.port</name>
      <value>2181</value>
  </property>
  ```

- 在Hive中创建表，并在HBase中创建同样的表

  ```mysql
  create table hive_hbase_student(
  id string,
  name string,
  age string)
  stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  with serdeproperties ("hbase.columns.mapping" = ":key, info:name, info:age")
  tblproperties ("hbase.table.name" = "default:student");
  #在hive中删除表后,hbase中存储的表也会被删除
  ```

- 在Hive中创建外部表，映射HBase中已经存在的表

  ```mysql
  create external table hive_hbase_student(
  id string,
  name string,
  age string)
  stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
  with serdeproperties ("hbase.columns.mapping" = ":key, info:name, info:age")
  tblproperties ("hbase.table.name" = "default:student");
  #对表的增删改操作会直接映射到hbase中,但由于是外部表,在hive中删除的表在hbase中依然存在
  ```

- 向关联了HBase表的Hive表导入数据

  ```mysql
  #关联了HBase表的Hive表无法直接使用load插入数据,必须借助中间表
  
  #1.将数据插入中间表
  load data local inpath '/home/admin/softwares/data/student.txt' into table tmp;
  
  #2.将数据从中间表插入到目标表
  insert into table hive_hbase_student select * from tmp;
  ```