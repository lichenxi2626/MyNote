# Superset

## 第一章 Superset入门

### 1.1 superset概述

- Superset是一个开源的、现代的、轻量级BI分析工具，可对接多种数据源，拥有丰富的图标展示形式，支持自定义仪表盘，用于数据可视化的展示
- 官网地址：http://superset.apache.org/

### 1.2 superset应用场景

- superset可对接多种常见的数据分析工具，如kylin、hive、mysql等



## 第二章 Superset安装

- superset是使用Python编写的web应用，需要Python3.6环境。CentOS7自带Python版本为2.7，因此需要单独安装

### 2.1 安装python环境

1. 上传并安装Miniconda（Miniconda用于管理切换一台机器上的不同版本的Python环境）

   ```shell
   bash Miniconda3-latest-Linux-x86_64.sh
   
   #安装过程中需要经过三次提示
   #第1次提示直接确认
   #第2次提示时输入安装路径
   /opt/module/miniconda3
   #第3次提示是否执行初始化,选择是,会自动配置环境变量
   ```

2. 使环境变量生效

   ```shell
   source ~/.bashrc
   ```

3. 取消base环境自动激活

   ```shell
   #安装miniconda后,每次打开终端会默认激活base环境,可以通过配置关闭
   conda config --set auto_activate_base false
   ```

4. 配置conda国内（清华）镜像

   ```shell
   conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
   conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
   conda config --set show_channel_urls yes
   ```

5. 创建Python3.6环境

   ```shell
   conda create --name superset python=3.6
   
   #环境相关命令
   #创建环境 conda create -n env_name
   #查看所有环境 conda info --envs
   #删除环境 conda remove -n env_name --all
   ```

6. 激活环境

   ```shell
   conda activate superset
   
   #激活后命令行会出现环境表示
   (superset) [atguigu@hadoop201 kylin-3.0.2]$
   
   #退出当前环境命令
   conda deactivate
   ```



### 2.2 安装superset

1. 安装superset所需依赖

   ```shell
   sudo yum install -y python-setuptools
   sudo yum install -y gcc gcc-c++ libffi-devel python-devel python-pip python-wheel openssl-devel cyrus-sasl-devel openldap-devel
   ```

2. 安装superset所需工具

   ```shell
   pip install --upgrade setuptools pip -i https://pypi.douban.com/simple/
   ```

3. 安装superset

   ```shell
   pip install apache-superset -i https://pypi.douban.com/simple/
   ```

4. 初始化superset数据库

   ```shell
   superset db upgrade
   ```

5. 创建管理员用户

   ```shell
   export FLASK_APP=superset
   flask fab create-admin
   #flask执行后需要依次输入用户名/几个信息(可省略)/密码(输出2次)
   ```

6. 初始化superset

   ```shell
   superset init
   ```

7. 安装gunicorn

   ```shell
   #gunicorn是一个Python Web Server,类似于Java中的TomCat
   pip install gunicorn -i https://pypi.douban.com/simple/
   ```

8. 启动superset

   ```shell
   #启动前确保激活了conda的Python3.6环境
   gunicorn --workers 5 --timeout 120 --bind hadoop201:8787  "superset.app:create_app()" --daemon
   
   # workers 指定进程个数(用于可视化展示无需过多进程)
   # timeout worker的超时判定时间,超时自动重启
   # bind 绑定superset的访问地址
   # deamon 后台运行
   ```

9. 停止superset

   ```shell
   ps -ef | awk '/superset/ && !/awk/{print $2}' | xargs kill -9
   ```

10. 编写superset启停脚本

    ```shell
    cd bin/
    vi supersetctl
    ```

    ```shell
    #!/bin/bash
    
    superset_status(){
        result=`ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | wc -l`
        if [[ $result -eq 0 ]]; then
            return 0
        else
            return 1
        fi
    }
    superset_start(){
            # 该段内容取自~/.bashrc，所用是进行conda初始化
            # >>> conda initialize >>>
            # !! Contents within this block are managed by 'conda init' !!
            __conda_setup="$('/opt/module/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
            if [ $? -eq 0 ]; then
                eval "$__conda_setup"
            else
                if [ -f "/opt/module/miniconda3/etc/profile.d/conda.sh" ]; then
                    . "/opt/module/miniconda3/etc/profile.d/conda.sh"
                else
                    export PATH="/opt/module/miniconda3/bin:$PATH"
                fi
            fi
            unset __conda_setup
            # <<< conda initialize <<<
            superset_status >/dev/null 2>&1
            if [[ $? -eq 0 ]]; then
                conda activate superset ; gunicorn --workers 5 --timeout 120 --bind hadoop201:8787 --daemon 'superset.app:create_app()'
            else
                echo "superset正在运行"
            fi
    
    }
    
    superset_stop(){
        superset_status >/dev/null 2>&1
        if [[ $? -eq 0 ]]; then
            echo "superset未在运行"
        else
            ps -ef | awk '/gunicorn/ && !/awk/{print $2}' | xargs kill -9
        fi
    }
    
    
    case $1 in
        start )
            echo "启动Superset"
            superset_start
        ;;
        stop )
            echo "停止Superset"
            superset_stop
        ;;
        restart )
            echo "重启Superset"
            superset_stop
            superset_start
        ;;
        status )
            superset_status >/dev/null 2>&1
            if [[ $? -eq 0 ]]; then
                echo "superset未在运行"
            else
                echo "superset正在运行"
            fi
    esac
    ```

    ```shell
    chmod 777 supersetctl
    ```

11. 使用脚本启动、停止superset

    ```shell
    supersetctl start
    supersetctl stop
    ```

12. web端访问superset

    ```shell
    http://hadoop201:8787
    #使用步骤5中创建的用户名密码登陆
    ```



## 第三章 Superset使用

### 3.1 对接MySQL数据源

1. 安装mysql依赖

   ```shell
   #先启动conda环境
   conda install mysqlclient
   
   #对接其他数据源需要查看官网安装相应依赖
   http://superset.apache.org/installation.html#database-dependencies
   ```

2. 重启superset

   ```shell
   supersetctl restart
   ```

3. 配置数据源

   ![image-20200707203431100](Superset.assets/image-20200707203431100.png)
   自定义数据库名并配置mysql的url，关联mysql数据库，测试连接（seems ok）![image-20200707203622294](Superset.assets/image-20200707203622294.png)
   保存提交
   ![image-20200707203651950](Superset.assets/image-20200707203651950.png)

4. 配置数据表
   ![image-20200707203831051](Superset.assets/image-20200707203831051.png)
   选择上一步添加的数据库，并添加数据库中的表
   ![image-20200707204023938](Superset.assets/image-20200707204023938.png)



### 3.2 制作仪表盘（看板）

1. 创建仪表盘
   ![image-20200707204205400](Superset.assets/image-20200707204205400.png)
   添加仪表盘标题
   ![image-20200707204303799](Superset.assets/image-20200707204303799.png)
   保存新建
   ![image-20200707204333833](Superset.assets/image-20200707204333833.png)
2. 创建图表
   ![image-20200707204454057](Superset.assets/image-20200707204454057.png)
   选择已创建的数据表，并选择图表样式
   ![image-20200707204854964](Superset.assets/image-20200707204854964.png)
   配置图表
   ![image-20200707205426632](Superset.assets/image-20200707205426632.png)
   保存图表到看板
   ![image-20200707205722934](Superset.assets/image-20200707205722934.png)
3. 调整图表自动刷新时间
   ![image-20200707205931822](Superset.assets/image-20200707205931822.png)
4. 调整看板布局
   ![image-20200707210008260](Superset.assets/image-20200707210008260.png)



