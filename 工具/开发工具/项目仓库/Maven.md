# Maven

## Maven基本使用

### 下载及安装

[下载地址](https://link.zhihu.com/?target=http%3A//maven.apache.org/download.cgi%23)

[配置环境变量](https://app.gitbook.com/@1184884206/s/java/gong-ju/mac/mac/mac-pei-zhi-huan-jing-bian-liang)

### 修改配置文件

#### 指定本地仓库

- 指定本地仓库位置的配置信息文件：

  ```sh
  $ vim apache-maven-3.2.2\conf\settings.xml
  ```

- 添加如下内容：[本地仓库路径，也就是RepMaven.zip的解压目录]

  ```sh
  <localRepository>maven解压目录</localRepository>
  ```

#### 指定远程仓库

159行添加阿里云仓库

```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

或者其他的源

```xml
<mirror>  
	<id>nexus</id>  
	<mirrorOf>*</mirrorOf>  
	<name>Nexus Mirror</name>  
	<url>http://nkss-ict.ichangtou.com.cn/content/groups/public/</url>  
</mirror>
```



### 在IDE中的设置

`file`-->`other settings`-->`perfermence for new project`

![image-20210919125630736](https://gitee.com/huawesome/my-picture/raw/master/img/202109191256807.png)

### 第一个Maven工程

- 目录结构

  ![image-20210918195359373](https://gitee.com/huawesome/my-picture/raw/master/img/202109181953424.png)

- POM文件内容

  ```xml
  <?xml version="1.0" ?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  	<modelVersion>4.0.0</modelVersion>
  
  	<artifactId>Hello</artifactId>
    
      <dependencies>
      <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.0</version>
          <scope>test</scope>
      </dependency>
  		</dependencies>
    </project>
  ```

- 编写主程序代码

在src/main/java/com/atguigu/maven目录下新建文件Hello.java，内容如下

```java
package com.atguigu.maven;
public class Hello {
    public String sayHello(String name){
        return "Hello "+name+"!";
    }
}
```

- 编写测试代码

在/src/test/java/com/atguigu/maven目录下新建测试文件HelloTest.java

```java
package com.atguigu.maven;    
import org.junit.Test;
import static junit.framework.Assert.*;
public class HelloTest {
    @Test
    public void testHello(){
        Hello hello = new Hello();
        String results = hello.sayHello("litingwei");
        assertEquals("Hello litingwei!",results);    
    }
}
```

- 运行几个基本的Maven命令

  ```sh
  $ mvn compile  # 编译
  $ mvn clean  # 清理
  $ mvn test  # 测试
  $ mvn package  # 打包
  ```

  > 注意：运行Maven命令时一定要进入pom.xml文件所在的目录！

### pom.xml配置文件详解

#### 节点分布

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
            http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 基本配置 -->
    <groupId>...</groupId>  # 项目组编号 例如，一家银行集团com.company.bank拥有所有银行相关项目。
    <artifactId>...</artifactId>  # 这是项目的ID。这通常是项目的名称。 例如，consumer-banking。
    <version>...</version>  # 这是项目的版本。与groupId一起使用，artifact在存储库中用于将版本彼此分 																离。 例如：com.company.bank:consumer-banking:1.0，					    															com.company.bank:consumer-banking:1.1
    <packaging>...</packaging>  # 项目打包方式，有以下值：pom, jar, maven-plugin, ejb, war, ear, 																		rar, par


    <!-- 依赖配置 -->
    <dependencies>...</dependencies>  # 项目相关依赖配置，如果在父项目写的依赖，会被子项目引用。一般会																					在父项目中定义子项目中所有共用的依赖。
    <parent>...</parent>  # 用于确定父项目的坐标位置。
    <dependencyManagement>...</dependencyManagement>
    <modules>...</modules>
    <properties>...</properties>  # 用于定义pom常量 如：<java.version>1.7</java.version>
  																	上面这个常量可以在pom文件的任意地方通过${Java.version}来引用

    <!-- 构建配置 -->
    <build>...</build>
    <reporting>...</reporting>

    <!-- 项目信息 -->
    <name>...</name>
    <description>...</description>
    <url>...</url>
    <inceptionYear>...</inceptionYear>
    <licenses>...</licenses>
    <organization>...</organization>
    <developers>...</developers>
    <contributors>...</contributors>

    <!-- 环境设置 -->
    <issueManagement>...</issueManagement>
    <ciManagement>...</ciManagement>
    <mailingLists>...</mailingLists>
    <scm>...</scm>
    <prerequisites>...</prerequisites>
    <repositories>...</repositories>
    <pluginRepositories>...</pluginRepositories>
    <distributionManagement>...</distributionManagement>
    <profiles>...</profiles>
</project>
```



## Maven项目构建

### 目录结构

> well,每个项目工程，都有非常繁琐的目录结构，每个目录都有不同的作用。请记住这一点，目录的划分是根据需要来的，每个目录有其特定的功能。目录本质上就是一个文件或文件夹路径而已。那么，我们换一个思路考虑: 一个项目的文件结构需要组织什么信息呢? 

让我们看一下功能的划分：

![目录结构](https://gitee.com/huawesome/my-picture/raw/master/img/202109181950845.png)

### 如何修改默认的目录配置

### 依赖原则

- 依赖路径最短优先言责
- 声明顺序优先原则
- 覆写优先原则

### 解决依赖冲突

## Maven项目生命周期与构建原理

### Maven对项目生命周期的抽象-三大项目生命周期

> Maven从项目的三个不同的角度，定义了单套生命周期，三套生命周期是相互独立的，它们之间不会相互影响。

- 默认构建生命周期(Default Lifeclyle): 该生命周期表示这项目的构建过程，定义了一个项目的构建要经过的不同的阶段。 
- 清理生命周期(Clean Lifecycle): 该生命周期负责清理项目中的多余信息，保持项目资源和代码的整洁性。一般拿来清空directory(即一般的     target)目录下的文件。 
- 站点管理生命周期(Site Lifecycle) :向我们创建一个项目时，我们有时候需要提供一个站点，来介绍这个项目的信息，如项目介绍，项目进度状态、项目组成成员，版本控制信息，项目javadoc索引信息等等。站点管理生命周期定义了站点管理过程的各个阶段。

![生命周期](https://gitee.com/huawesome/my-picture/raw/master/img/202109181953431.png)

### Maven对项目默认生命周期的抽象

### Maven各生命阶段行为绑定

### 如何查看Maven各个生命周期阶段和插件的绑定情况

### 项目中Run Package命令

## 参考

*XMind: ZEN - Trial Version*

```

```