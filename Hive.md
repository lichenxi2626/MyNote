# Hive

## 第一章 Hive 基本概念

### 1.1 概念

- Hive是基于Hadoop开发的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL的查询功能。
- Hive的本质：将HQL转化为MapReduce程序执行。

### 1.2 Hive的优缺点

- 优点
  1. 使用类SQL语法操作，语法简单易上手
  2. 无需编写MapReduce程序
  3. 可处理海量数据
  4. 支持自定义函数
- 缺点
  1. HQL（Hive Query Language）表达能力有限，无法实现迭代算法
  2. MapReduce作业的代码自动生成，效率较差，调优困难
  3. 执行延迟高，实时性差，处理少量数据不能体现时间优势

### 1.3 Hive架构原理

<img src="Hive.assets/wps1.png" alt="img" style="zoom:150%;" />

- Hive中表的具体数据存储在HDFS中，而元数据存储在MySQL的本地库中。

### 1.4 Hive与MySQL数据库

- 查询语言
  - Hive：使用类SQL语言HQL
  - MySQL：使用SQL语言
- 数据更新
  - Hive：主要用于操作数据仓库，因此主要使用查询功能，不建议使用增删改的操作
  - MySQL：可任意使用增删改查功能
- 执行延迟
  - Hive：没有索引，查询需要整表扫描，同时还要使用MapReduce框架，延迟较高
  - MySQL：低延迟
- 数据规模
  - Hive：支持对集群中大规模数据的计算操作
  - MySQL：仅支持小规模数据的处理



## 第二章 Hive 安装

### 2.1 Hive下载地址

```shell
#Hive官网地址
http://hive.apache.org/
#文档查看地址
https://cwiki.apache.org/confluence/display/Hive/GettingStarted
#下载地址
http://archive.apache.org/dist/hive/
#github地址
https://github.com/apache/hive
```



### 2.2 MySQL的安装

1. 官网下载MySQL镜像，官网地址：http://dev.mysql.com/downloads/mysql/

2. 检查系统是否已经安装了mysql或mariadb（mysql的衍生版本）

   ```shell
   rpm -qa | grep mysql
   rpm -qa | grep mariadb
   #若已经安装则需要卸载
   rpm -e --nodeps  mariadb-libs
   ```

3. 上传mysql安装包

4. 解压mysql安装包

   ```shell
   tar -xvf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
   ```

5. 依次安装rpm（因为RPM包间存在依赖关系，必须按照顺序安装）

   ``` shell
   #先安装可能需要的依赖
   yum install -y perl perl-devel
   yum install -y libaio
   #再安装rpm包
   rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
   rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
   rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
   rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
   rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
   ```

6. 删除 /etc/my.cnf 配置文件中 datadir 指向的目录下的所有内容

   ```shell
   #/etc/my.cnf中的内容
   [mysqld]
   datadir=/var/lib/mysql
   
   #删除datadir目录下的所有内容
   cd /var/lib/mysql
   rm -rf ./*
   ```

7. 初始化数据库

   ```shell
   #初始化数据库
   mysqld --initialize --user=mysql
   ```

8. 启动mysql服务

   ``` shell
   systemctl start mysqld.service
   ```

9. 查看自动生成的root用户的12位密码

   ```shell
   cat /var/log/mysqld.log
   ```

10. 登陆mysql数据库

    ```shell
    mysql -uroot -p
    #输入8中生成的初始密码
    ```

11. 首次登陆需要修改密码才能进行后续操作

    ```mysql
    #设置密码为123456
    set password = password("123456");
    ```

12. 设置root用户的远程访问

    ```mysql
    update mysql.user set host = '%' where user = 'root';
    flush privileges;
    #设置后查看
    select Host,User,authentication_string from user;
    ```

13. 查看并开启mysql服务自启动

    ```shell
    systemctl list-unit-files | grep mysqld.service #查看
    systemctl enable mysqld.service #开启自启动
    systemctl disable mysqld.service #关闭自启动
    ```

14. 修改配置文件 /etc/my.cnf，最终配置如下

    ```properties
    # For advice on how to change settings please see
    # http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
    
    [mysqld]
    #修改默认字符集为utf-8
    character_set_server=utf8
    collation-server=utf8_general_ci
    #设置大小写不敏感
    lower_case_table_names=1
    #开启缓存
    query_cache_type=1
    #
    # Remove leading # and set to the amount of RAM for the most important data
    # cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
    # innodb_buffer_pool_size = 128M
    #
    # Remove leading # to turn on a very important data integrity option: logging
    # changes to the binary log between backups.
    # log_bin
    #
    # Remove leading # to set options mainly useful for reporting servers.
    # The server defaults are faster for transactions and fast SELECTs.
    # Adjust sizes as needed, experiment to find the optimal values.
    # join_buffer_size = 128M
    # sort_buffer_size = 2M
    # read_rnd_buffer_size = 2M
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock
    
    # Disabling symbolic-links is recommended to prevent assorted security risks
    symbolic-links=0
    
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid
    ```



### 2.3 Hive的安装

1. 上传Hive安装包

2. 解压安装包到指定路径，并重命名

   ```shell
   tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /opt/module/
   mv /opt/module/apache-hive-3.1.2-bin /opt/module/hive-3.1.2
   ```

3. 为Hive追加环境变量

   ```shell
   vi /etc/profile.d/my_env.sh
   
   #在java、hadoop环境变量后追加hive环境变量
   #JAVA_HOME
   JAVA_HOME=/opt/module/jdk1.8.0_212
   #HADOOP_HOME
   HADOOP_HOME=/opt/module/hadoop-3.1.3
   #HIVE_HOME
   HIVE_HOME=/opt/module/hive-3.1.2
   #PATH
   PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
   export PATH JAVA_HOME HADOOP_HOME HIVE_HOME
   
   #重新加载环境变量
   source /etc/profile.d/my_env.sh
   ```

4. 解决Hadoop和Hive日志Jar包冲突问题

   ```shell
   #设置log4j-slf4j-impl-2.10.0.jar失效
   mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
   ```



### 2.4 配置Hive元数据到MySQL

1. 上传 MySQL 的 JDBC 驱动到 Hive 安装目录的 lib 目录下

   ```shell
   cp mysql-connector-java-5.1.48.jar $HIVE_HOME/lib
   ```

2. 创建 hive-site.xml 配置文件

   ```shell
   vi $HIVE_HOME/conf/hive-site.xml
   ```

   ```xml
   <!-- 添加以下内容 -->
   <?xml version="1.0"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
       
       <!-- jdbc连接的URL(mysql地址) -->
       <property>
           <name>javax.jdo.option.ConnectionURL</name>
           <value>jdbc:mysql://hadoop100:3306/metastore?useSSL=false</value>
       </property>
       <!-- jdbc连接的Driver-->
       <property>
           <name>javax.jdo.option.ConnectionDriverName</name>
           <value>com.mysql.jdbc.Driver</value>
       </property>
       <!-- jdbc连接的mysql用户的username-->
       <property>
           <name>javax.jdo.option.ConnectionUserName</name>
           <value>root</value>
       </property>
       <!-- jdbc连接的mysql用户的password -->
       <property>
           <name>javax.jdo.option.ConnectionPassword</name>
           <value>123456</value>
       </property>
       <!-- Hive默认在HDFS的工作目录 -->
       <property>
           <name>hive.metastore.warehouse.dir</name>
           <value>/user/hive/warehouse</value>
       </property>
       <!-- Hive元数据存储版本的验证 -->
       <property>
           <name>hive.metastore.schema.verification</name>
           <value>false</value>
       </property>
       <!-- 指定存储元数据要连接的地址(mysql地址) -->
       <property>
           <name>hive.metastore.uris</name>
           <value>thrift://hadoop100:9083</value>
       </property>
       <!-- 指定hiveserver2连接的端口号 -->
       <property>
           <name>hive.server2.thrift.port</name>
           <value>10000</value>
       </property>
       <!-- 指定hiveserver2连接的host -->
       <property>
           <name>hive.server2.thrift.bind.host</name>
           <value>hadoop100</value>
       </property>
       <!-- 元数据存储授权 -->
       <property>
           <name>hive.metastore.event.db.notification.api.auth</name>
           <value>false</value>
       </property>
   
   </configuration>
   ```



### 2.5 Tez引擎的安装

- Tez是Hive的一种运行引擎，性能优于MapReduce，这是因为Tez将多个具有依赖关系的MapReduce作业转换为1个作业，大幅减少了HDFS的读写，从而提升性能。

- Tez的安装部署

  1. 上传tez安装包（分本地包和HDFS包）

  2. 解压本地安装包到指定路径

     ```shell
     tar -zxvf tez-0.10.1-SNAPSHOT-minimal.tar.gz -C /opt/module/tez
     ```

  3. 启动hadoop集群，上传tez包到HDFS

     ```shell
     hadoop fs -mkdir /tez
     hadoop fs -put tez-0.10.1-SNAPSHOT.tar.gz /tez
     ```

  4. 在hadoop的配置文件目录下创建tez-site.xml

     ```shell
     vi %HADOOP_HOME/etc/hadoop/tez-site.xml
     ```

     ```xml
     <!-- 添加以下内容 -->
     <?xml version="1.0" encoding="UTF-8"?>
     <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
     <configuration>
         <!-- 设置hdfs中的上传的tez包路径 -->
         <property>
             <name>tez.lib.uris</name>
             <value>${fs.defaultFS}/tez/tez-0.10.1-SNAPSHOT.tar.gz</value>
         </property>
         <property>
             <name>tez.use.cluster.hadoop-libs</name>
             <value>true</value>
         </property>
         <property>
             <name>tez.am.resource.memory.mb</name>
             <value>1024</value>
         </property>
         <property>
             <name>tez.am.resource.cpu.vcores</name>
             <value>1</value>
         </property>
         <property>
             <name>tez.container.max.java.heap.fraction</name>
             <value>0.4</value>
         </property>
         <property>
             <name>tez.task.resource.memory.mb</name>
             <value>1024</value>
         </property>
         <property>
             <name>tez.task.resource.cpu.vcores</name>
             <value>1</value>
         </property>
     </configuration>
     ```

  5. 配置hadoop环境变量

     ```shell
     vim $HADOOP_HOME/etc/hadoop/shellprofile.d/tez.sh
     ```

     ```shell
     #添加以下内容
     hadoop_add_profile tez
     function _tez_hadoop_classpath
     {
         hadoop_add_classpath "$HADOOP_HOME/etc/hadoop" after
         #本地tez包解压目录
         hadoop_add_classpath "/opt/module/tez/*" after
         #本地tez包解压目录下的lib目录
         hadoop_add_classpath "/opt/module/tez/lib/*" after
     }
     ```

     ```shell
     #修改权限
     chmod 777 $HADOOP_HOME/etc/hadoop/shellprofile.d/tez.sh
     ```

  6. 修改 hive-stie.xml 文件，部署tez

     ```shell
     vim $HIVE_HOME/conf/hive-site.xml
     ```

     ```xml
     <!-- 添加以下内容 -->
     <property>
         <name>hive.execution.engine</name>
         <value>tez</value>
     </property>
     <property>
         <name>hive.tez.container.size</name>
         <value>1024</value>
     </property>
     ```

  7. 解决tez和hadoop的日志Jar包冲突

     ``` shell
     mv /opt/module/tez/lib/slf4j-log4j12-1.7.10.jar /opt/module/tez/lib/slf4j-log4j12-1.7.10.bak
     ```



### 2.6 Hive的启动

1. 登陆mysql

   ```shell
   mysql -uroot -p
   ```

2. 新建数据库，用于存储Hive的元数据

   ```mysql
   create database metastore;
   quit;
   ```

3. 初始化Hive元数据库

   ```shell
   schematool -initSchema -dbType mysql -verbose
   ```

   ```mysql
   #初始化完成后可以再次登陆mysql查看刚才创建的数据库中生成了多张表
   use metastore;
   show tables;
   ```

4. 编写Hive启动关闭脚本

   ```shell
   # hive启动需要启动metastore和hiveserver2两个进程,而这两个进程会占用shell窗口,非常不便
   # 因此需要自己编写脚本用于hive的启动和关闭
   vi $HIVE_HOME/bin/hivectl.sh
   ```

   ```shell
   #!/bin/bash
   #设置日志输出文件路径
   HIVE_LOG_DIR=$HIVE_HOME/logs
   if [ ! -d $HIVE_LOG_DIR ]
   then
   	mkdir -p $HIVE_LOG_DIR
   fi
   #检查进程是否运行正常，参数1为进程名，参数2为进程端口
   function check_process()
   {
       pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
       ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
       echo $pid
       [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
   }
   
   function hive_start()
   {
       metapid=$(check_process HiveMetastore 9083)
       cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
       # nohup开头 表示不挂起,即关闭shell终端窗口也保持运行
       # 2>&1 表示将错误重定向到标准输出上
       # &结尾 表示后台运行
       cmd=$cmd" sleep 4; hdfs dfsadmin -safemode wait >/dev/null 2>&1"
       [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
       server2pid=$(check_process HiveServer2 10000)
       cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
       [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
   }
   
   function hive_stop()
   {
       metapid=$(check_process HiveMetastore 9083)
       [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
       server2pid=$(check_process HiveServer2 10000)
       [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
   }
   
   case $1 in
   "start")
       hive_start
       ;;
   "stop")
       hive_stop
       ;;
   "restart")
       hive_stop
       sleep 2
       hive_start
       ;;
   "status")
       check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
       check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
       ;;
   *)
       echo Invalid Args!
       echo 'Usage: '$(basename $0)' start|stop|restart|status'
       ;;
   esac
   ```

   ```shell
   #修改脚本权限
   chmod 777 $HIVE_HOME/bin/hivectl.sh
   ```

5. 启动Hive

   ```shell
   #需要启动hadoop集群
   hivectl.sh start
   ```

6. 访问hive

   ```shell
   #方式1 Hive访问
   hive #启动客户端
   quit #退出客户端
   
   #方式2 HiveJDBC访问
   beeline -u jdbc:hive2://hadoop100:10000 -n atguigu #使用atguigu用户启动客户端
   !quit #退出客户端
   ```



### 2.7 Hive常用命令

```shell
#查看hive命令
hive -help

#不进入客户端执行sql语句
hive -e "select id from student;"

#不进入客户端执行sql脚本
echo "select * from student;" >> student.sql
hive -f student.sql
```



### 2.8 Hive的运行日志配置

- Hive的日志默认存放在 /tmp/atguigu/hive.log 下，可以手动修改日志的存储目录

- 修改运行日志存储目录

  1. 复制并重命名配置文件

     ```shell
     cp $HIVE_HOME/conf/hive-log4j2.properties.template $HIVE_HOME/conf/hive-log4j2.properties
     ```

  2. 修改配置文件 hive-log4j2.properties

     ```shell
     vi $HIVE_HOME/conf/hive-log4j2.properties
     ```

     ```shell
     #修改日志存放目录位置项
     property.hive.log.dir = /opt/module/hive-3.1.2/logs
     ```



### 2.9 Hive的参数配置方式

1. 配置文件配置

   ```shell
   cd HIVE_HOME/conf
   hive-default.xml.template #默认配置文件
   hive-site.xml #用户自定义配置文件,覆盖默认配置
   ```

2. 命令行启动配置

   ```shell
   #启动Hive客户端时,可以指定参数,覆盖配置文件中的配置
   #配置参数仅在当次启动的客户端中生效
   hive -hiveconf mapred.reduce.tasks=10;
   ```

3. hive客户端启动后声明

   ```mysql
   #查看配置项配置
   set mapred.reduce.tasks;
   
   #设置配置项的参数,仅在当次启动的客户端中生效
   set mapred.reduce.tasks=100;
   
   #设置日志级别
   set hive.server2.logging.operation.level = NONE;
   ```



## 第三章 Hive 数据类型

### 3.1 基本数据类型

| Hive数据类型 | Java数据类型 | 长度                                               |
| ------------ | ------------ | -------------------------------------------------- |
| TINYINT      | byte         | 1byte有符号整数                                    |
| SMALLINT     | short        | 2byte有符号整数                                    |
| **INT**      | int          | 4byte有符号整数                                    |
| **BIGINT**   | long         | 8byte有符号整数                                    |
| BOOLEAN      | boolean      | 布尔类型，true或者false                            |
| FLOAT        | float        | 单精度浮点数                                       |
| **DOUBLE**   | double       | 双精度浮点数                                       |
| **STRING**   | string       | 字符系列。可以指定字符集。可以使用单引号或双引号。 |
| TIMESTAMP    |              | 时间类型                                           |
| BINARY       |              | 字节数组                                           |

- Hive中的string类型对应数据库的varchar类型，但长度没有限制，因此在使用时无需声明长度。



### 3.2 集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                       |
| -------- | ------------------------------------------------------------ | ---------------------------------------------- |
| STRUCT   | 和c语言中的struct类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, last STRING},那么第1个元素可以通过字段.first来引用。 | struct()例如struct<street:string, city:string> |
| MAP      | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()例如map<string, int>                      |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用。 | Array()例如array\<string>                      |

- 示例

  1. 创建数据文本文件

     ```shell
     mkdir -p /opt/module/datas
     vi /opt/module/datas/test.txt
     
     #添加如下内容
     songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
     yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
     ```

  2. 启动Hive，创建表

     ```mysql
     create table test(
     name string,
     friends array<string>,
     children map<string, int>,
     address struct<street:string, city:string>
     )
     row format delimited fields terminated by ',' #列分隔符
     collection items terminated by '_' #STRUCT MAP ARRAY的元素分隔符
     map keys terminated by ':' #MAP中key和value的分隔符
     lines terminated by '\n'; #行分隔符
     ```

  3. 导入数据文本文件到表

     ```mysql
     load data local inpath '/opt/module/datas/test.txt' into table test;
     ```

  4. 查询表中的数据

     ```mysql
     select* from test
     select friends[1],children['xiao song'],address.city from test
     ```



### 3.3 类型转化

- 隐式转化

  1. 整数类型可以隐式转化为范围更大的整数类型
  2. 整数类型、FLOAT、STRING（符合数据类型格式）可以隐式转化为DOUBLE
  3. TINYINT、SMALLINT、INT可以隐式转化为FLOAT
  4. BOOLEAN不能转化为任何其他类型

- 显式转化：可以使用CAST关键字实现显式的强制转换

  ```mysql
  #举例:将STRING转化为INT
  CAST('1' AS INT)
  ```



## 第四章 DDL数据定义

### 4.1 创建数据库

- 语法

  ```mysql
  CREATE DATABASE [IF NOT EXISTS] database_name #IF NOT EXISTS:若已有表则不创建
  [COMMENT database_comment] #库描述
  [LOCATION hdfs_path] #指定库在HDFS的存储路径,默认为hive-site.xml配置文件中的路径/user/hive/warehouse
  [WITH DBPROPERTIES (property_name=property_value, ...)];
  ```

- 示例

  ```mysql
  #创建数据库db_hive1(避免穿件已存在的数据库),使用默认路径存储
  create database if not exists db_hive1;
  
  #创建数据库db_hive2(避免穿件已存在的数据库),使用指定路径存储
  create database if not exists db_hive2
  location '/db_hive2';
  ```



### 4.2 查询数据库

```mysql
#显示所有数据库
show databases;

#显示数据库信息(存储路径、所有者)
desc database extended database_name;
desc database extended db_hive1;

#切换数据库
use database_name;
use db_hive1;
```



### 4.3 修改数据库配置

```mysql
#修改数据库的创建时间
alter database db_hive1 set dbproperties('createtime'='20170830');
```



### 4.4 删除数据库

```mysql
#删除空数据库;
drop database database_name;
drop database db_hive1;

#删除非空数据库
drop database database_name cascade;
drop database db_hive cascade;
```



### 4.5 创建表

- 语法

  ```mysql
  CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name #管理表/外部表
  [(col_name data_type [COMMENT col_comment], ...)] #声明字段、字段类型、字段描述
  [COMMENT table_comment] #表描述
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] #创建分区表
  [CLUSTERED BY (col_name, col_name, ...) #创建分桶表
  [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] #对桶中列排序
  [ROW FORMAT row_format] #指定表字段的格式
  [STORED AS file_format] #指字段数据的存储格式
  [LOCATION hdfs_path] #hdfs的存储路径,默认路径为所在库路径下,defalut库的表路径为hive存储路径
  [TBLPROPERTIES (property_name=property_value, ...)] #指定表的属性
  [AS select_statement] #从其他表中查询
  ```

#### 4.5.1 创建管理表（内部表）

- 定义：Hive中默认创建的表为管理表，在Hive中删除管理表时，hdfs中存储的表数据也会一并删除。

  ```mysql
  #创建管理表(create后不添加关键字)
  create table if not exists student1(
  id int, name string
  )
  row format delimited fields terminated by '\t';
  
  #使用查询结果创建管理表
  create table if not exists student2 as select id, name from student;
  
  #根据已有表结构创建管理表
  create table if not exists student3 like student;
  
  #查询表的类型(查看Table Type字段)
  desc formatted student2;
  ```

#### 4.5.2 创建外部表

- 定义：Hive创建的外部表在删除时，仅删除元数据，hdfs中存储的数据会保留。

  ```mysql
  #创建外部表
  create external table if not exists dept(
  deptno int,
  dname string,
  loc int
  )
  row format delimited fields terminated by '\t';
  ```

- 外部表和管理表的转换

  ```mysql
  #转换表为外部表
  alter table student set tblproperties('EXTERNAL'='TRUE');
  #转换表为内部表
  alter table student set tblproperties('EXTERNAL'='FALSE');
  ```



### 4.6 修改表

- 重命名表

  ```mysql
  #将table_name1表重命名为table_name2
  ALTER TABLE table_name1 RENAME TO table_name2
  ```

- 修改列

  ```mysql
  #添加列
  ALTER TABLE table_name ADD COLUMNS (col_name data_type [COMMENT col_comment], ...) 
  #示例
  alter table dept add columns(deptdesc string);
  
  #更新列
  ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] 
  #示例
  alter table dept change column deptdesc dept_desc string;
  
  #替换全部列(替换原表中的所有字段)
  ALTER TABLE table_name REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
  #示例
  alter table dept replace columns(deptno string, dname string, loc string);
  
  #注意:更新和替换操作涉及修改数据类型时,需保证修改后的数据类型与修改前一致,或可以从修改前隐式转换
  ```



### 4.7 删除表

```mysql
#删除表
drop table table_name;
drop table dept;
```



## 第五章 DML数据操作

### 5.1 数据导入

#### 5.1.1 load 装载数据

- 语法

  ```mysql
  load data [local] inpath 'path' [overwrite] 
  into table student [partition (partcol1=val1,…)];
  # local 从本地加载数据(复制) / 从hdfs加载数据(剪切)
  # overwrite 导入时覆盖表中已有数据 / 追加已有数据
  ```

- 示例

  ```mysql
  #从本地加载表数据
  load data local inpath '/opt/module/hive/datas/student.txt' into table default.student;
  
  #从hdfs加载表数据
  load data inpath '/user/atguigu/hive/student.txt' into table default.student;
  ```



#### 5.1.2 insert 插入数据

```mysql
#逐条插入数据
insert into table student values(1,'wangwu'),(2,'zhaoliu');

#使用查询结果插入数据
insert into table student select id, name from student;

#先删除表中已有数据(分区表则删除指定分区数据),再使用查询结果插入数据
insert overwrite table student select id, name from student;
```



#### 5.1.3 as select 建表插入数据

```mysql
#创建表时使用查询结果创建并插入数据
create table if not exists student2 as select id, name from student1;
```



#### 5.1.4 location 建表加载数据

```mysql
#创建表时使用hdfs中指定路径,若路径下存有数据文件,则自动加载数据
create table if not exists student1(
    id int, 
    name string
)
row format delimited fields terminated by '\t'
location '/student1';
```



#### 5.1.5 import 导入数据

```mysql
#导入(hdfs中)使用export导出的数据包
import table student2  from '/user/hive/warehouse/export/student';
```



### 5.2 数据导出

#### 5.2.1 insert 导出到本地 / HDFS

```mysql
#将查询结果导出到本地(无格式)
insert overwrite local directory '/opt/module/hive/datas/export/student'
select * from student;

#将查询结果按指定格式格式导出到本地
insert overwrite local directory '/opt/module/hive/datas/export/student'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from student;
#可使用hadoop fs -put命令直接上传带格式的数据文件实现相同效果

#将查询结果按指定格式导出到hdfs
insert overwrite directory '/user/atguigu/student'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from student;
```



#### 5.2.2 hadoop 命令导出到本地

```shell
#直接使用hadoop命令从数据存储目录下载数据到本地(带格式)
hadoop fs -get -p /user/hive/warehouse/student /opt/module/datas/student
```



#### 5.2.3 hive 命令导出到本地

```shell
#使用hive的shell命令直接将查询结果重定向输出到本地(带格式)
hive -e 'select * from default.student;' > /opt/module/hive/datas/export/student.txt;
```



#### 5.2.4 export 导出到HDFS

```mysql
#使用export将表的元数据和数据一起导出到hdfs
export table default.student to '/user/hive/warehouse/export/student';
```



### 5.3 清除数据

```mysql
#清空表中的数据(仅对管理表有效)
truncate table student;
```



## 第六章 查询

### 6.1 基本查询

- 语法

  ```mysql
  SELECT [ALL | DISTINCT] select_expr, select_expr, ...
  FROM table_reference
  [WHERE where_condition]
  [GROUP BY col_list] #分组查询
  [ORDER BY col_list] #全排序
  [CLUSTER BY col_list | [DISTRIBUTE BY col_list] [SORT BY col_list] ] #指定排序方式
  [LIMIT number] #查看指定行
  ```

#### 6.1.1 全表查询和列查询

```mysql
#对emp表全表查询
select * from emp;

#查询emp表中的ename和sal列
select ename, sal from emp;

#查询emp表中的ename和sal列,并在输出结果中使用别名命名列
# as可省略
select ename as name, sal as salary from emp;
```



#### 6.1.2 运算符

- 算数运算符

| 运算符 | 描述           |
| ------ | -------------- |
| A+B    | A加B           |
| A-B    | A减B           |
| A*B    | A乘B           |
| A/B    | A除以B         |
| A%B    | A对B取模       |
| A&B    | A和B按位取与   |
| A\|B   | A和B按位取或   |
| A^B    | A和B按位取异或 |
| ~A     | A按位取反      |

- 比较运算符

| 操作符                  | 支持的数据类型 | 描述                                                         |
| ----------------------- | -------------- | ------------------------------------------------------------ |
| A=B                     | 基本数据类型   | 如果A等于B则返回TRUE，反之返回FALSE                          |
| **A<=>B**               | 基本数据类型   | **如果A和B都为NULL，则返回TRUE，如果一边为NULL，返回False**  |
| A<>B, A!=B              | 基本数据类型   | A或者B为NULL则返回NULL；如果A不等于B，则返回TRUE，反之返回FALSE |
| A<B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于B，则返回TRUE，反之返回FALSE |
| A<=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于等于B，则返回TRUE，反之返回FALSE |
| A>B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于B，则返回TRUE，反之返回FALSE |
| A>=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于等于B，则返回TRUE，反之返回FALSE |
| A [NOT] BETWEEN B AND C | 基本数据类型   | 如果A，B或者C任一为NULL，则结果为NULL。如果A的值大于等于B而且小于或等于C，则结果为TRUE，反之为FALSE。如果使用NOT关键字则可达到相反的效果。 |
| A IS NULL               | 所有数据类型   | 如果A等于NULL，则返回TRUE，反之返回FALSE                     |
| A IS NOT NULL           | 所有数据类型   | 如果A不等于NULL，则返回TRUE，反之返回FALSE                   |
| IN(数值1, 数值2)        | 所有数据类型   | 使用 IN运算显示列表中的值                                    |
| A [NOT] LIKE B          | STRING 类型    | B是一个SQL下的简单正则表达式，也叫通配符模式，如果A与其匹配的话，则返回TRUE；反之返回FALSE。B的表达式说明如下：‘x%’表示A必须以字母‘x’开头，‘%x’表示A必须以字母’x’结尾，而‘%x%’表示A包含有字母’x’,可以位于开头，结尾或者字符串中间。如果使用NOT关键字则可达到相反的效果。 |
| A RLIKE B, A REGEXP B   | STRING 类型    | B是基于java的正则表达式，如果A与其匹配，则返回TRUE；反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串A相匹配，而不是只需与其字符串匹配。 |

- 逻辑运算符

| 操作符 | 含义   |
| ------ | ------ |
| AND    | 逻辑并 |
| OR     | 逻辑或 |
| NOT    | 逻辑否 |



#### 6.1.3 常用函数

```mysql
#求总行数count
select count(*) cnt from emp;

#求最大值max
select max(sal) max_sal from emp;

#求最小值min
select min(sal) min_sal from emp;

#求和sum
select sum(sal) sum_sal from emp;

#求平均值avg
select avg(sal) avg_sal from emp;
```



#### 6.1.3 where 筛选查询

- where + 比较运算符 筛选查询

  ```mysql
  #查询工资大于2000的员工
  select ename, sal from emp
  where sal > 2000;
  
  #多个筛选条件使用逻辑运算符连接
  select ename, sal from emp
  where sal > 2000 and deptno = 10;
  ```

- where + like/rlike 模糊查询

  ```mysql
  #使用like查找相似值
  # % 表示0个或任意个字符
  # _ 表示1个字符
  
  #查找姓名以'S'开头的员工
  select ename, sal from emp
  where ename like 'S%';
  #查找名字第二位字母为'A'的员工
  select ename, sal from emp
  where ename like '_A%';
  ```

  ```mysql
  #使用rlike查找相似值
  # rlike 连接java的正则表达式
  
  #查找姓名中带有'S'的员工
  select ename, sal from emp
  where ename rlike '[S]';
  ```



#### 6.1.4 limit 限制返回行

```mysql
#返回查询结果的前5行内容
select * from emp limit 5;

#返回查询结果的第3行起(表的行数从0计算),共3行内容
select * from emp limit 2,3;
```



### 6.2 分组查询

#### 6.2.1 group by 分组

```mysql
#按照部门分组查询每个部门的平均工资
select deptno, avg(sal) avg_sql from emp
group by deptno;
```

#### 6.2.2 having 筛选分组

```mysql
# having子句可以对group by分组查询的结果进行再次筛选
#查询平均工资高于2000的部门
select deptno, avg(sal) avg_sal from emp
group by deptno
having avg_sal > 2000;
```



### 6.3 join 多表关联

- Hive在多表关联中，仅支持等值连接，不支持非等值连接。

```mysql
#内连接
select e.ename, d.deptno, d.dname
from emp e
join dept d
on e.deptno = d.deptno;

#左外连接
select e.ename, d.deptno, d.dname
from emp e
left join dept d
on e.deptno = d.deptno;

#右外连接
select e.ename, d.deptno, d.dname
from emp e
right join dept d
on e.deptno = d.deptno;

#满外连接(mysql中不支持,hive中支持)
select e.ename, d.deptno, d.dname
from emp e
full join dept d
on e.deptno = d.deptno;
```

- 一般情况下，hive会为每一次join操作启动1个mapreduce程序。当多表关联的多个join连接字段一致时，启动的mapreduce数量由关联字段数量决定，即相同的连接字段只启动1个mapreduce程序。
- 笛卡尔积：多表关联时不提供连接条件或连接条件无效时就会出现笛卡尔积，即A表中的每一条数据都会与B表中的每一条数据组成一行查询结果，要避免。



### 6.4 排序

#### 6.4.1 order by 全排序

```mysql
#只有1个Reducer时使用

#按照工资升序(默认)对查询结果全排序
select * from emp
order by sal;

#按照工资降序对查询结果全排序
select * from emp
order by sal desc;

#先按照部门号升序,再按照工资降序对查询结果排序
select * from emp
order by deptno, sal desc;
```



#### 6.4.2 distribute by 分区排序

```mysql
#设置引擎为mr(使用分区排序需要切换为mr引擎)
set hive.execution.engine=mr;
#设置引擎为tez
set hive.execution.engine=tez;
#设置reduce个数(最终分区个数)
set mapreduce.job.reduces=3;

#随机分区后按sort字段排序
select * from emp sort by deptno desc;
#说明:根据设置的reduce个数对每行随机分区,再对每个分区内的数据按照deptno的降序排序

#按distribute字段分区后按sort字段排序
select * from emp
distribute by deptno
sort by sal desc;
#分区规则:为deptno的hash值对分区数取模得到
```



#### 6.4.3 cluster by 排序

```mysql
#分区字段和排序字段一致时,可使用cluster by实现分区排序
set mapreduce.job.reduces=3;
select * from emp cluster by deptno;

#cluster by只能实现默认的升序排序,不能使用asc或desc关键字
```



## 第七章 分区表和分桶表

- 由于hive本身不支持索引，因此提供了分区表和分桶表方案用于提高查询效率

### 7.1 分区表

- 概念

  - 分区表是在创建表时，额外声明一个分区字段，该字段与其他字段一样作为表的一个列存储
  - 对于分区字段的每一个不同的值，在hdfs中会在表的存储目录下再创建1个分区目录，数据文件存储在分区目录下

- 创建分区表

  ```mysql
  create table dept_partition(
  deptno int, dname string, loc string
  )
  partitioned by (day string)
  row format delimited fields terminated by '\t';
  ```

- 添加数据到分区表

  ```mysql
  #方式1:从本地加载数据文件(不含分区字段)到指定分区表的指定分区
  load data local inpath '/opt/module/datas/dept_20200402.log' into table dept_partition partition(day='20200402');
  
  #方式2:使用hadoop命令上传数据文件到指定分区目录后修复分区
  hadoop fs -mkdir -p /user/hive/warehouse/hive1/dept_partition/day=20200401;
  hadoop fs /opt/module/datas/dept_20200401.log  /user/hive/warehouse/hive1/dept_partition/day=20200401;
  #对分区表执行修复命令
  msck repair table dept_partition;
  
  #方式3:上传数据后添加分区
  hadoop fs -mkdir -p /user/hive/warehouse/hive1/dept_partition/day=20200401;
  hadoop fs /opt/module/datas/dept_20200401.log  /user/hive/warehouse/hive1/dept_partition/day=20200401;
  #添加分区
  alter table dept_partition add partition(day='20200401');
  ```

- 查询分区表中的数据

  ```mysql
  #和普通表的查询完全相同
  select * from dept_partition where day='20200401';
  select * from dept_partition where day between '20200401' and '20200402';
  ```

- 添加、删除分区表的分区

  ```mysql
  #添加分区(添加多个分区使用' '连接)
  alter table dept_partition add partition(day='20200404');
  alter table dept_partition add partition(day='20200405') partition(day='20200406');
  
  #删除分区(删除多个分区使用','连接)
  alter table dept_partition drop partition (day='20200406');
  alter table dept_partition drop partition (day='20200404'), partition(day='20200405');
  alter table dept_partition drop partition (day>'20200404',day<'20200505');
  ```

- 查看分区表的全部分区

  ```mysql
  show partitions dept_partition;
  ```

- 二级分区表

  ```mysql
  #创建2级分区表
  create table dept_partition2(
  deptno int, dname string, loc string
  )
  partitioned by (day string, hour string) #先按day分区,再按hour分区,hdfs中生成相应的多级目录
  row format delimited fields terminated by '\t';
  
  #添加数据
  load data local inpath '/opt/module/datas/dept_20200401.log' 
  into table dept_partition2 partition(day='20200401', hour='12');
  ```



- **实现动态分区（重要）**

  ```mysql
  #配置相关参数
  #开启动态分区(默认为true)
  set hive.exec.dynamic.partition=true
  #设置非严格模式(默认为strict)
  set hive.exec.dynamic.partition.mode=nonstrict
  #设置所有mr节点上的最大动态分区数(默认1000)
  set hive.exec.max.dynamic.partitions=100000
  #设置每个mr节点上的最大动态分区数(默认100)
  hive.exec.max.dynamic.partitions.pernode=100000
  #设置可以创建的最大hdfs文件数(默认100000)
  hive.exec.max.created.files=100000
  #设置空分区生成时不抛出异常(默认false)
  hive.error.on.empty.partition=false
  ```

  ```mysql
  #配置完成参数后,再向分区表中添加数据时,若使用动态分区,系统会自动按照最后1个字段的值将数据插入对应分区,且该字段的值不会插入表中
  insert into table dept_partition_dy partition(location)
  select deptno, dname, loc from dept;
  ```

  

### 7.2 分桶表

- 概念

  - 和分区表一样，分桶表同样实现了对数据的整理划分，提高查询效率
  - 和分区表不同的是，分区是对表中数据在hdfs上的存储路径按字段值进行了划分，而分桶表是对每一个具体的数据文件按照字段值划分为多个数据文件存储。

- 创建分桶表

  ```mysql
  create table stu_buck(id int, name string)
  clustered by(id) #使用已声明的字段分桶
  into 4 buckets #配置分桶数
  row format delimited fields terminated by '\t';
  ```

- 添加数据到分桶表

  ```mysql
  #导入数据文件
  load data local inpath '/opt/module/hive/datas/student.txt' into table stu_buck;
  
  #分桶规则:根据分桶字段值的hash值对分桶数取模后按结果分桶
  ```



## 第八章 函数

### 8.1 系统内置函数

```mysql
#查看系统内置函数
show functions;

#显示指定函数的用法
desc function function_name;

#显示指定函数的详细用法
desc function extended function_name;
```



#### 8.1.1 nvl 空值处理

```mysql
#说明:当指定字段的值为null时,使用返回指定值
#查询员工的姓名和奖金,若奖金为null则返回0
select ename, nvl(comm, 0) from emp;
```



#### 8.1.2 case when 条件判断

- 作用：根据字段的不同值，使用不同的方式处理数据

- 示例

  ```mysql
  #根据以下数据统计不同部门的男女人数
  name	dept_id	sex
  悟空	A	男
  大海	A	男
  宋宋	B	男
  凤姐	A	女
  慧慧	B	女
  婷婷	B	女
  
  #查询语句
  select dept_id,
  sum(case when sex='男' then 1 when sex='女' then 0 end) sum_man,
  sum(case when sex='女' then 1 when sex='男' then 0 end) sum_woman
  from emp_sex
  group by dept_id;
  
  #查询结果如下
  +----------+----------+------------+
  | dept_id  | sum_man  | sum_woman  |
  +----------+----------+------------+
  | A        | 2        | 1          |
  | B        | 1        | 2          |
  +----------+----------+------------+
  ```

  

#### 8.1.3 concat collect 行转列

- 相关函数

  ```mysql
  #拼接任意多个字符串或1个集合,返回1个字符串
  concat(string A, string B, ...)
  
  #使用指定连接符拼接任意多个字符串或1个集合,返回1个字符串
  concat_ws(delimiter, string A, string B, ...)
  concat_ws(delimiter, array A)
  
  #配合group by使用,将某一字段的值去重汇总,返回1个数组(array)
  collect_set(col)
  ```

- 示例

  ```mysql
  #处理如下数据
  name	constellation	blood_type
  悟空	白羊座	A
  大海	射手座	A
  宋宋	白羊座	B
  八戒	白羊座	A
  凤姐	射手座	A
  苍苍	白羊座	B
  #返回如下结果
  射手座,A	大海|凤姐
  白羊座,A	悟空|八戒
  白羊座,B	宋宋|苍苍
  
  #查询语句如下
  select t.info, concat_ws('|',collect_set(t.name))
  from (
   select name, concat_ws(',', constellation, blood_type) info
   from person_info
  ) t
  group by t.info;
  ```



#### 8.1.4 split explode 列转行

- 相关函数

  ```mysql
  #将字符串按照指定分隔符切割,返回1个数组(array)
  split(string A, delimiter)
  
  #对于数组型字段,拆分每一个数组元素,返回多行
  explode(col)
  
  #侧视图:与UDTF函数配合使用,效果相当于生成1个虚拟表,在原表基础上,增加UDTF函数返回的多列多行数据
  lateral view explode(col) table_name as column_name
  ```

- 示例

  ```mysql
  #处理以下数据
  movie	category
  《疑犯追踪》	悬疑,动作,科幻,剧情
  《Lie to me》	悬疑,警匪,动作,心理,剧情
  《战狼2》	战争,动作,灾难
  #返回如下格式的结果
  《疑犯追踪》	悬疑
  《疑犯追踪》	动作
  《疑犯追踪》	科幻
  《疑犯追踪》	剧情
  《Lie to me》	悬疑
  《Lie to me》	警匪
  《Lie to me》	动作
  《Lie to me》	心理
  《Lie to me》	剧情
  《战狼2》	战争
  《战狼2》	动作
  《战狼2》	灾难
  
  #查询语句如下
  select info.movie, tmp.category_info
  from movie_info as info
  lateral view explode(split(category, ',')) tmp as category_info;
  ```



#### 8.1.5 窗口函数

- 需求：当我们需要对局部的查询结果使用聚合函数时，会使用group by对结果集进行分组再计算，但是使用分组查询时，select的字段必须是分组条件或聚合函数，当我们想要在结果中呈现其他字段时非常不便，因此hive引入了窗口函数。

- 结果集与窗口

  - 结果集：默认情况下，使用查询语句会返回1个包含1或多行查询结果的结果集，当查询字段中包含聚合函数（求和、求平均值等）时，默认对当前结果集整体计算。
  - 窗口：默认情况下，窗口大小就是结果集的大小，即1个结果集对应1个窗口。窗口函数可以改变窗口的大小，此时聚合函数不再以结果集为准，而是以窗口为准。

- **本质：窗口函数的实现原理实际还是group by分组查询，在分组查询后将查询结果与需求字段所在表使用分组条件进行关联，达到select分组条件之外字段的效果。**

- 窗口函数

  ```mysql
  #声明方式:在聚合函数后使用,指定聚合函数的作用窗口
  over()
  
  #over()中的参数
  current row -- 当前行
  n preceding -- 窗口内向前n行
  n following -- 窗口内向后n行
  unbounded preceding -- 窗口第一行
  unbounded following -- 窗口最后一行
  
  #相关函数
  lag(col, n, default_val) -- 返回窗口内向前第n行,col列的数据,若为空返回默认值
  lead(col, n, default_val) -- 返回窗口内向后第n行,col列的数据,若为空返回默认值
  ntile(n) -- 将窗口内的行分为n组,返回组号
  ```

- 示例

  ```mysql
  #处理如下数据
  name orderdate cost
  jack,2017-01-01,10
  tony,2017-01-02,15
  jack,2017-02-03,23
  tony,2017-01-04,29
  jack,2017-01-05,46
  jack,2017-04-06,42
  tony,2017-01-07,50
  jack,2017-01-08,55
  mart,2017-04-08,62
  mart,2017-04-09,68
  neil,2017-05-10,12
  mart,2017-04-11,75
  neil,2017-06-12,80
  mart,2017-04-13,94
  
  #需求1:查询2017年4月购买过的顾客及总人数
  select name, count(*) over() sum_people
  from business
  where month(orderdate)='04'
  group by name;
  #说明:按照日期筛选再根据姓名分组后查询结果只有2行记录,这时窗口函数的作用对象就是这2条结果
  #查询结果
  +-------+-------------+
  | name  | sum_people  |
  +-------+-------------+
  | jack  | 2           |
  | mart  | 2           |
  +-------+-------------+
  
  #需求2:查询每个顾客的购买明细及他们的各自的月购买总额
  select name, orderdate, cost,
  sum(cost) over(partition by name, month(orderdate)) sum_cost
  from business;
  #说明:窗口函数中可使用partition by划分结果集,此处先后以姓名、月份划分结果集后,使用sum对每个结果集内的cost求和
  #查询结果
  +-------+-------------+-------+-----------+
  | name  |  orderdate  | cost  | sum_cost  |
  +-------+-------------+-------+-----------+
  | jack  | 2017-01-01  | 10    | 111       |
  | jack  | 2017-01-05  | 46    | 111       |
  | jack  | 2017-01-08  | 55    | 111       |
  | jack  | 2017-02-03  | 23    | 23        |
  | jack  | 2017-04-06  | 42    | 42        |
  | mart  | 2017-04-13  | 94    | 299       |
  | mart  | 2017-04-08  | 62    | 299       |
  | mart  | 2017-04-09  | 68    | 299       |
  | mart  | 2017-04-11  | 75    | 299       |
  | neil  | 2017-05-10  | 12    | 12        |
  | neil  | 2017-06-12  | 80    | 80        |
  | tony  | 2017-01-04  | 29    | 94        |
  | tony  | 2017-01-07  | 50    | 94        |
  | tony  | 2017-01-02  | 15    | 94        |
  +-------+-------------+-------+-----------+
  
  #需求3:查询每个顾客的购买明细和按日期累加的总消费
  select name, orderdate, cost,
  sum(cost) over(partition by name order by orderdate) sum_cost
  from business;
  #说明:窗口函数中使用order by对每个窗口内按指定字段进行排序,同时对于每一行来说,默认窗口为当前结果集的首行到当前行的全部数据
  -- order by还可以和rows between共同使用用于指定窗口包括的具体行
  -- sum(cost) over(partition by name order by orderdate rows between 1 preceding and 1 following)
  -- 此处窗口范围为前一行、当前行、后一行共3行
  #查询结果
  +-------+-------------+-------+-----------+
  | name  |  orderdate  | cost  | sum_cost  |
  +-------+-------------+-------+-----------+
  | jack  | 2017-01-01  | 10    | 10        |
  | jack  | 2017-01-05  | 46    | 56        |
  | jack  | 2017-01-08  | 55    | 111       |
  | jack  | 2017-02-03  | 23    | 134       |
  | jack  | 2017-04-06  | 42    | 176       |
  | mart  | 2017-04-08  | 62    | 62        |
  | mart  | 2017-04-09  | 68    | 130       |
  | mart  | 2017-04-11  | 75    | 205       |
  | mart  | 2017-04-13  | 94    | 299       |
  | neil  | 2017-05-10  | 12    | 12        |
  | neil  | 2017-06-12  | 80    | 92        |
  | tony  | 2017-01-02  | 15    | 15        |
  | tony  | 2017-01-04  | 29    | 44        |
  | tony  | 2017-01-07  | 50    | 94        |
  +-------+-------------+-------+-----------+
  
  #需求4:查看顾客上次购买的时间
  select name, orderdate, cost,
  lag(orderdate, 1, null) over(partition by name order by orderdate) last_date
  from business;
  #查询结果
  +-------+-------------+-------+-------------+
  | name  |  orderdate  | cost  |  last_date  |
  +-------+-------------+-------+-------------+
  | jack  | 2017-01-01  | 10    | NULL        |
  | jack  | 2017-01-05  | 46    | 2017-01-01  |
  | jack  | 2017-01-08  | 55    | 2017-01-05  |
  | jack  | 2017-02-03  | 23    | 2017-01-08  |
  | jack  | 2017-04-06  | 42    | 2017-02-03  |
  | mart  | 2017-04-08  | 62    | NULL        |
  | mart  | 2017-04-09  | 68    | 2017-04-08  |
  | mart  | 2017-04-11  | 75    | 2017-04-09  |
  | mart  | 2017-04-13  | 94    | 2017-04-11  |
  | neil  | 2017-05-10  | 12    | NULL        |
  | neil  | 2017-06-12  | 80    | 2017-05-10  |
  | tony  | 2017-01-02  | 15    | NULL        |
  | tony  | 2017-01-04  | 29    | 2017-01-02  |
  | tony  | 2017-01-07  | 50    | 2017-01-04  |
  +-------+-------------+-------+-------------+
  
  #需求5:查看日期前20%的订单信息
  select t.name, t.orderdate, t.cost
  from (
    select name, orderdate, cost, ntile(5) over(order by orderdate) group_id
    from business
    ) t
  where group_id = 1;
  #说明:将表中的数据按日期分为5组(每组20%),返回第1组数据
  #查询结果
  +---------+--------------+---------+
  | t.name  | t.orderdate  | t.cost  |
  +---------+--------------+---------+
  | jack    | 2017-01-01   | 10      |
  | tony    | 2017-01-02   | 15      |
  | tony    | 2017-01-04   | 29      |
  +---------+--------------+---------+
  ```



#### 8.1.6 Rank

- 相关函数

  ```mysql
  #返回排名,字段值相等时按2个值处理
  rank()
  #返回排名,字段值相等时按1个值处理
  dense_rank()
  #返回行号
  row_number()
  ```

- 示例

  ```mysql
  #处理如下数据,实现学科排名
  name	subject	score
  悟空	语文	87
  悟空	数学	95
  悟空	英语	68
  大海	语文	94
  大海	数学	56
  大海	英语	84
  宋宋	语文	64
  宋宋	数学	86
  宋宋	英语	84
  婷婷	语文	65
  婷婷	数学	85
  婷婷	英语	78
  
  #查询语句
  select name, subject, score,
  rank() over(partition by subject order by score desc) rank,
  dense_rank() over(partition by subject order by score desc) dense_rank,
  row_number() over(partition by subject order by score desc) row_number
  from score;
  #注意:当一次查询出现多个窗口函数时,要确保多个窗口函数的分区和排序规则一致,否则结果会按照最后一个声明的规则执行,前面的结果会不符合预期
  ```



#### 8.1.7 其他常用函数

```mysql
#常用日期函数
unix_timestamp -- 返回当前或指定时间的时间戳	
from_unixtime -- 将时间戳转为日期格式
current_date -- 当前日期
current_timestamp -- 当前的日期加时间
to_date -- 抽取日期部分
year -- 获取年
month -- 获取月
day -- 获取日
hour -- 获取时
minute -- 获取分
second -- 获取秒
weekofyear -- 当前时间是一年中的第几周
dayofmonth -- 当前时间是一个月中的第几天
months_between -- 两个日期间的月份
add_months -- 日期加减月
datediff -- 两个日期相差的天数
date_add -- 日期加天数
date_sub -- 日期减天数
last_day -- 日期的当月的最后一天

#常用取整函数
round -- 四舍五入
ceil -- 向上取整
floor -- 向下取整

#常用字符串操作函数
upper -- 转大写
lower -- 转小写
length -- 长度
trim -- 前后去空格
lpad -- 向左补齐,到指定长度
rpad -- 向右补齐,到指定长度
regexp_replace -- SELECT regexp_replace('100-200', '(\\d+)', 'num')  使用正则表达式匹配目标字符串,匹配成功后替换！

#集合操作
size -- 集合中元素的个数
map_keys -- 返回map中的key
map_values -- 返回map中的value
array_contains -- 判断array中是否包含某个元素
sort_array -- 将array中的元素排序
```



### 8.2 自定义函数

- 自定义函数分类
  - UDF（一进一出）
  - UDAF（聚合函数，多进一出）
  - UDTF（一进多出）

#### 8.2.1 自定义UDF函数

- 示例：自定义函数计算给定字符串的长度

  1. 启动idea，创建maven工程

  2. pom.xml中导入依赖

     ```xml
     <dependencies>
         <dependency>
             <groupId>org.apache.hive</groupId>
             <artifactId>hive-exec</artifactId>
             <version>3.1.2</version>
         </dependency>
     </dependencies>
     ```

  3. 创建GenericUDF类继承类

     ```java
     /**
      * 继承 GenericUDF 类
      * 重写 initialize() evaluate() getDisplayString()三个方法
      */
     public class UDFTest extends GenericUDF {
     
         /**
          * 对输入参数的个数及类型做判断
          * @param objectInspectors 输入参数类型的鉴别器对象
          * @return 返回值类型的鉴别器对象
          * @throws UDFArgumentException
          */
         @Override
         public ObjectInspector initialize(ObjectInspector[] objectInspectors) throws UDFArgumentException {
             //1.判断输入参数的个数正确,此处需求为输入1个字符串,因此个数为1
             if (objectInspectors.length != 1){
                 throw new UDFArgumentLengthException("输入参数个数错误");
             }
             //2.判断输入参数的类型(primitive/struct/map/array),此处输入参数只有1个,因此判断数组第0个元素是否为基本数据类型
             if (!objectInspectors[0].getCategory().equals(ObjectInspector.Category.PRIMITIVE)){
                 //类型异常的第一个参数为输入参数数组的脚标,用于在异常中指出具体的参数
                 throw new UDFArgumentTypeException(0, "参数类型错误");
             }
             //3.返回函数返回值对应类型的鉴别器对象,此处返回值为字符串的长度,因此返回int型鉴别器对象
             return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
         }
     
         /**
          * 函数的具体逻辑实现
          * @param deferredObjects 输入参数组成的数组
          * @return
          * @throws HiveException
          */
         @Override
         public Object evaluate(DeferredObject[] deferredObjects) throws HiveException {
             //1.判断参数是否为空
             if (deferredObjects[0].get() == null){
                 return 0;
             }
             //2.不为空则返回字符串长度
             return deferredObjects[0].get().toString().length();
         }
     
         @Override
         public String getDisplayString(String[] strings) {
             return "";
         }
     }
     ```

  4. 打包上传到linux

  5. 启动hive，加载jar包

     ```mysql
     add jar /opt/module/datas/hivedemo.jar;
     ```

  6. 创建临时函数

     ```mysql
     create temporary function my_len as "com.atguigu.hive.UDFTest";
     ```

  7. 调用函数

     ```
     select my_len('helloworld');
     ```



#### 8.2.2 自定义UDTF函数

- 示例：按照指定分隔符切割字符串，返回多行数据

  1. 启动idea，创建maven工程

  2. pom.xml中导入依赖

     ```xml
     <dependencies>
         <dependency>
             <groupId>org.apache.hive</groupId>
             <artifactId>hive-exec</artifactId>
             <version>3.1.2</version>
         </dependency>
     </dependencies>
     ```

  3. 创建GenericUDTF类继承类

     ```java
     /**
      * 继承GenericUDTF类
      * 重写 initialize() process() close() 3个方法
      */
     public class UDTFTest extends GenericUDTF {
     
         private List<String> out = new ArrayList<String>();
     
         /**
          * 定义输出的列名和列的类型
          * @param argOIs
          * @return
          * @throws UDFArgumentException
          */
         @Override
         public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {
             //1.创建集合对象记录输出的列名,此处仅输出1个列
             ArrayList<String> fieldName = new ArrayList<String>();
             fieldName.add("word");
             //2.创建集合对象记录输出的列的类型
             ArrayList<ObjectInspector> fieldOIs = new ArrayList<ObjectInspector>();
             fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
             //3.返回列名和列类型的集合对象
             return ObjectInspectorFactory.getStandardStructObjectInspector(fieldName,fieldOIs);
         }
     
         /**
          * 函数的具体逻辑实现
          * @param objects 函数的输入参数组成的数组
          * @throws HiveException
          */
         @Override
         public void process(Object[] objects) throws HiveException {
             //1.判断输入参数个数
             if (objects.length != 2){
                 throw new HiveException("输入参数个数错误");
             }
             //2.处理输入参数,获得切割后数据的数组对象
             String data = objects[0].toString();
             String delimiter = objects[1].toString();
             String[] words = data.split(delimiter);
             //3.迭代写出
             for (String word : words) {
                 out.clear();
                 out.add(word);
                 forward(out);
             }
         }
     
         @Override
         public void close() throws HiveException {
         }
     }
     ```

  4. 打包上传到linux

  5. 启动hive，加载jar包

     ```mysql
     add jar /opt/module/datas/hivedemo.jar;
     ```

  6. 创建临时函数

     ```mysql
     create temporary function my_len as "com.atguigu.hive.UDTFTest";
     ```

  7. 调用函数

     ```mysql
     select my_split('a,b,c,d,e',',');
     ```



## 第九章 压缩和存储

### 9.1 hadoop压缩配置

- mr支持的压缩格式

| 压缩格式 | 工具  | 算法    | 文件扩展名 | 是否可切分 |
| -------- | ----- | ------- | ---------- | ---------- |
| DEFLATE  | 无    | DEFLATE | .deflate   | 否         |
| Gzip     | gzip  | DEFLATE | .gz        | 否         |
| bzip2    | bzip2 | bzip2   | .bz2       | 是         |
| LZO      | lzop  | LZO     | .lzo       | 是         |
| Snappy   | 无    | Snappy  | .snappy    | 否         |

- 压缩格式的编解码器

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

- 压缩参数配置，修改mapred-site.xml

| 参数                                             | 默认值                                                       | 阶段        | 建议                                         |
| ------------------------------------------------ | ------------------------------------------------------------ | ----------- | -------------------------------------------- |
| io.compression.codecs  （在core-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec, org.apache.hadoop.io.compress.Lz4Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器 |
| mapreduce.map.output.compress                    | false                                                        | mapper输出  | 这个参数设为true启用压缩                     |
| mapreduce.map.output.compress.codec              | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress       | false                                                        | reducer输出 | 这个参数设为true启用压缩                     |
| mapreduce.output.fileoutputformat.compress.codec | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2      |
| mapreduce.output.fileoutputformat.compress.type  | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK  |



### 9.2  hive中开启MapReduce压缩

- 开启Map输出数据的压缩

  ```mysql
  #开启hive中间传输数据的压缩功能
  set hive.exec.compress.intermediate=true;
  
  #开启mapreduce中map输出数据的压缩功能
  set mapreduce.map.output.compress=true;
  
  #设置mapreduce中map输出数据的压缩格式
  set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
  ```

- 开启Reduce输出数据的压缩

  ```mysql
  #开启hive最终输出数据的压缩功能
  set hive.exec.compress.output=true;
  
  #开启mapreduce最终输出数据的压缩功能
  set mapreduce.output.fileoutputformat.compress=true;
  
  #设置mapreduce最终输出数据的压缩格式
  set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
  
  #设置mapreduce最终输出数据压缩为块压缩
  set mapreduce.output.fileoutputformat.compress.type=BLOCK;
  ```



### 9.3 文件存储格式

#### 9.3.1 行式存储与列式存储

![image-20200428225556862](Hive.assets/image-20200428225556862.png)

- 行存储（TextFile，SequenceFile）：多字段查询效率高
- 列存储（ORC，Parquet）：单字段计算效率高

#### 9.3.2 存储格式

- TextFile格式
  - 默认的存储格式，基于行存储，数据不做压缩，磁盘占用空间大，数据解析资源需求高。
- ORC格式
  - 基于列存储，将数据切分为多个stripe存储，数据文件内维护了多级索引。
- Parquet格式
  - 基于列存储，使用二进制方式存储数据，包含元数据，因此支持自解析。



#### 9.3.3 建表方式

- TextFile

  ```mysql
  #建表语句
  create table log_text (
  track_time string,
  url string,
  session_id string,
  referer string,
  ip string,
  end_user_id string,
  city_id string
  )
  row format delimited fields terminated by '\t'
  stored as textfile; -- 默认的存储格式即为textfile,此行可省略
  ```

- ORC

  ```mysql
  #建表语句
  create table log_orc(
  track_time string,
  url string,
  session_id string,
  referer string,
  ip string,
  end_user_id string,
  city_id string
  )
  row format delimited fields terminated by '\t'
  stored as orc -- 设置存储格式为ORC
  tblproperties("orc.compress"="NONE"); -- 设置ORC存储不使用压缩
  
  #不可以使用load直接导入数据(无法解析),只能从其他表导入数据
  insert into table log_orc
  select * from log_text;
  ```

- Parquet

  ```mysql
  #建表语句
  create table log_parquet(
  track_time string,
  url string,
  session_id string,
  referer string,
  ip string,
  end_user_id string,
  city_id string
  )
  row format delimited fields terminated by '\t'
  stored as parquet
  tblproperties("parquet.compress"="NONE");
  
  #不可以使用load直接导入数据(无法解析),只能从其他表导入数据
  insert into table log_parquet
  select * from log_text;
  ```

- 对比

  - 空间占用：TextFile > Parquet > ORC
  - 查询效率：三者速度相近



### 9.4 存储和压缩结合

- 使用列存储和压缩结合方式建表

- 使用SNAPPY压缩存储ORC存储格式的表

  ```mysql
  #建表语句
  create table log_orc_snappy(
  track_time string,
  url string,
  session_id string,
  referer string,
  ip string,
  end_user_id string,
  city_id string
  )
  row format delimited fields terminated by '\t'
  stored as orc -- 设置表的存储方式为orc
  tblproperties("orc.compress"="SNAPPY"); -- 设置数据的压缩格式为snappy
  
  #使用insert从其他表添加数据
  insert into log_orc_snappy select * from log_text;
  ```

- 使用SNAPPY压缩存储Parquet存储格式的表

  ```mysql
  #建表语句
  create table log_parquet_snappy(
  track_time string,
  url string,
  session_id string,
  referer string,
  ip string,
  end_user_id string,
  city_id string
  )
  row format delimited fields terminated by '\t'
  stored as parquet -- 设置表的存储方式为parquet
  tblproperties("parquet.compression"="SNAPPY");  -- 设置数据的压缩方式为snappy
  
  #使用insert从其他表添加数据
  insert into log_parquet_snappy select * from log_text;
  ```



## 第十章 企业级调优

### 10.1 Fetch 抓取

- 概念：Fetch抓取通过设置，使指某些简单的查询语句不需要经过MapReduce计算，直接读取数据文件并输出查询结果。

- 设置Fetch级别，修改hive-site配置文件，添加以下配置项

  ```xml
  <property>
      <name>hive.fetch.task.conversion</name>
      <value>more</value>
  </property>
  <!--
  说明:Fetch级别包括
  none 不设置Fetch抓取,所有查询均经过mr计算输出
  minimal select *, 通过partition列过滤, limit使用Fetch抓取
  more select任意字段, 通过所有列过滤, limit使用Fetch抓取
  -->
  ```



### 10.2 严格模式设置

- 可以通过设置严格模式的相关参数来预防一些错误或占用极大资源的操作

```mysql
hive.strict.checks.no.partition.filter=false
#设置为true:查询分区表时,where子句中必须包含分区字段作为条件

hive.strict.checks.orderby.no.limit=false
#设置为true:使用order by排序后,必须使用limit子句限制输出行数

hive.strict.checks.cartesian.product=false
#设置为true:自动禁止可能出现笛卡尔积的查询,即多表关联查询时必须指明连接条件
```



### 10.3 小文件合并

1. 执行map前合并小文件以减少maptask个数，修改输入流类型

   ```mysql
   set hive.input.format= org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
   ```

2. map-only型任务结束时合并小文件，默认true

   ```mysql
   set hive.merge.mapfiles = true;
   ```

3. map-reduce任务结束时合并小文件，默认false

   ```mysql
   set hive.merge.mapredfiles = true;
   ```

4. 合并后文件的大小，默认256M

   ```mysql
   set hive.merge.size.per.task = 268435456;
   ```

5. 当输出文件的平均大小设置值时，启动一个单独的mr任务进行文件合并

   ```shell
   set hive.merge.smallfiles.avgsize = 16777216;
   ```



## 第十一章 Hive  实战

### 11.1 需求描述

- 根据某视频网站的数据信息，分析多项Top指标
- 提供两个数据文件，分别记录 video 和 user 的数据信息，表中字段信息如下
- video表

| 字段      | 备注                         | 详细描述               |
| --------- | ---------------------------- | ---------------------- |
| videoId   | 视频唯一id（String）         | 11位字符串             |
| uploader  | 视频上传者（String）         | 上传视频的用户名String |
| age       | 视频年龄（int）              | 视频在平台上的整数天   |
| category  | 视频类别（Array\<String>）   | 上传视频指定的视频分类 |
| length    | 视频长度（Int）              | 整形数字标识的视频长度 |
| views     | 观看次数（Int）              | 视频被浏览的次数       |
| rate      | 视频评分（Double）           | 满分5分                |
| Ratings   | 流量（Int）                  | 视频的流量，整型数字   |
| conments  | 评论数（Int）                | 一个视频的整数评论数   |
| relatedId | 相关视频id（Array\<String>） | 相关视频的id，最多20个 |

- user表

| **字段** | 备注         | 字段类型 |
| -------- | ------------ | -------- |
| uploader | 上传者用户名 | string   |
| videos   | 上传视频数   | int      |
| friends  | 朋友数量     | int      |



### 11.2 数据准备

#### 11.2.1 数据文件分析

- user表的数据格式规整，三字段均使用"\t"分隔，可直接读取

- video表的数据存储格式无法直接通过建表语句解析，需要数据清洗

  ```mysql
  #video数据文件中的摘取一行数据格式如下
  RX24KLBhwMI	lemonette	697	People & Blogs	512	24149	4.22	315	474	t60tW0WevkE	WZgoejVDZlo	Xa_op4MhSkg	MwynZ8qTwXA	sfG2rtAkAcg	j72VLPwzd_c	24Qfs69Al3U	EGWutOjVx4M	KVkseZR5coU	R6OaRcsfnY4	dGM3k_4cNhE	ai-cSq6APLQ	73M0y-iD9WE	3uKOSjE79YA	9BBu5N0iFBg	7f9zwx52xgA	ncEV0tSC7xM	H-J8Kbx9o68	s8xf4QX1UvA	2cKd9ERh5-8
  #清洗需求如下
  #1.目标字段至少为9,数据文件中字段总数不足9的行需要舍弃
  #2.第4字段为Array型数组字段category,数组中多个元素使用" & "切割,hive中仅能解析单字符的分割符,需要替换
  #3.第10字段为Array型数组字段relatedId,数组中多个元素使用"\t"分割,与列分隔符冲突,需要替换
  ```

  

#### 11.2.2 ETL 数据清洗

1. 启动idea，新建Maven工程

2. 修改pom.xml文件，导入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.hive</groupId>
           <artifactId>hive-exec</artifactId>
           <version>3.1.2</version>
       </dependency>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.12</version>
       </dependency>
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-slf4j-impl</artifactId>
           <version>2.12.0</version>
       </dependency>
       <dependency>
           <groupId>org.apache.hadoop</groupId>
           <artifactId>hadoop-client-api</artifactId>
           <version>3.1.3</version>
       </dependency>
       <dependency>
           <groupId>org.apache.hadoop</groupId>
           <artifactId>hadoop-client-runtime</artifactId>
           <version>3.1.3</version>
       </dependency>
   </dependencies>
   ```

3. 编写Mapper继承类，实现数据清洗

   ```java
   public class ETLMapper extends Mapper<LongWritable, Text, Text, NullWritable> {
   
       private Text k = new Text();
   
       @Override
       protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
   
           //1.切割输入数据
           String line = value.toString();
           String[] splits = line.split("\t");
   
           //2.剔除字段过少的行
           if (splits.length < 9){
               return;
           }
   
           //3.替换category列中元素的分隔符为&
           splits[3] = splits[3].replaceAll(" ", "");
   
           //4.遍历数组,替换relatedId列的分隔符
           StringBuffer outline = new StringBuffer();
           for (int i = 0; i < splits.length; i++) {
               if (i < 9){
                   //前9列使用\t分隔写出
                   if (i < splits.length - 1){
                       outline.append(splits[i]).append("\t");
                   }else{
                       outline.append(splits[i]);
                   }
               } else {
                   //替换relatedId列的分隔符为&,与category列一致
                   if (i < splits.length - 1){
                       outline.append(splits[i]).append("&");
                   }else{
                       outline.append(splits[i]);
                   }
               }
           }
   
           //5.写出清洗后的行数据
           k.set(outline.toString());
           context.write(k, NullWritable.get());
   
       }
   }
   ```

4. 编写Driver类

   ```java
   public class ETLDriver {
   
       public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
   
           Job job = Job.getInstance(new Configuration());
   
           job.setJarByClass(ETLDriver.class);
           job.setMapperClass(ETLMapper.class);
   
           job.setOutputKeyClass(Text.class);
           job.setOutputValueClass(NullWritable.class);
   
           FileInputFormat.setInputPaths(job, new Path(args[0]));
           FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
           //设置不经过Reduce阶段,直接由Map输出结果
           job.setNumReduceTasks(0);
   
           job.waitForCompletion(true);
   
       }
   }
   ```

5. 打包并上传至linux

6. 上传数据文件到hadoop

   ```shell
   hadoop fs -p mkdir /gulivideo/user
   hadoop fs -p mkdir /gulivideo/video
   hadoop fs -put user/* /gulivideo/user
   hadoop fs -put video/* /gulivideo/video
   ```

7. 执行jar包清洗数据

   ```shell
   hadoop jar etl.jar com.atguigu.etl.ETLDriver /gulivideo/video /gulivideo/output
   ```



#### 11.2.3 建表并加载数据

1. 准备video数据的原始表和orc表

   ```mysql
   #创建表gulivideo
   create table gulivideo(
       videoId string, 
       uploader string, 
       age int, 
       category array<string>, 
       length int, 
       views int, 
       rate float, 
       ratings int, 
       comments int,
       relatedId array<string>)
   row format delimited fields terminated by "\t"
   collection items terminated by "&"
   stored as textfile;
   
   #向表gulivideo中插入数据
   load data inpath '/gulivideo/output' into table gulivideo;
   
   #创建表gulivideo_orc
   create table gulivideo_orc(
       videoId string, 
       uploader string, 
       age int, 
       category array<string>, 
       length int, 
       views int, 
       rate float, 
       ratings int, 
       comments int,
       relatedId array<string>)
   stored as orc
   tblproperties("orc.compress"="SNAPPY");
   
   #向表gulivideo_orc中插入数据
   insert into gulivideo_orc select * from gulivideo;
   ```

2. 准备video数据的原始表和orc表

   ```mysql
   #创建表guliuser
   create table guliuser(
       uploader string,
       videos int,
       friends int)
   row format delimited 
   fields terminated by "\t" 
   stored as textfile;
   
   #向表gulivideo中插入数据
   load data inpath '/gulivideo/user' into table guliuser;
   
   #创建表guliuser_orc
   create table guliuser_orc(
       uploader string,
       videos int,
       friends int)
   row format delimited 
   fields terminated by "\t" 
   stored as orc
   tblproperties("orc.compress"="SNAPPY");
   
   #向表gulivideo_orc中插入数据
   insert into guliuser_orc select * from guliuser;
   ```



### 11.3 业务实现

#### 11.3.1 视频观看数TOP10

```mysql
#查询语句
select videoId, views
from gulivideo_orc
order by views desc
limit 10;

#输出结果
+--------------+-----------+
|   videoid    |   views   |
+--------------+-----------+
| dMH0bHeiRNg  | 42513417  |
| 0XxI-hvPRRA  | 20282464  |
| 1dmVU08zVpA  | 16087899  |
| RB-wUgnyGv0  | 15712924  |
| QjA5faZF1A8  | 15256922  |
| -_CSo1gOd48  | 13199833  |
| 49IDp76kjPw  | 11970018  |
| tYnn51C3X_w  | 11823701  |
| pv5zWaTEVkI  | 11672017  |
| D2kJZOfq7zk  | 11184051  |
+--------------+-----------+
```



#### 11.3.2 视频数最多的类别TOP10

```mysql
#查询语句
select t1.category_name, count(*) category_cnt
from (
  select videoId, category_name
  from gulivideo_orc
  lateral view explode(category) category_emp as category_name
) t1
group by category_name
order by category_cnt desc
limit 10;

#输出结果
+-------------------+---------------+
| t1.category_name  | category_cnt  |
+-------------------+---------------+
| Music             | 179049        |
| Entertainment     | 127674        |
| Comedy            | 87818         |
| Film              | 73293         |
| Animation         | 73293         |
| Sports            | 67329         |
| Games             | 59817         |
| Gadgets           | 59817         |
| People            | 48890         |
| Blogs             | 48890         |
+-------------------+---------------+
```



#### 11.3.3 观看数最高的TOP20个视频的所属类别中包含TOP20视频的数量

```mysql
#查询语句
select t2.category_name, count(*) category_cnt
from (
  select t1.videoId, category_name
  from (
    select videoId, category, views
    from gulivideo_orc
    order by views desc
    limit 20
  ) t1
  lateral view explode(t1.category) tmp as category_name
)t2
group by t2.category_name
order by category_cnt desc;

#输出结果
+-------------------+---------------+
| t2.category_name  | category_cnt  |
+-------------------+---------------+
| Comedy            | 6             |
| Entertainment     | 6             |
| Music             | 5             |
| Blogs             | 2             |
| People            | 2             |
| UNA               | 1             |
+-------------------+---------------+
```



#### 11.3.4 观看数TOP50视频的关联视频所属类别的视频数排名

```mysql
#查询语句
select t6.category_name, t6.category_cnt,
rank() over(order by t6.category_cnt desc) rank
from (
  select t5.category_name, count(*) category_cnt
  from (
    select t4.related_id, category_name
    from (
      select t2.related_id, t3.category
      from (
        select related_id
        from (
          select videoId, category, views, relatedId
          from gulivideo_orc
          order by views desc
          limit 50
        ) t1
        lateral view explode(t1.relatedId) tmp1 as related_id  
      ) t2
      left join gulivideo_orc t3
      on t2.related_id = t3.videoId
    ) t4
    lateral view explode(t4.category) tmp2 as category_name  
  ) t5
  group by category_name
) t6

#输出结果
+-------------------+------------------+-------+
| t6.category_name  | t6.category_cnt  | rank  |
+-------------------+------------------+-------+
| Comedy            | 237              | 1     |
| Entertainment     | 216              | 2     |
| Music             | 195              | 3     |
| Blogs             | 51               | 4     |
| People            | 51               | 4     |
| Animation         | 47               | 6     |
| Film              | 47               | 6     |
| News              | 24               | 8     |
| Politics          | 24               | 8     |
| Games             | 22               | 10    |
| Gadgets           | 22               | 10    |
| Sports            | 19               | 12    |
| DIY               | 14               | 13    |
| Howto             | 14               | 13    |
| UNA               | 13               | 15    |
| Places            | 12               | 16    |
| Travel            | 12               | 16    |
| Animals           | 11               | 18    |
| Pets              | 11               | 18    |
| Autos             | 4                | 20    |
| Vehicles          | 4                | 20    |
+-------------------+------------------+-------+
```



#### 11.3.5 Music类别中播放量最高的视频TOP10

```mysql
#查询语句
select t1.videoId, views
from (
  select videoId, views, category_name
  from gulivideo_orc
  lateral view explode(category) tmp1 as category_name
) t1
where t1.category_name="Music"
order by views desc
limit 10;

#输出结果
+--------------+-----------+
|  t1.videoid  |   views   |
+--------------+-----------+
| QjA5faZF1A8  | 15256922  |
| tYnn51C3X_w  | 11823701  |
| pv5zWaTEVkI  | 11672017  |
| 8bbTtPL1jRs  | 9579911   |
| UMf40daefsI  | 7533070   |
| -xEzGIuY7kw  | 6946033   |
| d6C0bNDqf3Y  | 6935578   |
| HSoVKUVOnfQ  | 6193057   |
| 3URfWTEPmtE  | 5581171   |
| thtmaZnxk_0  | 5142238   |
+--------------+-----------+
```



#### 11.3.6 每个类别的视频播放量TOP10

```mysql
#查询语句
select t2.videoId, t2.views, t2.category_name, t2.rank
from (
  select t1.videoId, t1.views, t1.category_name,
  rank() over(partition by category_name order by views desc) rank
  from (
    select videoId, views, category_name
    from gulivideo_orc
    lateral view explode(category) tmp1 as category_name
  ) t1
) t2
where t2.rank <= 10;

#输出结果
+--------------+-----------+-------------------+----------+
|  t2.videoid  | t2.views  | t2.category_name  | t2.rank  |
+--------------+-----------+-------------------+----------+
| 1dmVU08zVpA  | 16087899  | Entertainment     | 1        |
| RB-wUgnyGv0  | 15712924  | Entertainment     | 2        |
| vr3x_RRJdd4  | 10786529  | Entertainment     | 3        |
| lsO6D1rwrKc  | 10334975  | Entertainment     | 4        |
| ixsZy2425eY  | 7456875   | Entertainment     | 5        |
| RUCZJVJ_M8o  | 6952767   | Entertainment     | 6        |
| tFXLbXyXy6M  | 5810013   | Entertainment     | 7        |
| 7uwCEnDgd5o  | 5280504   | Entertainment     | 8        |
| 2KrdBUFeFtY  | 4676195   | Entertainment     | 9        |
| vD4OnHCRd_4  | 4230610   | Entertainment     | 10       |
| hr23tpWX8lM  | 4706030   | News              | 1        |
| YgW7or1TuFk  | 2899397   | News              | 2        |
| nda_OSWeyn8  | 2817078   | News              | 3        |
| 7SV2sfoPAY8  | 2803520   | News              | 4        |
| HBa9wdOANHw  | 2348709   | News              | 5        |
| xDh_pvv1tUM  | 2335060   | News              | 6        |
| p_YMigZmUuk  | 2326680   | News              | 7        |
| QCVxQ_3Ejkg  | 2318782   | News              | 8        |
| a9WB_PXjTBo  | 2310583   | News              | 9        |
| qSM_3fyiaxM  | 2291369   | News              | 10       |
-- 省略后续结果
```



#### 11.3.7  上传视频最多的TOP10用户的所有视频中播放量TOP20

```mysql
#查询语句
select t1.uploader, t2.videoId, t2.views
from (
  select uploader, videos
  from guliuser_orc
  order by videos desc
  limit 10
) t1
left join gulivideo_orc t2
on t1.uploader = t2.uploader
order by t2.views desc
limit 20;

#输出结果
+----------------+--------------+-----------+
|  t1.uploader   |  t2.videoid  | t2.views  |
+----------------+--------------+-----------+
| expertvillage  | -IxHBW0YpZw  | 39059     |
| expertvillage  | BU-fT5XI_8I  | 29975     |
| expertvillage  | ADOcaBYbMl0  | 26270     |
| expertvillage  | yAqsULIDJFE  | 25511     |
| expertvillage  | vcm-t0TJXNg  | 25366     |
| expertvillage  | 0KYGFawp14c  | 24659     |
| expertvillage  | j4DpuPvMLF4  | 22593     |
| expertvillage  | Msu4lZb2oeQ  | 18822     |
| expertvillage  | ZHZVj44rpjE  | 16304     |
| expertvillage  | foATQY3wovI  | 13576     |
| expertvillage  | -UnQ8rcBOQs  | 13450     |
| expertvillage  | crtNd46CDks  | 11639     |
| expertvillage  | D1leA0JKHhE  | 11553     |
| expertvillage  | NJu2oG1Wm98  | 11452     |
| expertvillage  | CapbXdyv4j4  | 10915     |
| expertvillage  | epr5erraEp4  | 10817     |
| expertvillage  | IyQoDgaLM7U  | 10597     |
| expertvillage  | tbZibBnusLQ  | 10402     |
| expertvillage  | _GnCHodc7mk  | 9422      |
| expertvillage  | hvEYlSlRitU  | 7123      |
+----------------+--------------+-----------+
```



```mysql
#课后练习1
select t.*,
sum(t.month_sum) over(partition by t.userId order by t.month) user_sum
from (
 select userId, concat(substring(visitDate,1,4),'-0',substring(visitDate,6,1)) month,
 sum(visitCount) month_sum
 from action
 group by userId,concat(substring(visitDate,1,4),'-0',substring(visitDate,6,1))
) t;

#课后练习2
select shop, size(collect_set(user_id)) user_cnt
from visit
group by shop;

select t.*
from (
select shop, user_id, count(*) times,
rank() over(partition by shop order by count(*) desc) rank
from visit
group by shop, user_id
order by shop, times desc) t
where t.rank <= 3;

```

