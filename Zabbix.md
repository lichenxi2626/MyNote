# Zabbix

## 第一章 Zabbix概述

- Zabbix是一款能够监控各种网络参数及服务器健康性、完整性的软件。可以对任何事件配置基于邮件的告警。
- Zabbix架构
  <img src="Zabbix.assets/image-20200713155648783.png" alt="image-20200713155648783" style="zoom:80%;" />
  1. agent：部署在需要监控的主机节点上，监测该检点的资源及应用运行状态
  2. server：收集agent监控的数据，判断是否满足触发器条件，并向用户通知
  3. database：存储所有的配置信息
  4. web：用户操作界面，展示监控信息
- Zabbix集群规划
  - hadoop201：agent，server，web，MySQL
  - hadoop202：agent
  - hadoop203：agent



## 第二章 Zabbix部署

### 2.1 Zabbix集群搭建

1. 关闭防火墙（3台节点，已关闭）

   ```shell
   sudo service iptables stop
   sudo chkconfig iptables off
   ```

2. 在zabbix-web的节点修改配置文件（hadoop201）

   ```shell
   sudo vi /etc/selinux/config
   ```

   ```shell
   SELINUX=disabled
   ```

   ```shell
   #修改后重启主机生效,或执行以下命令使配置临时生效
   sudo setenforce 0
   ```

3. 从阿里云镜像下载zabbix的yum源（3台节点）

   ```shell
   sudo rpm -ivh https://mirrors.aliyun.com/zabbix/zabbix/4.4/rhel/7/x86_64/zabbix-release-4.4-1.el7.noarch.rpm
   ```

   ```shell
   #修改下载的repo文件
   sudo vi /etc/yum.repos.d/zabbix.repo
   ```

   ```shell
   #替换路径为阿里云镜像
   :%s/http:\/\/repo.zabbix.com/https:\/\/mirrors.aliyun.com\/zabbix/g
   ```

4. 安装Zabbix（3台节点）

   ```shell
   #hadoop201执行
   sudo yum install -y zabbix-server-mysql zabbix-web-mysql zabbix-agent
   ```

   ```shell
   #hadoop202,203执行
   sudo yum install -y zabbix-agent
   ```

5. 在zabbix-mysql节点使用mysql创建数据库（hadoop201）

   ```shell
   mysql -uroot -p123456 -e"create database zabbix character set utf8 collate utf8_bin"
   ```

6. 导入建表语句到mysql（hadoop201）

   ```shell
   zcat /usr/share/doc/zabbix-server-mysql-4.4.10/create.sql.gz | mysql -uroot -p123456 zabbix
   #zabbix-server-mysql-4.4.10可能随时间变化版本,实际可到目录下查看后修改命令
   cd /usr/share/doc/
   ```

7. 修改zabbix-server配置文件（hadoop201）

   ```shell
   sudo vim /etc/zabbix/zabbix_server.conf
   ```

   ```shell
   #配置mysql相关配置
   DBHost=hadoop201
   DBName=zabbix
   DBUser=root
   DBPassword=123456
   ```

8. 修改zabbix-agent配置文件（3台节点）

   ```shell
   sudo vim /etc/zabbix/zabbix_agentd.conf
   ```

   ```shell
   Server=hadoop201
   #注释以下配置
   #ServerActive=127.0.0.1
   #Hostname=Zabbix server
   ```

9. 配置zabbix-web时区（hadoop201）

   ```shell
   sudo vim /etc/httpd/conf.d/zabbix.conf
   ```

   ```xml
   <Directory "/usr/share/zabbix">
       Options FollowSymLinks
       AllowOverride None
       Require all granted
   
       <IfModule mod_php5.c>
           php_value max_execution_time 300
           php_value memory_limit 128M
           php_value post_max_size 16M
           php_value upload_max_filesize 2M
           php_value max_input_time 300
           php_value max_input_vars 10000
           php_value always_populate_raw_post_data -1
           <!-- 修改时区为上海 -->
           php_value date.timezone Asia/Shanghai 
       </IfModule>
   </Directory>
   ```

10. 启动zabbix（3台节点）

    ```shell
    #hadoop201
    sudo systemctl start zabbix-server zabbix-agent httpd
    sudo systemctl enable zabbix-server zabbix-agent httpd
    #hadoop202,hadoop203
    sudo systemctl start zabbix-agent
    sudo systemctl enable zabbix-agent
    ```

11. 访问web端

    ```http
    http://hadoop201/zabbix
    ```



### 2.2 Web配置

1. 访问web端，点击下一步

   <img src="Zabbix.assets/image-20200716162153318.png" alt="image-20200716162153318"  />

   ![image-20200716162245518](Zabbix.assets/image-20200716162245518.png)

2. 配置数据库
   ![image-20200716162355733](Zabbix.assets/image-20200716162355733.png)

3. 配置zabbix-server地址端口号
   ![image-20200716162502920](Zabbix.assets/image-20200716162502920.png)
   ![image-20200716162536443](Zabbix.assets/image-20200716162536443.png)

4. 配置成功
   ![image-20200716162657778](Zabbix.assets/image-20200716162657778.png)



## 第三章 Zabbix使用 

### 3.1 Zabbix术语

- Host 主机：监控的目标节点，用IP或域名表示
- Item 监控项：监控的具体内容，是度量数据
- Trigger 触发器：设定的阈值，用于评估监控项数据的逻辑表达式
- Action 动作：对具体事件的反应操作，如发送邮件



### 3.2 部署Zabbix监控

#### 3.2.1 配置主机

1. 启动Zabbix服务，登陆Web端（用户名Admin，密码zabbix）

2. 修改中文操作界面
   ![image-20200716163540923](Zabbix.assets/image-20200716163540923.png)

3. 创建Host
   ![image-20200716163813029](Zabbix.assets/image-20200716163813029.png)

   ![image-20200716164026605](Zabbix.assets/image-20200716164026605.png)

4. 同理为hadoop202，hadoop203两台主机创建Host，注意选择同一群组
   ![image-20200716164226689](Zabbix.assets/image-20200716164226689.png)



#### 3.2.2 配置主机的监控项

1. 添加监控项
   ![image-20200716164648893](Zabbix.assets/image-20200716164648893.png)

   ![image-20200716164721799](Zabbix.assets/image-20200716164721799.png)
   配置监控项，并添加![image-20200716170134461](Zabbix.assets/image-20200716170134461.png)

2. 查看监控项状态
   ![image-20200716170444304](Zabbix.assets/image-20200716170444304.png)



#### 3.2.2 配置主机的触发器

1. 创建触发器
   ![image-20200716170729800](Zabbix.assets/image-20200716170729800.png)

   ![image-20200716170746465](Zabbix.assets/image-20200716170746465.png)

2. 配置触发器![image-20200716171508717](Zabbix.assets/image-20200716171508717.png)

   ![image-20200716171319935](Zabbix.assets/image-20200716171319935.png)



#### 3.2.4 配置动作

1. 创建动作
   ![image-20200716171709009](Zabbix.assets/image-20200716171709009.png)

2. 配置动作的触发器
   ![image-20200716172030776](Zabbix.assets/image-20200716172030776.png)

3. 配置动作的操作
   ![image-20200716172315371](Zabbix.assets/image-20200716172315371.png)

   ![image-20200716172742302](Zabbix.assets/image-20200716172742302.png)

4. 动作创建完成
   ![image-20200716172837111](Zabbix.assets/image-20200716172837111.png)



#### 5.2.5 配置报警媒介

1. 配置报警媒介（发送告警邮件的邮箱）
   ![image-20200716173102569](Zabbix.assets/image-20200716173102569.png)

   ![image-20200716174023616](Zabbix.assets/image-20200716174023616.png)

2. 配置用户邮箱
   ![image-20200716180002421](Zabbix.assets/image-20200716180002421.png)
   配置用户的报警媒介
   ![image-20200716180146517](Zabbix.assets/image-20200716180146517.png)



### 3.3 Zabbix模版

1. 创建模版（模版等价于主机，配置完成后直接在主机中应用模版即可）
   ![image-20200716182903663](Zabbix.assets/image-20200716182903663.png)

   ![image-20200716183008514](Zabbix.assets/image-20200716183008514.png)

2. 配置模版的监控项（与单独配置监控项一致）
   ![image-20200716183734374](Zabbix.assets/image-20200716183734374.png)

   ![image-20200716183905029](Zabbix.assets/image-20200716183905029.png)

3. 配置模版的触发器
   ![image-20200716184053599](Zabbix.assets/image-20200716184053599.png)
   配置触发器的监控项时需要选择主机为模版（模版为群组中的一台虚拟主机）
   ![image-20200716184331496](Zabbix.assets/image-20200716184331496.png)
   其余配置与常规的触发器配置一致
   ![image-20200716184430758](Zabbix.assets/image-20200716184430758.png)

4. 选择要应用模版的主机
   ![image-20200716184506791](Zabbix.assets/image-20200716184506791.png)
   选择相应模版后更新即可
   ![image-20200716184549709](Zabbix.assets/image-20200716184549709.png)

5. 主机成功启用模版
   ![image-20200716184717195](Zabbix.assets/image-20200716184717195.png)