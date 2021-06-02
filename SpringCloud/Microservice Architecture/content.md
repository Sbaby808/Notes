[TOC]

# 二、微服务架构编码

## 1.文档地址

+ Spring Cloud：
  + https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/
  + https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md
+ Spring Boot：
  + https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/

## 2.工程搭建

> 约定 > 配置 > 编码

### 2.1 聚合父工程

> Maven中**dependencyManagement**和**dependencies**：
>
> &emsp;Maven 使用 dependencyManagement 元素来提供了一种管理依赖版本号的方式。
>
> 通常会在一个组织或者项目的最顶层的父 POM 中看到 dependencyManagement 元素。
>
> &emsp;使用 pom.xml 中的 dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式地列出版本号。Maven 会沿着父子层次往上走，直到找到一个拥有 dependencyManagement 元素的项目，然后它就会使用这个 dependencyManagement 元素中指定的版本号。
>
> &emsp;这样做的好处就是：如果有多个子项目都引用同一依赖，则可以避免在每个使用的子项目里都声明一个版本号，这样当想升级或切换到另一个时，只需要在顶层父容器里更新，而不需要一个一个子项目的修改；另外如果某个子项目需要另外的一个版本，只需要声明version就可。
>
> &emsp;dependencyManagement 这里只是声明依赖，**并不实现引入**，因此子项目需要显式地声明需要用的依赖。
>
> &emsp;如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有子项目中写了该依赖，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom。
>
> &emsp;如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。
>
> **父工程POM参考：**
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
> 
>   <modelVersion>4.0.0</modelVersion>
> 
>   <groupId>com.atguigu.springcloud</groupId>
>   <artifactId>cloud2020</artifactId>
>   <version>1.0-SNAPSHOT</version>
>   <packaging>pom</packaging>
> 
> 
>   <!-- 统一管理jar包版本 -->
>   <properties>
>     <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
>     <maven.compiler.source>1.8</maven.compiler.source>
>     <maven.compiler.target>1.8</maven.compiler.target>
>     <junit.version>4.12</junit.version>
>     <log4j.version>1.2.17</log4j.version>
>     <lombok.version>1.16.18</lombok.version>
>     <mysql.version>5.1.47</mysql.version>
>     <druid.version>1.1.16</druid.version>
>     <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
>   </properties>
> 
>   <!-- 子模块继承之后，提供作用：锁定版本+子modlue不用写groupId和version  -->
>   <dependencyManagement>
>     <dependencies>
>       <!--spring boot 2.2.2-->
>       <dependency>
>         <groupId>org.springframework.boot</groupId>
>         <artifactId>spring-boot-dependencies</artifactId>
>         <version>2.2.2.RELEASE</version>
>         <type>pom</type>
>         <scope>import</scope>
>       </dependency>
>       <!--spring cloud Hoxton.SR1-->
>       <dependency>
>         <groupId>org.springframework.cloud</groupId>
>         <artifactId>spring-cloud-dependencies</artifactId>
>         <version>Hoxton.SR1</version>
>         <type>pom</type>
>         <scope>import</scope>
>       </dependency>
>       <!--spring cloud alibaba 2.1.0.RELEASE-->
>       <dependency>
>         <groupId>com.alibaba.cloud</groupId>
>         <artifactId>spring-cloud-alibaba-dependencies</artifactId>
>         <version>2.1.0.RELEASE</version>
>         <type>pom</type>
>         <scope>import</scope>
>       </dependency>
> 
>       <dependency>
>         <groupId>mysql</groupId>
>         <artifactId>mysql-connector-java</artifactId>
>         <version>${mysql.version}</version>
>       </dependency>
>       <dependency>
>         <groupId>com.alibaba</groupId>
>         <artifactId>druid</artifactId>
>         <version>${druid.version}</version>
>       </dependency>
>       <dependency>
>         <groupId>org.mybatis.spring.boot</groupId>
>         <artifactId>mybatis-spring-boot-starter</artifactId>
>         <version>${mybatis.spring.boot.version}</version>
>       </dependency>
>       <dependency>
>         <groupId>junit</groupId>
>         <artifactId>junit</artifactId>
>         <version>${junit.version}</version>
>       </dependency>
>       <dependency>
>         <groupId>log4j</groupId>
>         <artifactId>log4j</artifactId>
>         <version>${log4j.version}</version>
>       </dependency>
>       <dependency>
>         <groupId>org.projectlombok</groupId>
>         <artifactId>lombok</artifactId>
>         <version>${lombok.version}</version>
>         <optional>true</optional>
>       </dependency>
>     </dependencies>
>   </dependencyManagement>
> 
>   <build>
>     <plugins>
>       <plugin>
>         <groupId>org.springframework.boot</groupId>
>         <artifactId>spring-boot-maven-plugin</artifactId>
>         <configuration>
>           <fork>true</fork>
>           <addResources>true</addResources>
>         </configuration>
>       </plugin>
>     </plugins>
>   </build>
> 
> </project>
> ```

### 2.2 DevTools

1. Adding devtools to your project

   ```xml
   <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   ```

   

2. Adding plugin to your pom.xml

   ```xml
   一段配置黏贴到父工程当中的pom里
    
   <build>
     <plugins>
       <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
         <configuration>
           <fork>true</fork>
           <addResources>true</addResources>
         </configuration>
       </plugin>
     </plugins>
   </build>
    
   ```

   

3. Enabling automatic build

   > settings->Build,Exception,Deployment->Compiler->Automatic……/Display……/Build……/Compile……

4. Update the value of

5. restart idea
