# Zookeeper

## 第一章 Zookeeper入门

### 1.1 概述

- Zookeeper是一个基于观察者模式设计的分布式服务管理框架。
- Zookeeper = 文件系统 + 通知机制
- 官网：http://zookeeper.apache.org/

### 1.2 特点

1. Zookeeper集群包含1个领导者（Leader）和多个跟随者（Follower）
2. 半数可用机制：集群中只要保证半数以上的节点可用，集群就能提供正常服务
3. 强一致性：每个节点保存一份相同的数据副本，所有节点数据一致

### 1.3 数据结构

- Zookeeper的数据模型结构类似于Unix文件系统，采用树状结构，根目录下创建多个ZNode，每个ZNode默认存储上限为1MB

### 1.4 应用场景

- 统一命名服务
- 统一配置管理（重点）：同步集群中所有节点的配置文件信息
- 同一集群管理
- 服务器节点动态上下线
- 软负载均衡



## 第二章 Zookeeper的安装

### 2.1 本地模式安装

1. 安装JDK并配置环境变量

2. 上传Zookeeper安装包，解压到指定安装目录

   ```shell
   tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/
   ```

3. 复制配置文件zoo_sample.cfg

   ```shell
   cd /opt/module/zookeeper-3.5.7/conf
   cp zoo_sample.cfg zoo.cfg
   #zoo_sample.cfg是Zookeeper提供的配置文件参考,在调用的配置文件为zoo.cfg,因此需要复制或重命名后再做修改
   ```

4. 修改配置文件zoo.cfg

   ```shell
   vi zoo.cfg
   
   #修改以下内容,配置Zookeeper文件系统的数据存储目录
   dataDir=/opt/module/zookeeper-3.5.7/zkdata
   ```

5. 创建存储目录的文件夹

   ```shell
   mkdir zkdata
   ```

6. 启动Zookeeper

   ```shell
   bin/zkServer.sh start
   
   #查看进程
   jps
   #查看状态
   bin/zkServer.sh status
   ```

7. 启动客户端

   ```shell
   bin/zkCli.sh
   
   #退出客户端
   quit
   ```

8. 停止Zookeeper

   ```shell
   bin/zkServer.sh stop
   ```




### 2.2 配置参数说明

- 配置文件zoo.cfg中包含以下配置项

  ```shell
  #通信心跳间隔(毫秒)
  tickTime=2000
  #服务器之间或服务器与客户端之间的心跳通信间隔时间,决定了超时时间的配置(最小为tickTime*2)
  
  #LF初始通信时限
  initLimit=10
  #Follower在启动时等待响应的最大时限(initLimit*tickTime),即超过该心跳数时间仍未通信成功则连接失败
  
  #LF同步通信时限
  synLimit=5
  #Follower和Leader间可以接受的最大未响应时限(synLimit*tickTime),即超过该心跳数时间仍未响应则判断Follower死亡,从Zookeeper集群中删除该节点
  
  #数据文件存储目录
  dataDir=/opt/module/zookeeeper-3.5.7/zkdata
  
  #客户端连接端口号
  clientPort=2181
  ```



### 2.3 Zookeeper的四字命令

- Zookeeper提供了一系列查询命令，使用户可以直接使用客户端获取Zookeeper服务的相关信息。

- 使用步骤

  1. 修改配置文件zoo.cfg

     ```shell
     #在配置文件中追下以下配置
     4lw.commands.whitelist=*
     ```

  2. 使用nc命令向Zookeeper提交命令

     ```shell
     #格式为 nc [主机名] [端口号]
     nc hadoop100 2181
     ```

  3. 输入命令，回车确认

     ```shell
     conf
     ```

- 主要的四字命令

  | 命令 | 功能                                                         |
  | ---- | ------------------------------------------------------------ |
  | ruok | 测试服务是否处于正确的状态，若是则返回 imok ,否则不做任何响应 |
  | conf | （3.3.0版本起）打印出服务相关配置的详细信息                  |
  | cons | 列出所有连接到这台服务器的客户端全部会话详细信息。           |
  | crst | 重置所有的连接和会话统计信息                                 |
  | dump | （仅限Leader节点）列出重要的会话和临时节点                   |
  | envi | 打印出服务环境的详细信息                                     |



## 第三章 Zookeeper的内部原理

### 3.1 节点类型

- 持久节点：客户端和服务器断开连接后，节点保留
  - 持久目录节点（Persistent）
  - 持久编号目录节点（Persistent_sequential）：创建后znode名后会追加一个自增的顺序号
- 短暂节点：客户端和服务器断开连接后，节点删除
  - 临时目录节点（Ephemeral）
  - 临时编号目录节点（Ephemeral_sequential）：创建后znode名后会追加一个自增的顺序号
- 顺序号：所有znode在创建时会生成一个总0开始自增的10位顺序号，该号码由其父节点维护，用于记录父节点下多个节点的创建顺序。不论是否以编号显式命名节点，计数器都会增1，旧节点删除后也会继续以当前计数器中的顺序号为后续新建的znode编号。



### 3.2 Stat结构体

- zxid（事务ID）：每次对Zookeeper的修改操作都会生成一个zxid形式的时间戳，用于记录所有修改操作的发生顺序，zxid越小，发生越靠前。
- Zookeeper中使用Stat来描述一个节点（znode）的状态，Stat中包含以下信息
  1. cZxid：创建该znode的zxid
  2. ctime：创建该znode的时间
  3. mZxid：最后一次修改该znode的zxid
  4. mtime：最后一次修改该znode的时间
  5. pZxid：最后一次修改子节点的zxid
  6. cversion：子节点的修改版本号（次数）
  7. detaVersion：该znode的数据修改版本号（次数）
  8. aclVersion：访问控制列表的版本号（次数）
  9. ephemeralOwner：若该znode为临时节点，则记录znode拥有者的session id，否则为0
  10. dataLength：该znode的数据长度
  11. numChildren：该znode的子节点数



### 3.3 监听器原理（重点）

1. 首先启动main()线程
2. main线程中创建Zookeeper客户端，这时会启动两个线程，一个是用于网络通信的connect线程，一个是用于监听的listener线程
3. 通过connect线程将需要注册的监听事件（如具体的znode变化）发送给Zookeeper
4. Zookeeper接收后将监听事件注册到监听器中
5. Zookeeper的监听器监听到数据变化时，将这个消息发送给listener线程
6. listener接收到Zookeeper发送的消息后调用process()方法



### 3.4 选举机制（重点）

- 半数机制：集群中半数以上（不包含半数）的节点存活，集群可用。因此Zookeeper节点数适合配置为奇数。
- Zookeeper集群的所有节点中，1个节点为Leader，其余节点为Follower，Leader节点由内部的选举机制产生。
- 选举机制
  - 情况1：集群初次启动，无数据存储
    1. 集群启动时自动读取zoo.cfg配置文件，确认集群配置的总节点数，进而得到半数的节点数
    2. 每个节点的启动顺序有前后差距，当某一节点启动后，集群中的已启动节点数达到半数以上，会选出Leader
    3. 当前已经启动的节点中，编号最大的节点被选举为Leader
  - 情况2：集群非初次启动，含有存储数据
    1. Leader为上次启动时的节点不变
  - 情况3：集群运行中Leader节点故障死亡
    1. Leader节点死亡后，集群会再次判断当前已启动节点是否达到半数以上
    2. 若未达到则集群不可用，无法产生新的Leader，等待启动新的节点
    3. 启动的节点数达到半数以上时，首先比较这些节点的zxid（事务id），优先选举事务id最大的节点作为Leader（保证数据的完整性）
    4. 若zxid相同，则比较各节点的编号，编号最大的节点被选举为Leader
- 选举机制总结
  - 选举的前提：**半数以上节点可用**
  - 选举的规则：**先比较zxid，再比较节点编号**



### 3.5 数据写入流程

1. 客户端向Zookeeper集群的任意一个节点发送数据写入请求
2. Zookeeper判断该节点是否为Leader节点，若不是，则将请求转发给Leader节点
3. Leader节点收到请求后，将请求广播给所有的Follower节点
4. Follower节点收到请求后，将该请求加入待写队列，并向Leader节点发送响应
5. 当Leader节点收到半数以上的节点（包括自身）的响应信息后，向所有节点再次发送确认信息
6. 所有节点执行待写队列中的请求，正式写入数据
7. Zookeeper返回客户端成功信息



## 第四章 Zookeeper的使用

### 4.1 分布式安装部署（集群脚本）

- 在hadoop100、hadoop101、hadoop102三个节点上部署Zookeeper

  1. 完成2.1中的步骤，在hadoop100节点上安装Zookeeper并完成相关配置

  2. 在zkdata目录下创建文件myid

     ```shell
     cd /opt/module/zookeeeper-3.5.7/zkdata/
     touch myid
     ```

  3. 编辑myid文件，写入自定义的节点编号

     ```shell
     vi myid
     
     #仅需写入节点编号
     0
     ```

  4. 修改配置文件zoo.cfg

     ```shell
     vi /opt/module/zookeeeper-3.5.7/conf/zoo.cfg
     
     #添加如下配置
     server.0=hadoop100:2888:3888
     server.1=hadoop101:2888:3888
     server.2=hadoop102:2888:3888
     
     #参数说明
     #格式: server.[节点编号]=[主机名]:[端口号1]:[端口号2]
     #节点编号	自定义的数字,标识集群中的每个的节点
     #主机名	Zookeeper集群需要部署的主机地址
     #端口号1	Follower与Leader交互的端口
     #端口号2	Leader节点失效后,其余节点进行选举使用的临时交互端口
     ```

  5. 分发Zookeeper至其他主机

     ```shell
     xsync /opt/module/zookeeeper-3.5.7
     ```

  6. 在其他主机节点上修改myid（和zoo.cfg中配置一致）

     ```shell
     vi /opt/module/zookeeeper-3.5.7/zkdata/myid
     ```

  7. 在三台主机节点上依次启动Zookeeper

     ```shell
     bin/zkServer.sh start
     ```

  8. 查看节点状态和java进程

     ```shell
     bin/zkServer.sh status
     
     jps
     #zookeeper进程名:QuorumPeerMain
     ```

- 编写Zookeeper集群群起，群停脚本

  1. 编写脚本

     ```shell
     #在当前用户家目录的bin目录下编写脚本文件(无需配置环境变量)
     vi /home/atguigu/bin/zkctl
     
     #编辑脚本 传参为start/stop/staus
     #!/bin/bash
     if [ $# -lt 1 ]
     	then
     		echo "No args input"
     		exit
     fi
     for i in hadoop100 hadoop101 hadoop102
     	do
     		echo "----------$i----------"
     		ssh $i /opt/module/zookeeper-3.5.7/bin/zkServer.sh $1
     	done
     ```

  2. 修改脚本权限

     ```shell
     chmod 777 zkctl
     ```

  3. 分发脚本

     ```shell
     xsync zkctl
     ```

  

### 4.2 客户端的命令行操作

1. 启动客户端

   ```shell
   bin/zkCli.sh
   ```

2. 输入并执行命令

   ```shell
   # help
   # 显示所有命令
   help
   
   # ls path
   # 查看指定节点下的子节点
   ls /
   # -w 监听子节点变化
   ls -w /
   # -s 附加stat信息
   ls -s /
   
   # create
   # 创建持久节点
   create /lcx
   create /lcx "123"
   # -s 节点名带序列编号
   create -s /lcx
   # -e 创建短暂节点
   create -e /lcx
   
   # get path
   # 获取节点的值,若没有值则返回空
   get /lcx
   # -w 监听内容变化
   get -w /lcx
   # -s 附加stat信息
   get -s /lcx
   
   # set
   # 设置节点的值
   set /lcx "123456"
   
   # stat
   # 查看节点的状态
   stat /lcx
   
   # delete
   # 删除节点
   delete /lcx
   
   # deleteall
   # 递归删除节点
   deleteall /lcx
   
   # quit
   # 退出客户端
   quit
   ```



### 4.3 API应用

#### 4.3.1 环境准备

1. 启动IDEA，新建Maven工程

2. 修改pom.xml配置文件，添加如下依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>RELEASE</version>
       </dependency>
       <dependency>
           <groupId>org.apache.logging.log4j</groupId>
           <artifactId>log4j-core</artifactId>
           <version>2.8.2</version>
       </dependency>
       <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
       <dependency>
           <groupId>org.apache.zookeeper</groupId>
           <artifactId>zookeeper</artifactId>
           <version>3.5.7</version>
       </dependency>
   </dependencies>
   ```

3. 在src/main/resources目录下新建文件log4j.properties，添加如下内容

   ```java
   log4j.rootLogger=INFO, stdout  
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
   log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n  
   log4j.appender.logfile=org.apache.log4j.FileAppender  
   log4j.appender.logfile.File=target/spring.log  
   log4j.appender.logfile.layout=org.apache.log4j.PatternLayout  
   log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n  
   ```



#### 4.3.2 创建Zookeeper客户端

```java
public class ZookeeperTest {

    //声明字符串类型的Zookeeper集群的服务器:端口号列表,多个节点使用","连接
    String connectionString = "hadoop100:2181,hadoop101:2181,hadoop102:2181";
    //声明超时时限(毫秒):默认范围4000-40000ms,超过设置范围会自动按照范围内的最大/最小值配置
    int sessionTimeOut = 10000;
    //声明Zookeeper集群的对象
    private ZooKeeper zkClient;

    //Before方法在所有本类中方法执行前优先执行
    @Before
    public void before() throws IOException {
        
        //使用声明的服务器地址,超时时限,监视器对象获取Zookeeper集群对象
        zkClient = new ZooKeeper(connectionString, sessionTimeOut, new Watcher() {
            //Watcher为监视器接口,这里使用匿名实现类的方式提供监听器对象
            public void process(WatchedEvent watchedEvent) {
                //process()方法为监听器监听到数据变化时调用的方法,具体的功能(如返回提示信息)在该方法内实现
            }
        });
        
    }
}
```



#### 4.3.3 Zookeeper的API操作

```java
public class ZookeeperTest {

    String connectionString = "hadoop100:2181,hadoop101:2181,hadoop102:2181";
    int sessionTimeOut = 10000;
    private ZooKeeper zkClient;

    @Before
    public void before() throws IOException {
        zkClient = new ZooKeeper(connectionString, sessionTimeOut, new Watcher() {
            public void process(WatchedEvent watchedEvent) {
            }
        });
    }

    //查看路径下的子节点,等同于ls命令
    @Test
    public void testGetChildren() throws KeeperException, InterruptedException {
        //返回String型集合,记录路径下所有子节点名
        List<String> children = zkClient.getChildren("/", false);//boolean参数表示是否监听该节点
        for (String child : children) {
            System.out.println(child);
        }
    }

    //创建节点,等同于create命令
    @Test
    public void testCreate() throws KeeperException, InterruptedException {
        String java = zkClient.create(
                "/atguigu/java",	//创建的节点路径
                "JavaNO1".getBytes(),	//Byte数组类型,节点的值
                ZooDefs.Ids.OPEN_ACL_UNSAFE,	//节点权限
                CreateMode.PERSISTENT);	//节点类型(持久/临时,显式/隐式序列化编号)
        System.out.println(java);
    }

    //节点数据的读取和设置,等同于get和set命令
    @Test
    public void testGetSetData() throws KeeperException, InterruptedException {
        Stat stat = zkClient.exists("/atguigu/java", false);//boolean参数表示是否监听该节点
        if (stat == null){
            System.out.println("节点不存在");
        }else{
            //读取节点数据
            byte[] data = zkClient.getData("/atguigu/java", false, null);
            System.out.println(new String(data));
            //设置节点数据
            zkClient.setData("/atguigu/java",
                    "JavaNO2".getBytes(),
                    stat.getVersion());//乐观锁机制:提供版本号用于检测,设置为-1则不检测
        }
    }

    //删除节点,等同于delete命令,不支持递归删除,递归需要手动实现
    @Test
    public void testDelete() throws KeeperException, InterruptedException {
        Stat stat = zkClient.exists("/atguigu/java", false);//boolean参数表示是否监听该节点
        if (stat == null){
            System.out.println("节点不存在");
        }else {
            zkClient.delete("/atguigu/java", stat.getVersion());
        }
    }

    //获取节点信息,等同于stat命令,可用于判断节点是否存在
    @Test
    public void testStat() throws KeeperException, InterruptedException {
        //Stat类将节点的信息作为属性进行了封装
        Stat stat = zkClient.exists("/atguigu", false);//boolean参数表示是否监听该节点
        //若返回值为null则说明节点不存在
        stat.getPzxid();
        stat.getDataLength();
        stat.getVersion();
    }

    //关闭资源
    @After
    public void zkTest() throws IOException, InterruptedException {
        zkClient.close();
    }
}
```



#### 4.3.4 获取子节点并监听节点变化

```java
@Test
public void getChildren() throws Exception {

    //获取并监听根目录下所有子节点
    List<String> children = zkClient.getChildren("/", true);

    //延时阻塞
    Thread.sleep(Long.MAX_VALUE);
    
    //当节点下数据发生变化时,会输出信息到控制台,仅监听1次
}
```



### 4.4 监听服务器节点动态上下线

- 服务端

  ```java
  public class DistributeServer {
  
  	private static String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
  	private static int sessionTimeout = 2000;
  	private ZooKeeper zk = null;
  	private String parentNode = "/servers";
  	
  	//创建客户端与服务端的连接(获取Zookeeper集群对象)
  	public void getConnect() throws IOException{
  		zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
  			public void process(WatchedEvent event) {
  			}
  		});
  	}
  	
  	//注册服务器(在指定Zookeeper服务器上创建短暂节点)
  	public void registServer(String hostname) throws Exception{
  		String create = zk.create(parentNode + "/server", hostname.getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
  		System.out.println(hostname + " is online " + create);
  	}
  	
  	//业务功能(输出创建的节点信息)
  	public void business(String hostname) throws Exception{
  		System.out.println(hostname + " is working ...");
  		Thread.sleep(Long.MAX_VALUE);
  	}
  	
  	public static void main(String[] args) throws Exception {
  		//1.获取zk连接
  		DistributeServer server = new DistributeServer();
  		server.getConnect();		
  		//2.利用zk连接注册服务器信息
  		server.registServer(args[0]);		
  		//3.启动业务功能
  		server.business(args[0]);
  	}
  }
  ```

- 客户端

  ```java
  public class DistributeClient {
  
  	private static String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
  	private static int sessionTimeout = 2000;
  	private ZooKeeper zk = null;
  	private String parentNode = "/servers";
  
  	//创建客户端与服务端的连接(获取Zookeeper集群对象)
  	public void getConnect() throws IOException {
  		zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
  			//重写process()实现循环监听
               @Override
  			public void process(WatchedEvent event) {
  				//再次启动监听
  				try {
  					getServerList();
  				} catch (Exception e) {
  					e.printStackTrace();
  				}
  			}
  		});
  	}
  
  	// 获取服务器列表信息
  	public void getServerList() throws Exception {
  		//获取服务器子节点信息，并且对父节点进行监听
  		List<String> children = zk.getChildren(parentNode, true);
           //存储服务器信息列表
  		ArrayList<String> servers = new ArrayList<>();
           //遍历所有节点，获取节点中的主机名称信息
  		for (String child : children) {
  			byte[] data = zk.getData(parentNode + "/" + child, false, null);
  			servers.add(new String(data));
  		}
           //打印服务器列表信息
  		System.out.println(servers);
  	}
  
  	//业务功能
  	public void business() throws Exception{
  		System.out.println("client is working ...");
  Thread.sleep(Long.MAX_VALUE);
  	}
      
  	public static void main(String[] args) throws Exception {
  		//1.获取zk连接
  		DistributeClient client = new DistributeClient();
  		client.getConnect();
  		//2.获取servers的子节点信息，从中获取服务器信息列表
  		client.getServerList();
  		//3.业务进程启动
  		client.business();
  	}
  }
  ```

  

# HA

## 第一章 Hadoop HA 高可用

### 1.1 HA概述

- HA（High Availablity），意为高可用。是Hadoop为消除单点故障提供的一种方案机制。具体包括HDFS-HA和YARN-HA。
- 单点故障：NameNode由于意外或需要升级，出现暂时无法使用的情况。
- HDFS HA通过配置两个（hadoop2）或多个（hadoop3）NameNode节点，实现集群中对NameNode的热备。出现单点故障时，NameNode可以直接切换到另一节点。



### 1.2 HDFS-HA 工作机制

- 在集群中多个节点上启动NameNode。集群工作时，仅有1个NameNode处于Active状态，可以进行读写操作，另外的NameNode处于Standby状态，仅有读取权限，实时读取共享的Edits文件中的内容，实现多个NameNode的数据同步。
- 在HA的工作模式下，原来SecondaryNameNode的功能集成在Standby的NameNode中，因此不再需要额外启动SecondaryNameNode。
- 隔离机制：同一时刻下，只有Active的NameNode对外提供服务。



### 1.3 HDFS-HA 配置手动故障转移

1. 前提准备：Java、Hadoop集群已完成安装并配置了环境变量。NameNode节点间实现了免密登陆。

2. 创建目录用于存放HA集群

   ```shell
   sudo mkdir /opt/ha
   cd /opt
   sudo chmod 777 ha
   ```

3. 拷贝原Hadoop目录下的内容

   ```shell
   cp /opt/module/hadoop-3.1.3 /opt/ha/
   ```

4. 修改core-site.xml配置文件

   ```xml
   <configuration>
       <!-- 将多个NameNode的地址封装为集群并命名 -->
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://mycluster</value>
       </property>
       <!-- 指定hadoop运行时产生文件的存储目录 -->
       <property>
           <name>hadoop.tmp.dir</name>
           <value>/opt/ha/hadoop-3.1.3/data/tmp</value>
       </property>
       <!-- 声明journalnode服务器存储目录-->
       <property>
           <name>dfs.journalnode.edits.dir</name>
           <value>file://${hadoop.tmp.dir}/jn</value>
       </property>
   </configuration>
   ```

5. 修改hdfs-site.xml配置文件

   ```xml
   <configuration>
       <!-- 完全分布式集群名称 -->
       <property>
           <name>dfs.nameservices</name>
           <value>mycluster</value>
       </property>
       <!-- NameNode数据存储目录 -->
       <property>
           <name>dfs.namenode.name.dir</name>
           <value>file://${hadoop.tmp.dir}/name</value>
       </property>
       <!-- DataNode数据存储目录 -->
       <property>
           <name>dfs.datanode.data.dir</name>
           <value>file://${hadoop.tmp.dir}/data</value>
       </property>
   
       <!-- 集群中NameNode节点都有哪些 -->
       <property>
           <name>dfs.ha.namenodes.mycluster</name>
           <value>nn0,nn1,nn2</value>
       </property>
   
       <!-- nn0的RPC通信地址 -->
       <property>
           <name>dfs.namenode.rpc-address.mycluster.nn0</name>
           <value>hadoop100:9000</value>
       </property>
       <!-- nn1的RPC通信地址 -->
       <property>
           <name>dfs.namenode.rpc-address.mycluster.nn1</name>
           <value>hadoop101:9000</value>
       </property>
       <!-- nn2的RPC通信地址 -->
       <property>
           <name>dfs.namenode.rpc-address.mycluster.nn2</name>
           <value>hadoop102:9000</value>
       </property>
       
       <!-- nn0的http通信地址 -->
       <property>
           <name>dfs.namenode.http-address.mycluster.nn0</name>
           <value>hadoop100:9870</value>
       </property>
       <!-- nn1的http通信地址 -->
       <property>
           <name>dfs.namenode.http-address.mycluster.nn1</name>
           <value>hadoop101:9870</value>
       </property>
       <!-- nn2的http通信地址 -->
       <property>
           <name>dfs.namenode.http-address.mycluster.nn2</name>
           <value>hadoop102:9870</value>
       </property>
   
       <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
       <property>
           <name>dfs.namenode.shared.edits.dir</name>
           <value>qjournal://hadoop100:8485;hadoop101:8485;hadoop102:8485/mycluster</value>
       </property>
       
       <!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
       <property>
           <name>dfs.ha.fencing.methods</name>
           <value>sshfence</value>
       </property>
   
       <!-- 使用隔离机制时需要ssh无秘钥登录-->
       <property>
           <name>dfs.ha.fencing.ssh.private-key-files</name>
           <value>/home/atguigu/.ssh/id_rsa</value>
       </property>
   
       <!-- 访问代理类：client用于确定哪个NameNode为Active -->
       <property>		
           <name>dfs.client.failover.proxy.provider.mycluster</name>
           <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
       </property>
   </configuration>
   ```

6. 删除Hadoop目录下的data和logs目录

   ```shell
   cd /opt/ha/hadoop-3.1.3/
   rm -rf data logs
   ```

7. 将配置完成的hadoop整个拷贝到其他节点

   ```shell
   #先在其他节点上新建目录
   xsync /opt/ha/hadoop-3.1.3
   ```

8. 修改**所有**节点的HADOOP_HOME环境变量

   ```shell
   vi /etc/profile.d/my_env.sh 
   
   #修改HADOOP_HOME地址
   HADOOP_HOME=/opt/ha/hadoop-3.1.3
   ```

9. 在**所有**NameNode节点上启动JournalNode服务

   ```shell
   hdfs --daemon start jornalnode
   ```

10. 在任一NameNode节点上格式化并启动NameNode

    ```shell
    #正确的格式化过程不会跳出询问,若跳出需要确认data和logs是否删除
    #若删除后依旧没有解决,可以停止所有jps服务,然后删除/tmp/*
    hdfs namenode -format
    hdfs --daemon start namenode
    ```

11. 在其余节点上同步元数据信息

    ```shell
    hdfs namenode -bootstrapStandby
    ```

12. 在其余节点上启动NameNode

    ```shell
    hdfs --daemon start namenode
    ```

13. 查看web端三个节点状态

    ```shell
    hadoop100:9870
    hadoop101:9870
    hadoop102:9870
    #此时均显示为standby状态
    ```

14. 手动设置Active节点

    ```shell
    #设置nn0为Active节点
    hdfs haadmin -transitionToActive nn0
    #查看节点状态
    hdfs haadmin -getServiceState nn0
    #设置nn0为standby节点
    hdfs haadmin -transitionToStandby nn0
    ```



### 1.4 HDFS-HA 配置自动故障转移

#### 1.4.1 Zookeeper与ZKFC

- 要实现NameNode故障的自动转移，需要Zookeeper与多个NameNode间创建1个持久会话，用于监听NameNode的工作状态。
- ZKFC（Zookeeper Failover Controller）是Zookeeper的客户端，该进程随每个NameNode的启动和而创建，负责监听和管理该NameNode的状态。
- ZKFC的具体功能包括
  1. NameNode健康监测：定时检测所属NameNode的状态，并予以标识。
  2. Zookeeper会话管理：每个ZKFC在Zookeeper中维护一个短暂节点，当NameNode故障时，ZKFC终止会话，短暂节点被删除。
  3. 为NameNode获取Active状态：当ZKFC所属的NameNode状态健康，且无Active的NameNode节点时，ZKFC会向Zookeeper发起获取Active状态的请求。多个ZKFC同时发起请求时，先到先得。

#### 1.4.2 配置过程

1. 前提：完成1.3中的配置过程，Zookeeper集群搭建完成

2. 修改hdfs-site.xml配置文件，添加以下内容

   ```xml
   <property>
       <name>dfs.ha.automatic-failover.enabled</name>
       <value>true</value>
   </property>
   ```

3. 修改core-site.xml文件，添加以下内容

   ```xml
   <property>
       <name>ha.zookeeper.quorum</name>
       <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
   </property>
   ```

4. 同步修改到所有节点

   ```shell
   xsync hdfs-site.xml
   xsync core-site.xml
   ```

5. 使用脚本启动Zookeeper集群

   ```shell
   zkctl start
   ```

6. 初始化HA在Zookeeper中的状态

   ```shell
   hdfs zkfc -format
   ```

7. 启动hdfs服务

   ```shell
   start-dfs.sh
   ```

8. web端查看，此时Active的NameNode已经自动选出，其余节点为Standby状态



### 1.5 YARN-HA 配置（集群脚本）

- 配置多节点启动ResourceManager

1. 修改yarn-site.xml配置文件

   ```xml
   <configuration>
       
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
   
       <!-- 启用resourcemanager ha -->
       <property>
           <name>yarn.resourcemanager.ha.enabled</name>
           <value>true</value>
       </property>
   
       <!-- 声明HA resourcemanager的地址 -->
       <property>
           <name>yarn.resourcemanager.cluster-id</name>
           <value>cluster-yarn1</value>
       </property>
       <!-- 指定RM的逻辑列表 -->
       <property>
           <name>yarn.resourcemanager.ha.rm-ids</name>
           <value>rm0,rm1,rm2</value>
       </property>
   
       <!-- ===========rm0 配置============ -->
       <!-- 指定rm0的主机名 -->
       <property>
           <name>yarn.resourcemanager.hostname.rm0</name>
           <value>hadoop100</value>
       </property>
       <!-- 指定rm0的web端地址 -->
       <property>
           <name>yarn.resourcemanager.webapp.address.rm0</name>
           <value>hadoop100:8088</value>
       </property> 
       <!-- 指定rm0的内部通信地址 -->
       <property>
           <name>yarn.resourcemanager.address.rm0</name>
           <value>hadoop100:8032</value>
       </property>
       <!-- 指定AM向rm0申请资源的地址 -->
       <property>
           <name>yarn.resourcemanager.scheduler.address.rm0</name>  
           <value>hadoop100:8030</value>
       </property>
       <!-- 指定供NM连接的地址 -->  
       <property>
           <name>yarn.resourcemanager.resource-tracker.address.rm0</name>
           <value>hadoop100:8031</value>
       </property>
   
       <!-- ===========rm1 配置============ --> 
       <property>
           <name>yarn.resourcemanager.hostname.rm1</name>
           <value>hadoop101</value>
       </property>
   
       <property>
           <name>yarn.resourcemanager.webapp.address.rm1</name>
           <value>hadoop101:8088</value>
       </property>
       <property>
           <name>yarn.resourcemanager.address.rm1</name>
           <value>hadoop101:8032</value>
       </property>
       <property>
           <name>yarn.resourcemanager.scheduler.address.rm1</name>
           <value>hadoop101:8030</value>
       </property>
       <property>
           <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
           <value>hadoop101:8031</value>
       </property>
   
       <!-- ===========rm2 配置============ --> 
       <property>
           <name>yarn.resourcemanager.hostname.rm2</name>
           <value>hadoop102</value>
       </property>
       <property>
           <name>yarn.resourcemanager.webapp.address.rm2</name>
           <value>hadoop102:8088</value>
       </property>
       <property>
           <name>yarn.resourcemanager.address.rm2</name>
           <value>hadoop102:8032</value>
       </property>
       <property>
           <name>yarn.resourcemanager.scheduler.address.rm2</name>
           <value>hadoop102:8030</value>
       </property>
       <property>
           <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
           <value>hadoop102:8031</value>
       </property>
   
       <!--指定zookeeper集群的地址--> 
       <property>
           <name>yarn.resourcemanager.zk-address</name>
           <value>hadoop100:2181,hadoop101:2181,hadoop102:2181</value>
       </property>
   
       <!--启用自动恢复--> 
       <property>
           <name>yarn.resourcemanager.recovery.enabled</name>
           <value>true</value>
       </property>
   
       <!--指定resourcemanager的状态信息存储在zookeeper集群--> 
       <property>
           <name>yarn.resourcemanager.store.class</name>	
           <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
       </property>
   
       <!-- 环境变量的继承 -->
       <property>
           <name>yarn.nodemanager.env-whitelist</name>
           <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
       </property>
   
   </configuration>
   ```

2. 同步配置文件到其他节点

   ```shell
   xsync yarn-site.xml
   ```

3. 启动HDFS

   ```shell
   start-dfs.sh
   ```

4. 启动YARN

   ```shell
   start-yarn.sh
   ```

5. 查看服务状态

   ```shell
   #查看rm0节点的ResourceManager状态
   yarn rmadmin -getServiceState rm0
   
   #web端查看
   hadoop100:8080
   ```

- 编写HA集群群起，群停脚本

  ```shell
  #在当前用户家目录的bin目录下编写脚本文件(无需配置环境变量)
  vi /home/atguigu/bin/hactl
  
  #编辑脚本 传参为start/stop
  #!/bin/bash
  if [ $# -lt 1 ]
  	then
  		echo "Input args is null"
  		exit
  fi
  case $1 in
  "start")
  	echo "---------- hdfs start -----------"
  	ssh hadoop100 /opt/ha/hadoop-3.1.3/sbin/start-dfs.sh
  	echo "---------- yarn start -----------"
  	/opt/ha/hadoop-3.1.3/sbin/start-yarn.sh
  	;;
  "stop")
  	echo "---------- yarn stop -----------"
  	ssh hadoop100 /opt/ha/hadoop-3.1.3/sbin/stop-yarn.sh
  	echo "---------- hdfs stop -----------"
  	/opt/ha/hadoop-3.1.3/sbin/stop-dfs.sh
  	;;
  *)
  	echo "Input args not found"
  	;;
  esac
  ```

- 启动后的所有节点的jps进程包括

  ```shell
  ResourceManager
  QuorumPeerMain	#Zookeeper进程
  NameNode
  DFSZKFailoverController	#ZKFC进程
  JournalNode
  NodeManager
  Jps
  DataNode
  ```



### 1.6 HDFS Federation 架构

- NameNode架构的局限性
  1. Namespace（命名空间）有限
  2. 隔离性差，一个程序的执行可能影响到其他程序
  3. 无法拓展，单个NameNode的吞吐量上限决定了整个HDFS的吞吐量上限
- HDFS Federation架构
  - 不同于现在的元数据全部存储在1个NameNode节点中，HDFS Federation架构将不同类型的元数据分别存储在多个NameNode节点中，同时解决了NameNode的扩容和数据隔离问题。



## 附：该阶段HA的四个配置文件

### core-site.xml

```xml
<configuration>

    <!-- 将多个NameNode的地址合成一个集群mycluster -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
    </property>

    <!-- 指定hadoop运行时产生文件的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/ha/hadoop-3.1.3/data/tmp</value>
    </property>

    <!-- 声明journalnode服务器存储目录 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>file://${hadoop.tmp.dir}/jn</value>
    </property>
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>hadoop100:2181,hadoop101:2181,hadoop102:2181</value>
    </property>

    <!-- 兼容性配置 -->
    <property>
        <name>hadoop.proxyuser.atguigu.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.atguigu.groups</name>
        <value>*</value>
    </property>
    
    <!-- 修改web访问NameNode的默认用户 -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
    </property>
    
</configuration>
```

### hdfs-site.xml

```xml
<configuration>
    
    <!-- 完全分布式集群名称 -->
    <property>
        <name>dfs.nameservices</name>
        <value>mycluster</value>
    </property>
    <!-- NameNode数据存储目录 -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file://${hadoop.tmp.dir}/name</value>
    </property>
    <!-- DataNode数据存储目录 -->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file://${hadoop.tmp.dir}/data</value>
    </property>

    <!-- 声明集群中的所有NameNode节点 -->
    <property>
        <name>dfs.ha.namenodes.mycluster</name>
        <value>nn0,nn1,nn2</value>
    </property>

    <!-- nn1的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn0</name>
        <value>hadoop100:9000</value>
    </property>
    <!-- nn2的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn1</name>
        <value>hadoop101:9000</value>
    </property>
    <!-- nn3的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn2</name>
        <value>hadoop102:9000</value>
    </property>

    <!-- nn1的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.mycluster.nn0</name>
        <value>hadoop100:9870</value>
    </property>
    <!-- nn2的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.mycluster.nn1</name>
        <value>hadoop101:9870</value>
    </property>
    <!-- nn3的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.mycluster.nn2</name>
        <value>hadoop102:9870</value>
    </property>

    <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://hadoop100:8485;hadoop101:8485;hadoop102:8485/mycluster</value>
    </property>

    <!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>

    <!-- 使用隔离机制时需要ssh无秘钥登录-->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/home/atguigu/.ssh/id_rsa</value>
    </property>

    <!-- 访问代理类：client用于确定哪个NameNode为Active -->
    <property>		
        <name>dfs.client.failover.proxy.provider.mycluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>

    <!-- 兼容性配置 -->
    <property>
        <name>dfs.client.datanode-restart.timeout</name>
        <value>30s</value>
    </property>
    
</configuration>
```

### yarn-site.xml

```xml
<configuration>
    
    <!-- Reducer获取数据的方式 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 启用resourcemanager ha -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>

    <!-- 声明HA resourcemanager的地址 -->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster-yarn1</value>
    </property>
    <!-- 指定RM的逻辑列表 -->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm0,rm1,rm2</value>
    </property>

    <!-- =========== rm0配置 ============ --> 
    <!-- 指定rm0的主机名 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm0</name>
        <value>hadoop100</value>
    </property>
    <!-- 指定rm0的web端地址 -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm0</name>
        <value>hadoop100:8088</value>
    </property>
    <!-- 指定rm0的内部通信地址 -->
    <property>
        <name>yarn.resourcemanager.address.rm0</name>
        <value>hadoop100:8032</value>
    </property>
    <!-- 指定AM向rm0申请资源的地址 -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm0</name>  
        <value>hadoop100:8030</value>
    </property>
    <!-- 指定供NM连接的地址 -->  
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm0</name>
        <value>hadoop100:8031</value>
    </property>

    <!-- =========== rm1配置 ============ --> 
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop101</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>hadoop101:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>hadoop101:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>
        <value>hadoop101:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
        <value>hadoop101:8031</value>
    </property>

    <!-- =========== rm2配置 ============ --> 
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>hadoop102</value>
    </property>

    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>hadoop102:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>hadoop102:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>hadoop102:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>hadoop102:8031</value>
    </property>

    <!-- 指定zookeeper集群的地址 --> 
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop100:2181,hadoop101:2181,hadoop102:2181</value>
    </property>

    <!-- 启用自动恢复 --> 
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>

    <!-- 指定resourcemanager的状态信息存储在zookeeper集群 --> 
    <property>
        <name>yarn.resourcemanager.store.class</name>     
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>

    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>

    <!-- 配置日志聚集 -->
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
    
    <!-- 解决虚拟内存超标 -->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
    
</configuration>
```

### mapred-site.xml

```xml
<configuration>
    
    <!-- 指定MR运行在YARN上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    
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
    
</configuration>
```

