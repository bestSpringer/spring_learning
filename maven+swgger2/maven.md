# Maven

### 基本配置

#### 安装

* 解压安装包
* 配置系统环境变量
* 本地仓库，远程仓库（私服），中央仓库
* conf-->settings.xml 配置本地仓库路径
* src
  * main
    * java
    * resources
  * test
    * java
    * resources
  * main
    * webapp(静态资源)

#### 命令

* mvn tomcat:run
  * 进入到pom.xml路径，用tomcat启动一个maven项目

* mvn clean
  * 清空target目录，将编译好的classes文件删除
* mvn compile
  * 生成target目录，存储着编译后的classes文件
* mvn test
  * 生成target下面的test-classes，将测试代码编译
  * 将核心代码和测试代码同时编译
* mvn package
  * 核心代码和测试代码编译并打包，打war包放在target目录
* mvn install
  * 将war包放在本地仓库
* mvn deploy
  * 将jar包上传到私服

#### 生命周期

构建的步骤：编译->测试->打包->安装->发布

#### idea集成maven

idea中的config  settings配置maven安装目录和settings目录

### 依赖管理

#### scope标签

* provided
  
  * 在编译，测试时有效，例如servlet-api
  
  * 写代码的时候需要，但是运行的时候不需要
  * 例如：mvn tomcat:run 编写servlet时，tomcat里面有jar包jsp-api,servlet-api，pom文件里面也引入这两个jar包，pom文件引入的jar包是为了解决编码中包不存在的错误，然而真正执行时没有用pom引入的包，而是用tomcat自带的jar包
  
* test
  
  * 在测试时有效，例如junit
  * 单元测试运行的时候需要，编码正确，真正跑项目的时候不需要scope为test的jar包
  
* runtime

  * 在测试，运行时有效，例如JDBC驱动

* compile

  * 默认为compile，在编译时，测试时，运行时都有效，例如spring-core

#### maven修改运行环境

```java
//原来运行环境为tomcat6，修改tomcat运行环境
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <port>8888</port>
                </configuration>
            </plugin>
            
            //修改编译版本
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
</build>
```

#### dependencyManagement标签

背景：maven工程是可以分父子依赖的，依赖父项目后，子项目的依赖jar包会传递依赖，eg：当B依赖A项目后，A项目的jar包会传递到B项目，B项目开发者重新引入一套ssm的jar包，对于B来说是直接依赖，那么B项目的依赖包会直接覆盖掉A项目的jar包。

目的：为了防止上述问题的发生，可以在A项目中主要jar包的坐标锁住。

使用：只锁定版本，不真正引入jar包

```java
//声明jdk版本
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring-boot.version>2.3.1.RELEASE</spring-boot.version>
</properties>

<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>${spring-boot.version}</version>
            </dependency>
        </dependencies>
</dependencyManagement>

<dependencies>
        <!--web功能的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
</dependencies>      
```

#### jar包冲突

解决冲突的三种方式

* 声明优先原则
  * 哪个jar包的坐标在靠上的位置，优先使用哪个jar包的版本
* 路径近者优先原则
  * 直接依赖比传递依赖路径近
* 直接排除法
  * 要排除某个jar包的依赖包，在配置exclusions标签的时候，内部可以不写版本号，因为此时依赖包使用的版本号和默认jar包是一样的

#### jar包上传（私服）

将jar包上传到私服

```java
<distributionManagement>
        <repository>
            <id>releases</id>
            <url>http://localhost:8081/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
</distributionManagement>
```

配置完成之后，点击mvn deploy上传到私服

#### jar包下载（私服）

将jar包从私服上下载，在maven->conf->settings.xml文件里配置

```java
<profile>
      <id>dev</id>
      <repositories>
            <repository>
              <id>nexus</id>
              <url>http://localhost:8081/nexus/content/groups/public/</url>
              <releases><enabled>true</enabled></releases>
              <snapshots><enabled>false</enabled></snapshots>
            </repository>
        <repository>
              <id>nexus-snapshots</id>
              <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
              <releases><enabled>false</enabled></releases>
              <snapshots><enabled>true</enabled></snapshots>
            </repository>
        <repository>
              <id>nexus-releases</id>
              <url>http://localhost:8081/nexus/content/repositories/releases/</url>
              <releases><enabled>true</enabled></releases>
              <snapshots><enabled>false</enabled></snapshots>
            </repository>
      </repositories>
      <pluginRepositories>
            <pluginRepository>
              <id>nexus-plugin-releases</id>
              <url>http://localhost:8081/nexus/content/repositories/releases/</url>
              <releases><enabled>true</enabled></releases>
              <snapshots><enabled>false</enabled></snapshots>
            </pluginRepository>
            <pluginRepository>
              <id>nexus-3rd-plugin-releases</id>
              <url>http://localhost:8081/nexus/content/groups/public/</url>
              <releases><enabled>true</enabled></releases>
              <snapshots><enabled>false</enabled></snapshots>
            </pluginRepository>
      </pluginRepositories>
    </profile>

  <activeProfiles>
    <activeProfile>dev</activeProfile>
  </activeProfiles>
```

#### 父子项目

父项目：pom.xml  和 iml

子模块：module

启动

* 第一种：启动父项目
* 第二种：启动web模块，必须将service模块上传到仓库中
* 第三种：tomcat启动

#### Maven环境隔离

