# MySQL

***

## 第一章  MySQL简介

### 1.1 概念

- MySQL是一个关系型数据库管理系统，由瑞典MySQLAB公司开发，现已被Oracle公司收购；
- MySQL是开源、可定制的，采用了GPL协议；
- MySQL使用标准的SQL数据语言形式；
- MySQL支持多种系统和编程语言；
- MySQL支持大型数据库，支持5000万条记录的数据仓库，32位系统支持最大4GB的表文件，64位系统则支持最大8TB的表文件。

### 1.2 整体架构

- 连接层
- 服务层
- 引擎层
- 存储层



## 第二章 MySQL的安装和配置

### 2.1 在Linux上安装MySQL

1. 官网下载MySQL镜像，官网地址：http://dev.mysql.com/downloads/mysql/

2. 检查系统是否已经安装了mysql或mariadb（mysql的衍生版本）

   ```shell
   rpm -qa | grep mysql
   rpm -qa | grep mariadb
   #若已经安装则需要卸载
   rpm -e --nodeps  mariadb-libs
   ```

3. 使用finalshell上传mysql安装包到安装目录下

4. 解压mysql安装包

   ```shell
   tar -xvf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
   ```

5. 依次安装rpm（因为RPM包间存在依赖关系，必须按照顺序安装）

   ``` shell
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
   rm -rf *
   ```

7. 初始化数据库

   ```shell
   #安装初始化可能需要的依赖
   yum install -y perl perl-devel
   yum install -y libaio
   
   #初始化数据库
   mysqld --initialize --user=mysql
   ```

8. 查看自动生成的root用户的密码

   ``` shell
   cat /var/log/mysqld.log
   ```

9. 启动mysql服务

   ```shell
   systemctl start mysqld.service
   ```

10. 登陆mysql数据库

    ```shell
    mysql -uroot -p
    #输入8中生成的初始密码
    ```

11. 首次登陆需要修改密码才能进行后续操作

    ```mysql
    set password = password("新密码");
    ```



### 2.2 MySQL中的一些配置

#### 2.2.1 mysql服务自启动

1. 查看mysql服务的自启动状态

   ```shell
   systemctl list-unit-files | grep mysqld.service
   ```

2. 开启/关闭mysql服务的自启动

   ```shell
   systemctl enable mysqld.service
   systemctl disable mysqld.service
   ```



#### 2.2.2 修改默认字符集

1. 查询mysql中和字符集相关的信息

   ```mysql
   show variables like '%char%';
   ```

2. 修改 /etc/my.cnf 配置文件，永久修改默认字符集

   ```shell
   vi /etc/my.cnf
   #添加语句
   [mysqld]
   character_set_server=utf8
   collation-server=utf8_general_ci
   ```



#### 2.2.3 设置大小写不敏感

1. 查看当前数据库配置

   ```mysql
   show variables like '%lower_case_table_names%';
   #0--大小写敏感
   #1--大小写不敏感
   #2--创建的表和数据库根据语句格式存储，所有查找操作自动转换为小写进行
   ```

2. 修改 /etc/my.cnf 配置文件，永久修改大小写不敏感

   ``` shell
   vi /etc/my.cnf
   
   #[mysqld]节点下添加语句
   lower_case_table_names=1
   ```

   

#### 2.2.4 设置语法校验规则

1. 查看当前的语法规则

   ```mysql
   select @@sql_mode;
   ```

2. 修改 /etc/my.cnf 配置文件，永久修改语法规则

   ``` shell
   vi /etc/my.cnf
   
   #[mysqld]节点下添加语句
   sql_mode='' 
   #此处作用为修改语法规则为空(仅作为示例,语法规则不建议为空)
   ```

   

### 2.3 MySQL的用户管理

- mysql中的用户管理在mysql库的user表中

  ```mysql
  use mysql;
  select Host,User,authentication_string from user;
  ```

- 常用的指令

  ``` mysql
  create user zhang3 identified by '123456';
  #创建名称为zhang3的用户，密码为123456
  
  select host,user,password,select_priv,insert_priv,drop_priv from mysql.user;
  #查看用户和权限的相关信息
  
  set password=password('123456');
  #修改当前用户的密码
  # host表示连接类型,包括以下几种
  #	% 通过所有地址进行TCP连接
  #	IP地址 通过指定IP地址进行TCP连接
  #	localhost 本地方式通过命令行连接
  
  update mysql.user set authentication_string=password('123456') where user='li4';
  flush privileges;
  #修改li4的用户密码为123456
  #使用后必须使用刷新生效
  
  update mysql.user set user='li4' where user='wang5';
  #修改用户名wang5为li4
  
  drop user li4;
  #删除用户li4
  
  update mysql.user set host = '%' where user = 'root';
  flush privileges;
  #设置所有地址均能使用root用户远程连接
  ```



### 2.4 MySQL的权限管理

#### 2.4.1 授予权限

```mysql
grant select,insert,delete,drop on atguigudb.* to li4@localhost;
#授予li4用户在本地命令行的方式下的对atguigudb库中所有表的select,insert,delete,drop权限

grant all privileges on "." to zhang3@'%' identified by '123456';
#赋予zhang3用户在网络连接方式下的对所有库所有表的全部权限,密码为123456
```

#### 2.4.2 收回权限

```mysql
show grants
#查看当前用户权限

revoke select,insert,delete,drop on atguigu.* from li4@localhost;
#收回li4用户在本地命令行的方式下的对atguigu库中所有表的select,insert,delete,drop权限

revoke all privileges on mysql.* from zhang3@localhost;
#收回zhang3用户对mysql库全表的所有权限
```

- 权限修改后需要目标用户重新登录生效。



### 2.5 查看sql的执行周期

1. 查看profile是否开启

   ```mysql
   show variables like '%profiling%';
   ```

2. 开启profile

   ```mysql
   set profiling=1;
   ```

3. 查看近几次执行的操作

   ```mysql
   show profiles;
   ```

4. 使用 Query_ID 查看sql语句的具体执行步骤

   ```mysql
   show profile cpu,block io for query 2; #查询2号语句的执行步骤
   ```



### 2.6 查询缓存

1. 查看查询缓存的相关设置

   ```shell
   show variables like "%query_cache%";
   #查询结果说明
   #query_cache_type 缓存类型
   #	0 -- OFF 查询缓存关闭
   #	1 -- ON 查询缓存开启
   #	2 -- DEMAND 查询缓存使用SQL_CACHE关键字手动开启
   ```

2. 修改 /etc/my.cnf 配置文件，永久开启查询缓存

   ```shell
   vi /etc/my.cnf 
   ```

   ```shell
   #[mysqld]节点下添加语句
   query_cache_type=1
   ```

3. 重新启动mysqld服务

   ```shell
   service mysqld restart
   ```

4. （不使用缓存查询）

   ```mysql
   #使用SQL_NO_CACHE关键字不使用缓存查询
   select SQL_NO_CACHE * from employees;
   ```



### 2.7 MySQL存储引擎

- 查看存储引擎

  ```mysql
  #查看支持的存储引擎
  show engines;
  #查看当前默认的存储引擎
  show variables like "%storage_engine%";
  ```

- 存储引擎介绍

  - InnoDB：默认的事务型引擎，优先选择使用。
  - MyISAM：支持全文索引、压缩、空间函数等，读取性能强。
  - Archive：用于档案存储、日志和数据采集，仅支持 insert 和 select 操作。
  - Blackhole：无存储机制，仅记录日志。
  - CSV：可将CSV文件当作表处理，可使用文本编辑器或excel直接读取。
  - Memory：快速访问无需修改的数据。
  - Federated：跨服务器代理，默认禁用。

- MyISAM引擎和InnoDB引擎对比

| 对比项         | MyISAM                                           | InnoDB                                                       |
| -------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 外键           | 不支持                                           | 支持                                                         |
| 事务           | 不支持                                           | 支持                                                         |
| 行表锁         | 表锁，所有操作都会锁住整个表，不适合高并发的操作 | 行锁，操作时只锁操作的行，不对其它行有影响，适合高并发的操作 |
| 缓存           | 只缓存索引，不缓存真实数据                       | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 关注点         | 读性能                                           | 并发写、事务、资源                                           |
| 默认安装       | Y                                                | Y                                                            |
| 默认使用       | N                                                | Y                                                            |
| 自带系统表使用 | Y                                                | N                                                            |





## 第三章 索引优化分析

### 3.1 索引简介

- 索引（Index）是帮助MySQL高效获取数据的**数据结构**，以索引文件的形式存储在磁盘上
- 优点
  1. 提高数据检索的效率，降低数据库的IO成本
  2. 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗
- 缺点
  1. 降低表的增删改效率
  2. 占用磁盘空间



### 3.2 MySQL的索引结构

#### 3.2.1 BTree与B+Tree

- **MySQL采用的索引结构为B+Tree**
- BTree：所有的叶子节点（末端节点）和非叶子节点中都保存了指针和数据
- B+Tree：数据只保存在叶子节点中，所有非叶子节点中仅保存数据的指针信息
- B+Tree相比于BTree的优势
  1. **节省磁盘IO**：存储同样体量的数据，B+Tree更宽，而树高更低，也就意味着磁盘IO次数越少
  2. **查询效率稳定**：对于B+Tree的每一条数据的查询，都需要从根节点走到叶子节点，查询路径相同，效率稳定



#### 3.2.2 聚簇索引与非聚簇索引

- 聚簇索引是一种数据存储方式，使表中某一字段在磁盘中的排列与索引的排列顺序保持一致
- 优点
  1. 查询范围型数据时，可以从磁盘连续读取，节省IO
- 限制
  1. MySQL中只要innodb数据引擎支持聚簇索引
  2. 1个表只能有1个聚簇索引，一般为主键（只能保证一个字段的排序）



### 3.3 MySQL的索引分类

### 3.4 索引的创建时机





## 第五章 Explain 性能分析

## 第六章 批量数据脚本