# Kylin & Presto

## 第一章 Kylin

### 1.1 Kylin简介

#### 1.1.1 定义

- Kylin是一个开源的**分布式分析引擎**，提供了Hadoop/Spark之上的SQL查询接口及**多维分析**（OLAP）能力以支持超大规模数据的亚秒级查询。
- Kylin高速查询的实现原理是对数据进行多维度的**预计算**，在预计算范围内的查询可以快速返回结果。



#### 1.1.2 架构

![image-20200707215441951](Kylin & Presto.assets/image-20200707215441951.png)

- Rest Server：面向程序开发的入口点，实现Kylin的应用开发工作。应用程序可以提供查询、构建cube、获取元数据等功能。
- Query Engine：查询引擎，解析用户的查询请求，与其他组件交互，返回用户所需的结果。
- Routing：路由器，将Kylin无法执行的查询操作交给hive执行，由于hive执行效率过差，默认关闭该功能
- Metadata：元数据，Kylin执行的关键组件，主要为cube元数据，存储在hbase中
- Cube Build Engine：任务引擎，用于处理离线任务（shell脚本，JavaAPI，MR任务等）



#### 1.1.3 特点

1. 标准的SQL接口
2. 支持超大数据集
3. 亚秒级响应
4. 可伸缩性和高吞吐量：Kylin支持搭建集群
5. BI工具集成：可与ODBC、JDBC、RestAPI等工具集成，同时Kylin还提供了Zepplin插件用于访问Kylin



### 1.2 Kylin安装

1. 搭建并启动kylin运行所需环境

   ```shell
   #1.部署Hadoop、Hive、Zookeeper、Hbase
   #2.环境变量中配置HADOOP_HOME、HIVE_HOME、HBASE_HOME
   #3.启动hdfs、yarn(包括历史服务器)、hive、zookeeper、habse
   hadoopctl start
   hivectl start
   zkctl start
   start-habse.sh
   ```

2. 上传安装包并解压安装

   ```shell
   tar -zxvf apache-kylin-3.0.2-bin.tar.gz -C /opt/module/
   mv /opt/module/apache-kylin-3.0.2-bin /opt/module/kylin-3.0.2
   ```

3. 启动kylin

   ```shell
   bin/kylin.sh start
   ```

4. web端访问kylin

   ```shell
   http://hadoop201:7070/kylin
   #用户名 ADMIN
   #密码 KYLIN
   ```

5. 关闭kylin

   ```shell
   bin/kylin.sh stop
   ```



### 1.3 Kylin使用

Kylin的使用前需要设计好事实表与需要关联的维度表，即星型或雪花模型，并确定数据的聚合维度

#### 1.3.1 创建工程

1. 新建工程，填写名称后提交
   ![image-20200707223817858](Kylin & Presto.assets/image-20200707223817858.png)
2. 选择新建的工程
   ![image-20200707223900152](Kylin & Presto.assets/image-20200707223900152.png)



#### 1.3.2 获取数据源

1. 点击Data Source
   ![image-20200707223951428](Kylin & Presto.assets/image-20200707223951428.png)
2. 点击加载表
   ![image-20200707224205045](Kylin & Presto.assets/image-20200707224205045.png)
3. 选择所有需要读取的表（事实表+维度表），点击解析
   ![image-20200707224453863](Kylin & Presto.assets/image-20200707224453863.png)



#### 1.3.3 创建model

1. 新建Model
   ![image-20200707225204939](Kylin & Presto.assets/image-20200707225204939.png)

2. 自定义model名
   ![image-20200707225325604](Kylin & Presto.assets/image-20200707225325604.png)

3. 选择事实表
   ![image-20200707225416428](Kylin & Presto.assets/image-20200707225416428.png)

4. 添加维度表，可添加多个
   ![image-20200707225442051](Kylin & Presto.assets/image-20200707225442051.png)
   添加关联配置
   ![image-20200707225936636](Kylin & Presto.assets/image-20200707225936636.png)
   配置所有维度表后点击下一步
   ![image-20200707230134547](Kylin & Presto.assets/image-20200707230134547.png)

5. 指定维度字段（即分组聚合运算时的group by字段）
   ![image-20200707230332102](Kylin & Presto.assets/image-20200707230332102.png)

6. 指定度量值字段（即聚合运算的目标字段）
   ![image-20200707230545639](Kylin & Presto.assets/image-20200707230545639.png)

7. 指定事实表的分区字段及解析格式，保存

   ![image-20200707230800452](Kylin & Presto.assets/image-20200707230800452.png)



#### 1.3.4 构建cube

1. 新建Cube
   ![image-20200709115917040](Kylin & Presto.assets/image-20200709115917040.png)

2. 选择model，并命名cube
   ![image-20200709115814012](Kylin & Presto.assets/image-20200709115814012.png)

3. 选择聚合计算的维度
   ![image-20200709120117443](Kylin & Presto.assets/image-20200709120117443.png)
   ![image-20200709120302475](Kylin & Presto.assets/image-20200709120302475.png)
   ![image-20200709120324289](Kylin & Presto.assets/image-20200709120324289.png)

4. 选择度量值，并配置聚合方式
   ![image-20200709120511145](Kylin & Presto.assets/image-20200709120511145.png)
   ![image-20200709120612208](Kylin & Presto.assets/image-20200709120612208.png)
   ![image-20200709120631099](Kylin & Presto.assets/image-20200709120631099.png)

5. 配置Cube数据自动合并日期（每天构建Cube时，都会将结果保存到hbase的一张新表中，为避免小文件过多，可配置7天进行一次小合并，28天进行一次大合并，减少文件数）
   ![image-20200709120929756](Kylin & Presto.assets/image-20200709120929756.png)

6. 高级配置（可跳过）

   ![image-20200709121107120](Kylin & Presto.assets/image-20200709121107120.png)

7. Kylin配置项覆盖
   ![image-20200709121147680](Kylin & Presto.assets/image-20200709121147680.png)

8. 保存cube
   ![image-20200709121213919](Kylin & Presto.assets/image-20200709121213919.png)

9. 构建cube（执行构建后会开始预计算过程，并将计算结果保存到hbase中）
   ![image-20200709122111472](Kylin & Presto.assets/image-20200709122111472.png)
   选择构建的日期范围后提交

   ![image-20200709122134224](Kylin & Presto.assets/image-20200709122134224.png)
   查看构建进度
   ![image-20200709124144928](Kylin & Presto.assets/image-20200709124144928.png)



#### 1.3.5 使用进阶

- 在构建cube时，需要确保维度表中用于与事实表关联的字段唯一，即对于每日全量导入的维度表需要筛选当天的数据后再关联。可以通过hive对维度表创建当天数据的视图后，在构建model时使用创建的视图关联事实表解决。

  1. 启动hive，创建当日数据的视图

     ```mysql
     create view dwd_dim_user_info_his_view as select * from dwd_dim_user_info_his where end_date='9999-99-99';
     create view dwd_dim_sku_info_view as select * from dwd_dim_sku_info where dt=date_add(current_date,-1);
     ```

  2. 修改数据源，添加视图表

  3. 新建model，构建cude

- 实现每日自动执行cube构建。

  1. 编写执行kylin任务的shell脚本，构建指定的cube

     ```shell
     #!/bin/bash
     cube_name=order_detail_cube
     do_date=`date -d '-1 day' +%F`
     
     #获取00:00时间戳
     start_date_unix=`date -d "$do_date 08:00:00" +%s`
     start_date=$(($start_date_unix*1000))
     
     #获取24:00的时间戳
     stop_date=$(($start_date+86400000))
     
     curl -X PUT -H "Authorization: Basic QURNSU46S1lMSU4=" -H 'Content-Type: application/json' -d '{"startTime":'$start_date', "endTime":'$stop_date', "buildType":"BUILD"}' http://hadoop102:7070/kylin/api/cubes/$cube_name/build
     ```

  2. 将脚本提交到Azkaban工作流中调用



### 1.4 Kylin Cube构建原理

#### 1.4.1 维度和度量

- 维度：观察数据的角度，如地区，性别，商品类别等等，对应表中做统计时的分组字段。
- 度量：被聚合的统计值，对应表中做聚合运算的字段。



#### 1.4.2 Cube和Cuboid

- Cuboid：在一个数据模型中（事实表+维度表），根据一或多个维度（分组字段），对度量值（聚合字段）进行聚合运算，产生的物化视图（即查询结果），称为一个Cuboid。
- Cube：一个数据模型的所有不同的维度组合的多个Cuboid，称为一个Cube。一个Cube的Cuboid数量取决于维度的个数，公式为2^n-1。
- 示例：若1个数据模型需要从性别、地区、商品类别3个维度统计，则共有性别、地区、商品类别、性别+地区、性别+商品类别、地区+商品类别、性别+地区+商品类别共7个维度组合，即7个Cuboid，7个Cuboid整体是1个Cube。



#### 1.4.3 Cube存储原理

- kylin构建cube后，会将计算结果**存储在hbase中**，这样在做预计算范围内的查询时可以直接获取结果而不需要再做聚合运算。
- Hbase中，cube存储使用的rowkey由两部分连接组成
  - 第一部分用于判定维度字段是否作为分组字段
  - 第二部分用于存储维度字段的具体值



#### 1.4.4 Cube构建算法

- Cube的计算**并非**是每个Cuboid各自根据分组条件字段，因为这样会产生大量的重复计算。
- 逐层构建算法：从高维度→低维度的逐层计算
  即若一个数据模型具有3个观察维度，则先将3个维度字段均作为group by条件聚合计算，得到第1个cuboid结果；
  再使用第一个cuboid结果，分别取其中2个不同的维度（3种组合），做group by聚合计算，又得到3个cuboid结果；
  最后将第二次计算的cuboid结果，再分别根据3个维度字段进行group by聚合计算，得到最后3个cuboid结果。
- 快速构建算法：优化后的逐层构建算法
  由于逐层构建算法采用了传统的MapReduce计算流程，将所有的聚合过程都交给ReduceTask处理，因此会造成Reduce负载过高。快速构建算法会在MapTask中对数据进行Combine预聚合，减少Reduce端的工作负重。



### 1.5 Kylin Cube构建优化

#### 1.5.1 使用衍生维度

- 对于一个数据模型，在构建cube时，每增加一个维度，就会使cuboid数量增加近一倍，占用大量的hbase存储空间。
- 一个维度表中，若包含较多的观察维度，可以将其构建为衍生维度，即在创建model选择维度时，勾选Derived。这样在构建cube时，会直接以维度表的**主键**替代该维度字段做聚合计算。
- 优劣
  - 优点：减少了Cube构建的预计算工作量，节约了hbase磁盘空间
  - 缺点：由于没有按照实际需求的维度字段进行预计算，在实际查询时仍然需要进一步进行聚合计算，响应速度会有所损失。因此，对于聚合工作量较大的维度字段，不建议设置为衍生维度。



#### 1.5.2 使用聚合组

- 一个数据模型在构建cube时，会对维度字段的所有组合情况均创建一个cuboid并进行计算，但在实际使用中，某些维度组合可能的查询结果并不需要，这时可以使用聚合组对Cube进行剪枝。

- 实现的方式为构建Cube时，在高级配置（Advanced Setting中对相应维度进行配置）

- 配置的种类

  1. 强制维度（Mandatory）：定义为强制维度的字段，必须在每个cuboid中作为分组条件出现

     ```
     如Cube包含A,B,C三个维度
     若定义A为强制维度
     则所有的cuboid维度组合为A,AB,AC,ABC共5种
     ```

  2. 层级维度（Hierarchy）：定义2个维度具有层级关系，其中一个维度的使用必须依赖于另一个维度的使用

     ```
     如Cube包含A,B,C三个维度
     若定义A→B为层级维度
     则所有的cuboid维度组合为A,C,AB,AC,ABC共5种,即B仅能在A使用的前提下使用
     ```

  3. 联合维度（Joint）：定义2个维度具有绑定关系，即二者视为一个整体使用，不能独立作为分组条件

     ```
     如Cube包含A,B,C三个维度
     若定义A,B为联合维度
     则所有的cuboid维度组合为C,AB,ABC共3种
     ```



#### 1.5.3 RowKey优化

1. 查询时作为过滤条件的维度字段放在前边
2. 字段数值离散度大的维度放在小的维度前边



#### 1.5.4 并发粒度优化

- 在构建cube是可以通过覆盖配置参数调整读取cuboid数据的并行度

- 相关参数

  ```properties
  #一个cuboid在hbase中的单个分区大小,默认为5.0(G)
  kylin.hbase.region.cut=5.0
  
  #一个cuboid的数据最少被划分为多少个分区,默认为1,可配置为2
  kylin.hbase.region.count.min=1
  
  #一个cuboid的数据最多被划分为多少个分区,默认为500,可配置为100
  kylin.hbase.region.count.max=500
  ```



### 1.6 Kylin BI工具

#### 1.6.1 JDBC

1. 新建Maven项目，导入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.kylin</groupId>
           <artifactId>kylin-jdbc</artifactId>
           <version>3.0.2</version>
       </dependency>
   </dependencies>
   ```

2. 建立JDBC连接，执行SQL

   ```java
   public class TestKylin {
   
       public static void main(String[] args) throws Exception {
   
           //Kylin_JDBC 驱动
           String KYLIN_DRIVER = "org.apache.kylin.jdbc.Driver";
   
           //Kylin_URL
           String KYLIN_URL = "jdbc:kylin://hadoop201:7070/gmall";
   
           //Kylin的用户名
           String KYLIN_USER = "ADMIN";
   
           //Kylin的密码
           String KYLIN_PASSWD = "KYLIN";
   
           //添加驱动信息
           Class.forName(KYLIN_DRIVER);
   
           //获取连接
           Connection connection = DriverManager.getConnection(KYLIN_URL, KYLIN_USER, KYLIN_PASSWD);
   
           //预编译SQL
           PreparedStatement ps = connection.prepareStatement("SELECT sum(sal) FROM emp group by deptno");
   
           //执行查询
           ResultSet resultSet = ps.executeQuery();
   
           //遍历打印
           while (resultSet.next()) {
               System.out.println(resultSet.getInt(1));
           }
       }
   }
   ```



#### 1.6.2 Zeppelin

1. 上传并解压安装zeppelin

   ```shell
   tar -zxvf zeppelin-0.8.0-bin-all.tgz -C /opt/module/
   mv zeppelin-0.8.0-bin-all/ zeppelin-0.8.0
   ```

2. 为避免与zookeeper端口号冲突，修改配置文件

   ```shell
   cd /opt/module/zeppelin-0.8.0/conf
   mv zeppelin-site.xml.template zeppelin-site.xml
   vi zeppelin-site.xml
   ```

   ```xml
   <!-- 修改端口号(默认8080) -->
   <property>
     <name>zeppelin.server.port</name>
     <value>8089</value>
     <description>Server port.</description>
   </property>
   ```

3. 启动zepplin

   ```shell
   bin/zeppelin-daemon.sh start
   ```

4. web端访问

   ```
   http://hadoop201:8089
   ```

5. 配置zeppelin支持kylin
   ![image-20200709143647677](Kylin & Presto.assets/image-20200709143647677.png)
   搜索kylin并编辑配置
   ![image-20200709143819717](Kylin & Presto.assets/image-20200709143819717.png)
   配置Kylin地址和项目名称后保存
   ![image-20200709144222386](Kylin & Presto.assets/image-20200709144222386.png)

6. 新建note，命名并选择解释器
   ![image-20200709144535684](Kylin & Presto.assets/image-20200709144535684.png)

7. 输入sql语句，执行查询
   ![image-20200709144834621](Kylin & Presto.assets/image-20200709144834621.png)

8. 可使用图形化工具直接对查询结果进行数据可视化
   ![image-20200709144936830](Kylin & Presto.assets/image-20200709144936830.png)



## 第二章 Presto

### 2.1 Presto简介

#### 2.1.1 概念

- Presto是一个开源的分布式SQL查询引擎，用于处理GB到PB级数据的秒级查询场景。
- 官网地址 https://prestodb.github.io/



#### 2.1.2 架构

![image-20200709231617185](Kylin & Presto.assets/image-20200709231617185.png)



#### 2.1.3 特点

- 优点
  1. 基于内存计算，减少磁盘IO，做聚合计算时并非一次性将数据读到内存，而是边读边计算，同时清理内存。
  2. 能够同时连接多个数据源，实现跨数据源关联表查询
- 缺点
  1. 多表关联查询时会产生大量临时数据，降低效率



#### 2.1.4 Presto与Impala

- Impala性能稍强于Presto
- Presto支持的数据源种类更丰富



### 2.2 Presto安装

#### 2.2.1 Presto Server安装

1. 上传并解压安装包到指定目录

   ```shell
   tar -zxvf presto-server-0.196.tar.gz -C /opt/module/
   mv /opt/module/presto-server-0.196/ /opt/module/presto-0.196
   ```

2. 创建用于存储数据的目录

   ```shell
   cd /opt/module/presto-0.196
   mkdir data
   ```

3. 创建用于存储配置文件的目录

   ```shell
   cd /opt/module/presto-0.196
   mkdir etc
   ```

4. 编写JVM配置文件

   ```shell
   vi /opt/module/presto-0.196/etc/jvm.config
   ```

   ```properties
   -server
   -Xmx16G
   -XX:+UseG1GC
   -XX:G1HeapRegionSize=32M
   -XX:+UseGCOverheadLimit
   -XX:+ExplicitGCInvokesConcurrent
   -XX:+HeapDumpOnOutOfMemoryError
   -XX:+ExitOnOutOfMemoryError
   ```

5. 配置hive数据源

   ```shell
   cd /opt/module/presto-0.196/etc
   mkdir catalog
   cd catalog
   vi hive.properties
   ```

   ```properties
   connector.name=hive-hadoop2
   hive.metastore.uri=thrift://hadoop201:9083
   ```

6. 分发presto到所有节点

   ```shell
   cd /opt/module
   xsync presto-0.196
   ```

7. 在所有节点分别配置node属性

   ```shell
   cd /opt/module/presto-0.196/etc
   vi node.properties
   ```

   ```properties
   #hadoop201
   node.environment=production
   node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
   node.data-dir=/opt/module/presto-0.196/data
   ```

   ```properties
   #hadoop202
   node.environment=production
   node.id=ffffffff-ffff-ffff-ffff-fffffffffffe
   node.data-dir=/opt/module/presto-0.196/data
   ```

   ```properties
   #hadoop203
   node.environment=production
   node.id=ffffffff-ffff-ffff-ffff-fffffffffffd
   node.data-dir=/opt/module/presto-0.196/data
   ```

8. 在hadoop201配置主机为coordinator节点

   ```shell
   cd /opt/module/presto-0.196/etc
   vi config.properties
   ```

   ```properties
   coordinator=true
   node-scheduler.include-coordinator=false
   http-server.http.port=8881
   query.max-memory=50GB
   discovery-server.enabled=true
   discovery.uri=http://hadoop201:8881
   ```

9. 在hadoop202、hadoop203上配置为两台主机为worker节点

   ```shell
   cd /opt/module/presto-0.196/etc
   vi config.properties
   ```

   ```properties
   coordinator=false
   http-server.http.port=8881
   query.max-memory=50GB
   discovery.uri=http://hadoop201:8881
   ```

10. 启动hive metastore

    ```shell
    nohup hive --service metastore >/dev/null 2>&1 &
    #或使用自定义脚本
    hivectl start
    ```

11. 在三台节点分别启动presto server

    ```shell
    #前台启动
    bin/launcher run
    
    #后台启动
    bin/launcher start
    
    #关闭服务
    bin/launcher stop
    ```

12. 编写群起群停脚本

    ```shell
    #!/bin/bash
    if [ $# -lt 1 ]
            then
                    echo "No Args Input Error"
            exit
    fi
    case $1 in
    "start")
            for i in hadoop201 hadoop202 hadoop203
            do
                    echo "==========start $i presto=========="
                    ssh $i '/opt/module/presto-0.196/bin/launcher start'
            done
    ;;
    "stop")
            for i in hadoop201 hadoop202 hadoop203
            do
                    echo "==========stop $i persto=========="
                    ssh $i '/opt/module/presto-0.196/bin/launcher stop'
            done
    ;;
    *)
            echo "Input Args Error"
    ;;
    esac
    ```

    



#### 2.2.2 Presto Client安装

1. 上传并重命名presto-cli-0.196-executable.jar

   ```shell
   mv presto-cli-0.196-executable.jar  prestocli
   ```

2. 添加执行权限

   ```shell
   chmod 777 prestocli
   ```

3. 启动perstocli

   ```shell
   ./prestocli --server hadoop201:8881 --catalog hive --schema default
   ```

4. 可使用与hive类似的命令行操作

   ```mysql
   #查看数据库,使用schema替代database
   show schemas;
   
   #查询表后按q退出
   #presto默认不支持读取lzo格式压缩的表,需要进行相关配置
   ```



#### 2.2.3 Presto LZO压缩配置

1. 添加lzo依赖到presto插件目录

   ```shell
   cd /opt/module/presto-0.196/plugin/hive-hadoop2
   cp /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar ./
   ```

2. 分发到所有节点

   ```shell
   xsync hadoop-lzo-0.4.20.jar
   ```

3. 重启presto服务

   ```shell
   prestoctl stop
   prestoctl start
   ./prestocli --server hadoop201:8881 --catalog hive --schema default
   ```



#### 2.2.4 Presto 可视化Client安装

1. 上传并解压安装包

   ```shell
   unzip yanagishima-18.0.zip
   ```

2. 移动到指定路径

   ```shell
   mv yanagishima-18.0 /opt/module
   ```

3. 修改配置文件

   ```shell
   vi /opt/module/yanagishima-18.0/yanagishima.properties
   ```

   ```properties
   #追加以下配置
   jetty.port=7080
   #自定义数据源名,并对该数据源进行相关配置
   presto.datasources=atguigu-presto
   presto.coordinator.server.atguigu-presto=http://hadoop201:8881
   catalog.atguigu-presto=hive
   schema.atguigu-presto=default
   sql.query.engines=presto
   ```

4. 在安装目录下，启动yanagishima

   ```shell
   nohup bin/yanagishima-start.sh >y.log 2>&1 &
   
   #关闭yanagishima
   nohup bin/yanagishima-shutdown.sh >y.log 2>&1 &
   ```

5. web端访问

   ```
   http://hadoop201:7080
   ```

6. 执行查询
   ![image-20200713150928997](Kylin & Presto.assets/image-20200713150928997.png)



### 2.3 Presto优化之数据存储

1. 合理选择表的分区方式
2. 使用ORC列式存储表数据，Presto对ORC文件读取进行了特定优化，性能优于Parquet
3. 使用压缩格式存储表，推荐Snappy



### 2.4 Presto优化之SQL查询

1. select字段尽量明确，避免使用*
2. 过滤条件必须带有分区字段
3. 使用group by以多个字段作为分组条件时，以字段数据跨度的降序排列，如group by user_id，gender
4. 使用order排序的同时使用limit，减少计算压力
5. 多表join时，数据量大的表放在左边



### 2.5 与HQL语法区别

1. 字段与关键字冲突时，使用双引号区分（MySQL中使用飘号）

   ```sql
   select "name" from user_info;
   ```

2. 比较时间字段时，需要添加timestamp关键字

   ```sql
   select t from a where t > timestamp '2017-01-01 00:00:00';
   ```

3. 不支持insert overwrite语法，仅支持先delete再insert into

4. 对于Parquet格式存储的表文件，仅支持select，不支持insert