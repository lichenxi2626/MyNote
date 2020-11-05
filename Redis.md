# Redis

## 第一章 Redis介绍

### 1.1 Redis概述

- Redis是一个开源的key-value存储系统，value对应多种数据类型，是一种NoSQL数据库。
- Redis将数据缓存在内存中，并可以周期性的写入磁盘。
- 使用场景
  1. 配合关系型数据库做高速缓存
  2. 实时计算中存储临时数据
  3. 根据需求持久化存储特定结构的数据（排序、定时、秒杀、去重等）
- 官方网站
  - https://redis.io/
  - https://www.redis.net.cn/

### 1.2 对比Hbase

- 相同点
  - 基于key-value方式存储数据，不适合用于数据分析查询
- 不同点
  1. 数据量：hbase远大于redis
  2. 性能：redis存取效率更高，用于数据经常变化的场景
  3. 时效性：hbase用于存储长期的数据，redis用于高频访问的临时数据



### 1.3 安装redis

1. 上传并解压安装包到指定目录

   ```shell
   tar -zxvf redis-3.2.5.tar.gz -C /opt/module
   ```

2. 安装相关依赖

   ```shell
   sudo yum install gcc-c++
   ```

3. 进入安装目录，执行make命令

   ```shell
   cd /opt/module/redis-3.2.5
   make
   ```

4. 执行make install命令

   ```shell
   sudo make install
   
   #安装完成后查看redis相关命令(命令已自动配置到环境变量)
   cd usr/local/bin
   ```

5. 复制redis.conf目录，并修改配置文件

   ```shell
   cp /opt/module/redis-3.2.5/redis.conf ~/redis.conf
   vi ~/redis.conf
   ```

   ```shell
   #配置后台运行(默认no)
   daemonize yes
   ```

6. 使用修改的配置文件启动redis server

   ```shell
   redis-server ~/redis.conf
   ```

7. 启动redis客户端

   ```shell
   redis-cli
   ```

8. 关闭redis server

   ```shell
   #方式1:在客户端中关闭
   shutdown
   
   #方式2:使用客户端命令关闭
   redis-cli shutdown
   ```



## 第二章 Redis数据类型 

### 2.1 数据库操作

- redis默认具有16个数据库，下标从0开始，默认使用0号数据库
- 所有库的密码相同，统一管理
- **Redis中存储的是key-value型数据，key的数据类型均为String，而Redis数据类型特指value的类型，共有5种**

```shell
#1.切换当前数据库
select [dbid]
select 10

#2.查看当前库的所有key
keys *

#3.判断指定的key是否存在,每存在1个,返回值+1
exists [key]...
exists 1001

#4.删除一或多个key
del [key]...
del 1001

#5.为key设定有效时间(秒),过期自动删除
expire [key] [seconds]
expire 1001 10

#6.清空当前库
flushdb

#7.清空所有库(触发rdb存盘)
flushall
```



### 2.2 String

- String是Redis的最基本数据类型，等价于java中的String型kv对
- redis中的String类型时二进制安全的，即支持jpg图片或序列化的对象存储
- String型的value最大可达512M
- 由于redis为单线程，因此对value的计算操作具有**原子性**

```shell
#1.使用key查询的value
get [key]
get 1001

#2.添加string型kv对到当前库,key值已存在时会覆盖value
set [key] [value]
set 1001 zhangsan

#3.添加string型kv对到当前库,同时设置过期时间
setex [key] [seconds] [value]
setex 1002 10 lisi

#4.若key不存在,则添加kv对到当前数据库
setnx [key] [value]
setnx 1002 lisi

#5.指定key的value值+1(value必须为整形)
incr [key]
set 1003 100
incr 1003

#4.指定key的value值-1(value必须为整形)
decr [key]
set 1004 100
decr 1004

#5.指定key的value值±指定步长(value必须为整形)
incrby [key] [increament]
decrby [key] [decreament]
incrby 1003 10
decrby 1003 10

#6.同时添加多个kv对到数据库
mset [key1] [value1] [key2] [value2] ...
mset 1001 zhangsan 1002 lisi

#7.获取多个key的value
mget [key1] [key2] ...
mget 1001 1002

#8.使用bit索引位置操作数据
getbit [key] [offset]
setbit [key] [offset] [value]
```



### 2.3 List

- List数据结构为一个String型key对应多个String型数据组成的集合
- List中插入值的方式类似于队列（先进先出）
- List底层使用**双向链表**存储，对两端操作性能高，使用索引下标操作性能差

```shell
#1.从左边/右边向指定List插入一或多个值
lpush / rpush [key] [value1] [value2] [value3]...
lpush 1010 zhangsan lisi wangwu

#2.从List左边/右边取出一个值,取出后会删除该值(当从List中取出最后一个值时,key会随之删除)
lpop / rpop [key]
rpop 1010

#3.从key1的List右边取出值,插入key2的List左边
rpoplpush [key1] [key2]
rpoplpush 1010 1011

#4.获得List中连续n个元素(不删除)
lrange [key] [startindex] [endindex]
lrange 1010 0 -1 #-1表示遍历到最后一个元素

#5.获得List中指定索引位置的元素(不删除)
lindex [key] [index]
lindex 1010 0

#6.获取指定key的List长度
llen [key]
llen 1010
```



### 2.4 Set

- Set数据结构为一个String型key对应多个String型数据组成的集合，集合中的数据会**自动去重**
- Set中的数据是**无序的**
- Set底层使用value为null的hash表存储

```shell
#1.添加一或多个元素到Set
sadd [key] [value1] [value2] [value3]...
sadd 1020 zhangsan lisi wangwu

#2.取出指定Set的所有值
smembers [key]
smembers 1020

#3.判断指定Set中是否含有指定值
sismember [key] [value]
sismember 1020 zhangsan

#4.删除Set中的指定元素
srem [key] [value]
srem 1020 lisi

#5.返回2个Set的交集
sinter [key1] [key2]
sinter 1020 1021

#6.返回2个Set的并集
sunion [key1] [key2]
sunion 1020 1021

#7.返回2个Set的差集
sdiff [key1] [key2]
sdiff 1020 1021
```



### 2.5 Hash

- Hash数据结构为一个String型key对应多个String型kv对组成的集合
- Hash常用于存储对象，集合中的每个kv对对应对象的属性和值

```shell
#1.添加一个kv对到Hash集合
hset [key] [field] [value]
hset 1030 name zhangsan

#2.获得一个Hash集合中指定field的值
hget [key] [field]
hget 1030 name

#3.批量添加kv对到Hash集合
hmset [key] [field1] [value1] [field2] [value2]...
hmset 1031 name lisi age 20

#4.查看指定的Hash集合是否含有某个field
hexists [key] [field]
hexists 1030 age

#5.获取Hash集合中的所有属性和值
hgetall [key]
hgetall 1031

#6.对指定Hash集合的指定field的值+n(仅限整型)
hincrby [key] [field] [increament]
hincrby 1031 age 5
```



### 2.6 Zset

- 与Set类似，同样自带去重功能，同时对集合中的每个值额外关联的一个score，用于排序，默认升序，score可以重复
- 由于Zset是有序的，因此可以通过下标获取指定元素

```shell
#1.添加一或多个元素到Zset集合中
zadd [key] [score1] [value1] [score2] [value2]...
zadd 1040 100 zhangsan 90 lisi

#2.返回指定索引区间内的元素(及score)
zrange [key] [startindex] [endindex] withscores
zrange 1040 0 1 #索引范围为闭区间
zrange 1040 0 1 withscores

#3.降序返回指定索引区间内的元素(及score)
zrevrange [key] [startindex] [endindex] withscores
zrevrange 1040 0 1 withscores

#4.对Zset集合中的指定value的score+n(仅限整型)
zincrby [key] [increament] [value]
zincrby 1040 5 lisi

#5.删除Zset集合中的指定value
zrem [key] [value]
zrem 1040 lisi

#6.获取指定score区间内的元素个数(闭区间)
zcount [key] [min] [max]
zcount 1040 95 100

#7.获取Zset中指定value的索引
zrank [key] [value]
zrank 1040 lisi
```



## 第三章 Java客户端Jedis

### 3.1 配置Redis

1. 配置Redis密码

   ```shell
   vi ~/redis.conf
   ```

   ```shell
   #修改以下配置
   #1.关闭保护模式(若开启则仅能在本机客户端访问redis-server)
   protected-mode no
   
   #2.注释ip绑定(生产环境下需要绑定应用服务器地址)
   # bind 127.0.0.1
   
   #3.设置密码(出于安全考虑,若绑定了ip则无需设置)
   requirepass 123456
   ```

   ```shell
   #设置密码后使用客户端操作redis时,需要先输入密码
   auth 123456
   ```

2. 重启redis



### 3.2 Jedis访问Redis单点模式

1. 创建Maven工程，添加Redis依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>3.2.0</version>
       </dependency>
   </dependencies>
   ```

2. 编写代码

   ```java
   public class RedisTest {
       public static void main(String[] args) {
   
           //1.通过url及端口号获取jedis对象
           Jedis jedis = new Jedis("hadoop201", 6379);
   
           //2.指定命令
           System.out.println(jedis.ping());
   
           //3.获取全部key
           Set<String> keys = jedis.keys("*");
           for (String key : keys) {
               System.out.println(key);
           }
           
           //3.关闭jedis
           jedis.close();
   
       }
   }
   ```

   



## 第四章 Redis持久化

- Redis提供了2种不同的数据持久化方式
  1. RDB（Redis DataBase）
  2. AOF（Appen Of File）



### 4.1 RDB持久化

- 概述：根据设定的时间间隔，定期将内存中Redis的**全量数据集快照替换写入磁盘文件**中，重新启动Redis时会自动读取快照文件到内存中。

- 实现原理：Redis会单独创建一个子进程用于持久化（写时复制技术），不断将新的数据写入一个临时文件中，直到**触发存盘条件**后，该临时文件会替换上次的持久化完成的文件。

- 以下操作**会触发数据存盘**，将Redis内存中的数据写入磁盘

  1. 触发配置文件中的RDB策略条件
  2. flushall命令（先清空所有库，再写入磁盘，**数据无法找回**）
  3. save命令（阻塞Redis线程，待存盘完成后再继续）
  4. shutdown命令
  5. 控制台kill命令结束redis-server进程（等价于shutdown）

- 以下操作**不会触发数据存盘**，因此**操作与上次存盘间的数据不会保留**

  1. flunshdb
  2. 控制台kill -9命令
  3. 意外宕机

- 配置RDB持久化（RDB默认情况下为开启状态）

  1. 修改配置文件

     ```shell
     vi ~/redis.conf
     ```

     ```shell
     #修改以下配置
     #1.磁盘中RDB文件的名称
     dbfilename dump.rdb
     
     #2.RDB文件的存储目录
     dir /home/atguigu/redis-datas
     
     #3.配置rdb的自动存盘策略,多行策略为或的关系,满足其一则触发存盘
     save 900 1  #说明:距上次存盘已达900秒且有1个key产生变化
     save 300 10
     save 60 10000
     
     #4.若redis存盘出现异常,则终止redis内存中的写操作(默认yes)
     stop-writes-on-bgsave-error yes
     
     #5.rdb存盘时,使用压缩(默认yes)
     rdbcompression yes
     
     #6.rdb存盘时,对存盘文件进行校验确保数据正确,消耗约10%的redis性能(默认yes)
     rdbchecksum yes
     ```

  2. 启动redis

     ```shell
     redis-server ~/redis.conf
     redis-cli
     #客户端启动后会自动到配置目录下读取rdb文件,恢复数据
     ```



### 4.2 AOF持久化

- 概述：以日志为载体记录redis的每个**写操作**，这些操作被不断**追加到日志文件中**，重新启动Redis时会自动读取日志文件，通过依次执行日志中的操作达到重建数据的效果。

- AOF文件的重写：因为AOF日志文件采用了追加的方式，为避免日志文件过大，增加了重写机制。当日志文件达到一定大小时，会触发重写，根据key值分别聚合所有的写操作，为每个key值只保留一条表示value结果的set语句，最后替换原来的日志文件。

- 配置AOF持久化

  1. 修改配置文件

     ```shell
     vi ~/redis.conf
     ```

     ```shell
     #1.开启AOF持久化(默认为no)
     appendonly yes
     
     #2.配置AOF日志文件名
     appendfilename "appendonly.aof"
     
     #3.配置日志文件的存储目录(与RDB公用同一目录)
     dir /home/atguigu/redis-datas
     
     #4.配置AOF同步频率(权衡数据安全和性能)
     # appendfsync always  #每次写操作追加进日志(数据绝对安全)
     appendfsync everysec  #每秒内的写操作一同追加进日志(数据较安全)
     # appendfsync no      #操作系统判定同步时机(数据不安全)
     
     #5.配置日志文件的重写条件
     auto-aof-rewrite-percentage 100
     auto-aof-rewrite-min-size 64mb
     #说明
     #重写条件为 (aof_size >= base_size * (1 + 100%)) && (aof_size >= 64mb)
     # base_size:上次重写后的aof文件大小
     # aof_size:当前aof文件大小
     ```

  2. 启动redis

     ```shell
     redis-server ~/redis.conf
     redis-cli
     #同时存在RDB和AOF文件时,系统默认读取AOF日志文件的数据
     ```

- 在redis进行高速写入时，AOF文件可能产生损坏，可使用redis自带的修复命令修复

  ```shell
  redis-check-aof  --fix  appendonly.aof
  ```

- 不停机配置AOF持久化

  ```shell
  #在redis客户端中执行
  config set appendonly yes
  config set dir /home/atguigu/redis-datas
  #由于配置仅对当前进程生效,下次启动前仍需要手动修改配置文件
  ```



### 4.3 RDB与AOF对比

- RDB
  - 优点：节省磁盘空间，读取速度快
  - 缺点：写时拷贝需要消耗一定性能，可能造成数据丢失
- AOF
  - 优点：数据安全性高，日志文件可读且可修改（处理误操作）
  - 缺点：磁盘空间占用高，读取速度慢，存在bug
- 使用场景
  1. 推荐2个都开启
  2. 若可以容忍小范围数据丢失，则单独开启RDB



## 第五章 Redis主从复制

### 5.1 主从复制概述

- 主从复制：主机数据更新后，备机根据一定的配置和策略，自动同步主机中数据的Master/Slaver机制
- 作用
  1. Master以写为主，Slaver仅支持读操作，实现读写分离（需要借助代码或插件）
  2. 实现数据的容灾性，可快速恢复



### 5.2 配置主从模式

1. 编写三份redis配置文件

   ```shell
   mkdir /home/atguigu/redisconf
   vi redisconf/redis6379.conf
   ```

   ```shell
   #配置后台执行
   daemonize yes
   #配置端口号
   port 6379
   #配置rdb文件目录
   dir /home/atguigu/redis-datas
   #配置rdb文件名
   dbfilename dump6379.rdb
   #配置rdb的存盘策略
   save 300 10
   #配置pid文件名
   pidfile /var/run/redis_6379.pid
   #关闭保护模式(开启远端访问)
   protected-mode no
   ```

   ```shell
   vi redisconf/redis6380.conf
   ```

   ```shell
   #配置后台执行
   daemonize yes
   #配置端口号
   port 6380
   #配置rdb文件目录
   dir /home/atguigu/redis-datas
   #配置rdb文件名
   dbfilename dump6380.rdb
   #配置rdb的存盘策略
   save 300 10
   #配置pid文件名
   pidfile /var/run/redis_6380.pid
   #关闭保护模式(开启远端访问)
   protected-mode no
   ```

   ```shell
   vi redisconf/redis6381.conf
   ```

   ```shell
   #配置后台执行
   daemonize yes
   #配置端口号
   port 6381
   #配置rdb文件目录
   dir /home/atguigu/redis-datas
   #配置rdb文件名
   dbfilename dump6381.rdb
   #配置rdb的存盘策略
   save 300 10
   #配置pid文件名
   pidfile /var/run/redis_6381.pid
   #关闭保护模式(开启远端访问)
   protected-mode no
   #配置优先度
   slave-priority 10
   ```

2. 分别在三个终端窗口启动redis

   ```shell
   redis-server ~/redisconf/redis6379.conf
   ```

   ```shell
   redis-server ~/redisconf/redis6380.conf
   ```

   ```shell
   redis-server ~/redisconf/redis6381.conf
   ```

3. 分别在三个端口启动redis客户端

   ```shell
   redis-cli
   ```

   ```shell
   redis-cli -p 6380
   ```

   ```shell
   redis-cli -p 6381
   ```

4. 分别在将6380、6381两个终端中执行命令，设置为6379终端的从机（星型主从模型）

   ```shell
   slaveof hadoop201 6379
   ```

   ```shell
   slaveof hadoop201 6379
   ```

   ```shell
   #设置完成后可适用命令查看主从关系
   info replication
   
   #主从关系确定后,从机会向主机发起同步申请,主机执行存盘,再将RDB文件发送给从机读取,此后主机的每次写操作都会直接同步给从机
   #主机shutdown后,所有从机等待主机重启,主机重启后恢复主从关系
   #从机shutdown后,主机和其他从机照常运行,从机重启后失去主机目标,需要重新设置
   ```

5. 在6381终端中执行命令，设置6380终端为主机（树型主从模型）

   ```shell
   slaveof hadoop201 6380
   
   #串型模型可减轻主机的写压力,但中间节点宕机会导致后续从机都无法同步数据
   #中间的从机虽然作为后续从机的主机,但依然只有读权限
   ```

   在树型模型中，主机节点宕机后，从机节点可通过命令提升为主机，该从机的后续节点自动识别新主机

   ```shell
   slaveof no one
   ```



### 5.3 配置哨兵模式

- 哨兵进程可监控主机状态，若主机故障，可自动选择从机节点提升为新的主机节点

- 配置哨兵进程

  1. 编写配置文件

     ```shell
     vi redisconf/sentinel.conf
     ```

     ```shell
     protected-mode no
     daemonize yes
     sentinel monitor mymaster hadoop201 6379 1
     #说明
     # mymaster 监控的主机的自定义名(任意)
     # 1 转移主机需要的最少哨兵数,即只要1个哨兵认为主机故障就转移主机(哨兵进程可配置多个)
     ```

  2. 启动哨兵（先启动所有主机从机）

     ```shell
     redis-sentinel ~/redisconf/sentinel.conf
     ```

  3. 结束主机服务，查看其它从机信息

     ```shell
     shutdown
     ```

     ```shell
     info replication
     ```

- 新主机选择规则（依次判断）

  1. 选择redis.conf文件中，slave-priority（优先度，默认100）较小的主机
  2. 选择偏移量大（数据更完整）的从机
  3. 选择runid（绑定redis实例的内置随机数）最小的从机

- 旧主机恢复重新启动后，哨兵会向其发送slaveof命令，使其成为新主机的从机



### 5.4 Jedis访问Redis主从模式

1. 新建Maven工程，导入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>3.2.0</version>
       </dependency>
   </dependencies>
   ```

2. 编写代码

   ```java
   public class RedisTest {
   
       //1.初始化单例jedisSentinelPool对象
       public static JedisSentinelPool jedisSentinelPool = null;
   
       public static Jedis getJedis(){
           if (jedisSentinelPool == null){
               
               //1.1 添加配置
               JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
               jedisPoolConfig.setMaxTotal(20);
               jedisPoolConfig.setBlockWhenExhausted(true);
               jedisPoolConfig.setMaxWaitMillis(2000);
               jedisPoolConfig.setMaxIdle(5);
               jedisPoolConfig.setMinIdle(5);
               jedisPoolConfig.setTestOnBorrow(true);
   
               //1.2 配置集群地址,可直接通过哨兵服务端口(26379)访问主从集群
               Set<String> sentinelSet = new HashSet<>();
               sentinelSet.add("hadoop201:26379");
   
               //1.3 创建redis池
               jedisSentinelPool = new JedisSentinelPool("mymaster", sentinelSet, jedisPoolConfig);
           }
           
           //1.4 获取redis实例对象
           return jedisSentinelPool.getResource();
       }
   
       public static void main(String[] args) {
   
           //5.使用redis实例对象访问redis主从集群
           Jedis jedis = RedisTest.getJedis();
           jedis.set("k111","v111");
           jedis.set("k222","v222");
           jedis.close();
   
       }
   
   }
   ```

   



## 第六章 Redis集群

- Redis主从模式实际只实现了数据的备份，但当数据量过大时，依然只有Master节点执行所有的写操作，就会出现容量和效率的问题，这时可以采用Redis集群，同时实现**并发的写操作、数据库扩容和高可用**。
- 一个Redis集群中可以含有多个主节点，同时每个主节点依然保留了主从模式，实现了数据的备份，每个主节点中存储的数据各不相同。



### 6.1 Redis集群搭建

1. 安装ruby环境

   ```shell
   sudo yum -y install ruby-libs ruby ruby-irb ruby-rdoc rubygems
   ```

2. 上传并安装redis-3.2.0.gem

   ```shell
   gem install --local redis-3.2.0.gem
   ```

3. 编写6份配置文件（示例为6个不同端口模拟6个节点）

   ```shell
   cd ~/redisconf
   vi redis6371.conf
   ```

   ```shell
   #配置后台执行
   daemonize yes
   #配置端口号
   port 6371
   #配置rdb文件目录
   dir "/home/atguigu/redis-datas"
   #配置rdb文件名
   dbfilename "dump6371.rdb"
   #配置rdb的存盘策略
   save 300 10
   #配置pid文件名
   pidfile "/var/run/redis_6371.pid"
   #关闭保护模式(开启远端访问)
   protected-mode no
   
   #开启集群模式
   cluster-enabled yes
   #配置元数据文件的文件名
   cluster-config-file nodes-6371.conf
   #配置超时时间(ms),超时判定故障
   cluster-node-timeout 15000
   ```

   ```shell
   # 6372 6373 6374 6375 6376 五个端口配置同理
   ```

4. 启动所有节点服务（确保存储目录下没有数据文件）

   ```shell
   redis-server redis6371.conf
   redis-server redis6372.conf
   redis-server redis6373.conf
   redis-server redis6374.conf
   redis-server redis6375.conf
   redis-server redis6376.conf
   ```

5. 将所有节点合并为集群

   ```shell
   cd /opt/module/redis-3.2.5/src
   
   ./redis-trib.rb create --replicas 1 \
   hadoop201:6371 \
   hadoop201:6372 \
   hadoop201:6373 \
   hadoop201:6374 \
   hadoop201:6375 \
   hadoop201:6376
   # replicas:每个master的副本数,此处共6个节点,因此合并后产生3个master节点和3个slaver节点
   # 命令执行后提示输入yes
   ```

   ```shell
   #若合并时出现异常
   ERR Slot 0 is already busy
   #启动每个redis-cli,执行以下命令后再重新执行合并命令
   flushall
   cluster reset
   ```

6. 使用集群模式启动客户端

   ```shell
   redis-cli -c -p 6371
   # -c 使用集群模式启动
   # -p 选择任意本地节点的端口号
   ```

7. 集群相关命令

   ```shell
   #1.查看节点负责的所有插槽存储的所有key
   keys *
   
   #2.查看集群所有节点信息
   cluster nodes
   ```



### 6.2 Redis集群分配规则

- Redis在首次合并集群时，会遵循以下规则自动分配master和slaver节点
  1. master节点数至少为3，即启动的redis-server数至少为3
  2. 根据配置的副本数，计算master数
     master数=总节点数/(副本数+1)
  3. 为每个节点分配角色时，尽量使每个master运行在不同的主机节点，每个slaver和对应的master运行在不同节点



### 6.3 Redis插槽

- ### hash slot（插槽）

  - 一个redis集群包含16384个插槽用于存储数据，编号为0-16383

  - 每条数据插入redis集群时，会根据公式CRC16(key)%16384来计算该key被分配到哪个插槽

  - 集群中的每个master节点分别存储一部分插槽的数据

    ```shell
    #示例 集群包含3个master节点
    #节点1存储0-5460号插槽的数据
    #节点2存储5461-10922号插槽的数据
    #节点3存储10923-16383号插槽的数据
    ```

- 相关命令

  ```shell
  #1.集群模式下,不同插槽的数据不能在一条命令中调用,因为某节点宕机时会影响事务的原子性
  mset k1 v1 k2 v2 #错误
  mget k1 k2 #错误
  
  #2.可使用{}定向数据插入的槽,系统不再根据key分配插槽,而是根据{}中的数据进行计算分配
  mset k1{g1} v1 k2{g1} v2
  
  #3.计算key分配的槽
  cluster keyslot [key]
  cluster keyslot k1
  
  #4.获取指定插槽的key的数据量
  cluster countkeysinslot [slot]
  cluster countkeysinslot 12706
  
  #获取指定插槽中的count个数据
  cluster getkeysinslot [slot] [count]
  cluster getkeysinslot 12076 100
  ```



### 6.4 集群故障恢复

- redis集群模式中，自带哨兵模式的高可用效果

  - 若主节点宕机，则某个从节点成为新的主节点，原主节点恢复后自动成为从节点
  - 若从节点宕机，集群照常运行，从节点恢复后自动重新匹配原主节点

- 若负责某一区段插槽的所有主从节点都发生宕机，则根据配置文件中的配置项进行响应

  ```shell
  cluster-require-full-coverage yes
  # yes(默认)	则集群停止提供服务,直到节点恢复
  # no	集群以现有的数据(除去缺失的slot),继续提供服务
  ```



### 6.5 Jedis访问Redis集群

1. 新建Maven工程，导入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>3.2.0</version>
       </dependency>
   </dependencies>
   ```

2. 编写代码

   ```java
   public class RedisCluster {
   
       private static JedisCluster jedisCluster = null;
   
       //返回jedisCluster单例
       public static JedisCluster getJedisCluster(){
   
           if (jedisCluster == null){
   
               //1.获取集群的节点,主从节点不限,配2-3个足矣
               Set<HostAndPort> hostAndPorts = new HashSet<>();
               hostAndPorts.add(new HostAndPort("hadoop201", 6371));
               hostAndPorts.add(new HostAndPort("hadoop201", 6372));
               hostAndPorts.add(new HostAndPort("hadoop201", 6373));
   
               //2.配置项池
               JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
               jedisPoolConfig.setMaxTotal(20);
               jedisPoolConfig.setBlockWhenExhausted(true);
               jedisPoolConfig.setMaxWaitMillis(2000);
               jedisPoolConfig.setMaxIdle(5);
               jedisPoolConfig.setMinIdle(5);
               jedisPoolConfig.setTestOnBorrow(true);
   
               //3.创建jedisCluster对象
               jedisCluster = new JedisCluster(hostAndPorts, jedisPoolConfig);
   
           }
   
           return jedisCluster;
   
       }
   
       public static void main(String[] args) {
   
           JedisCluster jedisCluster = RedisCluster.getJedisCluster();
   
           jedisCluster.set("k111", "v111");
           
           //注意:集群模式无需close资源
       }
   }
   ```