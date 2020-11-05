# Azkaban

## 第一章 Azkaban概述

### 1.1 工作流调度系统

- 一个完整的数据分析系统通常由大量任务单元组成，为了更合理的组织这些任务单元，需要一个调度系统进行调度执行，这就是工作流调度系统。



### 1.2 常见的工作调度系统

- 简单的任务调度：使用linux中的crontab定义
- 复杂的调度任务：开发调度平台或使用开源调度系统（Azkaban，Ooize，Cascading,Hamake等）



### 1.3 工作调度系统对比

| 特性               | Hamake               | Oozie             | Azkaban                        | Cascading |
| ------------------ | -------------------- | ----------------- | ------------------------------ | --------- |
| 工作流描述语言     | XML                  | XML (xPDL based)  | text file with key/value pairs | Java API  |
| 依赖机制           | data-driven          | explicit          | explicit                       | explicit  |
| 是否要web容器      | No                   | Yes               | Yes                            | No        |
| 进度跟踪           | console/log messages | web page          | web page                       | Java API  |
| Hadoop job调度支持 | no                   | yes               | yes                            | yes       |
| 运行模式           | command line utility | daemon            | daemon                         | API       |
| Pig支持            | yes                  | yes               | yes                            | yes       |
| 事件通知           | no                   | no                | no                             | yes       |
| 需要安装           | no                   | yes               | yes                            | no        |
| 支持的hadoop版本   | 0.18+                | 0.20+             | currently unknown              | 0.18+     |
| 重试支持           | no                   | workflownode evel | yes                            | yes       |
| 运行任意命令       | yes                  | yes               | yes                            | yes       |
| Amazon EMR支持     | yes                  | no                | currently unknown              | yes       |



### 1.4 Azkaban与Oozie

|              | Azkaban                                         | Oozie                                             |
| ------------ | ----------------------------------------------- | ------------------------------------------------- |
| 功能         | 调度mr，pig，java，脚本工作流任务，执行定时任务 | 同Azkaban                                         |
| 工作流定义   | 使用properites文件定义                          | 使用xml文件定义                                   |
| 工作流传参   | 支持直接传参                                    | 支持参数和EL表达式                                |
| 定时任务执行 | 基于时间                                        | 基于时间和输入数据                                |
| 资源管理     | 严格控制用户权限                                | 无严格的权限控制                                  |
| 工作流执行   | 支持单机模式和集群模式                          | 支持多用户和多用户流                              |
| 工作流管理   | 支持浏览器和ajax操作工作流                      | 支持命令行，HTTP REST，Java API，浏览器操作工作流 |
| **选型依据** | hadoop发行版为Apache                            | hadoop发行版为CDH                                 |



### 1.5 Azkaban特点

- Azkaban是由Linkedin开源的一个批量工作流任务调度器。用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban定义了一种KV文件格式来建立任务之间的依赖关系，并提供一个易于使用的web用户界面维护和跟踪你的工作流。



## 第二章 Azkaban入门

### 2.1 单机模式部署（测试用）

1. 上传Azkaban单机版安装包到linux并解压安装

   ```shell
   tar /opt/software/azkaban-solo-server-3.84.4.tar.gz -C /opt/module
   ```

2. 启动Azkaban

   ```shell
   #Azkaban必须在安装目录下启动
   cd /opt/module/azkaban-solo-server-3.84.4
   bin/start-solo.sh
   ```

3. 打开浏览器查看 http://hadoop100:8081

4. 关闭Azkaban

   ```shell
   cd /opt/module/azkaban-solo-server-3.84.4
   bin/shutdown-solo.sh
   ```



### 2.2 集群模式部署

1. Azkaban集群模式部署需要安装3个安装包，db、web包仅需安装1份，server包需要在所有集群节点安装。

2. 上传3个Azkaban安装包并解压

   ```shell
   mkdir /opt/module/azkaban
   tar -zxvf azkaban-db-3.84.4.tar.gz -C /opt/module/azkaban
   tar -zxvf azkaban-exec-server-3.84.4.tar.gz -C /opt/module/azkaban
   tar -zxvf azkaban-web-server-3.84.4.tar.gz -C /opt/module/azkaban
   ```

3. 登陆MySQL，进行相关配置

   ```mysql
   #1.创建Azkaban数据库
   create database azkaban;
   
   #2.创建Azkaban用户并赋予权限
   create user 'azkaban'@'%' identified by '123456';
   grant select, insert, update, delete on azkaban.* to 'azkaban'@'%' with grant option;
   flush privilege;
   
   #3.导入azkaban相关表
   use azkaban;
   source /opt/module/azkaban/azkaban-db-3.84.4/create-all-sql-3.84.4.sql
   ```

4. 修改配置文件，更改MySQL包的大小

   ```shell
   sudo vim /etc/my.cnf
   ```

   ```shell
   #在[mysqld]中添加配置
   max_allowed_packet=1024M
   ```

5. 重启MySQL服务

   ```shell
   sudo systemctl restart mysqld
   ```

6. 修改Excutotr Server的配置文件

   ```shell
   vim /opt/module/azkaban/azkaban-exec-server-3.84.4/conf/azkaban.properties
   ```

   ```properties
   #修改以下配置
   #1.默认时区
   default.timezone.id=Asia/Shanghai
   #2.WebServer地址
   azkaban.webserver.url=http://hadoop100:8081
   #3.mysql配置
   database.type=mysql
   mysql.port=3306
   mysql.host=hadoop100
   mysql.database=azkaban
   mysql.user=azkaban
   mysql.password=123456
   mysql.numconnections=100
   #4.最后添加配置
   executor.metric.reports=true
   executor.metric.milisecinterval.default=60000
   ```

7. 同步Excutor Server到所有Azkaban节点

   ```shell
   xsync /opt/module/azkaban/azkaban-exec-server-3.84.4
   ```

8. 在所有节点上启动并激活Excutor Server

   ```shell
   #注意:由于Azkaban启动时,加载配置文件默认使用了相对路径,因此必须到安装目录下启动脚本
   cd /opt/module/azkaban/azkaban-exec-server-3.84.4
   bin/start-exec.sh
   #激活
   curl -G "localhost:$(<./executor.port)/executor?action=activate" && echo
   #激活成功后出现如下提示 
   {"status":"success"}
   ```

9. 修改Web Server的配置文件

   ```shell
   vim /opt/module/azkaban/azkaban-web-server-3.84.4/conf/azkaban.properties
   ```

   ```properties
   #修改以下配置
   #1.修改默认时区
   default.timezone.id=Asia/Shanghai
   #2.配置mysql
   database.type=mysql
   mysql.port=3306
   mysql.host=hadoop100
   mysql.database=azkaban
   mysql.user=azkaban
   mysql.password=123456
   mysql.numconnections=100
   #3.修改配置
   azkaban.executorselector.filters=StaticRemainingFlowSize,CpuStatus
   ```

10. 修改Web Server的用户配置文件

    ```shell
    vim /opt/module/azkaban/azkaban-web-server-3.84.4/conf/azkaban-users.xml
    ```

    ```xml
    <!-- 在<azkaban-users>标签中添加user -->
    <user password="atguigu" roles="metrics,admin" username="atguigu"/>
    ```

11. 启动Web Server

    ```shell
    #注意:由于Azkaban启动时,加载配置文件默认使用了相对路径,因此必须到安装目录下启动脚本
    cd /opt/module/azkaban/azkaban-web-server-3.84.4
    bin/start-web.sh
    ```

12. 打开浏览器查看 http://hadoop100:8081，使用Web Server中的用户登录

13. 关闭Azkaban

    ```shell
    #关闭web server
    bin/shutdown-exec.sh
    #关闭excutor server
    bin/shutdown-web.sh
    ```



### 2.3 工作流

#### 2.3.1 创建工作流

1. 本地新建first.project文件，添加以下内容

   ```yaml
   azkaban-flow-version: 2.0
   ```

2. 本地新建basic.flow文件，添加以下内容（注意缩进）

   ```yaml
   nodes:
     - name: jobA
       type: command
       config:
         command: echo "This is an echoed text."
   ```

3. 将两文件打包压缩为zip文件（注意文件编码格式均使用utf-8）

4. 打开浏览器，进入web端页面，新建项目![image-20200510202853088](Azkaban.assets/image-20200510202853088.png)

5. 新建完成后，点击Upload上传zip包，Azkaban会自动解析工作流![image-20200510203030696](Azkaban.assets/image-20200510203030696.png)

6. 执行工作流![image-20200510203110310](Azkaban.assets/image-20200510203110310.png)

#### 2.3.2 工作流中的作业依赖

1. 修改2.3.1中的basic.flow文件

   ```yaml
   nodes:
     - name: jobC
       type: command
       # jobC依赖于JobA和JobB
       dependsOn:
         - jobA
         - jobB
       config:
         command: echo "I’m JobC"
   
     - name: jobA
       type: command
       config:
         command: echo "I’m JobA"
   
     - name: jobB
       type: command
       config:
         command: echo "I’m JobB"
   ```

2. 打包压缩后上传到新的项目执行



#### 2.3.3 全局配置

1. 修改2.3.1中的basic.flow文件

   ```yaml
   #声明全局变量,应用到工作流中的所有作业
   ---
   config:
     words.to.print: "This is for test!"
   
   nodes:
     - name: jobA
       type: command
       config:
         command: echo ${words.to.print}
   ```

2. 打包压缩后上传到新的项目执行



#### 2.3.4 失败重试配置

1. 修改2.3.1中的basic.flow文件

   ```yaml
   nodes:
     - name: JobA
       type: command
       config:
         command: sh /not_exists.sh
         #重试次数
         retries: 3
         #重试时间间隔(毫秒)
         retry.backoff: 10000
   ```

2. 打包压缩后上传到新的项目执行



#### 2.3.5 子工作流

1. 修改2.3.1中的basic.flow文件

   ```yaml
   nodes:
     - name: jobC
       type: command
       # jobC依赖embedded_flow
       dependsOn:
         - embedded_flow
       config:
         command: echo "I’m JobC"
   
     - name: embedded_flow
       type: flow
       config:
         prop: value
       nodes:
         - name: jobB
           type: noop
           dependsOn:
             - jobA
   
         - name: jobA
           type: command
           config:
             command: pwd
   ```

2. 打包压缩后上传到新的项目执行



## 第三章 Azkaban进阶

- Azkaban工作流的编写以key-value的形式定义，包含如下几项配置
  - name：作业名
  - type：作业类型
  - config：作业的相关配置
  - dependsOn：作业依赖

### 3.1 作业类型

#### 3.1.1 命令行

- 配置方式

  ```yaml
  type: command
  ```

- 示例

  ```yaml
  nodes:
    - name: jobA
      type: command
      config:
        command: echo "Command 1"
  ```



#### 3.1.2 JavaProcess（Java进程）

- 配置方式

  ```yaml
  type: javaprocess
  ```

- 示例

  1. 编写first.project文件

     ```yaml
     azkaban-flow-version: 2.0
     ```

  2. 编写basic.flow文件

     ```yaml
     nodes:
       - name: test_java
         type: javaprocess
         config:
           #最小堆
           Xms: 96M
           #最大堆
           Xmx: 200M
           #此处为自定义的的javaprocess的全类名
           java.class: com.atguigu.azkaban.AzkabanTest
     ```

  3. 启动IDEA，新建Maven工程

  4. 创建类，写入主方法

     ```java
     public class AzkabanTest {
         public static void main(String[] args) {
             System.out.println("javaprocess test");
         }
     }
     ```

  5. 打jar包后与project文件、flow文件一起压缩上传后执行



### 3.2 条件工作流

#### 3.2.1 运行时参数

- 当具有依赖关系的作业以作业运行时产生的参数作为运行条件时，可将该参数输出到$JOB_OUTPUT_PROP_FILE中，再以${jobName:param}引用该参数。

- 条件支持的运算符

  ```
  ==	等于
  !=	不等于
  >	大于
  >=	大于等于
  <	小于
  <=	小于等于
  &&	与
  ||	或
  !	非
  ```

- 示例

  1. 编写first.project文件

     ```yaml
     azkaban-flow-version: 2.0
     ```

  2. 编写basic.flow文件

     ```yaml
     nodes:
      - name: JobA
        type: command
        config:
          #执行脚本获取参数
          command: sh write_to_props.sh
     
      - name: JobB
        type: command
        dependsOn:
          - JobA
        config:
          command: echo "This is JobB."
        #以JobA输出参数作为运行条件
        condition: ${JobA:param1} == "AAA"
     
      - name: JobC
        type: command
        dependsOn:
          - JobA
        config:
          command: echo "This is JobC."
        #以JobA输出参数作为运行条件
        condition: ${JobA:param1} == "BBB"
     ```

  3. 编写write_to_props.sh

     ```shell
     echo '{"param1":"AAA"}' > $JOB_OUTPUT_PROP_FILE
     ```

  4. 将3个文件打包压缩上传后运行



#### 3.2.2 预定义宏

- Azkaban提供了一系列预定义的条件参数，对依赖作业的运行结果进行判断，再以判断结果作为作业的启动条件

  1. all_success：全部成功
  2. all_done：全部完成
  3. all_failed：全部失败
  4. one_success：至少成功1个
  5. one_failed：至少失败1个

- 示例

  1. 上个示例中的修改basic.flow

     ```yaml
     nodes:
       - name: JobA
         type: command
         config:
           command: sh /opt/module/write_to_props.sh
     
       - name: JobB
         type: command
         dependsOn:
           - JobA
         config:
           command: echo "This is JobB."
         condition: ${JobA:param1} == "AAA"
     
       - name: JobC
         type: command
         dependsOn:
           - JobA
         config:
           command: echo "This is JobC."
         condition: ${JobA:param1} == "BBB"
     
       - name: JobD
         type: command
         dependsOn:
           - JobB
           - JobC
         config:
           command: echo "This is JobD."
         condition: one_success
     
       - name: JobE
         type: command
         dependsOn:
           - JobB
           - JobC
         config:
           command: echo "This is JobE."
         condition: all_success
     
       - name: JobF
         type: command
         dependsOn:
           - JobB
           - JobC
           - JobD
           - JobE
         config:
           command: echo "This is JobF."
         condition: all_done
     ```

  2. 打包压缩上传后执行



### 3.3 定时执行

- Azkaban可以通过类似于crontab的方式配置工作流的定时执行
- 示例
  1. 执行任务时点击Schedule
     ![image-20200510213904370](Azkaban.assets/image-20200510213904370.png)
  2. 配置具体的执行时间，注意配置时区
     <img src="Azkaban.assets/image-20200510214021400.png" alt="image-20200510214021400" style="zoom:50%;" />
  3. 确认后提交
     <img src="Azkaban.assets/image-20200510214101485.png" alt="image-20200510214101485" style="zoom: 50%;" />



### 3.4 失败报警

#### 3.4.1 邮件报警（默认）

1. 注册邮箱，开启stmp服务，获取smtp服务的host地址

2. 修改Web Server的配置文件

   ```shell
   vim /opt/module/azkaban/azkaban-web-server-3.84.4/conf/azkaban.properties
   ```

   ```properties
   #修改如下配置
   #配置用于发送报警邮件的邮箱登陆信息
   mail.sender=atguigu@126.com
   mail.host=smtp.126.com
   mail.user=atguigu@126.com
   mail.password=password
   #配置发送的目标邮箱
   job.failure.email=atguigu@126.com
   job.success.email=atguigu@126.com
   ```

3. 重新启动Web Server

4. 编写测试的flow文件

   ```yaml
   nodes:
     - name: jobA
       type: command
       config:
         command: echo "This is an echoed text."
         failure.emails: atguigu@126.com
         success.emails: atguigu@126.com
         notify.emails: atguigu@126.com 
   ```

5. 打包压缩上传后测试



#### 3.4.2 自定义报警



## 第四章 参考资料

### 4.1 Azkaban配置项

- https://azkaban.readthedocs.io/en/latest/configuration.html

### 4.2 YAML语法

- https://www.runoob.com/w3cnote/yaml-intro.html

