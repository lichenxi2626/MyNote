# Hadoop

***

## 第一章 大数据概论

### 1.1 大数据概念

- 大数据指无法在一定时间范围内用常规软件工具进行捕捉、管理和处理的数据集合。是海量、高增长率、多样化的信息资产。

### 1.2 大数据特点

- Volume 大量
- Velocity 高速
- Variety 多样
- Value 低价值密度

### 1.3 大数据应用场景

- 大数据广泛应用于物流、仓储、零售、旅游、广告、保险、金融、房地产、人工智能等众多领域。

### 1.4 大数据发展前景

### 1.5 大数据部门业务流程分析

- 产品人员提出需求→数据部门搭建数据平台、分析数据指标→数据可视化

### 1.6 大数据部门组织结构

- 平台组：平台搭建、集群性能监控、调优
- 数据仓库组：数据清洗、数据分析、数据仓库建模
- 实时组：实施指标分析、性能调优
- 数据挖掘组：算法、推荐系统、用户画像
- 报表开发组：JavaEE、前端



## 第二章 从Hadoop框架讨论大数据生态

### 2.1 Hadoop概念

- **Hadoop是由Apache基金会开发的分布式系统基础架构，用于解决海量数据的存储和分析计算问题。**
- 广义上的Hadoop指Hadoop生态圈。

### 2.2 Hadoop发展历史

- Lucene框架→Nutch→Hadoop
- 创始人：Doug Cutting

### 2.3 Hadoop三大发行版本

- Apache：基础版本
  - 官网http://hadoop.apache.org/releases.html
  - 下载https://archive.apache.org/dist/hadoop/common/
- Cloudera：集成了大量大数据框架，对应产品CDH
  - 官网https://www.cloudera.com/downloads/cdh/5-10-0.html
  - 下载http://archive-primary.cloudera.com/cdh5/cdh/5/
- Hortonworks：文档更好，对应产品HDP
  - 官网https://hortonworks.com/products/data-center/hdp/
  - 下载https://hortonworks.com/downloads/#data-platform

### 2.4 Hadoop的优势

- 高可靠性：Hadoop底层维护有多个数据副本，即使Hadoop某个计算元素或存储出现故障，也不会导致数据丢失
- 高扩展性：集群间分配任务数据，可方便的扩展数以千计的节点
- 高效性：在MapReduce的思想下，Hadoop是并行工作的，以加快任务处理的速度
- 高容错性：能够自动将失败的任务重新分配

### 2.5 Hadoop的组成

- Hadoop1.x
  - MapReduce：计算+资源调度
  - HDFS：数据存储
  - Common：辅助工具
- Hadoop2.x/3.x
  - MapReduce：计算
  - Yarn：资源调度
  - HDFS：数据存储
  - Common：辅助工具

#### 2.5.1 HDFS架构概述

- NameNode（nn）：存储文件的**元数据**（文件名、路径、属性等），以及每个文件的**块列表和块所在的DataNode**。
- DataNode（dn）：在本地文件系统存储文件存储文件**块数据**，以及**块数据的校验和**。
- Secondary NameNode（2nn）：定时对NameNode**元数据备份**。

#### 2.5.2 YARN架构概述

- ResourceManager（RM）
  - 处理客户端请求
  - 监控各个NodeManager
  - 启动或监控ApplicationMaster
  - 资源的分配与调度
- NodeManager（NM）
  - 管理单个节点上的资源
  - 处理来自ResouceManager的命令
  - 处理来自ApplicationMaster的命令
- ApplicationMaster（AM）
  - 数据的切分
  - 为应用程序申请资源并分配给内部的任务
  - 任务的监控与容错
- Container
  - 封装节点上的多维度资源（内存、CPU、磁盘、网络等）

#### 2.5.3 MapReduce架构概述

- Map：并行处理输入的数据
- Reduce：对Map结果进行汇总

#### 2.5.4 MapReduce工作过程

1. 运行1个MapReduce任务时，MapReduce等待启动m个MapTask和n个ReduceTask共（m+n）个Task
2. MapReduce启动一个ApplicationMaster，该ApplicationMaster首先会向ResourceManager申请自己的资源（如内存）
3. ResourceManager找到有空余资源的NodeManager后，在该节点上运行ApplicationMaster，同时产生一个Container，封装该ApplicationMaster所需的资源
4. ApplicationMaster继续向RecourceMananer申请MapReduce等待启动的Task的资源
5. ResourceManager继续寻找空余资源的NodeManager，并在这些节点上运行各个Task，同时每个Task都会产生一个Container，封装该Task所需的资源
6. 在所有的Task运行结束后，ApplicationMaster会向ResourceManager申请注销，ResourceManager再对所有Container封装的资源进行回收

### 2.6 大数据技术生态体系

<img src="Hadoop.assets/wps1.png" alt="img" style="zoom:150%;" />

### 2.7 推荐系统框架

<img src="Hadoop.assets/wps2.png" alt="img" style="zoom:150%;" />



## 第三章 Hadoop运行环境搭建

### 3.1 虚拟机环境的准备

1. 最小化安装Linux系统虚拟机，并安装必要环境

   ```shell
   yum install -y epel-release
   yum install -y psmisc nc net-tools rsync vim lrzsz ntp libzstd openssl-static tree iotop
   ```

2. 修改静态IP

   ```shell
   vim /etc/sysconfig/network-scripts/ifcfg-ens33
   
   DEVICE=ens33
   TYPE=Ethernet
   ONBOOT=yes
   BOOTPROTO=static #设置IP为静态
   NAME="ens33"
   IPADDR=192.168.145.101 #修改ip
   PREFIX=24
   GATEWAY=192.168.1.2
   DNS1=192.168.1.2
   ```

3. 修改主机名

   ```shell
   vim /etc/hostname
   ```

4. 配置主机名映射

   ```shell
   vim /etc/hosts
   
   #添加映射的IP地址和主机名
   192.168.145.100 hadoop100
   192.168.145.101 hadoop101
   192.168.145.102 hadoop102
   192.168.145.103 hadoop103
   192.168.145.104 hadoop104
   192.168.145.105 hadoop105
   ```

5. 配置win10系统的主机映射C:\Windows\System32\drivers\etc\hosts

6. 关闭防火墙

   ```shell
   systemctl stop firewalld #关闭防火墙服务
   systemctl disable firewalld #关闭防火墙自启动
   ```

7. 创建atguigu用户，并分配root权限

   ```shell
   useradd atguigu
   passwd atguigu
   
   vi /etc/sudoers
   #找到root行在下方插入
   root    ALL=(ALL)     ALL
   #在下方插入
   atguigu   ALL=(ALL)  NOPASSWD:ALL
   ```

8. 创建文件夹

   ```shell
   mkdir /opt/module /opt/software
   ```



### 3.2 安装JDK

1. 上传JDK安装包到 /opt/software

2. 解压JDK安装包到指定目录

   ```shell
   tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
   ```

3. 配置JDK环境变量

   ```shell
   vim /etc/profile.d/my_env.sh #文件名可自取,但后缀必须为.sh
   
   #编写并保存
   #JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   export PATH=$PATH:$JAVA_HOME/bin #多个环境变量使用":"连接
   
   #刷新配置文件并查看环境变量是否生效
   source /etc/profile.d/my_env.sh
   echo $JAVA_HOME
   ```

4. 查看JDK版本

   ```shell
   java -version
   ```



### 3.3 安装Hadoop

1. 上传Hadoop包到 /opt/software

2. 解压Hadoop

   ```shell
   tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
   ```

3. 配置Hadoop环境变量

   ```shell
   vim /etc/profile.d/my_env.sh #文件名可自取,但后缀必须为.sh
   
   #在JDK配置后编写并保存
   #HADOOP_HOME
   HADOOP_HOME=/opt/module/hadoop-3.1.3
   PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   export PATH JAVA_HOME HADOOP_HOME
   
   #刷新配置文件并查看环境变量是否生效
   source /etc/profile.d/my_env.sh
   echo $HADOOP_HOME
   ```

4. 查看Hadoop版本

   ```shell
   hadoop version
   ```



### 3.4 Hadoop目录结构

```shell
drwxr-xr-x. 2 atguigu atguigu   4096 9月  12 2019 bin
#bin:存放对Hadoop相关服务进行操作的脚本
drwxr-xr-x. 3 atguigu atguigu   4096 9月  12 2019 etc
#etc:存放Hadoop的配置文件
drwxr-xr-x. 2 atguigu atguigu   4096 9月  12 2019 include
drwxr-xr-x. 3 atguigu atguigu   4096 9月  12 2019 lib
#lib:存放Hadoop的本地库
drwxr-xr-x. 4 atguigu atguigu   4096 9月  12 2019 libexec
-rw-rw-r--. 1 atguigu atguigu 147145 9月   4 2019 LICENSE.txt
-rw-rw-r--. 1 atguigu atguigu  21867 9月   4 2019 NOTICE.txt
-rw-rw-r--. 1 atguigu atguigu   1366 9月   4 2019 README.txt
drwxr-xr-x. 3 atguigu atguigu   4096 9月  12 2019 sbin
#sbin:存放启动或停止Hadoop相关服务的脚本
drwxr-xr-x. 4 atguigu atguigu   4096 9月  12 2019 share
#share:存放Hadoop依赖的jar包、文档和官方案例
```



## 第四章 Hadoop运行模式

- Hadoop运行模式包括：本地模式、伪分布式模式及完全分布式模式
- Hadoop官网：http://hadoop.apache.org/

### 4.1 本地运行模式

#### 4.1.1 官方Grep案例

1. 创建input目录

   ```shell
   cd /opt/module/hadoop-3.1.3
   mkdir input
   ```

2. 将Hadoop的xml文件复制到input中

   ```shell
   cp etc/hadoop/*.xml input
   ```

3. 执行share目录下的MapReduce程序grep

   ```shell
   hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep input output 'dfs[a-z.]+'
   ```

4. 查看输出结果

   ```shell
   cat output/*
   ```

#### 4.1.2 官方WordCount案例

1. 创建wcinput目录

   ```shell
   cd /opt/module/hadoop-3.1.3
   mkdir wcinput
   ```

2. 在wcinput目录下创建wc.input文件并编辑

   ```shell
   vi wcinput/wc.input
   
   #输入以下内容并保存
   hadoop yarn
   hadoop mapreduce
   atguigu
   atguigu
   ```

3. 执行share目录下的MapReduce程序wordcount

   ```shell
   hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount wcinput wcoutput
   ```

4. 查看输出结果

   ```shell
   cat wcoutput/part-r-00000
   ```



### 4.2 伪分布式运行模式

#### 4.2.1 启动HDFS并运行MapReduce程序

1. 配置hadoop-env.sh

   ```shell
   vi /opt/module/hadoop-3.1.3/etc/hadoop/hadoop-env.sh
   
   #添加JDK的安装路径
   export JAVA_HOME=/opt/module/jdk1.8.0_212
   ```

2. 配置core-site.xml

   ```xml
   vi /opt/module/hadoop-3.1.3/etc/hadoop/core-site.xml
   
   <!-- configuration中添加以下内容 -->
   <configuration>
       <!-- 指定HDFS中NameNode的地址 -->
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://hadoop101:9820</value>
       </property>
   
       <!-- 指定Hadoop运行时产生文件的存储目录 -->
       <property>
           <name>hadoop.tmp.dir</name>
           <value>/opt/module/hadoop-3.1.3/data/tmp</value>
       </property>
   </configuration>
   ```

3. 配置hdfs-site.xml

   ```xml
   vi /opt/module/hadoop-3.1.3/etc/hadoop/hdfs-site.xml
   
   <!-- configuration中添加以下内容 -->
   <configuration>
       <!-- 指定HDFS副本的数量 -->
       <property>
           <name>dfs.replication</name>
           <value>1</value>
       </property>
   </configuration>
   ```

4. 格式化NameNode（首次启动前需要格式化）

   ```shell
   hdfs namenode -format
   
   #若本地hadoop根目录下已经存在data和logs目录,则必须先删除再初始化
   #这是因为NameNode在初始化时会产生新的集群id,导致NameNode的新集群id和DataNode中的原集群id不匹配
   ```

5. 启动NameNode

   ```shell
   hdfs --daemon start namenode
   ```

6. 启动DataNode

   ```shell
   hdfs --daemon start datanode
   ```

7. 查看集群

   ```shell
   #查看启动的java进程,jps是JDK中的命令
   jps
   
   #web端查看HDFS文件系统
   http://hadoop101:9870
   ```

   - 常用端口号

   | Daemon                  | App                           | Hadoop2     | Hadoop3 |
   | ----------------------- | ----------------------------- | ----------- | ------- |
   | NameNode Port           | Hadoop HDFS NameNode          | 8020 / 9000 | 9820    |
   |                         | Hadoop HDFS NameNode HTTP UI  | 50070       | 9870    |
   |                         | Hadoop HDFS NameNode HTTPS UI | 50470       | 9871    |
   | Secondary NameNode Port | Secondary NameNode HTTP       | 50091       | 9869    |
   |                         | Secondary NameNode HTTP UI    | 50090       | 9868    |
   | DataNode Port           | Hadoop HDFS DataNode IPC      | 50020       | 9867    |
   |                         | Hadoop HDFS DataNode          | 50010       | 9866    |
   |                         | Hadoop HDFS DataNode HTTP UI  | 50075       | 9864    |
   |                         | Hadoop HDFS DataNode HTTPS UI | 50475       | 9865    |

   

#### 4.2.2 启动Yarn并运行MapRedce程序

1. 配置yarn-site.xml

   ```xml
   vi /opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml
   
   <!-- configuration中添加以下内容 -->
   <configuration>
       <!-- Reducer获取数据的方式 -->
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
   
       <!-- 指定YARN的ResourceManager的地址 -->
       <property>
           <name>yarn.resourcemanager.hostname</name>
           <value>hadoop101</value>
       </property>
       <property>
           <name>yarn.nodemanager.env-whitelist</name>		
           <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
       </property>
   </configuration>
   ```

2. 配置mapred-site.xml

   ```xml
   vi /opt/module/hadoop-3.1.3/etc/hadoop/mapred-site.xml
   
   <!-- configuration中添加以下内容 -->
   <configuration>
       <!-- 指定MR运行在YARN上 -->
       <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
       </property>
   </configuration>
   ```

3. 启动NameNode和DataNode

4. 启动ResourceManager

   ```shell
   yarn --daemon start resourcemanager
   ```

5. 启动NodeManager

   ```shell
   yarn --daemon start nodemanager
   ```

6. 查看集群

   ```shell
   #查看java进程
   jps
   
   #web端查看Yarn
   http://hadoop101:8088
   ```

7. 操作集群

   ``` shell
   hdfs dfs -mkdir -p /input
   #在hdfs根目录上创建一个input文件夹
   
   hdfs dfs -put wcinput/wc.input /input
   #将本地路径下的文件wcinput/wc.input上传到hdfs的/input中
   
   hdfs dfs -ls /input/
   #查看hdfs的input目录
   
   hdfs dfs -cat input/wc.input
   #查看hdfs的wc.input文件
   
   hadoop jar
   share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input/ /output
   #在hdfs上执行MapReduce中的wordcount程序
   
   hdfs dfs -cat /output/*
   #查看结果
   
   hdfs dfs -get /output/part-r-00000 ./output/
   #下载运行结果到本地
   
   hdfs dfs -rm -r /output
   #删除hdfs中的结果
   ```



#### 4.2.3 配置文件

- 默认配置文件：Hadoop启动时先加载默认配置文件

  | 默认配置文件       | jar包中的位置                                              |
  | ------------------ | ---------------------------------------------------------- |
  | core-default.xml   | hadoop-common-3.1.3.jar/ core-default.xml                  |
  | hdfs-default.xml   | hadoop-hdfs-3.1.3.jar/ hdfs-default.xml                    |
  | yarn-default.xml   | hadoop-yarn-common-3.1.3.jar/ yarn-default.xml             |
  | mapred-default.xml | hadoop-mapreduce-client-core-3.1.3.jar/ mapred-default.xml |

- 自定义配置文件：默认配置文件加载完成后加载，用于修改默认配置文件中的属性

  | 自定义配置文件  | Hadoop本地目录中的位置     |
  | --------------- | -------------------------- |
  | core-site.xml   | etc/hadoop/core-site.xml   |
  | hdfs-site.xml   | etc/hadoop/hdfs-site.xml   |
  | yarn-site.xml   | etc/hadoop/yarn-site.xml   |
  | mapred-site.xml | etc/hadoop/mapred-site.xml |



### 4.3 完全分布式运行模式

#### 4.3.1 虚拟机准备

- 安装虚拟机，关闭防火墙，修改静态ip、主机名称
- 安装JDK、Hadoop，并配置环境变量
- 修改Hadoop中的hadoop-env.sh文件，配置JAVA_HOME路径
- 集群规划
  - 配置共3台主机，各自包含1个DataNode和1个NodeManager
  - 将NameNode，ResourceManager，SecondaryNameNode分别配置到3台主机上



#### 4.3.2 编写集群分发脚本

- scp（secure copy）安全拷贝：实现服务器与服务器之间的数据拷贝。

  - 基本语法

    ```shell
    scp -r $pdir/$fname $user@hadoop$host:$pdir/$fname
    #-r 递归复制
    #当未指定用户时,默认使用当前用户,因此要保证未指定用户的主机上创建了当前用户
    ```

  - 举例

    ```shell
    sudo scp -r /opt/module root@hadoop102:/opt/module
    
    sudo scp -r atguigu@hadoop101:/opt/module root@hadoop103:/opt/module
    
    #注意修改拷贝后文件的所有者和所属组
    #配置文件拷贝后需要source
    ```

- rsync远程同步工具：只对差异文件做更新，传输速度更快

  - 基本语法

    ```shell
    rsync -av $pdir/$fname $user@hadoop$host:$pdir/$fname
    #-a 归档复制
    #-v 显示复制过程
    ```

  - 举例

    ```shell
    rsync -av /opt/software/ hadoop102:/opt/software
    ```

- xsync脚本：集群分发

  1. 在/home/atguigu/bin目录下创建脚本

     ```shell
     #/home/atguigu/bin为atguigu用户的环境变量目录
     vi /home/atguigu/bin/sxync
     
     #编写如下编码
     #!/bin/bash
     #1. 判断参数个数
     if [ $# -lt 1 ]
     then
       echo Not Enough Arguement!
       exit;
     fi
     #2. 遍历集群所有机器
     for host in hadoop101 hadoop102
     do
       echo ====================  $host  ====================
       #3. 遍历所有目录，挨个发送
       for file in $@
       do
         #4 判断文件是否存在
         if [ -e $file ]
         then
           #5. 获取父目录
           pdir=$(cd -P $(dirname $file); pwd)
           #6. 获取当前文件的名称
           fname=$(basename $file)
           ssh $host "mkdir -p $pdir"
           rsync -av $pdir/$fname $host:$pdir
         else
           echo $file does not exists!
         fi
       done
     done
     ```

  2. 修改脚本执行权限

     ```shell
     sudo chmod 777 xsync
     ```

  3. 测试脚本

     ```shell
     sudo xsync /lcx.txt
     ```

     

#### 4.3.3 集群配置

1. 修改core-site.xml

   ```xml
   <!-- configuration中添加以下内容 -->
   <configuration>
   
       <!-- 指定NameNode的地址 -->
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://hadoop100:9820</value>
       </property>
   
       <!-- 指定hadoop数据的存储目录  
   官方配置文件中的配置项是hadoop.tmp.dir,用来指定hadoop数据的存储目录,此次配置用的hadoop.data.dir是自己定义的变量,因为在hdfs-site.xml中会使用此配置的值来分别指定namenode和datanode存储数据的目录
    -->
       <property>
           <name>hadoop.data.dir</name>
           <value>/opt/module/hadoop-3.1.3/data</value>
       </property>
   
       <!-- 下面是兼容性配置 -->
       <!-- 配置该atguigu(superUser)允许通过代理访问的主机节点 -->
       <property>
           <name>hadoop.proxyuser.atguigu.hosts</name>
           <value>*</value>
       </property>
       <!-- 配置该atguigu(superuser)允许代理的用户所属组 -->
       <property>
           <name>hadoop.proxyuser.atguigu.groups</name>
           <value>*</value>
       </property>
       <!-- 配置该atguigu(superuser)允许代理的用户-->
       <property>
           <name>hadoop.proxyuser.atguigu.users</name>
           <value>*</value>
       </property>
   </configuration>
   ```

2. 修改hdfs-site.xml

   ```xml
   <!-- configuration中添加以下内容 -->
   <configuration>
       <!-- NameNode数据的存储目录 -->
       <property>
           <name>dfs.namenode.name.dir</name>
           <value>file://${hadoop.data.dir}/name</value>
       </property>
       <!-- DataNode数据的存储目录 -->
       <property>
           <name>dfs.datanode.data.dir</name>
           <value>file://${hadoop.data.dir}/data</value>
       </property>
       <!-- SecondaryNameNode数据的存储目录 -->
       <property>
           <name>dfs.namenode.checkpoint.dir</name>
           <value>file://${hadoop.data.dir}/namesecondary</value>
       </property>
       <!-- 兼容性配置 -->
       <property>
           <name>dfs.client.datanode-restart.timeout</name>
           <value>30</value>
       </property>
       <property>
       <!-- NameNode web端访问地址 -->
       <property>
           <name>dfs.namenode.http-address</name>
           <value>hadoop100:9870</value>
       </property>
       <!-- SeconddaryNameNode web端访问地址 -->
       <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>hadoop102:9868</value>
       </property>
   </configuration>
   ```

3. 修改yarn-site.xml

   ```xml
   <!-- configuration中添加以下内容 -->
   <configuration>
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
       <!-- 指定ResourceManager的地址-->
       <property>
           <name>yarn.resourcemanager.hostname</name>
           <value>hadoop101</value>
       </property>
       <!-- 环境变量的继承 -->
       <property>
           <name>yarn.nodemanager.env-whitelist</name>			<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
       </property>
   </configuration>
   ```
   
4. 修改mapred-site.xml

   ```xml
   <!-- configuration中添加以下内容 -->
   <configuration>
       <!-- 指定MapReduce程序运行在Yarn上 -->
       <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
       </property>
   </configuration>
   ```

5. 将修改后的配置文件分发到集群

   ```shell
   xsync /opt/module/hadoop-3.1.3/etc
   ```

   

#### 4.3.4 集群单点启动

1. 格式化NameNode

   ```shell
   #在hadoop100上格式化,注意格式化前删除data和logs目录
   hdfs namenode -format
   ```

2. 启动NameNode

   ```shell
   #在hadoop100上启动namenode
   hdfs --daemon start namenode
   ```

3. 启动SecondaryNameNode

   ```shell
   #在hadoop102上启动secondarynamenode
   hdfs --daemon start secondarynamenode
   ```

4. 启动DataNode

   ```shell
   #分别在hadoop100,hadoop101,hadoop102上启动datanode
   hdfs --daemon start datanode
   ```

5. 启动ResourceManager

   ```shell
   #在hadoop101上启动resourcemanager
   yarn --daemon start resourcemanager
   ```

6. 启动NodeManager

   ```shell
   #分别在hadoop100,hadoop101,hadoop102上启动nodemanager
   yarn --daemon start nodemanager
   ```




#### 4.3.5 SSH免密登陆

- ssh直接连接主机

  ```shell
  ssh hadoop100
  #需要输入密码
  ```

- ssh配置免密登陆

  1. 在访问方的主机上生成公钥和私钥

     ```shell
     ssh-keygen -t rsa
     #连敲3次回车确认
     #执行后会在家目录下生成./ssh目录(隐藏的).其中存放了公钥和私钥文件
     ```

  2. 将公钥拷贝到需要被访问的主机上

     ```shell
     ssh-copy-id hadoop100#通过ssh访问本地主机也需要配置免密登陆,方便后续脚本的执行
     ssh-copy-id hadoop101
     ssh-copy-id hadoop102
     ```

  3. 为使3台主机能互相访问，在需要ssh免密登陆的主机上都要执行上述操作

- .ssh目录下的文件

  | 文件            | 作用                           |
  | --------------- | ------------------------------ |
  | known_hosts     | 记录ssh访问过的主机的公钥      |
  | id_rsa          | 记录生成的私钥                 |
  | id_rsa.pub      | 记录生成的公钥                 |
  | authorized_keys | 存放授权过无密登录的主机的公钥 |



#### 4.3.6 群起/停止集群

- 配置集群的群起、停止

  1. 配置workers文件并同步至所有节点

     ```shell
     vi /opt/module/hadoop-3.1.3/etc/hadoop/workers
     
     #在配置文件中追加集群的所有主机并保存,配置前提是已经在/etc/hosts添加了主机名IP映射
     hadoop100
     hadoop101
     hadoop102
     
     #将配置文件同步至所有节点
     xsync  /opt/module/hadoop-3.1.3/etc/hadoop/workers
     ```

  2. 群起、群停集群

     ```shell
     #hadoop安装目录下的sbin目录中存有控制集群启动、停止的多个shell脚本
     
     #在任意节点执行以下操作
     start-dfs.sh #启动hdfs(启动NN、2NN、所有节点的DN)
     stop-dfs.sh #停止heds
     
     #在配置了ResourceManager的节点执行以下操作
     start-yarn.sh #启动yarn(启动RM、所有节点的NM)
     stop-yarn.sh #停止yarn
     ```

- 编写shell脚本群起、群停集群

  ```shell
  #用户家目录下的/bin目录中的内容会自动配置到环境变量中
  
  #编辑集群控制脚本
  vi /home/atguigu/bin/clusterctl
  
  #输入以下内容并保存
  #!/bin/bash
  if [ $# -lt 1 ]
  	then
  		echo "Input args is null"
  	exit
  fi
  case $1 in
  "start")
      echo "---------- hdfs start -----------"
      ssh hadoop100 /opt/module/hadoop-3.1.3/sbin/start-dfs.sh
      echo "---------- yarn start -----------"
      ssh hadoop101 /opt/module/hadoop-3.1.3/sbin/start-yarn.sh
      ssh hadoop100 mapred --daemon start historyserver #启动历史服务器
  ;;
  "stop")
      echo "---------- yarn stop -----------"
      ssh hadoop101 /opt/module/hadoop-3.1.3/sbin/stop-yarn.sh
      ssh hadoop100 mapred --daemon stop historyserver #关闭历史服务器
      echo "---------- hdfs stop -----------"
      ssh hadoop100 /opt/module/hadoop-3.1.3/sbin/stop-dfs.sh
  ;;
  *)
  	echo "Input args not found"
  ;;
  esac
  
  #修改脚本权限
  chmod 777 clusterctl
  
  #分发给所有主机
  xsync /home/atguigu/bin/clusterctl
  
  #通过start stop参数执行脚本
  clusterctl start
  clusterctl stop
  ```

- 编写脚本查看集群状态

  ```shell
  #编辑集群状态查询脚本
  vi /home/atguigu/bin/jpsall
  
  #输入以下内容并保存
  for i in hadoop100 hadoop101 hadoop103
  do
  	echo "----------- $i ----------"
  	ssh $i jps
  done
  
  #修改脚本权限
  chmod 777 jpsall
  
  #分发给所有主机
  xsync /home/atguigu/bin/jpsall
  
  #执行脚本
  jpsall
  ```



#### 4.3.7 配置历史服务器

1. 配置mapred-site.xml

   ```xml
   vi /opt/module/hadoop-3.1.3/etc/hadoop/mapred-site.xml
   
   <!-- 添加如下内容 -->
   <!-- 历史服务器端地址 -->
   <property>
       <name>mapreduce.jobhistory.address</name>
       <value>hadoop100:10020</value>
   </property>
   
   <!-- 历史服务器web端地址 -->
   <property>
       <name>mapreduce.jobhistory.webapp.address</name>
       <value>hadoop100:19888</value>
   </property>
   ```

2. 分发配置

   ```shell
   xsync /opt/module/hadoop-3.1.3/etc/hadoop/mapred-site.xml
   ```

3. 启动历史服务器

   ```shell
   #在历史服务器配置的主机上启动
   mapred --daemon start historyserver
   ```

4. 确认启动

   ```shell
   #确认启动主机服务
   jps
   
   #确认启动web端
   http://hadoop100:19888/jobhistory
   ```



#### 4.3.8 配置日志的聚集（虚拟机内存超标问题）

- 概念：应用运行完成后，将程序运行日志上传到HDFS系统上

- 作用：方便查看程序运行的情况

- 配置过程

  1. 配置yarn-site.xml

     ```xml
     vi /opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml
     
     <!-- 添加如下内容 -->
     <property>
         <name>yarn.log-aggregation-enable</name>
         <value>true</value>
     </property>
     <property>  
         <name>yarn.log.server.url</name>  
         <value>http://hadoop100:19888/jobhistory/logs</value>  
     </property>
     <property>
         <name>yarn.log-aggregation.retain-seconds</name>
         <value>604800</value>
     </property>
     <!-- 解决MR任务运行过程中的虚拟内存超标问题 -->
     <property>
         <name>yarn.nodemanager.vmem-check-enabled</name>
         <value>false</value>
     </property>
     ```

  2. 分发配置文件

     ```shell
     xsync /opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml
     ```

  3. 重新启动集群

     ```shell
     clusterctl stop
     clusterctl start
     ```

  4. 查看日志

     ```shell
     #登陆历史服务器查看日志
     http://hadoop100:19888/jobhistory
     ```

     

#### 4.3.9 配置集群时间同步

1. 关闭所有节点的ntp服务（使用root用户操作以下所有步骤）

   ```shell
   #在所有节点上执行
   sudo - root
   systemctl stop stpd
   ```

2. 修改ntp配置文件

   ```shell
   #在作为时间服务器的主机中配置
   vi /etc/ntp.conf
   
   #修改以下内容
   
   #原内容:
   #restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
   #修改后:删除注释,修改网段,修改后可以向该网段下的所有节点提供时间同步
   restrict 192.168.145.0 mask 255.255.255.0 nomodify notrap
   
   #原内容:
   server 0.centos.pool.ntp.org iburst
   server 1.centos.pool.ntp.org iburst
   server 2.centos.pool.ntp.org iburst
   server 3.centos.pool.ntp.org iburst
   #修改后:节点失去网络连接后,依然使用本地时间作为时间服务器为其他节点提供时间同步
   #server 0.centos.pool.ntp.org iburst
   #server 1.centos.pool.ntp.org iburst
   #server 2.centos.pool.ntp.org iburst
   #server 3.centos.pool.ntp.org iburst
   server 127.127.1.0
   fudge 127.127.1.0 stratum 10
   ```

3. 修改ntpd文件

   ```shell
   #在作为时间服务器的主机中配置
   vi /etc/sysconfig/ntpd
   
   #追加如下内容,使硬件时间与系统时间一起同步
   SYNC_HWCLOCK=yes
   ```

4. 启动ntpd服务

   ```shell
   systemctl start ntpd
   ```

5. 配置定时任务同步时间

   ```shell
   #在其他的所有主机中配置
   #编辑定时任务
   crontab -e
   #每隔10分钟与hadoop100主机同步时间
   */10 * * * * /usr/sbin/ntpdate hadoop100
   ```



## 第五章 Hadoop3和Hadoop2的主要区别

1. 最低Java版本
   - hadoop2：JDK7
   - hadoop3：JDK8
2. 引入纠删码
   - 解决数据量过大时，磁盘空间存储能力不足的问题
   - Hadoop中默认的3副本方案对于存储空间的有200%的额外开销，纠删码可以将数据冗余控制在约50%左右，且具有和3副本方案相同的容错力
3. 重写shell脚本
4. 支持超过2个NameNode
5. 服务的默认端口号变化

| Daemon                  | App                           | Hadoop2     | Hadoop3 |
| ----------------------- | ----------------------------- | ----------- | ------- |
| NameNode Port           | Hadoop HDFS NameNode          | 8020 / 9000 | 9820    |
|                         | Hadoop HDFS NameNode HTTP UI  | 50070       | 9870    |
|                         | Hadoop HDFS NameNode HTTPS UI | 50470       | 9871    |
| Secondary NameNode Port | Secondary NameNode HTTP       | 50091       | 9869    |
|                         | Secondary NameNode HTTP UI    | 50090       | 9868    |
| DataNode Port           | Hadoop HDFS DataNode IPC      | 50020       | 9867    |
|                         | Hadoop HDFS DataNode          | 50010       | 9866    |
|                         | Hadoop HDFS DataNode HTTP UI  | 50075       | 9864    |
|                         | Hadoop HDFS DataNode HTTPS UI | 50475       | 9865    |



# HDFS

***

## 第一章 HDFS概述

### 1.1 HDFS的背景和定义

- 产生背景：随之数据量越来越大，在一个操作系统下无法存储全部的数据，需要数据分配到更多的操作系统管理的磁盘中。为了方便的管理多台主机上的文件，出现了分布式文件管理系统。HDFS是分布式文件管理系统的一种。
- 定义
  - HDFS（Hadoop Distributed File System）是文件系统，用于存储文件，通过目录树定位文件：HDFS是分布式的，由很多服务器连接共同实现功能，集群中的每个服务器有各自的角色。
  - 使用场景：一次写入，多次读取，不支持文件的修改。

### 1.2 HDFS的优缺点

- 优点
  1. 高容错性（3副本方案），可靠性高
  2. 适合处理大数据（PB级数据量，文件数量达百万级）
- 缺点
  1. 不适合低延时的数据访问
  2. 无法高效的对大量的小文件进行存储（需要占用大量的NameNode资源存储元数据，同时延长寻址时间）
  3. 不支持同一文件的多线程写入操作
  4. 仅支持数据文件的追加操作，不支持修改

### 1.3 HDFS组成架构

- NameNode
- DataNode
- Client
- SecondaryNameNode

### 1.4 HDFS文件块大小

- HDFS上的文件在物理上分块（Block）存储，文件块的大小可通过配置参数指定，默认大小为128M
- 文件块的大小设置
  - 过小：增加寻址时间
  - 过大：增加文件传输时间
  - 文件块的大小取决于磁盘文件传输速率，一般设置为磁盘每秒传输数据量大小的相近整数



## 第二章 HDFS的Shell操作

### 2.1 基本语法

```shell
#方式一
hadoop fs [命令]
#方式二
hdfs dfs [命令]
```

### 2.2 命令

```shell
#追加一个文件到已经存在的文件末尾
-appendToFile <localsrc> ... <dst>
hadoop fs -appendToFile liubei.txt /sanguo/shuguo/kongming.txt

#显示文件内容
-cat [-ignoreCrc] <src> ...
hadoop fs -cat /sanguo/shuguo/kongming.txt

-checksum <src> ...

#修改文件权限
-chgrp [-R] GROUP PATH...
-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...
-chown [-R] [OWNER][:[GROUP]] PATH...
hadoop fs  -chmod  666  /sanguo/shuguo/kongming.txt
hadoop fs  -chown  atguigu:atguigu   /sanguo/shuguo/kongming.txt

#从本地文件系统中拷贝文件到HDFS路径去
-copyFromLocal [-f] [-p] <localsrc> ... <dst>
hadoop fs -copyFromLocal README.txt /

#从HDFS拷贝到本地
-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>
hadoop fs -copyToLocal /sanguo/shuguo/kongming.txt ./

-count [-q] <path> ...

#从HDFS的一个路径拷贝到HDFS的另一个路径
-cp [-f] [-p] <src> ... <dst>
hadoop fs -cp /sanguo/shuguo/kongming.txt /zhuge.txt

-createSnapshot <snapshotDir> [<snapshotName>]
-deleteSnapshot <snapshotDir> <snapshotName>
-df [-h] [<path> ...]

#统计文件夹的大小信息
-du [-s] [-h] <path> ...
hadoop fs -du -h /user/atguigu/test

-expunge

#从HDFS拷贝到本地
-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>
hadoop fs -get /sanguo/shuguo/kongming.txt ./

-getfacl [-R] <path>

#下载多个文件并合并(得到一个结果文件)
-getmerge [-nl] <src> <localdst>
hadoop fs -getmerge /user/atguigu/test/* ./zaiyiqi.txt

#输出命令的参数
-help [cmd ...]
hadoop fs -help rm

#查看目录信息
-ls [-d] [-h] [-R] [<path> ...]
hadoop fs -ls /

#创建目录
-mkdir [-p] <path> ...
hadoop fs -mkdir -p /sanguo/shuguo

#从本地剪切粘贴到HDFS
-moveFromLocal <localsrc> ... <dst>
hadoop fs  -moveFromLocal  ./kongming.txt  /sanguo/shuguo

#从HDFS剪切粘贴到本地
-moveToLocal <src> <localdst>
hadoop fs  -moveToLocal  ./kongming.txt  /sanguo/shuguo

#在HDFS目录中移动文件/重命名
-mv <src> ... <dst>
hadoop fs -mv /zhuge.txt /sanguo/shuguo/

#从本地文件系统中拷贝文件到HDFS路径去
-put [-f] [-p] <localsrc> ... <dst>
hadoop fs -put ./zaiyiqi.txt /user/atguigu/test/

-renameSnapshot <snapshotDir> <oldName> <newName>

#删除文件或文件夹
-rm [-f] [-r|-R] [-skipTrash] <src> ...
hadoop fs -rm /user/atguigu/test/jinlian2.txt

#删除空目录
-rmdir [--ignore-fail-on-non-empty] <dir> ...
hadoop fs -rmdir /test

-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]

#设置HDFS中文件的副本数量
-setrep [-R] [-w] <rep> <path> ...
hadoop fs -setrep 5 /README.txt

-stat [format] <path> ...

#显示一个文件的末尾
-tail [-f] <file>
hadoop fs -tail /sanguo/shuguo/kongming.txt

-test -[defsz] <path>
-text [-ignoreCrc] <src> ...
-touchz <path> ...
-usage [cmd ...]
```

### 2.3 设置web端的HDFS操作权限

```xml
<!-- 修改core-site.xml配置文件 -->
vi core-site.xml

<!-- 修改http访问的静态用户为atguigu -->
<property>
    <name>hadoop.http.staticuser.user</name>
    <value>atguigu</value>
</property>
```



## 第三章 HDFS的客户端操作

### 3.1 HDFS客户端环境准备

1. 在windows环境变量中配置hadoop依赖的家目录和bin目录

2. 在idea中创建maven工程

3. 修改pom.xml配置文件，导入相关的依赖

   ```xml
   <dependencies>
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
           <artifactId>hadoop-client</artifactId>
           <version>3.1.3</version>
       </dependency>
       <dependency>
           <groupId>org.apache.hadoop</groupId>
           <artifactId>hadoop-client</artifactId>
           <version>3.1.3</version>
       </dependency>
       <dependency>
           <groupId>org.apache.hadoop</groupId>
           <artifactId>hadoop-client-runtime</artifactId>
           <version>3.1.3</version>
       </dependency>
   </dependencies>
   ```

4. 在main/resources目录下新建log4j2.xml文件，并添加以下内容

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

   

### 3.2 HDFS的API操作

#### 3.2.1 上传文件

```java
//获取文件系统需要导入以下包
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

@Test
public void copyFromLocalFile() throws IOException, InterruptedException {

    //1.获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");//NameNode地址
    Configuration conf = new Configuration();
    String user = "atguigu";//操作的用户名
    FileSystem fs = FileSystem.get(uri, conf, user);

    //2.上传文件
    fs.copyFromLocalFile(false, true, new Path("D:/123.txt"), new Path("/atguigu"));
    //形参列表为(是否删除原文件,是否覆盖已有的同名文件,原文件路径,目标路径)

    //3关闭资源
    fs.close();  
}
```

- 在main/resources新建hdfs-site.xml文件修改配置信息

  ```xml
  <!-- 在配置文件中加入以下内容 -->
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
      <!-- 修改保存副本数为1 -->
      <property>
          <name>dfs.replication</name>
          <value>1</value>
      </property>
  </configuration>
  ```

- 配置参数优先级

  1. 客户端代码中的配置参数
  2. 客户端resources中的配置文件
  3. 服务器的自定义配置文件
  4. 服务器的默认配置文件

#### 3.2.2 下载文件

```java
@Test
public void copyToLocalFile() throws IOException, InterruptedException {

    //1.获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");
    Configuration conf = new Configuration();
    String user = "atguigu";
    FileSystem fs = FileSystem.get(uri, conf, user);

    //2.下载文件
    fs.copyToLocalFile(false, new Path("/123.txt"), new Path("D:/123.txt"), true);
    //形参列表为(是否删除原文件,原文件路径,目标文件路径(含文件名),是否取消CRC文件校验)

    //3.关闭资源
    fs.close();
}
```

#### 3.2.3 创建目录

```java
@Test
public void delete() throws IOException, InterruptedException {

    //1.获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");
    Configuration conf = new Configuration();
    String user = "atguigu";
    FileSystem fs = FileSystem.get(uri, conf, user);
	
    //2.创建目录
    fs.mkdirs(new Path("/user"));
	
    //3.关闭资源
    fs.close();
}
```

#### 3.2.4 删除文件或目录

```java
@Test
public void delete() throws IOException, InterruptedException {

    //1.获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");
    Configuration conf = new Configuration();
    String user = "atguigu";
    FileSystem fs = FileSystem.get(uri, conf, user);
	
    //2.删除文件或目录
    fs.delete(new Path("/user"), true);
    //形参列表为(删除的文件或目录路径,是否递归删除)
	
    //3.关闭资源
    fs.close();
}
```

#### 3.2.5 修改文件名/移动文件

```java
@Test
public void delete() throws IOException, InterruptedException {

    //1.获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");
    Configuration conf = new Configuration();
    String user = "atguigu";
    FileSystem fs = FileSystem.get(uri, conf, user);
	
    //2.重命名文件
    fs.rename(new Path("/123.txt"), new Path("/456.txt"));
	
    //3.关闭资源
    fs.close();
}
```

#### 3.2.6 查看文件详情

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.LocatedFileStatus;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.RemoteIterator;

@Test
public void listFiles() throws IOException, InterruptedException {

    //1.获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");
    Configuration conf = new Configuration();
    String user = "atguigu";
    FileSystem fs = FileSystem.get(uri, conf, user);

    //2.获取文件详情集合
    RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
    //形参列表为(查看的路径,是否递归查看)
    
    //遍历集合
    while(listFiles.hasNext()){
        LocatedFileStatus status = listFiles.next();
        //查看rwx权限
        System.out.println(status.getPermission());
        //查看所属者
        System.out.println(status.getOwner());
        //查看所属组
        System.out.println(status.getGroup());
        //查看文件大小
        System.out.println(status.getLen());
        //查看文件名
        System.out.println(status.getPath().getName());
        System.out.println("========================");
    }

    //3.关闭资源
    fs.close();
}
```

#### 3.2.7 判断文件和目录

```java
//递归打印文件和目录

public void getFileTree(String path, FileSystem fs) throws IOException {
    //获取路径下所有文件的信息数组
    FileStatus[] fileStatuses = fs.listStatus(new Path(path));
    //遍历数组
    for (FileStatus fileStatus : fileStatuses) {
        if (fileStatus.isFile()){//若为文件直接输出信息
            System.out.println("File-->" + fileStatus.getPath());
        }else{//若为目录则递归调用此方法
            System.out.println("Dir-->" + fileStatus.getPath());
            getFileTree(fileStatus.getPath().toString(), fs);
        }
    }
}

@Test
public void test() throws IOException, InterruptedException {
    //获取文件系统
    URI uri = URI.create("hdfs://hadoop100:9820");
    Configuration configuration = new Configuration();
    String user = "atguigu";
    FileSystem fs = FileSystem.get(uri, configuration, user);
	//调用方法递归查看根目录下的所有目录和文件
    getFileTree("/", fs);
}
```



## 第四章 HDFS数据流

### 4.1 HDFS数据写入过程

#### 4.1.1 文件写入过程

1. 客户端通过HDFS向NameNode发起文件写入请求，NameNode检查目标文件和所属目录是否存在；
2. NameNode返回是否可写入的信息；
3. 客户端对文件进行切割（默认一块128MB），向NameNode询问存储第一个文件块的DataNode节点；
4. NameNode返回三个DataNode节点DN1，DN2，DN3；
5. 客户端通过FSDataOutputStream模块请求向DN1发送数据，DN1收到后连通DN2，DN2连通DN3，形成通信管道；
6. DN1，DN2，DN3逐级应答客户端，等待接收数据；
7. 客户端将第一个文件块进一步拆解为一个个Package（64KB）发送给DN1，DN1每收到一个Package会发送给DN2，DN2再发送给DN3；
8. 第一个文件块全部传输完成后，客户端再次请求上传第二个文件块，重复3-7步，直到整个文件全部写入HDFS。

#### 4.1.2 网络节点距离计算

- 机房
  - 集群D1
    - 机架R1
      - 主机N1
      - 主机N2
      - 主机N3
    - 机架R2
      - 主机N1
      - 主机N2
      - 主机N3
  - 集群D2
    - 机架R1
      - 主机N1
      - 主机N2
      - 主机N3
    - 机架R2
      - 主机N1
      - 主机N2
      - 主机N3
  - 节点距离：两个节点到达最近的共同祖先的距离之和
    - D1.R1.N1 和 D1.R1.N1 的距离为0
    - D1,R1.N1 和 D1.R1.N2 的距离为2
    - D1.R1.N1 和 D1.R2.N1 的距离为4
    - D1.R1.N1 和 D2.R1.N3 的距离为6

#### 4.1.3 机架感知

- 官方说明
  - http://hadoop.apache.org/docs/r3.1.3/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Data_Replication
- 副本节点的选择（3副本方案）
  - 第一个副本在客户端所处的节点上存储，若客户端不在集群中，则随机选择；
  - 第二个副本存储在与第一个副本存储节点不同机架上的随机节点上；
  - 第三个副本存储在第二个副本所处机架的随机节点上。



### 4.2 HDFS数据读取过程

1. 客户端通过HDFS向NameNode请求下载文件，NameNode通过查询元数据，找到文件的所有块所在的DataNode；
2. 根据网络节点的就近原则，客户端从一台DataNode服务器请求读取第一个文件块数据；
3. DataNode传输文件块数据给客户端；
4. 客户端接收后先在本地缓存，再写入目标文件；
5. 客户端集群向NameNode请求下一个文件块的读取，重复第2-4步，直到整个文件读取完成。



## 第五章 NameNode和SecondaryNameNode

### 5.1 NN和2NN的工作机制

<img src="Hadoop.assets/wps1-1586787278225.png" alt="img" style="zoom:150%;" />

1. NameNode启动
   1. 首次启动NameNode格式化后，创建Fsimage和Edits文件。非首次启动时，NameNode直接将Fsimage的镜像文件和编辑日志加载到内存中；
   2. 客户端对元数据进行增删改请求；
   3. NameNode记录操作日志，更新滚动日志（先记录）；
   4. NameNode在内存中对元数据进行增删改操作（再操作）。
2. SecondaryNameNode工作
   1. SecondaryNameNode询问NameNode是否需要建立检查点（CheckPoint），并从NameNode带回询问结果；
   2. SecondaryNameNode请求执行CheckPoint；
   3. NameNode滚动当前的编辑日志（将当前日志交给2NN，创建新的编辑日志）；
   4. NameNode将编辑日志和镜像文件复制给SecondaryNameNode；
   5. SecondaryNameNode将编辑日志和镜像文件进行处理并合并生成fsimage.chkpoint；
   6. SecondaryNameNode将fsimage.chkpoint复制到NameNode并重命名为fsimage替换原有的元数据



### 5.2 Fsimage和Edits解析

- 概念
  - Fsimage文件：HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目录和文件inode的序列化信息；
  - Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写入操作首先会被记录到Edits文件中；
  - 每次NameNode启动的时候都会将Fsimage文件读入内存，同时加载Edits里面的更新操作，保证内存中的元数据信息是最新、同步的，也可以说NameNode在启动时自动的将Fsimage和Edits文件进行了合并。

- oiv查看Fsimage文件

  - 语法

    ```shell
    hdfs oiv -p [文件类型] -i [镜像文件] -o [转换后的输出路径]
    ```

  - 举例

    ```shell
    cd /opt/module/hadoop-3.1.3/data/name/current
    hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml
    cat /opt/module/hadoop-3.1.3/fsimage.xml
    ```

- oev查看Edits文件

  - 语法

    ```shell
    hdfs oev -p [文件类型] -i [编辑日志] -o [转换后的输出路径]
    ```

  - 举例

    ```shell
    cd /opt/module/hadoop-3.1.3/data/name/current
    hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-3.1.3/edits.xml
    cat /opt/module/hadoop-3.1.3/edits.xml
    ```



### 5.3 CheckPoint时间设置

- 默认配置下，SecondaryNameNode每小时自动发起一次CheckPoint，可修改hdfs-site文件配置该参数。

  ```xml
  <!-- 配置自动发起检查点的间隔时间 单位:秒 -->
  <property>
      <name>dfs.namenode.checkpoint.period</name>
      <value>3600</value>
  </property>
  ```

- 默认配置下，SecondarynameNode每隔1分钟会检查一次NameNode距离上一检查点的操作次数，当该次数达到100万次时，自动发起CheckPoint，可修改hdfs-site文件配置该参数

  ```xml
  <!-- 配置自动发起检查点的操作数 -->
  <property>
      <name>dfs.namenode.checkpoint.txns</name>
      <value>1000000</value>
  </property>
  <!-- 配置检查操作数的间隔时间 单位:秒 -->
  <property>
      <name>dfs.namenode.checkpoint.check.period</name>
      <value>60</value>
  </property>
  ```



### 5.4 集群安全模式

- 概述

  1. NameNode启动时，首先将镜像文件载入内存，然后执行编辑日志中的各项操作。完成以上操作后NameNode会创建一个新的Fsimage和一个空的编辑日志，同时开始监听DataNode的请求。**整个过程中，NameNode一直处于安全模式，NameNode的文件系统对于客户端来说是只读的。**
  2. DataNode启动。系统中的数据块的位置并不是由NameNode维护的，而是以块列表的形式存储在DataNode中。在系统正常工作期间，NamaNode会在内存中保留所有块位置的映射信息。在安全模式下，各个DataNode会向NameNode发送最新的块列表信息。
  3. 当满足最小副本条件（NameNode中的所有元数据都匹配到了n个对应的文件块，n默认为1）后，NameNode会在30秒后退出安全模式。

- 语法

  ```shell
  #查看安全模式状态
  hdfs dfsadmin -safemode get
  #进入安全模式状态
  hdfs dfsadmin -safemode enter
  #离开安全模式状态
  hdfs dfsadmin -safemode leave
  #等待安全模式状态
  hdfs dfsadmin -safemode wait
  ```



### 5.5 NameNode多目录配置

- 概念：NameNode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性

- 配置过程

  1. 修改hdfs-site.xml配置文件

     ```xml
     <!-- 多个路径使用,连接 -->
     <property>
         <name>dfs.namenode.name.dir</name>
         <value>file:///${hadoop.data.dir}/name1,file:///${hadoop.data.dir}/name2</value>
     </property>
     ```

  2. 分发配置文件

  3. 停止集群并重新格式化

     ```shell
     #删除所有节点的hadoop安装目录下的data和logs数据
     rm -rf data/ logs/
     
     #格式化集群
     hdfs namenode -format
     
     #重新启动集群
     satrt-dfs.sh
     ```



## 第六章 DataNode

### 6.1 DataNode工作机制

- 一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，另一个是元数据（包括数据块的长度，块数据的校验和CheckSum、时间戳）
- DataNode启动后主动向NameNode进行注册，通过注册后，周期性的（默认1小时）向NameNode上报所有的块信息
- DataNode默认3秒心跳一次，心跳会向DataNode发送一个信号，该信号返回时会携带NameNode交给DataNode的命令（复制、删除等操作）。默认情况下，当NameNode超过10分30秒没有收到某个DataNode的心跳，则认为该节点不可用。
- 集群在运行过程中可以安全的服役退役新老节点。



### 6.2 数据完整性

1. 当DataNode读取Block时，会计算ChcekSum；
2. 若计算后的CheckSum和创建时的值不一样，说明Block已经损坏；
3. 若数据损坏，Client会读取其他DataNode上的Block；
4. 常见的校验算法 （默认）crc（32） md5（128） shal（160）；
5. DataNode在其文件创建后周期验证CheckSum。



### 6.3 掉线时限

- 当NameNode长时间未收到来自DataNode的通信时会判定该DataNode死亡，停止该节点的使用

- 判定时间计算公式

  ```java 
  TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval
  // dfs.namenode.heartbeat.recheck-interval--心跳检查时间
  // dfs.heartbeat.interval--心跳间隔时间
  ```

- 可通过修改hdfs-site.xml配置文件修改掉线时限相关参数

  ```xml
  <!-- 心跳检查时间 单位:毫秒 -->
  <property>
      <name>dfs.namenode.heartbeat.recheck-interval</name>
      <value>300000</value>
  </property>
  <!-- 心跳间隔时间 单位:秒 一般不做修改 -->
  <property>
      <name>dfs.heartbeat.interval</name>
      <value>3</value>
  </property>
  ```



### 6.4 服役新数据节点

1. 在新节点上配置hadoop及java环境，并修改IP和主机名称

2. 修改etc/hadoop下的各个site配置文件

3. 确认hadoop根目录下不含data及logs目录

4. 启动DataNode，即可自动关联到集群

   ```shell
   hdfs --daemon start datanode
   yarn --daemon start nodemanager
   ```

5. 修改workers配置文件，添加新主机，用于集群的群体操作

   ```shell
   vi vi /opt/module/hadoop-3.1.3/etc/hadoop/workers
   
   #在配置文件中追加集群的所有主机并保存
   hadoop100
   hadoop101
   hadoop102
   hadoop103
   
   #将配置文件同步至所有节点
   xsync  workers
   ```

6. 若数据不均衡，可使用命令实现集群的再平衡

   ```shell
   sbin/start-balancer.sh
   ```



### 6.5 退役旧数据节点

#### 6.5.1 白名单与黑名单

- 白名单：白名单中的节点都允许访问NameNode，白名单中删除的节点会直接退出集群。

- 黑名单：黑名单中的节点都不允许访问NameNode，黑名单中添加的节点会在数据迁移后退出集群。

- 配置过程

  1. 编写白名单、黑名单文件

     ```shell
     #白名单
     vi /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     #添加主机名
     hadoop100
     hadoop101
     hadoop102
     
     #黑名单
     vi /opt/module/hadoop-3.1.3/etc/hadoop/blacklist
     #暂时为空
     ```

  2. 修改hdfs-site.xml配置文件

     ```xml
     vi /opt/module/hadoop-3.1.3/etc/hadoop/hdfs-site.xml
     
     <!-- 添加以下配置 -->
     <!-- 白名单路径 -->
     <property>
         <name>dfs.hosts</name>
         <value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
     </property>
     <!-- 黑名单路径 -->
     <property>
         <name>dfs.hosts.exclude</name>
         <value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
     </property>
     ```
   ```
  
   ```
3. 分发配置文件
  
  4. 重启集群

#### 6.5.2 黑名单退役

1. 编辑blacklist

   ```shell
   vi /opt/module/hadoop-3.1.3/etc/hadoop/blacklist
   #添加退役主机
   hadoop103
   ```

2. 刷新hdfs和yarn

   ```shell
   hdfs dfsadmin -refreshNodes
   yarn dfsadmin -refreshNodes
   ```

   

#### 6.5.2 白名单退役

- 白名单退役过程类似黑名单退役的操作，但不推荐使用，因为数据不会迁移，会造成数据的丢失。



### 6.6 DataNode多目录配置

- DataNode也可以配置为多个目录，但不用于NameNode，DataNode的多个目录中存储的数据各不相同

- 配置过程

  1. 修改hdfs-site.xml配置文件

     ``` xml
     <!-- 修改以下配置,多目录间使用,连接 -->
     <property>
         <name>dfs.datanode.data.dir</name>
         <value>file:///${hadoop.data.dir}/data1,file:///${hadoop.data.dir}/data2</value>
     </property>
     ```

  2. 分发配置文件

  3. 停止集群并重新格式化

     ```shell
     #删除所有节点的hadoop安装目录下的data和logs数据
     rm -rf data/ logs/
     
     #格式化集群
     hdfs namenode -format
     
     #重新启动集群
     satrt-dfs.sh
     ```



## 第七章 小文件存档

- HDFS的弊端：存储小文件

  - 每个文件均按块存储，每个块的元数据又存储在NameNode的内存中，因此大量的小文件会耗尽NameNode的内存，导致HDFS存储小文件效率很差。
  - 解决方法：使用HAR进行文件存档，可以将文件存入HDFS块。简单的说，HDFS存档文件实际还是一个一个的独立文件，但对NameNode而言却是一个整体。

- 举例

  1. 将多个小文件归档

     ```shell
     #将input目录下的所有小文件归档到output中
     bin/hadoop archive -archiveName input.har –p /user/atguigu/input /user/atguigu/output
     ```

  2. 查看归档后的文件

     ```shell
     #使用har://+文件路径查看归档文件
     hadoop fs -ls har:///user/atguigu/output/input.har
     ```

  3. 解除归档

     ```shell
     hadoop fs -cp har:/// user/atguigu/output/input.har/* /user/atguigu
     ```



## 第八章 纠删码

### 8.1 查看当前支持的纠删码策略

- 查看

  ```shell
  hdfs ec -listPolicies
  ```

- 策略介绍

  - RS-10-4-1024k：使用RS编码，每10个数据单元（cell），生成4个校验单元，共14个单元，也就是说：这14个单元中，只要有任意的10个单元存在（不管是数据单元还	是校验单元，只要总数=10），就可以得到原始数据。每个单元的大小是1024k。
  - RS-3-2-1024k：使用RS编码，每3个数据单元，生成2个校验单元，共5个单元，也就是说：这5个单元中，只要有任意的3个单元存在（不管是数据单元还是校验单元，只要总数=3），就可以得到原始数据。每个单元的大小是1024k。
  - RS-6-3-1024k：使用RS编码，每6个数据单元，生成3个校验单元，共9个单元，也就是说：这9个单元中，只要有任意的6个单元存在（不管是数据单元还是校验单元，只要总数=6），就可以得到原始数据。每个单元的大小是1024k。
  - RS-LEGACY-6-3-1024k：策略和RS-6-3-1024k一样，只是编码的算法用的是rs-legacy。
  - XOR-2-1-1024k：使用XOR编码（速度比RS编码快），生成2个数据单元，1个校验单元，共3个单元，也就是说：这3个单元中，只要有任意的2个单元存在（不管是数据单元还是校验单元，只要总数=2），就可以得到原始数据。每个单元的大小是1024k。

### 8.2 设置纠删码策略

- 纠删码策略的作用对象为一个具体的路径，在该路径下存储的文件都会使用该策略进行存储。

- 操作步骤

  1. 开启hdfs对需要使用的纠删码策略的支持

     ```shell
     #开启RS-3-2-1024k纠删码策略
     hdfs ec -enablePolicy -policy RS-3-2-1024k
     ```

  2. 创建目录，并配置纠删码策略

     ```shell
     hdfs dfs -mkdir /input
     hdfs ec -setPolicy -path /input -policy RS-3-2-1024k
     ```

  3. 查看目录的纠删码策略

     ``` shell
     hdfs ec -getPolicy -path /input
     ```



# MapReduce

## 第一章 MapReduce概述

### 1.1 MapReduce定义

- MapReduce是一个**分布式运算程序的编程框架**，是用户开发“基于Hadoop的数据分析应用”的核心框架。
- MapReduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序。



### 1.2 MapReduce的优缺点

- 优点
  1. MapReduce易于编程：通过实现一些简单的接口就可以完成一个分布式程序。
  2. 扩展性好：当计算资源不足时，可以直接增加机器扩展计算能力。
  3. 高容错性：当某一节点故障时，相关的计算任务会自动转移到另一节点上运行，直到完成。
  4. 适合PB级及以上的海量数据的离线处理。
- 缺点
  1. 不擅长实时计算：无法在毫秒或秒级返回结果。
  2. 不擅长流式计算：输入的数据集是静态的，不能动态变化。
  3. 不擅长DAG（有向图）计算：对于存在依赖关系的多个作业，每个MapReduce作业都会将输出结果写入磁盘，从而产生大量的磁盘IO，导致性能低下。



### 1.3 MapReduce核心思想

![img](Hadoop.assets/wps1-1586874569338.png)

- 分布式运算程序分为至少两个阶段
  - 第一阶段MapTask并发实例，并行运行，互不干扰
  - 第二阶段ReduceTask并发实例，并行运行，运行依赖于上一阶段MapTask的输出结果
- MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段。若业务逻辑复杂，则需要多个MapReduce程序串行运行。



### 1.4 MapReduce进程

- 完整的MapReduce程序在分布式运行时有三类实例进程
  1. MrAppMaster：负责程序的过程调度和状态协调
  2. MapTask：负责Map阶段整个数据的处理流程
  3. ReduceTask：负责Reduce阶段整个数据的处理流程



### 1.5 WordCount源码

官方的WordCount案例包含Mapper类、Reducer类和Driver类。数据类型为Hadoop封装的序列化类型。



### 1.6 常用数据序列化类型

| Java类型 | Hadoop Writable类型 |
| -------- | ------------------- |
| boolean  | BooleanWritable     |
| byte     | ByteWritable        |
| int      | IntWritable         |
| float    | FloatWritable       |
| long     | LongWritable        |
| double   | DoubleWritable      |
| Sting    | Text                |
| Map      | MapWritable         |
| Array    | ArrayWritable       |



### 1.7 MapReduce编程规范

1. Mapper阶段
   - 自定义类继承Mapper类
   - Mapper的输入数据为KV对
   - Mapper的业务逻辑重写在方法map()中
   - Mapper的输出数据为KV对
   - **map()方法对每一个KV对都调用一次**
2. Reducer阶段
   - 自定义类继承Reducer类
   - Reducer的输入数据与Mapper中的输出数据类型一致，也是KV对
   - Reducerd的业务逻辑重写在reduce()中
   - **reduce()方法对每一组K值相同的KV对调用一次**
3. Driver阶段
   - 相当于客户端，用于将程序提交到Yarn集群，提交的具体内容为封装了MapReduce程序相关运行参数的job对象



### 1.8 WordCount编写案例

1. 创建Maven工程

2. 在pom.xml文件中添加以下依赖

   ```xml
   <dependencies>
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
           <artifactId>hadoop-client</artifactId>
           <version>3.1.3</version>
       </dependency>
   </dependencies>
   <!-- 以下为打包插件,根据Hadoop环境是否包含以上jar包选择添加 -->
   <build>
       <plugins>
           <plugin>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>2.3.2</version>
               <configuration>
                   <source>1.8</source>
                   <target>1.8</target>
               </configuration>
           </plugin>
           <plugin>
               <artifactId>maven-assembly-plugin </artifactId>
               <configuration>
                   <descriptorRefs>
                       <descriptorRef>jar-with-dependencies</descriptorRef>
                   </descriptorRefs>
                   <archive>
                       <manifest>
                           <!-- 该位置配置为工程Driver类 -->
                           <mainClass>com.atguigu.mr.WordcountDriver</mainClass>
                       </manifest>
                   </archive>
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

3. 在src/main/resources下新建log4j2.xml文件，并添加以下内容

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

4. 编写Mapper类

   ```java
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   import java.io.IOException;
   
   /**
    * 自定义继承Mapper类
    * 泛型参数说明
    * 前两位为map()方法传入的kv对的数据类型
    * 后两位为map()方法写出的kv对的数据类型
    */
   public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
   
       /**
        * 重写Mapper类中的map()方法
        * @param key 表示输入的k,此处为记录上次读取到的文件位置的游标
        * @param value 表示输入的v,此处为读取的文件的一行内容
        * @param context 负责Mapper调度运行
        */
       @Override
       protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
           //1.声明写出的kv对,数据类型与泛型后两位一致
           Text k = new Text();
           IntWritable v = new IntWritable();
           //2.使用指定的分隔符拆分每行字符串,得到拆分后的字符串数组
           String line = value.toString();
           String[] words = line.split(" ");
           //3.为kv对赋值并使用context写出
           for (String word : words) {
               k.set(word);
               v.set(1);
               context.write(k, v);
           }
       }
   }
   ```

5. 编写Reducer类

   ```java
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   import java.io.IOException;
   
   /**
    * 自定义类继承Reducer类
    * 泛型参数说明
    * 前两位表示reduce()方法传入参数的数据类型,该类型必须与Mapper中map()方法写出的数据类型一致
    * 后两位表示reduce()方法传入参数的数据类型
    */
   public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
   
       /**
        * 重写Reducer中的Reduce()方法
        * @param key 表示key，此处为map()中拆分后的一个单词
        * @param values 一个迭代器对象，此处的迭代器包含了map()中拆分后的一个单词对应的所有kv对
        * @param context 负责Reducer的调度运行
        */
       @Override
       protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
           //1.声明写出的kv对,数据类型与泛型后两位一致
           IntWritable v = new IntWritable();
           //2.使用迭代器统计一个单词的出现次数
           int sum = 0;
           for (IntWritable value : values) {
               sum += value.get();
           }
           //3.为kv对赋值并使用context写出
           v.set(sum);
           context.write(key, v);
       }
   }
   ```

6. 编写Driver类

   ```java
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   import java.io.IOException;
   
   public class WordCountDriver {
       public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
           //1.创建job对象
           Configuration conf = new Configuration();
           Job job = Job.getInstance(conf);
           //2.关联jar
           job.setJarByClass(WordCountDriver.class);
           //3.关联Mapper和Reducer类
           job.setMapperClass(WordCountMapper.class);
           job.setReducerClass(WordCountReducer.class);
           //4.设置Mapper的输出key和value的类型
           job.setMapOutputKeyClass(Text.class);
           job.setMapOutputValueClass(IntWritable.class);
           //5.设置最终输出的key和value的类型
           job.setOutputKeyClass(Text.class);
           job.setOutputValueClass(IntWritable.class);
           //6.设置输入和输出的路径
           FileInputFormat.setInputPaths(job, new Path(args[0]));
           FileOutputFormat.setOutputPath(job, new Path(args[1]));
           //7.提交job
           job.waitForCompletion(true);
       }
   }
   ```

7. 对Maven工程打包，上传至Linux

   ```
   Maven -> lifecycle -> package
   ```

8. 启动集群，执行WordCount程序

   ```shell
   hadoop jar  wordcount.jar
    com.atguigu.wordcount.WordcountDriver /user/atguigu/input /user/atguigu/output
   ```



## 第二章 Hadoop序列化

### 2.1 序列化概述

- 概念
  - 序列化：把内存中的对象，转换成字节序列（或其他数据传输协议）以便于持久化的存储到磁盘和网络传输。
  - 反序列化：把字节序列（或其他数据传输协议）、磁盘的持久化数据，转换成内存中的对象。
  - **在Hadoop中主要用于将Map阶段写出的对象信息序列化写入磁盘，再在Reduce阶段通过反序列化读取。**
- Java序列化：Java序列化是一个重量级序列化框架，一个对象被序列化后，会附带很多额外信息，不便于在网络中高效传输。因此，Hadoop开发了一套自己的序列化机制。
- Hadoop序列化
  - 紧凑：高效利用存储空间
  - 快速：读写数据的额外开销较小
  - 可扩展：随通信协议升级而升级
  - 互操作：支持多语言交互



### 2.2 自定义bean对象实现序列化接口（Writable）

```java
import org.apache.hadoop.io.Writable;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

//1.自定义的Bean需要继承可序列化接口Writable
public class FlowBean implements Writable {
    
    //2.声明需要序列化的属性
    private Long upFlow;
    private Long downFlow;
    private Long sumFlow;
    
    //3.必须提供无参构造器
    public FlowBean() {
    }
    
    //4.提供各属性的get/set方法
    public Long getUpFlow() {
        return upFlow;
    }
    public void setUpFlow(Long upFlow) {
        this.upFlow = upFlow;
    }
    public Long getDownFlow() {
        return downFlow;
    }
    public void setDownFlow(Long downFlow) {
        this.downFlow = downFlow;
    }
    public Long getSumFlow() {
        return sumFlow;
    }
    public void setSumFlow(Long sumFlow) {
        this.sumFlow = sumFlow;
    }

    //5.重写toString()方法,指定最后写出呈现的格式,用于reduece()输出时调用
    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }

    //6.重写write()和readFields()方法,分别用于序列化和反序列化对象的属性
    public void write(DataOutput dataOutput) throws IOException {//序列化
        //属性的序列化顺序可任意指定,但必须与反序列化中的顺序一一对应
        dataOutput.writeLong(upFlow);
        dataOutput.writeLong(downFlow);
        dataOutput.writeLong(sumFlow);
    }
    public void readFields(DataInput dataInput) throws IOException {//反序列化
        this.upFlow = dataInput.readLong();
        this.downFlow = dataInput.readLong();
        this.sumFlow = dataInput.readLong();
    }
}
```



### 2.3 SumFlow案例编写

- 需求：按照手机号码统计使用的总上传流量、总下载流量和总流量。数据信息如下

  ```
  1	13736230513	192.196.100.1	www.atguigu.com	2481	24681	200
  2	13846544121	192.196.100.2			264	0	200
  3 	13956435636	192.196.100.3			132	1512	200
  4 	13966251146	192.168.100.1			240	0	404
  5 	18271575951	192.168.100.2	www.atguigu.com	1527	2106	200
  6 	84188413	192.168.100.3	www.atguigu.com	4116	1432	200
  7 	13590439668	192.168.100.4			1116	954	200
  8 	15910133277	192.168.100.5	www.hao123.com	3156	2936	200
  9 	13729199489	192.168.100.6			240	0	200
  10 	13630577991	192.168.100.7	www.shouhu.com	6960	690	200
  11 	15043685818	192.168.100.8	www.baidu.com	3659	3538	200
  12 	15959002129	192.168.100.9	www.atguigu.com	1938	180	500
  13 	13560439638	192.168.100.10			918	4938	200
  14 	13470253144	192.168.100.11			180	180	200
  15 	13682846555	192.168.100.12	www.qq.com	1938	2910	200
  16 	13992314666	192.168.100.13	www.gaga.com	3008	3720	200
  17 	13509468723	192.168.100.14	www.qinghua.com	7335	110349	404
  18 	18390173782	192.168.100.15	www.sogou.com	9531	2412	200
  19 	13975057813	192.168.100.16	www.baidu.com	11058	48243	200
  20 	13768778790	192.168.100.17			120	120	200
  21 	13568436656	192.168.100.18	www.alibaba.com	2481	24681	200
  22 	13568436656	192.168.100.19			1116	954	200
  ```

- 步骤

  1. 新建Maven工程，修改pom.xml文件，新建log4j2.xml文件

  2. 新建FlowBean类，封装上传流量、下载流量、总流量三个属性（见2.2）

  3. 编写Mapper类

     ```java
     import org.apache.hadoop.io.LongWritable;
     import org.apache.hadoop.io.Text;
     import org.apache.hadoop.mapreduce.Mapper;
     import java.io.IOException;
     
     public class FlowMapper extends Mapper<LongWritable, Text, Text, FlowBean> {
     
         @Override
         protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
             //声明写出的kv对对象
             Text k = new Text();
             FlowBean v = new FlowBean();
             //拆分一行字符串
             String line = value.toString();
             String[] split = line.split("\t");
             //抓取目标字段数据，赋给kv对
             k.set(split[1]);
             v.setUpFlow(Long.parseLong(split[split.length - 3]));
             v.setDownFlow(Long.parseLong(split[split.length - 2]));
             v.setSumFlow(v.getUpFlow() + v.getDownFlow());
             //写出kv对
             context.write(k, v);
         }
         
     }
     ```

  4. 编写Reducer类

     ```java
     import org.apache.hadoop.io.Text;
     import org.apache.hadoop.mapreduce.Reducer;
     import java.io.IOException;
     
     public class FlowReducer extends Reducer<Text, FlowBean, Text, FlowBean> {
         @Override
         protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {
             //声明写出的kv对对象
             FlowBean v = new FlowBean();
             //计算总上传流量、总下载流量
             long sumUpFlow = 0;
             long sumDownFlow = 0;
             for (FlowBean value : values) {
                 sumUpFlow += value.getUpFlow();
                 sumDownFlow += value.getDownFlow();
             }
             //将计算结果赋值给kv对
             v.setUpFlow(sumUpFlow);
             v.setDownFlow(sumDownFlow);
             v.setSumFlow(v.getUpFlow() + v.getDownFlow());
             //写出kv对
             context.write(key, v);
         }
         
     }
     ```

  5. 编写Driver类

     ```java
     import org.apache.hadoop.conf.Configuration;
     import org.apache.hadoop.fs.Path;
     import org.apache.hadoop.io.Text;
     import org.apache.hadoop.mapreduce.Job;
     import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
     import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
     import java.io.IOException;
     
     public class FlowDriver {
         public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
             //1.创建job对象
             Configuration configuration = new Configuration();
             Job job = Job.getInstance(configuration);
             //2.关联jar
             job.setJarByClass(FlowDriver.class);
             //3.关联Mapper和Reducer类
             job.setMapperClass(FlowMapper.class);
             job.setReducerClass(FlowReducer.class);
             //4.设置Mapper的输出key和value的类型
             job.setMapOutputKeyClass(Text.class);
             job.setMapOutputValueClass(FlowBean.class);
             //5.设置最终输出的key和value的类型
             job.setOutputKeyClass(Text.class);
             job.setOutputValueClass(FlowBean.class);
             //6.设置输入和输出的路径
             FileInputFormat.setInputPaths(job, new Path(args[0]));
             FileOutputFormat.setOutputPath(job, new Path(args[1]));
             //7.提交job
             job.waitForCompletion(true);
         }
     }
     ```

  6. 打包Maven工程，上传至Linux后执行

     ```shell
     hadoop jar  sumflow.jar
      com.atguigu.sumflow.FlowDriver /user/atguigu/input /user/atguigu/output
     ```



## 第三章 MapReduce框架原理

### 3.1 InputFormat数据输入

#### 3.1.1 切片与MapTask并行度

- Map阶段会并发多个MapTask，MapTask的数量（并行度）取决于数据切片的数量，每有一个数据切片就分配一个MapTask进行处理。
- 数据块与数据切片
  - 数据块：HDFS在物理上对一个文件的存储进行了切分，默认大小128M
  - 数据切片：Map阶段对文件在逻辑上进行切分，只记录切片的元数据信息。



#### 3.1.2 Job提交流程源码和切片过程

- job提交流程

```java
waitForCompletion();
submit();

// 1.建立连接
	connect();	
		//创建提交Job的代理
		new Cluster(getConfiguration());
			//判断是本地yarn还是远程
			initialize(jobTrackAddr, conf); 

// 2.提交job
submitter.submitJobInternal(Job.this, cluster)
	// 1)创建给集群提交数据的Stag路径
	Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);

	// 2)获取jobid ，并创建Job路径
	JobID jobId = submitClient.getNewJobID();

	// 3)拷贝jar包到集群
	copyAndConfigureFiles(job, submitJobDir);	
	rUploader.uploadFiles(job, jobSubmitDir);

	// 4)计算切片，生成切片规划文件
	writeSplits(job, submitJobDir);
	maps = writeNewSplits(job, jobSubmitDir);
	input.getSplits(job);

	// 5)向Stag路径写XML配置文件
	writeConf(conf, submitJobFile);
	conf.writeXml(out);

	// 6)提交Job,返回提交状态
	status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
```

- FileInputFormat切片过程
  1. 程序寻找存储数据的目录
  2. 遍历目录下每一个需要处理的文件
  3. 对于每一个文件
     1. 获取文件大小
     2. 计算切片大小，默认大小等于文件块大小
     3. 开始切片，并将切片的元数据信息（起始位置、长度、所在节点）写入切片规划文件
  4. 将切片规划文件提交给Yarn的MrAppMaster，再开启与切片数相应的MapTask



#### 3.1.3 FileInputFormat切片规则

- 切片方式：按照文件内容长度切片

- 切片大小：源码中有计算切片大小的公式，默认与文件块大小一致

  ```java
  Math.max(minSize, Math.min(maxSize, blockSize));
  
  //mapreduce.input.fileinputformat.split.minSize
  //默认值为1,参数调整后当blocksSize<minSize时,以minSize的大小进行切分,单位为字节
  
  //mapreduce.input.fileinputformat.split.maxSize
  //默认值为Long.MAXValue,参数调整后当blocksSize>maxSize时,以maxSize的大小进行切分,单位为字节
  
  //需要注意的是 每次切片后都要计算余下的部分是否大于本次切片的1.1倍,若不大于则将这不足1.1倍的数据切为1片,这样可以少启动1个MapTask,节省资源
  ```

- 切片时不考虑整个数据集，而是针对每一个文件单独做切片处理（即默认一个切片上不会含有两个数据文件的内容）



#### 3.1.3 FileInputFormat实现类

- 继承关系

  ```java
  InputFormat//负责Map端数据输入的抽象类
  	|--FileInputFormat//抽象类 驶向了getSplits()方法,提供了默认切片规则
      	|--TextInputFormat//haoop默认使用的实现类,按行读取数据
      	|--CombineFileInputFormat
      	|--NLineInputFormat
      	|--KeyValueInputFormat
  ```



- TextInputFormat类

  - key：相对于整个文件起始位置的字节偏移量（即每行的起始位置），类型为LongWritable
  - value：一行的具体内容，类型为Text

  

- CombineFileInputFormat类

  - 应用场景：用于处理大量小文件，将多个小文件从逻辑上规划到一个切片中，交给一个MapTask处理

  - kv对数据类型同TextInputFormat类

  - 切片机制

    1. 设置setMaxInputSplitSize的值（单位为字节），决定切片大小判定基准
    2. 开始虚拟存储过程，比较小文件与基准值的大小
       - 若文件大小<基准值，则直接将该文件虚拟存储为1个文件
       - 若文件大小>基准值且文件大小<2*基准值，则将该文件均分后虚拟存储为2个文件
       - 若文件大小>2*基准值，则先虚拟存储一个等于基准值大小的文件，再判定剩余部分的大小，重复这一过程，最终虚拟存储为n个文件
    3. 开始切片过程，将虚拟存储的文件按依次合并，每合并一次，比较一次合并后的文件与基准值的大小
       - 若合并后的文件大小>=基准值，则将该文件作为1个逻辑切片，交给1个MapTask处理
       - 若合并后的文件大小<基准值，则继续合并下一个文件，直到>=基准值时，生成1个逻辑切片，交给1个MapTask处理

  - 举例

    ```java
    //由于默认使用TextInputFormat提供的切片规则,需要再Driver类中手动进行设置
    //1.设置job对象的数据输入实现类
    job.setInputFormatClass(CombineTextInputFormat.class);
    //2.设置切片基准值,此处为4M
    ConbineTextInputFormat.setMaxInputSplitSize(job,4194304);
    ```

  

- NLineInputFormat类

  - 应用场景：不再使用默认的块大小，而是按行数对数据文件进行切片

  - kv对数据类型同TextInputFormat类

  - 切片机制

    1. 设置切片行数
    2. 读取文件，每读取够指定的行数就生成1个切片，读到末尾若行数不足则也单独生成1个切片

  - 举例

    ```java
    //在Driver类中添加如下设置
    //设置每个切片包含3行数据
    NLineInputFormat.setNumLinesPerSplit(job, 3);
    //设置job对象的数据输入实现类
    job.setInputFormatClass(NLineInputFormat.class);  
    ```

    

- KeyValueInputFormat类

  - 应用场景：将数据文件每行开头字段作为key值处理的任务

  - key与value

    - key：根据指定分隔符切割后的的每行的第一个分隔字段
    - value：每行除去key和第一个分隔符后的内容

  - 举例

    ```java
    //在Driver类中添加如下设置
    
    //1.获取job对象前设置分隔符" "
    Configuration conf = new Configuration();
    conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, " ");
    Job job = Job.getInstance(conf);
    //2.配置job对象的数据输入实现类
    job.setInputFormatClass(KeyValueTextInputFormat.class);
    ```



### 3.2 Shuffle洗牌机制

#### 3.2.1 Shuflle机制图解

- Shuffle指map方法后，reduce方法前，hadoop对数据进行的一系列处理过程。

<img src="Hadoop.assets/wps1-1587120217813.png" alt="img" style="zoom:150%;" />

1. 在数据输入阶段，hadoop为每一个数据切片启动一个相应的MapTask，在经过map()方法处理后，输出了一系列kv对组成的结果，这时MapReduce计算进入Shuffle阶段。
2. 首先，kv对依次存入内存的环形缓冲区（默认100M），每当缓冲区占用达80%（80M）时，这部分数据会在内存中根据分区规则进行**分区**，再在各个分区内**排序**（对key值快速排序），最后写入磁盘中，生成一个带有分区且分区内有序的溢写文件。
3. （可选步骤）每个溢写文件在写入磁盘前，可根据自定义的Combiner规则对该分区的数据进行合并，目的是将ReduceTask的任务分担给MapTask，以减小ReduceTask的作业压力。
4. 1个MapTask生成的全部溢写文件在落盘完成后，提取每个溢写文件中同一分区的数据合并，然后进行**排序**（对key值做归并排序），再次落盘，生成一个带有分区且分区内有序的大文件，至此该MapTask工作结束。
5. （可选步骤）同样的，在分区数据合并时，也可以自定义的Combiner规则对该分区的数据进行合并。
6. 进入ReduceTask工作阶段，由于1个ReduceTask处理1个分区的数据，每个ReduceTask从所有MapTask写出的大文件中依次检索并拉取属于自己处理的分区的数据，缓存到内存中。
7. ReduceTask将内存中属于自己分区的数据再次进行合并并整体**排序**（对key值做归并排序），然后写入磁盘，得到1个文件。
8. （可选步骤）ReduceTask默认会为每一个key值执行一次reduce()方法，当我们有另外的需求（比如为有类似特征的一组key值共用一个reduce方法）时，可以通过GroupingComparator分组实现。
9. Shuffle阶段到此结束，ReduceTask开始数据的逻辑运算过程。



#### 3.2.2 Partition分区

- 需求：当我们需要根据一定规则区分数据，将MapReduce计算后的结果输出到不同的目标文件时，就要使用分区功能。

- 分区的两个主要参数

  1. 分区数：直接决定了Reduce阶段启动的ReduceTask个数，1个分区对应1个ReduceTask，而分区数本身则通常由我们实际的需求决定。
  2. 分区规则
     - 默认规则：hadoop通过计算key的hashCode对ReduceTask个数（也就是分区数）取模的结果将不同的kv对输出到不同的结果文件中，这样的输出结果不可控。
     - 自定义规则：通过继承Partitiner类并重写类中的getPartition()方法自定义分区规则，使分区结果符合需求。

- 具体步骤

  1. 自定义类继承Partitioner类，重写getPartition()方法

     ```java
     import org.apache.hadoop.mapreduce.Partitioner;
     
     //泛型参数为map()方法中输出的kv对类型
     public class TestPartitioner extends Partitioner<Object, Object> {
         // 形参列表依次为输出的kv对类型,分区数
         @Override
         public int getPartition(Object o, Object o2, int i) {
             //分区逻辑代码
             if (){
                 ...
             	//返回分区号,分区号从0开始
             	return 0; 
             }else {
                 ...
                 return 1;
             }
             
         }
     }
     ```

  2. 在Driver类中配置自定义类和分区数

     ```java
     job.setPartitionerClass(TestPartitioner.class);
     job.setNumReduceTask(5);
     //若设置ReduceTask=0,则没有Reduce阶段,最后输出MapTask个数的结果文件
     //若设置ReduceTask=1,则不管自定义类中划分多少个分区,最后都只输出1个结果文件
     //若设置1 < ReduceTask < getPartition的分区结果数,则会导致数据无输出目标文件,报Exception
     //若设置ReduceTask > getPartition的分区结果数,则生成几个空结果文件
     ```

- 实例

  1. 使用2.3中的手机号-流量数据模型，需求为按照手机号开头三位分区输出结果文件

  2. 编写Mapper、Reducer、Driver类

  3. 编写继承Partitioner的自定义类

     ```java
     public class PhoneNumPartitioner extends Partitioner<Text, FlowBean> {
         public int getPartition(Text text, FlowBean flowBean, int i) {
             String phoneNum = text.toString();
             int partNum = 0;
             if (phoneNum.startsWith("136")){
                 partNum = 0;
             }else if (phoneNum.startsWith("138")){
                 partNum = 1;
             }else {
                 partNum = 2;
             }
             return partNum;
         }
     }
     ```

  4. 在Driver类中添加分区配置语句

     ```java
     job.setPartitionerClass(PhoneNumPartitioner.class);
     job.setNumReduceTasks(3);//匹配实际的分区数
     ```

     

#### 3.2.3 WritableComparable可排序接口

- 需求：对于MapReduce阶段的每一次数据的写出和合并，Hadoop都会调用比较器针对key值进行排序，这是Hadoop的默认行为。这就需要保证**key值必须是可比较、可排序的**。Hadoop提供的常用数据类型（如Text）均是可以排序的，这是因为它们都实现了WritableComparable接口。因此当我们使用自定义类作为key的数据类型时，自定义类也必须实现这一接口，并重写CompareTo()方法，给出指定的排序规则。

- WritableComparable接口继承自Writable, Comparable两个接口，因此在使用时还需要重写write()和readFiles()方法

- 具体步骤

  ```java
  //作为key的自定义类实现WritableComparable接口
  //泛型参数为该自定义类本身
  public class FlowBean implements WritableComparable<FlowBean> {
  
  	//省略代码
      //...
  
  	@Override
  	public int compareTo(FlowBean bean) {
  		//比较逻辑,此处为按照sumFlow升序排列
  		return sumFlow - bean.getSumFlow();
  	}
  }
  ```

  

#### 3.2.4 Combiner合并

- 需求：在整个MapReduce过程中，MapTask阶段主要承担了数据的拆分、分区和排序工作，而计算的部分集中在ReduceTask阶段（如WordCount案例），这样会造成ReduceTask阶段的作业压力和数据的传输量过大。为了分担ReduceTask的工作，我们可以通过Combiner让MapTask对数据进行预处理。简单来说，就是用MapTask对局部的数据进行本应在ReduceTask中进行的处理工作。

- Combiner的实现过程

  1. 自定义类，继承Reducer类，实现reduce()方法
  2. 在Driver类中设置Combiner
  3. 注意：Combiner使用的前提是不能改变最终的业务逻辑，且业务必须包含reduce阶段（否则Combiner没有意义）

- 举例：WordCount案例

  1. 编写Mapper、Reducer、Driver类

  2. 编写自定义类，继承Reducer类

     ```java
     //方式1
     public class WordCombiner extends Reducer<Text, IntWritable, Text, IntWritable> {
         @Override
         protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
             int sum = 0;
             for (IntWritable value : values) {
                 sum += value.get();
             }
             context.write(key, new IntWritable(sum));
         }
     }
     ```

  3. 在Driver类中设置Combiner

     ```java
     //方式1
     job.setCombinerClass(WordCombiner.class);
     //方式2:
     //由于在该案例中,Combiner和Reducer的业务逻辑完全一致,可以跳过步骤2,直接使用Reducer类作为Combiner
     job.setCombinerClass(WordReducer.class);
     ```



#### 3.2.5 GroupingComparator分组比较器

- 需求：在reduce()方法执行前，hadoop会调用分组比较器针对key值对数据进行分组，将同一组数据的value装入迭代器中，再传入reduce()方法，即一组数据调用一次reduce()方法。**默认的分组比较器的实现方式为调用key所在类的compareTo()方法，比较结果相等的kv对被分到同一组。**当我们需要的另外的分组规则时，就需要自定义分组排序。

- 具体步骤

  1. 自定义类继承GroupingComparator接口，提供构造器，实现compare()方法
  2. 在Driver类中设置GroupingComparator分组比较器

- 举例：处理订单信息，按照订单号分组

  1. 自定义GroupingComparator接口实现类

     ```java
     public class OrderGroupingComparator extends WritableComparator {
         //提供构造器,传入key所属类的class对象
     	protected OrderGroupingComparator() {
     		super(OrderBean.class, true);
     	}
         //重写compare()方法,
     	@Override
     	public int compare(WritableComparable a, WritableComparable b) {
     		OrderBean aBean = (OrderBean) a;
     		OrderBean bBean = (OrderBean) b;
     		return aBean.getOrder_id().compareTo(bBean.getOrder_id());
     	}
     }
     ```

  2. 在Driver中设置分组比较器

     ```java
     job.setGroupingComparatorClass(OrderGroupingComparator.class)
     ```



### 3.3 OutputFormat数据输出

#### 3.3.1 OutputFormat接口

- OutputFormat是MapReduce的输出基类，所有MapReduce的输出类都实现了该接口。
  - TextOutputFormat类：默认的输出实现类，**将每条记录写为文本行**，输出的kv对会自动的调用所在类的toString方法；
  - SquenceFileOutputFormat类：输出的数据可以**作为后续MapReduce任务的输入数据**，输出格式紧凑、易于压缩
  - 自定义OutputFormat实现类：根据需求由用户自定义输出形式。

#### 3.3.2 自定义OutputFormat类

- 需求：当我们需要自定义最终文件的**输出路径和输出格式**时，需要自定义OutputFormat。

- 具体步骤

  1. 编写Mapper、Reducer、Driver类
  2. 自定义FileOutputFormat类的子类
  3. 改写RecordWriter，重写数据输出的write()方法

- 举例

  1. 需求：过滤log日志，将包含atguigu的网站和其他网站分开输出

     ```
     http://www.baidu.com
     http://www.google.com
     http://cn.bing.com
     http://www.atguigu.com
     http://www.sohu.com
     http://www.sina.com
     http://www.sin2a.com
     http://www.sin2desa.com
     http://www.sindsafa.com
     ```

  2. 编写Mapper类、Reducer类、Driver类

  3. 自定义FilieOutputFormat的子类

     ```java
     public class OPOutputFormat extends FileOutputFormat {
         //重写getRecordeWriter()方法
         @Override
         public RecordWriter getRecordWriter(TaskAttemptContext job) throws IOException, InterruptedException {
             //返回自定义的RecordWriter类的对象
             return new OPRecordWriter(job);
         }
     }
     ```

  4. 自定义RecordWriter类的子类

     ```java
     public class OPRecordWriter extends RecordWriter<Text, NullWritable> {
         //声明输出用的流对象
         FSDataOutputStream atguiguout;
         FSDataOutputStream otherout;
         //创建带参构造器
         public OPRecordWriter(TaskAttemptContext job){
             try {
                 //使用job的配置信息创建文件系统对象
                 FileSystem fs = FileSystem.get(job.getConfiguration());
     
                 //使用文件系统对象创建流，指定流的输出路径及输出的文件名
                 Path path1 = new Path("D:/output/atguigu.log");
                 Path path2 = new Path("D:/output/other.log");
                 atguiguout = fs.create(path1);
                 otherout = fs.create(path2);
             } catch (IOException e) {
                 e.printStackTrace();
             }
     
         }
         //重写write()方法,声明写出的数据规则和格式
         public void write(Text text, NullWritable nullWritable) throws IOException, InterruptedException {
             if (text.toString().contains("atguigu")){
                 atguiguout.write(text.toString().getBytes());
                 atguiguout.write("\r\n".getBytes());
             }else{
                 otherout.write(text.toString().getBytes());
                 otherout.write("\r\n".getBytes());
             }
         }
         //重写close()方法,关闭流
         public void close(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
             atguiguout.close();
             otherout.close();
         }
     }
     ```

  5. 在Driver类中配置输出类

     ```java
     //配置自定义的输出类
     job.setOutputFormatClass(OPOutputFormat.class);
     //输出路径依然需要配置,因为会输出SUCCESS文件
     FileInputFormat.setInputPaths(job, new Path("D:/logs.txt"));
     FileOutputFormat.setOutputPath(job, new Path("D:/output"));
     ```



### 3.4 Join应用

- 需求：有时候我们需要从不同的数据文档中整理、计算数据，且这些文档中具有某种关联（类似于sql中不同表的关联），我们就需要实现对多种文档的多种处理方式。

#### 3.4.1 Reduce Join

- 分工

  - Map端：为来自不同表或文件的kv对，**打标签以区分源文件**，**然后使用连接字段（共有字段）作为key**，其余字段作为value，输出给Reduce端。
  - Reduce端：通过key自动分组后，**根据标签对迭代器下的value进行分离**，再处理数据，合并并输出。

- 案例：对以下两文件合并，输出“订单号-产品名-产品数”格式的内容

  ```java
  //order.txt
  1001	01	1
  1002	02	2
  1003	03	3
  1004	01	4
  1005	02	5
  1006	03	6
  //pd.txt
  01	小米
  02	华为
  03	格力
  ```

  1. 自定义OrderBean类，封装所有字段信息

     ```java
     public class OrderBean implements Writable {
     
         private String orderId;//订单号
         private String pid;//商品id
         private Integer amount;//商品数
         private String pname;//商品名
         private String flag;//来源文档标签
     
         public String getOrderId() {
             return orderId;
         }
     
         public void setOrderId(String orderId) {
             this.orderId = orderId;
         }
     
         public String getPid() {
             return pid;
         }
     
         public void setPid(String pid) {
             this.pid = pid;
         }
     
         public Integer getAmount() {
             return amount;
         }
     
         public void setAmount(Integer amount) {
             this.amount = amount;
         }
     
         public String getPname() {
             return pname;
         }
     
         public void setPname(String pname) {
             this.pname = pname;
         }
     
         public String getFlag() {
             return flag;
         }
     
         public void setFlag(String flag) {
             this.flag = flag;
         }
     
         public OrderBean() {
         }
     
         //定义最终输出的内容格式
         @Override
         public String toString() {
             return orderId + "\t" + pname + "\t" + amount;
         }
     
     
         public void write(DataOutput dataOutput) throws IOException {
             dataOutput.writeUTF(orderId);
             dataOutput.writeUTF(pid);
             dataOutput.writeInt(amount);
             dataOutput.writeUTF(pname);
             dataOutput.writeUTF(flag);
         }
     
         public void readFields(DataInput dataInput) throws IOException {
             this.orderId = dataInput.readUTF();
             this.pid = dataInput.readUTF();
             this.amount = dataInput.readInt();
             this.pname = dataInput.readUTF();
             this.flag = dataInput.readUTF();
         }
     
     }
     ```

  2. 自定义Mapper类，为不同来源的数据打标签，用于reduce阶段区分处理

     ```java
     //输出的key应取两文件(两表)的共有字段
     public class ReduceJoinMapper extends Mapper<LongWritable, Text, Text, OrderBean> {
     	
         //声明切片名,记录数据文件的标签
         private String splitName;
         //声明用于输出的kv对变量
         private Text k = new Text();
         private OrderBean v = new OrderBean();
     	
         //重写setup方法,该方法在1个MapTask中只执行1次,且在map方法前最先执行
         @Override
         protected void setup(Context context) throws IOException, InterruptedException {
             //由于1个MapTask处理1个切片信息,而1个切片中的数据来源于1个文件,因此可以通过提取切片信息读取来源文件名,用于区分数据来源文件
             InputSplit inputSplit = context.getInputSplit();
             FileSplit currentSplit = (FileSplit)inputSplit;
             String name = currentSplit.getPath().getName();
             splitName = name;
         }
     
         @Override
         protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
             String line = value.toString();
             //注意:不同文件的分隔符可能不同,此处为相同,处理方式一致
             String[] split = line.split("\t");
             //根据切片名,对数据用不同的方式封装写出
             if (splitName.contains("order")){
                 k.set(split[1]);
                 v.setOrderId(split[0]);
                 v.setPid(split[1]);
                 v.setAmount(Integer.parseInt(split[2]));
                 v.setPname("");
                 v.setFlag("order");
             }else {
                 k.set(split[0]);
                 v.setOrderId("");
                 v.setPid(split[0]);
                 v.setAmount(0);
                 v.setPname(split[1]);
                 v.setFlag("name");
             }
             context.write(k, v);
         }
     }
     ```

  3. 自定义Reducer类，根据数据标签，合并数据，得到目标内容

     ```java
     public class ReduceJoinReducer extends Reducer<Text, OrderBean, OrderBean, NullWritable> {
     	
         //声明变量记录数据中用于被合并的部分(从表),此处为商品名
         String name = "";
         //由于map中输出的两种文件的数据的处理方式不同,而reduce方法中的迭代器只能遍历一次
         //因此这里声明集合对象,用于临时装入合并的部分(主表)
         List<OrderBean> list =  new ArrayList<OrderBean>();
     
         @Override
         protected void reduce(Text key, Iterable<OrderBean> values, Context context) throws IOException, InterruptedException {
     		
             //根据标签分类处理数据
             for (OrderBean value : values) {
                 if ("order".equals(value.getFlag())){
                     //将主表数据深拷贝进集合
                     //注意:由于reduce中提供的迭代器对象在迭代时只是对1个对象重复赋值再输出,地址值不会改变,因此这里必须使用深拷贝
                     OrderBean newOrder = new OrderBean();
                     try {
                         BeanUtils.copyProperties(newOrder, value);
                     } catch (IllegalAccessException e) {
                         e.printStackTrace();
                     } catch (InvocationTargetException e) {
                         e.printStackTrace();
                     }
                     list.add(newOrder);
                 }else {
                     //对从表的数据进行提取
                     name = value.getPname();
                 }
             }
     
             //遍历集合,将从表数据写入主表中输出
             for (OrderBean order : list) {
                 order.setPname(name);
                 context.write(order, NullWritable.get());
             }
     
             //由于Reducer类只初始化1次,而reduce方法会对每个组各调用1次,因此需要在一次方法执行后清空集合
             list.clear();
         }
     }
     ```

  4. 自定义Driver类

     ```java
     public class Driver {
     
         public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
             Job job = Job.getInstance(new Configuration());
     
             job.setJarByClass(Driver.class);
             job.setMapperClass(ReduceJoinMapper.class);
             job.setReducerClass(ReduceJoinReducer.class);
     
             job.setMapOutputKeyClass(Text.class);
             job.setMapOutputValueClass(OrderBean.class);
             job.setOutputKeyClass(OrderBean.class);
             job.setOutputValueClass(NullWritable.class);
     
             FileInputFormat.setInputPaths(job, new Path("D:/input"));
             FileOutputFormat.setOutputPath(job, new Path("D:/output"));
     
             job.waitForCompletion(true);
         }
     }
     ```

     

#### 3.4.2 Map Join

- Map Join用于处理多个数据来源中，多个表间存在数据倾斜问题的场景。直接跳过Reduce阶段，由多个MapTask直接完成数据处理的工作。

- 案例：此处依然使用Reduce Join中的订单案例

  1. 自定义OrderBean类，同3.4.1的定义方式

  2. 自定义Mapper类，将pd.txt作为缓存文件被MapTask读取，再重写map()方法

     ```java
     public class MapJoinMapper extends Mapper<LongWritable, Text, Text, NullWritable> {
     
         //声明用于输出的key对象
         private Text k = new Text();
         //声明集合对象,用于存放pd.txt中的数据
         private HashMap<String, String> map = new HashMap<String, String>();
     
         @Override
         protected void setup(Context context) throws IOException{
     
             //获取缓存文件路径
             URI[] cacheFiles = context.getCacheFiles();
             Path path = new Path(cacheFiles[0]);
             //通过配置对象获取文件系统
             FileSystem fs = FileSystem.get(context.getConfiguration());
             //使用文件系统创建流
             FSDataInputStream fsDataInputStream = fs.open(path);
             BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(fsDataInputStream,"utf-8"));
             //读取文件内容,存入集合
             String line;
             while ((line = bufferedReader.readLine()) != null ){
                 String[] splits = line.split("\t");
                 map.put(splits[0],splits[1]);
             }
             //关闭流
             bufferedReader.close();
         }
     
         @Override
         protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
     
             //读取order.txt的一行内容,并切分
             String line = value.toString();
             String[] splits = line.split("\t");
             //按照指定的输出格式输出结果
             String out = splits[0] + "\t" + map.get(splits[1]) + "\t" + splits[2];
             k.set(out);
             context.write(k, NullWritable.get());
     
         }
     }
     ```

  3. 自定义Driver类，配置缓存文件，配置跳过Reduce阶段

     ```java
     public class Driver {
     
         public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException, URISyntaxException {
             Job job = Job.getInstance(new Configuration());
     
             job.setJarByClass(Driver.class);
             job.setMapperClass(MapJoinMapper.class);
     
             //只需配置最终的输出数据类型
             job.setOutputKeyClass(Text.class);
             job.setOutputValueClass(NullWritable.class);
     
             FileInputFormat.setInputPaths(job, new Path("D:/input/"));
             FileOutputFormat.setOutputPath(job, new Path("D:/output/"));
     
             //配置缓存文件路径
             job.addCacheFile(new URI("file:///d:/pd.txt"));
     
             //配置ReduceTask数量为0,直接跳过Reduce阶段
             job.setNumReduceTasks(0);
     
             job.waitForCompletion(true);
         }
     }
     ```



### 3.5 计数器应用

- Hadoop为每个作业维护若干的内置计数器，用于描述作业过程中的多项指标。如：记录已处理的字节数和记录数，使用户监控已处理的输入数据量和产生的输出数据量。
- 计数器API
  - 采用枚举的方式统计计数
  - 采用计数器组、计数器名的方式统计



### 3.6 数据清洗（ETL）

- 需求：由于实际生产环境中，源数据中通常夹杂许多无意义、不符合要求的数据，这时就需要在包含核心业务的MapReduce程序执行前，对数据进行清洗。清洗的过程一般只需要运行Map阶段。

- 实例：去除日志文件中字段长度小于等于11的日志内容

  1. 自定义Mapper类

     ```java
     public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
     	
         //声明用于输出的变量
     	Text k = new Text();
     	
     	@Override
     	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
     		
     		// 1.获取1行数据
     		String line = value.toString();
     		
     		// 2.使用自定义方法解析日志内容
     		boolean result = parseLog(line,context);
     		
     		// 3.日志不合法退出
     		if (!result) {
     			return;
     		}
     		
     		// 4.将合法日志赋给key
     		k.set(line);
     		
     		// 5.写出数据
     		context.write(k, NullWritable.get());
     	}
     
     	// 自定义日志的过滤规则
     	private boolean parseLog(String line, Context context) {
     
     		// 1.截取
     		String[] fields = line.split(" ");
     		
     		// 2.日志长度大于11的为合法
     		if (fields.length > 11) {
     			//使用系统计数器对数据计数
     			context.getCounter("map", "true").increment(1);
     			return true;
     		}else {
     			context.getCounter("map", "false").increment(1);
     			return false;
     		}
     	}
     }
     ```

  2. 自定义Driver类

     ```java
     public class LogDriver {
     
     	public static void main(String[] args) throws Exception {
     
             // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
             args = new String[] { "e:/input/inputlog", "e:/output1" };
     
     		// 1 获取job信息
     		Configuration conf = new Configuration();
     		Job job = Job.getInstance(conf);
     
     		// 2 加载jar包
     		job.setJarByClass(LogDriver.class);
     
     		// 3 关联map
     		job.setMapperClass(LogMapper.class);
     
     		// 4 设置最终输出类型
     		job.setOutputKeyClass(Text.class);
     		job.setOutputValueClass(NullWritable.class);
     
     		// 设置reducetask个数为0
     		job.setNumReduceTasks(0);
     
     		// 5 设置输入和输出路径
     		FileInputFormat.setInputPaths(job, new Path(args[0]));
     		FileOutputFormat.setOutputPath(job, new Path(args[1]));
     
     		// 6 提交
     		job.waitForCompletion(true);
     	}
     }
     ```



# Yarn

## 第一章 Yarn资源调度器

### 1.1 Yarn基本架构

- Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于操作系统上运行的应用程序。
- Yarn架构图示

<img src="Hadoop.assets/wps1-1587385762838.png" alt="img" style="zoom:150%;" />



### 1.2 Yarn工作机制

1. MapReduce程序提交至客户端节点
2. YarnRunner向ResourceManager申请一个Application
3. RM将该应用程序的资源路径返回YarnRunner
4. 该程序将运行所需的资源提交到HDFS
5. 程序资源提交完成后，申请运行MrApplicationMaster
6. RM将用户的请求初始化为一个Task
7. 该Task在一个NodeManager上创建Container（一个NM可创建多个Container），并生成MrApplicationMaster
8. Container从HDFS上拷贝资源到本地
9. MrApplicationMaster继续向RM申请资源启动MapTask
10. RM将MapTask任务分配给多个NodeManager，多个NodeManager接收任务后分别创建Container
11. MrApplicationMaster向NodeManager分配MapTask启动的脚本，NodeManager执行MapTask
12. MrApplicationMaster等待所有MapTask执行完毕后，向RM申请资源启动ReduceTask
13. ReduceTask从MapTask获取数据
14. ReduceTask全部执行完成后，MrApplicationMaster向RM申请注销，释放所有Task占用的资源。



### 1.3 资源调度器

- Hadoop作业调度器主要有三种：FIFO，Capacity Scheduler，Fair Scheduler。Hadoop3中默认使用Capacity 

  Scheduler

- FIFO（先进先出调度器）

  - 规则：集群中的所有用户的所有作业提交到一个队列中，Hadoop按照提交的顺序依次运行。
  - 缺点：当队首运行大型作业时，一些资源占用较小的小型作业等待时间过长。而当队首运行小型作业时，资源又存在大量的浪费。因此该机制已被弃用。

- Capacity Scheduler（容量调度器）

  - 规则
    1. 集群中同时存在多个作业队列（每个队列遵循FIFO规则），为每个队列分别分配一定的资源量（主要标准为内存资源）。
    2. 当有新作业提交时，hadoop自动的根据不同队列中的资源占比（队列任务数 / 队列总资源），将新作业分配到占比最低的队列中。
    3. 当一个队列中的资源空闲时，可以暂时性的将资源共享给其余队列（资源量在配置的区间范围内），待其余队列的作业执行结束后收回资源。
  - 缺点：多队列机制减少了FIFO中作业无法并行的问题，但没有从根本上解决，每个队列中依旧只有队首作业可以执行。

- Fair Scheduler（公平调度器）

  - 规则
    1. 多队列，每个队列的资源量同样可以人为分配
    2. 一个队列中的多个作业可以同时执行，通过最大最小算法为一个队列中的每个作业分配资源
    3. 可为每个作业设置最小资源量，调度器会保证该作业分获的资源不低于设置值。
    4. 多队列间支持紫苑的临时共享
  - 最大最小算法
    - 非加权：
      1. 计算 队列总资源量 / 队列作业数，将资源均分给多个作业。
      2. 当分配后的作业出现资源剩余时，计算 剩余资源总量 / 资源不足作业总数， 将剩余资源均分给其他作业
      3. 重复第2步，直至没有作业存在资源剩余或所有作业都拿到了所需资源。
    - 加权：
      1. 计算 （队列资源总量 / 加权系数总和）* 加权系数，将资源分配给多个作业
      2. 当分配后的作业出现资源剩余时，计算 （剩余资源总量 / 资源不足作业加权系数总和）* 加权系数， 将剩余资源均分给其他作业
      3. 重复第2步，直至没有作业存在资源剩余或所有作业都拿到了所需资源。



### 1.4 容量调度器

- Yarn默认使用了容量调度器的调度机制，但在默认配置中只配置了default一条队列，要实现多队列调度，需要自行配置其余队列。

- 生产环境中实际配置的队列数量一般决定于按照一定规则划分的数量

  - 按照框架划分队列（hive、spark、flink）
  - 按照业务划分队列（登陆模块、购物车、物流、业务部门）
  
  

#### 1.4.1 配置自定义队列

1. 修改 etc/hadoop/capacity-scheduler.xml 配置文件内容

   ```xml
   <!-- 根据dufault队列的配置,自行配置队列hive -->
   <!-- 指定多队列,使用,连接 -->
   <property>
       <name>yarn.scheduler.capacity.root.queues</name>
       <value>default,hive</value>
   </property>
   <!-- 指定default队列的额定容量 -->
   <property>
       <name>yarn.scheduler.capacity.root.default.capacity</name>
       <value>40</value>
   </property>
   <!-- 指定hive队列的额定容量,value为分配的资源百分比 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.capacity</name>
       <value>60</value>
   </property>
   <!-- 指定default队列允许单用户占用的资源占比 -->
   <property>
       <name>yarn.scheduler.capacity.root.default.user-limit-factor</name>
       <value>1</value>
   </property>
   <!-- 指定hive队列允许单用户占用的资源占比 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
       <value>1</value>
   </property>
   <!-- 指定default队列的最大容量,向其他队列借用资源时的队列总资源上限 -->
   <property>
       <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
       <value>60</value>
   </property>
   <!-- 指定hive队列的最大容量 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
       <value>80</value>
   </property>
   <!-- 指定default队列的状态 -->
   <property>
       <name>yarn.scheduler.capacity.root.default.state</name>
       <value>RUNNING</value>
   </property>
   <!-- 指定hive队列的状态 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.state</name>
       <value>RUNNING</value>
   </property>
   <!-- 指定default队列允许哪些用户提交job -->
   <property>
       <name>yarn.scheduler.capacity.root.default.acl_submit_applications</name>
       <value>*</value>
   </property>
   <!-- 指定hive队列允许哪些用户提交job -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
       <value>*</value>
   </property>
   <!-- 指定default队列允许哪些用户进行管理 -->
   <property>
       <name>yarn.scheduler.capacity.root.default.acl_administer_queue</name>
       <value>*</value>
   </property>
   <!-- 指定hive队列允许哪些用户进行管理 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
       <value>*</value>
   </property>
   <!-- 指定default队列允许哪些用户提交配置优先级的job -->
   <property>       
       <name>yarn.scheduler.capacity.root.default.acl_application_max_priority</name>
       <value>*</value>
   </property>
   <!--指定hive队列允许哪些用户提交配置优先级的job -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
       <value>*</value>
   </property>
   <!-- 指定default队列允许job运行的最大时间-->
   <property>
       <name>yarn.scheduler.capacity.root.default.maximum-application-lifetime</name>
       <value>-1</value>
   </property>
   <!-- 指定hive队列允许job运行的最大时间 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
       <value>-1</value>
   </property>
   <!-- 指定default队列允许job运行的默认时间 -->
   <property>
       <name>yarn.scheduler.capacity.root.default.default-application-lifetime</name>
       <value>-1</value>  
   </property>
   <!-- 指定hive队列允许job运行的最大时间 -->
   <property>
       <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
       <value>-1</value>
   </property>
   ```
   
2. 配置完成后重启Yarn，查看web端Scheduler项，可以看到配置的队列



#### 1.4.2 提交程序到指定队列执行

1. 在自定义Driver类中添加配置，指定队列名

   ```java
   Configuration configuration = new Configuration();
   
   //过时但依然可用
   //configuration.set("mapred.job.queue.name", "hive");
   //hadoop3中的新配置字段
   conf.set("mapreduce.job.queuename","hive");
   
   Job job = Job.getInstance(configuration);
   ```



### 1.5 任务的推测执行

- 需求：一个作业包含多个MapTask和ReduceTask，当其中一个或几个任务执行缓慢时，可能是由于一些客观原因（硬件老化，软件bug）造成的，这是无法人为避免的情况。

- hadoop提供了推测执行机制，当一个任务的执行速度远低于其他任务时，会自动为该任务启动一个备份任务，与原任务同时执行，最终采用先执行完的任务的结果（空间换时间）。

- 推测执行的条件

  1. 对于1个Task，只能启动1个备份Task

  2. 当前作业已完成的Task不得少于5%

  3. mapred-site.xml配置文件中开启推测实行（默认为开启）

     ```xml
     <property>
         <name>mapreduce.map.speculative</name>
         <value>true</value>
         <description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
     </property>
     
     <property>
         <name>mapreduce.reduce.speculative</name>
         <value>true</value>
         <description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
     </property>
     ```

  4. Task间不能存在严重的负载倾斜，因为这时一定会出现相对执行缓慢的Task，并非其他原因造成

  5. Task任务不能是持续性任务，例如持续接收写出的Task执行周期必然长于其他Task



### 1.6 Yarn命令行操作

```shell
#查看appID
yarn application -list

#查看进程日志
yarn logs -applicationId application_1596550537820_0001

#结束app进程
yarn application -kill application_1596550537820_0001
```



# 优化



## 第一章 Hadoop 数据压缩

### 1.1 概述

- 压缩是提高Hadoop运行效率的一种优化策略。通过对Mapper、Reducer运行过程的数据进行压缩，减少磁盘IO
- 压缩减少了磁盘IO，但增加了CPU运算负担，因此适用于IO密集型作业，不适用于运算密集型作业。



### 1.2 MapReduce支持的压缩编码

| 压缩格式 | 算法    | 扩展名   | hadoop是否自带  | 是否支持切分 | 配置项                                 |
| -------- | ------- | -------- | --------------- | ------------ | -------------------------------------- |
| DEFLATE  | DEFLATE | .deflate | 是              | 否           | 只需配置压缩格式                       |
| gzip     | DEFLATE | .gz      | 是              | 否           | 只需配置压缩格式                       |
| bzip2    | bzip2   | .bz2     | 是              | 是           | 只需配置压缩格式                       |
| LZO      | LZO     | .lzo     | 否              | 是           | 除配置压缩格式外需要建索引指定输入格式 |
| Snappy   | Snappy  | .snappy  | 是（hadoop3起） | 否           | 只需配置压缩格式                       |

- 压缩格式和编解码器（codec）

| 压缩格式 | 编解码器                                   |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |



### 1.3 压缩方式的选择

#### 1.3.1 gzip

- 优点：压缩率较高，速度较快，hadoop自带
- 缺点：不支持切片
- 应用：每个文件压缩后大小可控制在1文件块大小内时，可使用gzip压缩

#### 1.3.2 bzip2

- 优点：压缩率最高，支持切片，hadoop自带
- 缺点：速度最慢
- 应用：数据过大且对速度要求不高，后续调用机会较少的数据，可使用bzip2压缩

#### 1.3.3 LZO

- 优点：压缩率较高，速度较快，支持切片
- 缺点：hadoop不自带，需要额外安装，使用时需要建立索引，还要指定输出格式
- 应用：因为本身支持切片，当数据文件压缩后大小依然大于1文件块大小较多时，可使用LZO压缩

#### 1.3.4 Snappy

- 优点：速度最快，压缩率较高，hadoop自带
- 缺点：不支持切片
- 应用：对于Map端输出的数据，因为大小必定不足1文件块大小，且作业过程中对压缩速度要求较高，可使用Snappy压缩，再传递给Reduce端



### 1.4 压缩位置的选择

- 输入端采用压缩（bzip2,LZO）
- Mapper输出端采用压缩（Snappy）
- Reducer输出端采用压缩（bzip2,LZO）



### 1.5 压缩参数的配置

- 可修改mapred-site.xml配置文件中的相关配置实现不同位置的数据压缩

| 参数                                             | 默认值                                                       | 阶段        | 建议                                         |
| ------------------------------------------------ | ------------------------------------------------------------ | ----------- | -------------------------------------------- |
| io.compression.codecs  （在core-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec, org.apache.hadoop.io.compress.Lz4Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器 |
| mapreduce.map.output.compress                    | false                                                        | mapper输出  | 这个参数设为true启用压缩                     |
| mapreduce.map.output.compress.codec              | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress       | false                                                        | reducer输出 | 这个参数设为true启用压缩                     |
| mapreduce.output.fileoutputformat.compress.codec | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2      |
| mapreduce.output.fileoutputformat.compress.type  | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK  |



### 1.6 应用

#### 1.6.1 数据流的压缩和解压缩

- 数据流的压缩

  ```java
  @Test
  public void compresstest() throws IOException, ClassNotFoundException {
  
      //1.获取编解码器对象
      Class<?> codecClass = Class.forName("org.apache.hadoop.io.compress.DefaultCodec");
      CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(codecClass, new Configuration());
  
      //2.获取输入流
      FileInputStream fis = new FileInputStream(new File("D:/input1/input.txt"));
  
      //3.获取输出流,使用编解码器对象包装流
      FileOutputStream fos =
          new FileOutputStream(new File("D:/input1/input.txt" + codec.getDefaultExtension()));
      CompressionOutputStream os = codec.createOutputStream(fos);
  
      //4.拷贝文件
      IOUtils.copyBytes(fis,os,1024*1024, false);
  
      //5.关闭流
      os.close();
      fos.close();
      fis.close();
      
  }
  ```

- 数据流的解压缩

  ```java
  @Test
  public void decompresstest() throws IOException {
  
      //1.获取编解码器类的对象
      String path = "D:/input1/input.txt.deflate";
      CompressionCodecFactory factory = new CompressionCodecFactory(new Configuration());
      CompressionCodec codec = factory.getCodec(new Path(path));
  
      //2.获取输入流,使用编解码器对象包装流
      FileInputStream fis = new FileInputStream(new File(path));
      CompressionInputStream is = codec.createInputStream(fis);
  
      //3.获取输出流
      FileOutputStream fos = new FileOutputStream(new File("D:/input1/input.txt"));
  
      //4.拷贝文件
      IOUtils.copyBytes(is, fos, 1024*1024, false);
  
      //5.关闭流
      fos.close();
      is.close();
      fis.close();
      
  }
  ```



#### 1.6.2 Map输出端的数据压缩

- 只需要在Driver类中开启并指定压缩格式

  ```java
  Configuration conf = new Configuration();
  
  //开启map端输出压缩
  conf.setBoolean("mapreduce.map.output.compress", true);
  //设置map端输出的压缩方式
  conf.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);
  
  Job job = Job.getInstance(conf);
  ```

  

#### 1.6.3 Reduce输出端的数据压缩

- 只需要在Driver类中开启并指定压缩格式

  ```java
  //开启reduce端输出压缩
  FileOutputFormat.setCompressOutput(job, true);
  //设置reduce端输出放入压缩方式
  FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
  ```



## 第二章 Hadoop 企业优化

### 2.1 MapReduce性能问题

MapReduce性能瓶颈在于以下两点

1. 计算机性能（CPU、内存、磁盘、网络）
2. IO操作
   1. 数据倾斜
   2. MapTask和ReduceTask数量不合理
   3. Map和Reduce分工不均
   4. 大量小文件读写
   5. 不支持切分的过大单个文件
   6. 溢写次数过多
   7. 合并次数过多

### 2.2 MapReduce优化方案

#### 2.2.1 数据输入

- 合并小文件
- 使用CombineTextInputFormat输入数据

#### 2.2.2 Map阶段

- 减少溢写次数：调整io.sort.mb和sort.spill.percent
- 减少合并次数：调整io.sort.factor
- Shuffle阶段使用Combine预处理数据

#### 2.2.3 Reduce阶段

- 调整Map和Reduce任务数
- 实现Map、Reduce共同执行：调整slowstart.completedmaps
- 不适用Reduce，直接由Map完成数据的处理

#### 2.2.4 IO传输

- 使用数据压缩
- 使用SequenceFile二进制文件

#### 2.2.5 数据倾斜

- 自定义分区
- 使用Combine
- 使用Map Join

#### 2.2.6 参数调优（重要）

- 资源相关参数

| 配置参数（mapred-default.xml）                | 参数说明                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb                       | 一个MapTask可使用的资源上限（单位:MB），默认为1024。<br />如果MapTask实际使用的资源量超过该值，则会被强制杀死。<br />**生产环境需要考虑文件是否支持切片，配置为1个文件大小的8倍** |
| mapreduce.reduce.memory.mb                    | 一个ReduceTask可使用的资源上限（单位:MB），默认为1024。<br />如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。<br />**生产环境考虑ruduce前的单个文件大小，配置为1个文件大小的8倍** |
| mapreduce.map.cpu.vcores                      | 每个MapTask可使用的最多cpu core数目，默认值: 1               |
| mapreduce.reduce.cpu.vcores                   | 每个ReduceTask可使用的最多cpu core数目，默认值: 1            |
| mapreduce.reduce.shuffle.parallelcopies       | 每个Reduce去Map中取数据的并行数。默认值是5                   |
| mapreduce.reduce.shuffle.merge.percent        | Buffer中的数据达到多少比例开始写入磁盘。默认值0.66           |
| mapreduce.reduce.shuffle.input.buffer.percent | Buffer大小占Reduce可用内存的比例。默认值0.7                  |
| mapreduce.reduce.input.buffer.percent         | 指定多少比例的内存用来存放Buffer中的数据，默认值是0.0        |

| 配置参数（yarn-default.xml）             | 参数说明                                                     |
| ---------------------------------------- | ------------------------------------------------------------ |
| yarn.scheduler.minimum-allocation-mb     | 给应用程序Container分配的最小内存，默认值：1024              |
| yarn.scheduler.maximum-allocation-mb     | 给应用程序Container分配的最大内存，默认值：8192              |
| yarn.scheduler.minimum-allocation-vcores | 每个Container申请的最小CPU核数，默认值：1                    |
| yarn.scheduler.maximum-allocation-vcores | 每个Container申请的最大CPU核数，默认值：4                    |
| yarn.nodemanager.resource.memory-mb      | 给Containers分配的最大物理内存，默认值：8192<br />**生产环境下根据服务器性能，128g内存服务器配置为100g** |

| 配置参数（mapred-default.xml）   | 参数说明                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| mapreduce.task.io.sort.mb        | Shuffle的环形缓冲区大小，默认100m<br />**生产环境下配置为200m，减少溢写次数** |
| mapreduce.map.sort.spill.percent | 环形缓冲区溢出的阈值，默认80%<br />**生产环境下配置为90-95%，减少溢写次数** |

- 容错相关参数

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。 |



### 2.3 小文件优化方案（uber模式）

- 默认情况下，每个Task都会启动1个JVM来运行，当一个Task任务的数据量很小时，会造成资源的浪费。

- 通过开启uber模式，可以实现jvm的重用，使多个Task运行在1个JVM中

- 步骤

  1. 修改mapred-site.xml配置文件

     ```xml
     <!-- 开启uber模式 -->
     <property>
         <name>mapreduce.job.ubertask.enable</name>
         <value>true</value>
     </property>
     <!-- uber模式中最大的MapTask数量,可向下修改  --> 
     <property>
         <name>mapreduce.job.ubertask.maxmaps</name>
         <value>9</value>
     </property>
     <!-- uber模式中最大的ReduceTask数量，可向下修改(设置为0时没有Reduce阶段) -->
     <property>
         <name>mapreduce.job.ubertask.maxreduces</name>
         <value>1</value>
     </property>
     <!-- uber模式中最大的输入数据量，如果不配置，则使用dfs.blocksize的值，可向下修改 -->
     <property>
         <name>mapreduce.job.ubertask.maxbytes</name>
         <value></value>
     </property>
     ```

  2. 启动任务



