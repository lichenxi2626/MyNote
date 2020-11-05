# CDH

## 1. Cloud Manager安装

### 1.1 服务器准备

1. 购买阿里云服务器，配置主机名及开放的端口

   ```
   将以下内容存储为json文件，导入到阿里云安全组规则中
   ```

   ```json
   [{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-07-09T07:12:49Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"27017/27017","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-07-03T09:10:19Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"10000/10000","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-22T08:38:38Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"21000/21000","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-22T03:44:21Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8983/8983","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-11T05:35:45Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"5432/5432","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-08T05:23:59Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8989/8989","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-03T09:12:23Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"16020/16020","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-03T08:24:21Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"2181/2181","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-05-02T09:52:10Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8081/8081","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-04-04T07:52:06Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"16030/16030","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-04-04T06:34:03Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"16010/16010","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-26T10:04:35Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"88/88","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-26T06:57:12Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"25020/25020","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-26T06:56:49Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"25010/25010","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-17T03:17:15Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"19888/19888","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-04T05:44:29Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"18080/18080","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-02T05:16:07Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"9866/9866","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-01T05:09:21Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8000/8000","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-03-01T04:36:26Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"9870/9870","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-01-06T03:07:38Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8900/8900","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-01-03T08:44:47Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"9042/9042","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-12-25T12:36:10Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8080/8080","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-11-11T02:55:14Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"7050/7051","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-11-11T02:47:24Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8050/8051","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-10-07T12:04:54Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"9092/9092","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-10-06T04:34:45Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"3000/3000","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-10-06T04:33:36Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"4000/4000","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-26T06:04:33Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"18089/18089","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-24T04:06:17Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8042/8042","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-18T11:03:01Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"50010/50010","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-18T10:04:12Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8020/8020","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-18T09:24:30Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"9083/9083","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-18T07:45:53Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8088/8088","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-16T08:25:10Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"50070/50070","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-16T08:24:59Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"8888/8888","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-16T08:24:49Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"3306/3306","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2019-09-16T08:24:26Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"7180/7180","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"System created rule.","SourcePortRange":"","Priority":110,"CreateTime":"2019-09-16T08:21:00Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"22/22","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"System created rule.","SourcePortRange":"","Priority":110,"CreateTime":"2019-09-16T08:21:00Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"3389/3389","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"TCP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"System created rule.","SourcePortRange":"-1/-1","Priority":110,"CreateTime":"2019-09-16T08:20:59Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"ingress","SourceGroupName":"","PortRange":"-1/-1","DestGroupOwnerAccount":"","SourceCidrIp":"0.0.0.0/0","IpProtocol":"ICMP","DestCidrIp":"","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""},{"SourceGroupId":"","Policy":"Accept","Description":"","SourcePortRange":"","Priority":1,"CreateTime":"2020-01-06T03:05:06Z","Ipv6SourceCidrIp":"","NicType":"intranet","DestGroupId":"","Direction":"egress","SourceGroupName":"","PortRange":"8900/8900","DestGroupOwnerAccount":"","SourceCidrIp":"","IpProtocol":"TCP","DestCidrIp":"0.0.0.0/0","DestGroupName":"","SourceGroupOwnerAccount":"","Ipv6DestCidrIp":""}]
   ```

2. 配置公网ip到本地hosts文件

   ```
   C:\Windows\System32\drivers\etc\hosts
   ```

   ```
   39.101.64.34 hadoop101
   39.101.65.8 hadoop102
   39.101.66.218 hadoop103
   ```

3. 启动xshell，访问阿里云集群



### 1.2 配置ssh免密登陆

1. 使用内网ip配置集群所有节点的hosts文件

   ```
   vi /etc/hosts
   ```

   ```
   172.22.9.45 hadoop101  hadoop101
   172.22.9.47 hadoop102  hadoop102
   172.22.9.46 hadoop103  hadoop103
   ```

2. 配置ssh免密登陆

   ```shell
   #分别三台主机执行以下所有命令
   #1.生成公钥和私钥,三次回车确认
   ssh-keygen -t rsa
   #2.将公钥分发到目标主机
   ssh-copy-id hadoop101
   ssh-copy-id hadoop102
   ssh-copy-id hadoop103
   ```

3. 编写集群分发脚本

   ```shell
   vi /bin/xsync
   ```

   ```shell
   #!/bin/bash
   #1. 判断参数个数
   if [ $# -lt 1 ]
   then
     echo Not Enough Arguement!
     exit;
   fi
   #2. 遍历集群所有机器
   for host in hadoop102 hadoop103
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

   ```shell
   chmod +x xsync
   ```

   ```shell
   #在所有节点安装群发脚本所需的命令
   yum install -y rsync
   ```

   



### 1.3 安装JDK

1. 上传jdk安装包到集群，并安装

   ```shell
   rpm -ivh oracle-j2sdk1.8-1.8.0+update181-1.x86_64.rpm
   ```

2. 配置jdk环境变量

   ```shell
   vim /etc/profile.d/my_env.sh
   ```

   ```shell
   #JAVA_HOME
   export JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
   export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   ```shell
   source /etc/profile.d/my_env.sh
   ```

   ```shell
   #查看java版本
   java -version
   ```

3. 分发jdk和环境变量到所有节点

   ```shell
   xsync /usr/java/
   xsync /etc/profile.d/my_env.sh
   #记得source环境变量
   ```



### 1.4 安装MySQL

1. 确认是否已经安装了mysql或mariadb

   ```shell
   rpm -qa | grep mysql
   rpm -qa | grep mariadb
   #卸载这两个数据库
   rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64
   yum remove mysql-libs
   ```

2. 下载mysql依赖并安装

   ```shell
   yum -y install libaio
   yum -y install autoconf
   wget https://downloads.mysql.com/archives/get/p/23/file/MySQL-shared-compat-5.6.24-1.el6.x86_64.rpm
   wget https://downloads.mysql.com/archives/get/p/23/file/MySQL-shared-5.6.24-1.el7.x86_64.rpm
   rpm -ivh MySQL-shared-5.6.24-1.el7.x86_64.rpm
   rpm -ivh MySQL-shared-compat-5.6.24-1.el6.x86_64.rpm
   ```

   ```shell
   #安装完成后删除2个下载的rpm包
   rm MySQL-shared-5.6.24-1.el7.x86_64.rpm
   rm MySQL-shared-compat-5.6.24-1.el6.
   ```

3. 上传并解压mysql-libs压缩包

   ```shell
   yum -y install unzip
   unzip mysql-libs.zip
   ```

4. 安装mysql服务器

   ```shell
   cd mysql-libs
   rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
   #查看并保存生成的随机密码 RzR5XrVxESRwSXIl
   cat /root/.mysql_secret
   #启动mysql
   service mysql start
   ```

5. 安装mysql客户端

   ```shell
   rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
   ```

6. 访问mysql，修改配置

   ```shell
   mysql -uroot -pRzR5XrVxESRwSXIl
   #修改密码
   set password = password("123456");
   #设置root用户远程访问
   update mysql.user set host = '%' where user = 'root';
   #删除root用户的其他host
   delete from mysql.user where host != '%';
   #刷新
   flush privileges;
   #设置后查看
   select Host,User,authentication_string from mysql.user;
   ```

7. 创建CM部署所需的数据库

   ```mysql
   #创建scm用户
   GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';
   #创建4个数据库
   CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
   CREATE DATABASE hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
   CREATE DATABASE oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
   CREATE DATABASE hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
   #退出mysql
   quit;
   ```

8. 解压mysql-connector-java-5.1.27.tar.gz，并将连接器jar包拷贝到指定目录下

   ```shell
   tar -zxvf mysql-connector-java-5.1.27.tar.gz
   cd mysql-connector-java-5.1.27
   #重命名jar包
   mv mysql-connector-java-5.1.27-bin.jar mysql-connector-java.jar 
   #创建目录,复制jar包到目录下
   mkdir /usr/share/java
   cp mysql-connector-java.jar /usr/share/java/
   ```

9. 分发mysql连接器到所有节点（内含jdbc驱动）

   ```shell
   xsync /usr/share/java/mysql-connector-java.jar
   ```



### 1.5 安装CM

1. 上传CM安装包到hadoop101

   ```shell
   cm6.3.1-redhat7.tar.gz
   ```

2. 创建安装目录

   ```shell
   mkdir /opt/cloudera-manager
   ```

3. 解压安装包

   ```shell
   tar -zxvf cm6.3.1-redhat7.tar.gz
   cd cm6.3.1/RPMS/x86_64/
   #将三个安装包移动到安装目录下
   mv cloudera-manager-agent-6.3.1-1466458.el7.x86_64.rpm /opt/cloudera-manager/
   mv cloudera-manager-server-6.3.1-1466458.el7.x86_64.rpm /opt/cloudera-manager/
   mv cloudera-manager-daemons-6.3.1-1466458.el7.x86_64.rpm /opt/cloudera-manager/
   ```

4. 安装CM环境和客户端

   ```shell
   #复制3个安装包到所有节点
   xsync /opt/cloudera-manager/
   
   #所有节点安装CM环境
   cd /opt/cloudera-manager/
   rpm -ivh cloudera-manager-daemons-6.3.1-1466458.el7.x86_64.rpm
   
   #所有节点安装CM客户端
   yum install -y bind-utils psmisc cyrus-sasl-plain cyrus-sasl-gssapi fuse portmap fuse-libs /lib/lsb/init-functions httpd mod_ssl openssl-devel python-psycopg2 MySQL-python libxslt
   rpm -ivh cloudera-manager-agent-6.3.1-1466458.el7.x86_64.rpm
   ```

5. 修改agent的server节点

   ```shell
   #在所有节点修改配置
   vim /etc/cloudera-scm-agent/config.ini
   ```

   ```ini
   #服务端host均修改为hadoop101
   server_host=hadoop101
   ```

6. 安装服务端

   ```shell
   #在hadoop101安装服务端
   rpm -ivh cloudera-manager-server-6.3.1-1466458.el7.x86_64.rpm 
   ```

7. 修改服务端配置文件

   ```shell
   vim /etc/cloudera-scm-server/db.properties 
   ```

   ```properties
   com.cloudera.cmf.db.type=mysql
   com.cloudera.cmf.db.host=hadoop101:3306
   com.cloudera.cmf.db.name=scm
   com.cloudera.cmf.db.user=scm
   com.cloudera.cmf.db.password=scm
   com.cloudera.cmf.db.setupType=EXTERNAL
   ```

8. 上传CDH离线包到指定目录

   ```shell
   cd /opt/cloudera/parcel-repo
   ls
   ```

   ```shell
   CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel
   CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1
   manifest.json
   ```

   ```shell
   #重命名sha1文件
   mv CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1 CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha
   ```

9. 启动CM集群服务

   ```shell
   #hadoop101中启动server
   systemctl start cloudera-scm-server
   
   #所有节点启动agent
   systemctl start cloudera-scm-agent
   
   #可查看启动日志
   tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
   ```



### 1.6 CM集群部署

1. 登陆CM

   ```shell
   http://hadoop101:7180/cmf/login
   #用户名密码默认均为admin
   ```

2. 接受条款和相关协议

   ![image-20200824183634386](CDH.assets/image-20200824183634386.png)

   ![image-20200824183658244](CDH.assets/image-20200824183658244.png)

   ![image-20200824183732622](CDH.assets/image-20200824183732622.png)

3. 安装集群
   ![image-20200824183924113](CDH.assets/image-20200824183924113.png)

   自定义任意的集群名称
   ![image-20200824184007559](CDH.assets/image-20200824184007559.png)

   选择主机节点
   ![image-20200824184108274](CDH.assets/image-20200824184108274.png)

   选择CDH版本为6.3.2
   ![image-20200824184232816](CDH.assets/image-20200824184232816.png)

   等待安装完成后继续
   ![image-20200824184554280](CDH.assets/image-20200824184554280.png)

4. 检查网络性能和主机
   ![image-20200824184811738](CDH.assets/image-20200824184811738.png)

   根据提示修改集群配置
   ![image-20200824185044011](CDH.assets/image-20200824185044011.png)

   ```shell
   #在所有节点执行命令
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
   ```

   重新运行后通过检查，点击继续
   ![image-20200824185402963](CDH.assets/image-20200824185402963.png)

5. 自定义安装服务
   ![image-20200824185518313](CDH.assets/image-20200824185518313.png)

   勾选HDFS，YARN，ZooKeeper
   ![image-20200824185610895](CDH.assets/image-20200824185610895.png)

6. 配置hdfs，yarn和zookeeper安装节点
   ![image-20200824190215807](CDH.assets/image-20200824190215807.png)

   配置hdfs存储目录（可使用默认路径）
   ![image-20200824190358644](CDH.assets/image-20200824190358644.png)

   等待安装完成后继续，完成安装
   ![image-20200824190811642](CDH.assets/image-20200824190811642.png)



## 2. 数据采集模块安装

### 2.1 配置HDFS HA

1. 选择HDFS，修改配置
   ![image-20200824191125836](CDH.assets/image-20200824191125836.png)

2. 修改权限
   ![image-20200824191234604](CDH.assets/image-20200824191234604.png)

3. 启动HDFS高可用
   ![image-20200824191402588](CDH.assets/image-20200824191402588.png)

   自定义命名空间（任意）
   ![image-20200824191529660](CDH.assets/image-20200824191529660.png)

   配置多个NameNode节点（免费版仅支持2台）
   ![image-20200824191740889](CDH.assets/image-20200824191740889.png)

   配置jn存储目录后点击继续
   ![image-20200824191931018](CDH.assets/image-20200824191931018.png)

   等待初始化完成HA搭建
   ![image-20200824192523741](CDH.assets/image-20200824192523741.png)

   确认实例
   ![image-20200824192629219](CDH.assets/image-20200824192629219.png)



### 2.2 配置 YARN HA

1. 选择yarn，配置HA
   ![image-20200824192802063](CDH.assets/image-20200824192802063.png)

   ![image-20200824192827437](CDH.assets/image-20200824192827437.png)

2. 选择备用节点
   ![image-20200824192857027](CDH.assets/image-20200824192857027.png)

3. 等待安装完成，确认实例

   ![image-20200824193000331](CDH.assets/image-20200824193000331.png)

   ![image-20200824193026133](CDH.assets/image-20200824193026133.png)



### 2.3 安装Kafka

1. 添加Kafka服务
   ![image-20200824193214820](CDH.assets/image-20200824193214820.png)

   ![image-20200824193256982](CDH.assets/image-20200824193256982.png)

2. 配置kafka安装节点
   ![image-20200824193340285](CDH.assets/image-20200824193340285.png)

3. 配置最小内存为512M，继续
   ![image-20200824193430832](CDH.assets/image-20200824193430832.png)

4. 等待完成安装

5. （可选）安装过程中报错的解决方法
   确认所有节点的broker.id
   ![image-20200824193811949](CDH.assets/image-20200824193811949.png)

   在所有服务器节点中创建或修改配置文件

   ```shell
   vi /var/local/kafka/data/meta.properties
   ```

   ```properties
   #添加以下配置,注意broker_id与CM中查看的一致
   version=0
   broker.id=49
   ```



## 3.离线数仓环境搭建

### 3.1 安装hive

1. 添加Hive服务
   ![image-20200824194135346](CDH.assets/image-20200824194135346.png)

   ![image-20200824194202063](CDH.assets/image-20200824194202063.png)

2. 配置hive安装节点（hive on yarn仅需安装1台客户端）
   ![image-20200824194339746](CDH.assets/image-20200824194339746.png)

3. 配置Mysql用于存储hive的元数据，测试连接后继续
   ![image-20200824194447262](CDH.assets/image-20200824194447262.png)

4. 配置hive metastore端口（默认）
   ![image-20200824194558809](CDH.assets/image-20200824194558809.png)

5. 等待安装完成



## 4. 实时数仓环境搭建

### 4.1 安装spark

1. 添加spark服务
   ![image-20200824194850019](CDH.assets/image-20200824194850019.png)

   ![image-20200824194919067](CDH.assets/image-20200824194919067.png)

2. 配置历史服务和网关
   ![image-20200824195002650](CDH.assets/image-20200824195002650.png)

3. 安全配置（可跳过）
   ![image-20200824195034295](CDH.assets/image-20200824195034295.png)

4. 等待安装完成



## 5. 相关配置

### 5.1 开启HDFS集群的域名访问

dfs.client.use.datanode.hostname

![image-20200824195521073](CDH.assets/image-20200824195521073.png)


### 5.2 配置YARN的物理核虚拟核占比

yarn.nodemanager.resource.cpu-vcores

![image-20200824200226965](CDH.assets/image-20200824200226965.png)



### 5.3 配置单个contain的最大虚拟CPU数

yarn.scheduler.maximum-allocation-vcores

![image-20200824200915765](CDH.assets/image-20200824200915765.png)



### 5.4 配置Yarn单节点和单Container内存

yarn.scheduler.maximum-allocation-mb

![image-20200824201145516](CDH.assets/image-20200824201145516.png)

yarn.nodemanager.resource.memory-mb

![image-20200824201315001](CDH.assets/image-20200824201315001.png)



### 5.5 关闭Spark的动态资源分配

spark.dynamicAllocation.enabled

![image-20200824201506484](CDH.assets/image-20200824201506484.png)

hive中也需要修改此参数

![image-20200824201739251](CDH.assets/image-20200824201739251.png)

- **Spark3.0中若配置了自动缩减分区，则需要开启此配置项配合使用**



### 5.6 配置容量调度器

1. 选择容量调度器
   ![image-20200824201943937](CDH.assets/image-20200824201943937.png)
2. 配置多队列
   ![image-20200824202226356](CDH.assets/image-20200824202226356.png)



### 5.7 重启集群生效配置

1. 点击过期配置重启
   ![image-20200824202346145](CDH.assets/image-20200824202346145.png)
2. 确认所有配置修改
   ![image-20200824202414248](CDH.assets/image-20200824202414248.png)
3. 重启
   ![image-20200824202439690](CDH.assets/image-20200824202439690.png)



## 6. 安装OOZIE

1. 添加oozie服务

   ![image-20200824203202192](CDH.assets/image-20200824203202192.png)

2. 选择支持的框架
   ![image-20200824203229967](CDH.assets/image-20200824203229967.png)

3. 选择安装的节点
   ![image-20200824203330142](CDH.assets/image-20200824203330142.png)

4. 配置mysql

   ![image-20200824203810123](CDH.assets/image-20200824203810123.png)

5. 配置相关目录（默认）
   ![image-20200824203906462](CDH.assets/image-20200824203906462.png)

6. 等待安装完成



## 7. 安装HUE

1. 添加HUE服务
   ![image-20200824204124894](CDH.assets/image-20200824204124894.png)

   ![image-20200824204154080](CDH.assets/image-20200824204154080.png)

2. 选择支持的组件
   ![image-20200824204216891](CDH.assets/image-20200824204216891.png)

3. 选择安装的节点
   ![image-20200824204302100](CDH.assets/image-20200824204302100.png)

4. 配置Mysql连接
   ![image-20200824204424637](CDH.assets/image-20200824204424637.png)

5. 等待安装完成

6. 访问hue的web页面

   ```shell
   hadoop103:8888
   #用户名和密码均使用admin
   ```



## 8. 安装KUDU

1. 添加kudu服务
   ![image-20200824204927243](CDH.assets/image-20200824204927243.png)
2. 配置安装的节点
   ![image-20200824205319083](CDH.assets/image-20200824205319083.png)
3. 配置4个存储目录
   ![image-20200824205449715](CDH.assets/image-20200824205449715.png)
4. 等待安装完成



## 9. 安装Impala

1. 添加Impala服务
   ![image-20200824205636433](CDH.assets/image-20200824205636433.png)
2. 选择支持的组件
   ![image-20200824205711851](CDH.assets/image-20200824205711851.png)
3. 配置安装的节点
   ![image-20200824205836999](CDH.assets/image-20200824205836999.png)
4. 配置服务和存储目录（默认）
   ![image-20200824205908733](CDH.assets/image-20200824205908733.png)
5. 等待安装完成

