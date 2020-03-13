# 通过model创建工程模块

首先新建一个maven项目，然后删除其中的src目录。此时整个项目就变成了一个空的骨架，这时的pom.xml文件就作为了以后model的总pom.xml文件了，承载了一些公共的依赖和配置。

建好父项目之后，直接在工程上面右键 -->new -->model即可，剩下的配置和创建过程就和平时的工程创建是一样的。

当新建一个model后，在最外层骨架的pom.xml中就会多一个model标签，同时父工程中的jar包，子model完全可以使用，反之则不行。

# 使用maven创建web工程

1. 新建一个maven项目
2. 选择webapp骨架
3. 补齐目录结构
4. 配置本地tomcat服务器（结束）

# Maven的作用

简单来说，maven是一个项目构建工具，在Java开发中可以很便捷的管理jar包等。

Maven通过坐标的方式来从本地仓库中获取jar包，坐标的构成一般为：公司/组织名（groupId）+项目名/子项目名（artifactId）+版本号组成

对于maven的设置可以在maven开发包中的setting.xml中进行同意设置，包括配置阿里云镜像，修改本地仓库操作。

maven通过本地仓库管理jar包，可以让jar包和项目分离，从而减小项目体积，并且多个项目可以公用一个仓库，因此项目越多的时候越能体现出maven的高效和便捷。

# Maven的三种仓库

- 本地仓库
- 中央仓库
- 私服（远程仓库）

本地仓库主要存放我们工程用到的jar包，是通过我们自己维护的。而本地仓库的jar包就是从中央仓库中下载的。

中央仓库是由第三方的组织或者官方进行维护的。因此我们从中央仓库下载的jar包基本都是标准的jar包。

私服是由公司搭建的，由公司内部维护的一个仓库，其中的jar包也是从中央仓库中下载的。配置私服主要是为了公司jar包安全和便于统一管理。

在项目中如果配置私服，那么工程加载jar包的顺序如下：

首先工程还是会去本地仓库中查找相应的jar包，如果没有找到，则会去私服中查找，如果仍然没有找到，就会到中央仓库进行查找，找到后会下载到私服，私服有了jar包，本地仓库就会从私服中下载jar包。最后本地仓库中也有了jar包，则项目使用本地仓库中的jar包。

# 配置阿里云的仓库镜像

将下面的配置代码复制到setting.xml中的mirrors标签中即可

```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

# Maven的其他优点

Maven由纯Java语言开发，因此支持跨平台的使用。同时使用maven开发，可以得到一个很清晰的工程结构，使用maven可以将一个大型项目按照模块拆分成若干个工程，不同的团队只需维护自己的模块即可。

# Maven的常用命令

- clear：清理，清理编译出来的目录（target）
- compile：编译，只编译main目录，不编译test目录
- test-compile：编译test目录
- test：运行test目录中的代码
- package：打包 ，Java项目打成jar包，web项目打成war包
- install：将项目发布到本地仓库，可以使用此命令将自己写好的jar包供其他团队使用
- tomcat:run：一建构建项目

# Maven命令的声明周期

- Clean：生命周期：clean
- Default：生命周期：compile、test-compile、test、package、install
- Site：生命周期：site命令，用于描述工程的静态文档

不同声明周期的命令可以一起执行，如

```
mvn clean compile
```

# Maven命令的执行顺序

相同生命周期的命令会有执行顺序的影响，比如某些命令执行的时候它只会执行自己的命令，但是另外一些命令执行的时候，其他命令也会被指定。以Default生命周期的命令为说明：

install > package > test > test-compile > compile

如上的顺序说明，如果执行install命令的时候，那么后面的命令都会被执行一次，但是如果执行compile的时候，影响效果只限于compile命令自己的效果。

# Maven添加tomcat插件

如果不使用本地的tomcat服务器，可以直接使用Maven提供的tomcat进行运行，只需在pom.xml中添加tomcat插件即可

```xml
<plugins>
    <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
            <port>8081</port>
            <path>/</path>
        </configuration>
    </plugin>
</plugins>
```

如上，即可添加tomcat插件，运行时直接在maven命令中执行tomcat:run即可

这里有一个值得注意的问题，当直接运行并访问的时候，可能会出现xxx is not a servlet的500错误，此时为jar包冲突所致，修改servlet依赖的使用范围为provide即可。

# Maven中scope标签的使用

scope表示的为jar包的依赖作用范围，也就是规定了引入了jar包在哪些范围中是有效的。

- compile(默认值)：在编译、运行、测试、打包下都有效
- provided：编译、测试时有效，运行、打包时无效
- test：仅在测试时有效
- runtime：测试、运行、打包时有效
- syatem：不推荐使用，使用此范围时，不会去本地仓库寻找依赖

| 作用域   | 编译  | 测试  | 运行  | 打包  | 示例        |
| -------- | :---: | :---: | :---: | :---: | ----------- |
| compile  | **√** |   √   | **√** | **√** | spring-core |
| provided | **√** |   √   | **×** | **×** | servlet-api |
| test     | **×** | **√** | **×** |   ×   | junit       |
| runtime  | **×** | **√** |   √   |   √   | JDBCDriver  |

# 开启tomcat热部署

热部署：在tomcat运行的时候，不用停止tomcat也可以发布web项目上去。

要开启tomcat热部署，需要修改tomcat包中conf/tomcat-users.xml文件，向其中添加

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="12345" roles="manager-gui,manager-script"/>
```

# Maven中jar包的冲突解决

### 方式一

第一声明优先原则：在pom.xml文件中，坐标位置靠上的jar包依赖是优先声明的，此时这个包里的依赖包会优先进入项目中，如果其他jar包也同样依赖了上面这个jar的依赖包，那么将会以上面优先引入的依赖包为准。

### 方式二

路径近者优先原则

直接依赖：项目中直接导入的jar包，就是该项目的直接依赖包，也就是在pom.xml中直接声明的依赖。

传递依赖：项目中并没有明确引入依赖的jar包，但是确实又存在于工程之中的jar包，此为传递依赖

直接依赖的优先级高于传递依赖（直接依赖路劲近于传递依赖）

### 方式三

直接排除法

若在某个jar包中，其中的依赖包不想使用默认版本，可以直接将其排除

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.3.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

# Maven版本锁定和统一版本管理

使用版本锁定，可以让子工程进行继承父工程时，核心版本不被改变（即使是直接依赖也不行）

版本统一管理可以让工程的版本升级更加高效

如下演示了统一版本管理和版本锁定

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ctbu</groupId>
    <artifactId>web-review</artifactId>
    <packaging>pom</packaging>
    <version>1.0</version>
    <modules>
        <module>servlet-01</module>
        <module>singleton01</module>
        <module>container</module>
        <module>spring-web</module>
    </modules>


	<!--统一版本管理-->
    <properties>
        <spring.version>5.2.3.RELEASE</spring.version>
        <mysql.version>5.1.47</mysql.version>
        <mybatis.version>3.5.3</mybatis.version>
    </properties>

	<!--版本锁定-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

	<!--工程依赖-->
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
    </dependencies>
</project>
```

# 使用Maven进行模块化开发

maven支持将项目进行拆分，也就是将一个项目分成多个模块，模块可以统一继承自某一个父项目，父项目的jar包可以被子模块继承。如果各个模块之间需要相互引用，可以将模块的坐标写入pom.xml文件，供其他模块引用即可。

### 父工程

使用maven创建一个父工程，只需保留父工程的pom.xml即可，父工程的作用主要就是管理模块的公共依赖。

### 子模块

那经典的Java三层架构来说，dao层、service层、controller层每一个都能独立成为一个模块。他们之间通过maven坐标的方式进行通信，其中service、dao模块在建立的时候可以不用web的骨架了，因为出去了controller之后，他们就是一个纯的Java项目。

当一个A模块引入了B模块之后，A中就相当于拥有了B中的完整代码和配置文件。

# Maven父子工程的 3 种启动方式

- 直接启动root父工程（用tomcat插件）
- 启动核心模块（使用tomcat插件，此时需要将root打包到本地仓库中）
- 使用本地tomcat启动

# 

