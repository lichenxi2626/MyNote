# Git

## 第一章 Git简介及安装

### 1.1 Git概述

- Git是一个开源的分布式版本控制系统，用于高效的处理和管理项目
- Git的功能
  1. 协同开发：多人共同开发项目，可同步代码状态
  2. 冲突解决：多人对同一段代码做修改并提交时，会提示冲突并引导开发人员手动合并
  3. 版本记录：对于每一次提交到仓库的代码，Git都会留痕处理，保留代码的所有历史状态
  4. 历史追查：对于历史代码，可以查询签名等明细信息
  5. 代码备份：除本地外，代码会在本地仓库中备份
  6. 权限管理：对不同用户分别赋予项目的读写权限
  7. 版本还原：对于仓库中项目的所有历史版本，都可以随时实现还原
  8. 分支管理：对于需要不断小范围更新并开发大版本更新的项目，可以通过分支开发，同时进行，最后合并



### 1.2 Git安装

- 官网下载安装包并安装
  https://git-for-windows.github.io/



## 第二章 Git命令

### 2.1 配置用户名及邮箱

```shell
#配置全局用户名(默认使用的签名)
git config --global user.name "lichenxi"
#配置全局邮箱(默认使用的签名)
git config --global user.email "lichenxi2626@126.com"
#全局配置存储路径为 C:\Users\li\.gitconfig

#配置当前项目的用户名
git config user.name "lichenxi"
#配置当前项目的邮箱
git config user.email "lichenxi2626@126.com"
#项目配置的存储路径为 ./.git/config
```



### 2.2 版本管理

- 对于一个项目，在本地中由3部分构成
  1. 工作区：项目代码的直接存储目录
  2. 暂存区：存放在项目根目录下的.git/index文件中，保留工作区代码的所有临时修改记录
  3. 本地库：存放在项目根目录下的.git目录中，保留项目暂存区提交的项目的所有历史版本

```shell
#1.工作区操作
#进入项目文件夹,创建本地库
git init

#2.添加工作区的文件到暂存区
git add [文件名]
git add src

#3.将暂存区代码提交到本地库,并添加版本注释
git commit
git commit -m [注释]
git commit -m [v1.0]

#4.查看本地库所有历史版本
git log
git log --pretty=oneline #简易版
git reflog #查看项目版本更改的历史记录及版本号(7位字符串)
git reflog [文件名] #查看指定文件更改的历史记录及版本号(7位字符串)

#5.从本地库拉取代码,回退工作区及暂存区的项目版本
git reset --hard HEAD^ #回退到本地库的上1个提交的版本
git reset --hard HEAD~n #回退到本地库的上n个提交的版本
git reset --hard [版本号] #使用历史版本的版本号回退到历史版本

#6.从暂存区拉取代码,还原工作区的数据(注意空格)
git checkout -- [文件名]
git checkout -- src

#7、删除文件(先从工作区删除,再提交,但本地库实际还是保留了历史代码)
rm test.java
git add test.java
git commit
```



### 2.3 分支创建

- 当项目需要进入新版本的长工期开发，同时又需要修改旧版本bug时，可以通过创建分支隔离开发

```shell
#1.创建分支
git branch [分支名]

#2.查看项目的全部分支
git branch -v

#3.切换分支
git checkout [分支名]
git checkout -b [分支名] #创建并切换到分支

#4.将指定分支合并到当前分支
git checkout master #切回主干
git merge [分支名]
#若合并的两个分支具有冲突代码,则会自动提示,并进入MERGINT状态,此时需要修改提示文件,并重新提交
#提示如下
Auto-merging src/bigdata.java
CONFLICT (content): Merge conflict in src/bigdata.java
Automatic merge failed; fix conflicts and then commit the result.
#打开提示的代码文件(自动对比冲突的部分)
<<<<<<< HEAD
master
=======
branch
>>>>>>> branch01
#修改后保存
master
#重新提交,成功后结束MERGING状态
```



## 第三章 Github

### 3.1 github概述

- Github是一个Git项目托管网站，主要提供基于Git的版本托管服务
- 网址 https://github.com/
- 注册帐号（lichenxi2626@126.com）



### 3.2 github使用

1. 登陆github，创建仓库
   ![image-20200712190059039](Git.assets/image-20200712190059039.png)
   设置仓库名，读权限，确认创建
   ![image-20200712190253840](Git.assets/image-20200712190253840.png)

2. 获得仓库url地址

   ![image-20200712190705168](Git.assets/image-20200712190705168.png)

3. 启动Git Bash，执行命令

   ```shell
   #1.本地仓库映射到github仓库
   git remote add [远端代号] [远端地址] #远端代号一般使用origin
   git remote add origin https://github.com/lichenxi2626/GmallTest.git
   
   #2.将本地仓库代码推送到github仓库
   #推送时需要输入github仓库所属用户的账号密码
   git push [远端代号] [本地分支名称]
   git push origin master
   
   #3.从github仓库拉取代码到本地仓库
   #拉取后会同时更新到本地的本地库、暂存区及工作区
   git pull [远端代号] [远端分支名称]
   git pull origin master
   
   #4.从github仓库克隆项目到本地,远端代号为origin
   #可自行创建目录并在目录下启动git bash直接执行,无需在本地创建git仓库
   git clone [远端地址] [新项目本地路径]
   git clone https://github.com/lichenxi2626/GmallTest.git ./
   
   # pull与clone对比
   # pull操作是从远端仓库将指定分支的项目代码的最新版本拉取到本地,更新本地代码
   # clone操作是从远端仓库将整个仓库复制到本地,包括所有的分支和历史版本
   ```



### 3.3 github用户权限

- 对于public的github仓库，push和pull操作默认只能由远端仓库创建者使用（需要输入帐号密码），而其他用户仅能使用clone命令复制仓库到本地（只读权限）
- 若其他用户需要获取push及pull权限，可由仓库创建者引入合作者用户
  1. 在指定仓库下邀请其他用户作为该仓库的合作者
     ![image-20200712194120769](Git.assets/image-20200712194120769.png)
  2. 被邀请用户接受邀请后，即称为合作者，拥有该仓库文件的读写权限
- 对于创建者和合作者以外的用户，除clone命令外还可以
  1. fork--将其他用户的public仓库整体复制到自己的github仓库中
     ![image-20200712223941523](Git.assets/image-20200712223941523.png)
  2. pull request--将修改后的fork项目提交给创建者检验，创建者可在确认后将pull来的代码更新到自己的github仓库
     ![image-20200712224138381](Git.assets/image-20200712224138381.png)



## 第四章 IDEA中使用Git

### 4.1 配置和创建Git工程

1. 启动IDEA，配置git.exe路径
   ![image-20200713092337189](Git.assets/image-20200713092337189.png)

2. 在GitHub中创建仓库，获取url地址（参考3.2）

3. 使用Git创建工程
   <img src="Git.assets/image-20200713095820535.png" alt="image-20200713095820535" style="zoom: 80%;" />
   使用url创建
   <img src="Git.assets/image-20200713100054635.png" alt="image-20200713100054635" style="zoom:80%;" />

4. 编写代码时可能出现提交代码提示，选择不再提示
   <img src="Git.assets/image-20200713100459326.png" alt="image-20200713100459326" style="zoom:80%;" />

5. 配置GitHub用户信息

   ![image-20200713100826261](Git.assets/image-20200713100826261.png)



### 4.2 提交和推送

1. 编写代码
2. 右键文件，添加文件到暂存区
   ![image-20200713101034772](Git.assets/image-20200713101034772.png)
3. 从暂存区更新到本地库
   <img src="Git.assets/image-20200713101137236.png" alt="image-20200713101137236" style="zoom:80%;" />
   进行提交相关配置
   <img src="Git.assets/image-20200713101538148.png" alt="image-20200713101538148" style="zoom:80%;" />
4. 推送到GitHub远端仓库
   ![image-20200713101703527](Git.assets/image-20200713101703527.png)
   
   <img src="Git.assets/image-20200713101746393.png" alt="image-20200713101746393" style="zoom:80%;" />
5. 查看github仓库更新



## 第五章 Git工作流

### 5.1 GitFlow工作流

- 集中式工作流：一个项目中，远端仓库仅存有Master分支，而本地也仅存有Master分支，项目组内所有成员的对项目代码的修改都提交到该分支中。
- gitflow工作流：一个项目中，功能开发、发布准备、维护等工作各自维护一个独立的分支，开发工作互不影响。
- giflow工作流图示
  <img src="Git.assets/image-20200713111729790.png" alt="image-20200713111729790" style="zoom: 67%;" />
  1. master分支：主干分支，与正在运行的生产环境一致
  2. develop分支：管理在开发过程中的分支
  3. hotfix分支：用于紧急修复bug的分支
  4. release分支：用于测试develop分支，测试完成后合并到主干分支
  5. feature分支：对develop分支的进一步细化，当不同工作组同时进行多个功能的开发时可使用各自的分支，开发完成后合并到develop分支



### 5.2 IDEA中分支的使用

#### 5.2.1 创建分支

1. 创建分支
   <img src="Git.assets/image-20200713113811235.png" alt="image-20200713113811235" style="zoom:80%;" />

   命名并切换到分支
   <img src="Git.assets/image-20200713113852836.png" alt="image-20200713113852836" style="zoom:80%;" />

2. 编辑代码，提交到本地仓库

3. push到远端仓库
   <img src="Git.assets/image-20200713114209559.png" alt="image-20200713114209559" style="zoom:80%;" />

4. 查看github远端仓库中，上传的新分支代码



#### 5.2.2 切换分支

- 使用checkout切换本地库中的分支
  <img src="Git.assets/image-20200713114928883.png" alt="image-20200713114928883" style="zoom:80%;" />

- fetch刷新远端仓库的分支列表

  ![image-20200713115340826](Git.assets/image-20200713115340826.png)
  刷新后可查看到远端仓库的所有分支
  <img src="Git.assets/image-20200713115431043.png" alt="image-20200713115431043" style="zoom:80%;" />



#### 5.2.3 合并分支

1. 切换到主分支
   <img src="Git.assets/image-20200713115611459.png" alt="image-20200713115611459" style="zoom:80%;" />
2. 选择需要合并的分支，进行合并
   <img src="Git.assets/image-20200713115717742.png" alt="image-20200713115717742" style="zoom:80%;" />
3. 若产生代码冲突，可手动修改