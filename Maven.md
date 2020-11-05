# Maven

***

## 第一章 为什么使用Maven

### 1.1 大量的第三方jar包

- 在JavaEE开发中，我们需要导入大量的jar包。而对于每一个工程与工程中，存在大量重复导入的jar包，造成工作区文件的冗余。
- 通过使用Maven，每个jar包只需要在本地仓库中保存一份，而对于需要导入该jar包的工程本身，只需要维护一个引用（Maven中称为坐标），极大地节约了存储空间。

### 1.2 jar包之间存在依赖关系

- 不同的jar包之间并非完全独立存在，一些jar包需要在其他的jar包支持下工作，这就是jar包之间的依赖关系。
- jar包之间存在依赖关系，而我们不可能在导入每一个需要的jar包时再人为地依次导入所有的依赖包，因此引入了Maven。Maven可以自动的将一个jar包和它所依赖的jar包一次性全部自动导入，解决了人工导入的弊端。

### 1.3 jar包之间存在冲突问题

- 当工程下jar包数量过多时，因为同时存在大量的依赖关系，功能类似但版本不同jar包之间会产生冲突。
- Maven提供了最短路径优先和先声明优先两条依赖原则，自动的处理jar包间的冲突问题。

### 1.4 获取jar包方式繁杂

- JavaEE开发中使用的jar包种类繁多，而因为没有统一的规范，要自己在网络上获取这些不同的jar包是非常困难的。
- Maven提供了一套统一规范的jar包管理体系。当我们需要使用某个jar包实现相关功能时，只需要以坐标的方式依赖一个jar包，Maven就会自动从中央仓库中下载该jar包及其全部的依赖jar包，一次性满足准确性和规范性。

### 1.5 项目拆分和分布式部署

- 对于规模较大的JavaEE项目，在开发中需要对工程的不同模块进行拆分、分工，同时要保证拆分后的工程依然可以相互访问和调用。
- Maven具备依赖管理管理机制，当上层模块依赖于下层模块时，下层模块中定义的API都可以为上层模块访问和调用。
- 当一个工程的不同模块需要各自运行在独立的服务器上时（分布式部署），同样需要使用Maven实现、



## 第二章 Maven的介绍

### 2.1 自动化构建工具

- **Maven是一款自动化构建工具，专注服务于Java平台的项目构建和依赖管理**。

### 2.2 构建的概念

- 构建不等于创建
  - 对于Java代码，将代码**编译**为字节码文件的过程即为构建过程；
  - 对于Web哦工程，将Web工程编译的结果**部署**到服务器上的过程即为构建过程；
  - 对于实际项目，将我们编写的Java代码、框架配置文件等其他资源文件作为原材料，然后生产出一个可以运行的项目的过程即为构建的过程。

### 2.3 构建的环节

1. 清理（clean）：删除以前的编译结果，为重新编译做好准备；
2. 编译（compile）：将Java源程序编译为字节码文件；
3. 测试（test）：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性；
4. 报告：在每一次测试后以标准的格式记录并展示测试结果；
5. 打包（package）：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java工程对应jar包，Web工程对象war包；
6. 安装（install）：在Maven环境下将打包的结果（jar包或war包）安装到本地仓库中；
7. 部署（deploy）：将打包的结果部署到远程仓库或将war包部署到服务器上运行。

### 2.4 自动化构建

- 对于上述的构建环节，在IDEA中都可以找到相应的操作，但这些步骤大部分完全可以交给机器完成，Maven就实现了从构建过程起点一直执行到终点的自动化构建过程。



## 第三章 Maven的使用

### 3.1 安装Maven核心程序

1. 配置java环境变量，Maven是使用Java开发的，因为需要将jdk的bin目录配置到环境变量中；
2. 解压Maven核心程序并配置环境变量，同样的需要将Maven安装目录下的bin目录配置到环境变量中；
3. cmd中执行mvn -v查看Maven版本信息并确认安装正确。

### 3.2 第一个Maven工程

- Maven约定的目录结构

  ```java
  Hello//目录名
  	--src
      	--main//存放主程序
      		--java//存放源代码文件
      		--resources//存放配置文件和资源文件
      	--test//存放测试程序
      		--java//存放源代码文件
      		--resources//存放配置文件和资源文件
      --pom.xml//核心配置文件
  ```

- 命令行中的Maven指令

  ```
  mvn clean
  mvn compile
  mvn test-compile
  mvn test
  mvn package
  mvn install
  ```

### 3.3 Maven的联网

- 配置本地仓库位置

  1. 打开Maven的核心配置文件\apache-maven-3.2.2\conf\settings.xml

  2. 插入语句声明本地仓库目录

    ```xml
    <localRepository>D:\develop\Maven\LocalRepository</localRepository>
    ```

- 配置阿里云镜像

  1. 打开Maven的核心配置文件\apache-maven-3.2.2\conf\settings.xml

  2. 在<mirrors>标签中插入配置

    ```xml
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    ```
  
- 配置maven默认编译器版本为jdk1.8

  1. 打开Maven的核心配置文件\apache-maven-3.2.2\conf\settings.xml

  2. 在<profiles>标签中插入配置

     ```xml
     <profile>  
         <id>jdk-1.8</id> 
         <activation>  
             <activeByDefault>true</activeByDefault>  
             <jdk>1.8</jdk>  
         </activation>  
         <properties>  
             <maven.compiler.source>1.8</maven.compiler.source>  
             <maven.compiler.target>1.8</maven.compiler.target>  
             <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
         </properties>
     </profile>
     ```



### 3.4 IDEA中配置Maven

1. 以管理员身份启动IDEA，并打开Settings；
2. 设置Maven的安装目录及本地仓库![](maven.assets/image-20200330194955760.png)
   - Maven home directory：可以指定本地 Maven 的安装目录所在，因为我已经配置了 M2_HOME 系统参数，所以直接这样配置 IntelliJ IDEA 是可以找到的。但是假如你没有配置的话，这里可以选择你的 Maven 安装目录。此外，这里不建议使用IDEA默认的。
   - User settings file / Local repository：我们还可以指定 Maven 的 settings.xml 位置和本地仓库位置。
3. 配置Maven自动导入依赖的jar包![](maven.assets/image-20200330195043156.png)
   - Import Maven projects automatically：表示 IntelliJ IDEA 会实时监控项目的 pom.xml 文件，进行项目变动设置，勾选上。
   - Automatically download：在 Maven 导入依赖包的时候是否自动下载源码和文档。默认是没有勾选的，也不建议勾选，原因是这样可以加快项目从外网导入依赖包的速度，如果我们需要源码和文档的时候我们到时候再针对某个依赖包进行联网下载即可。IntelliJ IDEA 支持直接从公网下载源码和文档的。
   - VM options for importer：可以设置导入的 VM 参数。一般这个都不需要主动改，除非项目真的导入太慢了我们再增大此参数。

### 3.5 IDEA中创建Maven Module

1. File→new Module→Maven→Next

   ![image-20200330195222583](maven.assets/image-20200330195222583.png)

2. 配置坐标→Next

   ![image-20200330195506714](maven.assets/image-20200330195506714.png)

3. 目录结构同3.2中的目录结构

### 3.6 打包插件

- Maven中的打包操作默认只将当前Module工程打包，当我们需要把当前Module工程所依赖的jar包一并打包时，需要在pom.xml中加入打包插件语句。

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
              <archive>
                    <manifest>
                     <!-- 指定主类 -->
                        <mainClass>xxx.xxx.XXX</mainClass>
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



## 第四章 Maven的核心概念

### 4.1 POM

- Project Object Model：项目对象模型。将Java工程的相关信息封装为对象作为便于操作和管理的模型记录在pom.xml文件中，是Maven工程的核心配置。

### 4.2 约定的目录结构

- 根据 约定>配置>编码 的原则，我们确定了Maven中的文件保存目录的结构，即

  ```java
  Hello//目录名
  	--src
      	--main//存放主程序
      		--java//存放源代码文件
      		--resources//存放配置文件和资源文件
      	--test//存放测试程序
      		--java//存放源代码文件
      		--resources//存放配置文件和资源文件
      --pom.xml//核心配置文件
  ```

### 4.3 坐标

- 通过三个向量，使我们在Maven仓库中唯一的确定一个Maven工程

  - groupId：公司或组织的域名倒序+项目名
  - artifactId：项目的模块名
  - version：模块的版本

- 代码实现

  ```xml
  <groupId>com.atguigu.maven</groupId>
  <artifactId>Hello</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  ```

- 如何在仓库中定位jar包

  - 前提：**Maven工程必须执行过Install操作才会装进仓库**

  - 方式一：将三个向量依次连接

    ``` xml
    com.atguigu.maven+Hello+0.0.1-SNAPSHOT
    ```

  - 方式二：将三个向量作为字符串连接为目录结构

    ```xml
    com/atguigu/maven/Hello/0.0.1-SNAPSHOT/Hello-0.0.1-SNAPSHOT.jar
    ```

### 4.4 依赖管理

- 当jar包A需要用到jar包B中的类时，我们就说A对B有依赖。

- 当A依赖B，B依赖C时，称A对B、B对C为直接依赖，称A对C为间接依赖

- 配置文件中声明依赖的格式

  ```xml
  <dependency>
      <!--坐标-->
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.0</version>
      <!--依赖的范围-->
      <scope>test</scope>
  </dependency>
  ```

#### 4.4.1 依赖的范围

- **compile**（默认）：主程序、测试程序、服务器运行都需要使用
  - main目录下的Java代码**可以**访问该依赖
  - test目录下的Java代码**可以**访问该依赖
  - 部署到Tomcat服务器上运行时**会**放在WEB-INF的lib目录下
- **test**：仅在测试时需要使用
  - main目录下的Java代码**不可以**访问该依赖
  - test目录下的Java代码**可以**访问该依赖
  - 部署到Tomcat服务器上运行时**不会**放在WEB-INF的lib目录下
- **provided**：服务器会提供相关API
  - main目录下的Java代码**可以**访问该依赖
  - test目录下的Java代码**可以**访问该依赖
  - 部署到Tomcat服务器上运行时**不会**放在WEB-INF的lib目录下
- runtime、import、system等

#### 4.4.2 依赖的传递性

- **对于间接依赖的情况，如A依赖B而B依赖C时，只有B中声明C的范围为compile时C才对A可见。**

#### 4.4.3 依赖传递的原则

- 当依赖中同时存在不同版本的相同jar包时，依赖的传递遵守以下原则
  - **路径最短者优先**
  - **路径相同时先声明者优先**（声明的先后取决于pom.xml文件中依赖声明的先后）

#### 4.4.4 依赖的排除

- 为了确保程序可以正确的执行有可能出现的重复的依赖，我们可以不依靠依赖传递的原则，手动的排除掉不想传递的依赖；

- 代码实现

  ``` xml
  <dependency>
      <groupId>com.atguigu.maven</groupId>
      <artifactId>OurFriends</artifactId>
      <version>1.0-SNAPSHOT</version>
      <!--依赖排除-->
      <exclusions>
          <exclusion>
              <groupId>commons-logging</groupId>
              <artifactId>commons-logging</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  
  <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.1.2</version>
  </dependency>
  ```

#### 4.4.5 统一管理jar包的版本

- 当我们在Maven工程中需要导入同一个版本的多个不同的jar包时，为了后续维护、切换版本的便利，可以将依赖的version声明为一个变量，用于统一管理。

- 代码实现

  ```xml
  <!--统一管理当前模块的jar包的版本-->
  <properties>
      <spring.version>4.0.0.RELEASE</spring.version>
  </properties>
  <!--依赖多个同版本的jar包-->
  <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>${spring.version}</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>${spring.version}</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jdbc</artifactId>
          <version>${spring.version}</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-orm</artifactId>
          <version>${spring.version}</version>
      </dependency>
  </dependencies>
  ```


### 4.5 仓库

- 分类
  - 本地仓库：为当前本机电脑上的所有Maven工程服务
  - 远程仓库
    - 私服：架设在当前局域网环境下，为当前局域网范围内的所有Maven工程服务。
    - 中央仓库：架设在Internet上，为全世界所有的Maven工程服务。
    - 中央仓库的镜像：架设在各个大洲，为中央仓库分担流量、减轻压力，同时更快的相应用户的请求。
- 仓库中的文件
  - Maven的插件
  - 我们自己开发的项目的模块
  - 第三方框架或工具的jar包

### 4.6 生命周期

- Maven的生命周期定义了各个构建环节的执行顺序，通过这个清单，Maven可以自动化的执行构建的相应命令
- 三套相互独立的生命周期，可独立调用，互不影响
  - Clean Lifecycle：进行构建前进行清理；
  - Default Lifecycle：构建的核心部分，编译，测试，大包，安装，部署等等；
  - Site Lifecycle：生成项目报告，站点，发布站点。
- 每套生命周期又由一个个阶段组成，命令行中的指令即为一个相应的阶段。**运行一个周期中的任意阶段时，从该生命周期开始到该阶段的所有阶段都会被执行**。

#### 4.6.1 Clean Lifecycle

Clean生命周期包含三个阶段

- pre-clean 执行clean前的准备工作
- clean 移除所有上一次构建生成的文件
- post-clean 执行clean后需要立刻完成的工作

#### 4.6.2 Site Lifecycle

Site生命周期包括四个阶段

- pre-site 执行生成站点文档前的准备工作
- site 生成项目的站点文档
- post-site 执行生成站点文档后需要立刻完成的工作，以及部署的准备工作
- site-deploy 将生成的站点文档部署到特定的服务器上

#### 4.6.3 **Default Lifecycle**

Defalut生命周期中的**常用**阶段

- process-resources 复制并处理资源文件至目标目录，准备打包
- compile 编译项目的源代码
- process-test-resources 复制并处理资源文件至目标测试目录
- test-compile 编译测试源代码
- test 使用适合的单元测试框架运行测试，测试代码不会被打包或部署
- package 将编译完成的代码打包为可发布的格式（jar、war、pom）
- install 将包安装至本地仓库，为其他项目提供依赖
- deploy 将最终的包复制到远程的仓库，或部署到服务器上运行

### 4.7 插件和目标

- Maven定义了抽象的声明周期，而**生命周期的每一个具体的阶段都是由一个具体的插件完成的**。
- 一个插件可以实现多个功能，这些功能即为插件的**目标**。
- Maven声明周期的每一个阶段分别对应了某个插件的某一功能。



## 第五章 Maven的继承

### 5.1 继承的需求

- 通过Maven的依赖传递机制，我们可以传递声明范围为compile的依赖，减少依赖的冗余。但对于其他范围类型的依赖（如仅在test中访问的的junit等）却不能传递，但实际上存在对这些jar包统一管理的需求。
- Maven中提供了继承机制，将不同Maven工程下共有的依赖提取到父工程中统一的管理，用于解决这一需求。

### 5.2 继承的实现

1. 创建父工程，仅需配置并保留pom.xml文件，将父工程的打包方式声明为pom

   ```xml
   <groupId>com.atguigu.maven</groupId>
   <artifactId>Parent</artifactId>
   <!--打包方式声明语句(不显式声明时默认为jar)-->
   <packaging>pom</packaging>
   <version>1.0-SNAPSHOT</version>
   ```

2. 在父工程中将需要统一管理的依赖使用dependencyManagement标签进行声明

   ```xml
   <!--依赖管理-->
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.0</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

3. 在子工程中使用parent标签引用父工程实现继承

   ```xml
   <!--继承声明-->
   <parent>
       <!--声明父工程坐标-->
       <groupId>com.atguigu.maven</groupId>
       <artifactId>Parent</artifactId>
       <version>1.0-SNAPSHOT</version>
   	<!--指定从当前pom.xml文件出发寻找父工程的pom.xml文件的相对路径-->
   	<relativePath>../Parent/pom.xml</relativePath>
   </parent>
   
   <!--依赖中声明要继承的父工程中的具体依赖,不声明范围和版本号-->
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
       </dependency>
   </dependencies>
   ```



## 第六章 聚合

### 6.1 聚合的需求

- 在实际的项目开发中，讲一个工程划分为模块后，不同模块的依赖关系非常复杂，若不能正确的理清依赖关系按照正确的顺序安装则会出现错误。
- 使用Maven中的聚合可以将有依赖关系的不同module进行自动的安装、清理工作，不需要再确认其中具体的依赖关系。

### 6.2 聚合的实现

- 通常将聚合的声明代码定义在父工程中，用于统一批量的管理

```xml
<!--聚合-->
<modules>
    <!--使用相对路径依次声明具有依赖关系的module-->
    <module>../MakeFriend</module>
    <module>../OurFriends</module>
    <module>../HelloFriend</module>
    <module>../Hello</module>
</modules>
```



## 第七章 创建Web工程

1. 创建Maven工程，修改pom.xml文件

   ```xml
   <groupId>com.atguigu.maven</groupId>
   <artifactId>MavenWeb</artifactId>
   <packaging>war</packaging>
   <version>1.0-SNAPSHOT</version>
   ```

2. 在Project Structure中选择对应的Module，添加Web目录

3. 在Web目录下创建index.jsp页面

4. 部署到Tomcat上运行



## 附： Maven相关网站

我们可以通过以下网站搜索需求的jar包的依赖信息（坐标等）

- http://mvnrepository.com/
- http://search.maven.org/