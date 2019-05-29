---
title: IOT实验室Web项目开发手册
date: 2019-1-7 15:25:42
tags: Coding
---

本文可作为实验室Web项目通用的开发指南。其内容涵盖一个基本Web项目的搭建方法、开发规范、插件说明以及常见问题。
<!-- more -->

本文读者应该具备的技能：Java语言基础、数据库基础（SQL语句）、基本的前端开发能力（HTML+CSS+JS）、Java EE基础（了解Servlet、JSP等入门级Web开发技术或者有SSM、SSH等高级Web框架使用经验）。



## 搭建项目

**操作系统**：Wndows10

**IDE**：使用IntelliJ IDEA（ https://www.jetbrains.com/idea ），可在官网进行学生认证免费使用，或者使用license server激活（启动时需要联网）。

**项目构建**：Maven

**版本控制**：Git

**后台**：Spring Boot+Mybatis

**数据库**：MySQL



### Maven

Maven在项目中最核心的作用就是管理jar包。项目中需要引入大量第三方包和插件，用传统的下载jar包再手动引入的方式会十分繁琐，并且项目多人共同开发时可能引发jar包冲突。使用Maven的话引入jar包只需统一在pom.xml文件里配置即可，Maven会自动在中央仓库下载并引入。

##### 安装:

1.官网下载 <http://maven.apache.org/download.html> zip包，解压到相应路径，例如 D:\software\apache-maven-3.5.0

2.配置环境变量，右键“计算机”，选择“属性”，之后点击“高级系统设置”，点击“环境变量”：

新建系统变量   MAVEN_HOME  变量值：D:\software\apache-maven-3.5.0

编辑系统变量  Path         添加变量值： %MAVEN_HOME%\bin

3.终端输入 mvn -v 即可检查是否安装成功

![1559099761353](/home/qiuhang/.config/Typora/typora-user-images/1559099761353.png)

##### 配置文件：

编辑 {MAVEN_HOME }/conf 下的 settings.xml 可进行各种设置。

- 设置本地仓库位置，默认是在 user.home 下面，仓库储存的是从中央仓库下载的包

```xml
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
```

- 将中央仓库设置为阿里云的，国外的库可能会很慢

```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror> 
 </mirrors>
```



### Git

用于对程序进行版本控制、多人协作开发

##### 安装：

直接下载安装程序 https://git-scm.com/downloads  按默认选项安装

安装完毕在开始菜单打开 Git->Git Bash ，设置完用户信息即可使用：

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

##### 工作原理：

- 本地仓库分成工作目录、缓存区（stage）、HEAD，工作目录就是当前编辑的文件，stage临时保存你的改动，HEAD是个指针，指向你最近一次提交的结果。
- 远程仓库可以备份你的代码，并与他人共享，常用的是GitHub
- 初始化仓库只有一个master分支，有新的工作任务可以新建分支，在分支上开发，完成后再合并到主分支,分支上的改动未merge时对master不可见。
- 可参考 http://www.bootcss.com/p/git-guide/

##### 常用命令：

git init 在当前目录初始化一个git仓库，也就是可以对这个目录进行版本控制了

git add . 把当前所有修改更新到stage里

git commit -m ’代码提交信息‘  提交stage里的修改到本地仓库

git push orgin master  把本地库的修改提交到远程库的maser分支中（也可换成别的分支）

git pull  把远程库的代码更新到本地

git clone xxxxx 克隆一份代码到本地仓库

git checkout master/branch  切换到某个分支

git log 查看当前分支上面的日志信息

git reset --hard HEAD^ 回退到上个版本



### MySQL

常用的数据库还有Oracle、SQLServer等等，前者太庞大，后者太小，一般情况使用中型数据库MySQL即可。

##### 安装：

1.下载zip包  https://dev.mysql.com/downloads/mysql/ ，解压，例如 C:\Program Files\MySQL\mysql-5.7.23-winx64

2.配置环境变量，右键“计算机”，选择“属性”，之后点击“高级系统设置”，点击“环境变量”：

新建系统变量   MYSQL_HOME  变量值：C:\Program Files\MySQL\mysql-5.7.23-winx64

编辑系统变量  Path         添加变量值： %MYSQL_HOME%\bin;

3.生成data文件：以管理员身份运行cmd，进入 C:\Program Files\MySQL\mysql-5.7.23-winx64\bin 下

执行命令：mysqld --initialize-insecure --user=mysql  在C:\Program Files\MySQL\mysql-5.7.23-winx64目录下生成data目录

##### 使用：

1.启动服务：执行命令：net start mysql 

2.登录MySQL：执行命令mysql -u root -p   默认没有密码，回车即可

3.修改root用户密码：update mysql.user set authentication_string=password("123456") where user

="root";     此处引号中的内容是密码

##### 常见Q&A：

Q: 启动服务失败

A：执行命令 mysqld -install  即可（不需要my.ini配置文件也可以）然后去资源管理器中把mysql进程全结束了，重新启动即可。



Q: mysql插入字段有中文就会报错

A: 执行命令  show variables like 'character_set_%' ;  检查得知是因为数据库编码为 latin

ALTER database xxx character set utf8;  这样修改编码即可



Q: mysql的服务已经停止，无法启动

A：（1）如果之前装过mysql的，把mysql的server卸载掉，连带MySQL Server 5.7\data文件一起清干净

​      （2）重新装好后，输入 net start mysql 还是无法启动服务，这个时候，输入以下指令：

​          mysqld --remove 删除mysql服务

​          mysqld --install 安装服务

​          mysqld --initialize  初始化

​           net  start mysql 

​           然后发现服务已经启动

​        （3）输入mysql -u root -p 启动mysql ，然后会要求你输入密码，注意由于是初始化的，所以系统自动配置一个初始化密码，这个密码在哪里找到呢？在 MySQL Server 5.7.2\data 这个路径下有一个计算机名字加err的文件，这个文件是错误日志，打开它，找到一个temporary password的记录条，冒号后面的就是初始化密码了。

 	（4）输入初始化密码以后，修改密码，输入指令  alter user  'root'@'localhost' identified by '密码'； 就能修改成功了。



Q：使用纯数字表名

A：表名纯数字的话要用``包起来(esc下面那个键)



### Spring Boot+Mybatis

Spring Boot特点：只使用一个核心配置文件，取消了一系列xml配置，甚至连web.xml都没有, 全部使用注解的方式完成MVC的功能；框架内置Tomcat服务器，运行启动类中的main函数即可启动，整体非常简洁。

Mybatis是java Web最常用的持久层框架（ORM），与Spring Boot配合可以使用注解轻松地编写SQL语句对数据库进行CRUD操作。

##### 使用IDEA搭建一个基本的后台

1.新建工程

![img](https://images2017.cnblogs.com/blog/1252282/201712/1252282-20171208162710921-1687430293.png)

![img](https://images2017.cnblogs.com/blog/1252282/201712/1252282-20171208162941718-320506362.png)

2.勾上web，其他的不用

![img](https://images2017.cnblogs.com/blog/1252282/201712/1252282-20171208163055015-1494041380.png)

3.Finish

![img](https://images2017.cnblogs.com/blog/1252282/201712/1252282-20171208163208171-1829970561.png)

4.完善一下目录结构

![img](https://images2017.cnblogs.com/blog/1252282/201712/1252282-20171208164809406-2123095565.png)

5.在maven的pom.xml添加相关的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tqh</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <mybatis-spring-boot.version>1.2.0</mybatis-spring-boot.version>
        <mysql-connector.version>5.1.39</mysql-connector.version>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-->使用thymeleaf模板<-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!-->引入Mybatis依赖<-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot.version}</version>
        </dependency>
        <!-->引入MySQL依赖<-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-connector.version}</version>
        </dependency>
        <!-->热部署工具<-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

6.配置核心文件application.properties

```properties
#服务使用的端口
server.port=9090
#mvc配置
spring.mvc.view.prefix=classpath:/templates/
spring.mvc.view.suffix=.html
spring.mvc.static-path-pattern=/static/**
#thymeleaf配置
#spring.thymeleaf.prefix = classpath:/static/
#spring.thymeleaf.suffix = .html
#数据库配置
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#Mybatis配置
mybatis.typeAliasesPackage=com.tqh.demo.model

```

##### 测试

1.在MySQL数据库新建一个schema，命名为test（如上方配置文件所示）；在test里新建一张person表，字段为id、name、age，插入一些数据，不赘述。

2.在model包新建一个Person类（也可以使用IDEA根据数据库表字段自动生成）

```java
package com.tqh.demo.model;

public class Person {
    private Integer id;
    private String name;
    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "id=" + id +", name='" + name + '\'' +", age=" + age;
    }
}
```

3.在Mapper包创建一个UserMapper

```java
package com.tqh.demo.mapper;

import com.tqh.demo.model.Person;
import org.apache.ibatis.annotations.Select;
@Repository
public interface UserMapper { 
　　@Select("SELECT * FROM person WHERE id = #{id}") 
　　Person selectUser(int id); 
}
```

4.在service包创建一个UserService

```java
package com.tqh.demo.service;
import com.tqh.demo.model.Person;

public interface UserService {
  public Person selectUser(int id);
}
```

然后在service.impl包里创建一个对应实现类

```java
package com.tqh.demo.service.impl;
import com.tqh.demo.model.Person;
import com.tqh.demo.service.UserService;

@Service
public class UserServiceImpl implements UserService {
  @Autowired
  UserMapper userMapper;

  @Override
  public Person selectUser(int id) {
      return userMapper.selectUser(id);
  }
}
```

5.在controller包新建一个UserController控制器

```java
package com.tqh.demo.controller;

import com.tqh.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
   @Autowired
   private UserService userService;

   @RequestMapping("/showUser/{id}")
   public String selectUser (@PathVariable int id){
   	 return userService.selectUser(id).toString();
   }
}
```

6.编写启动类DemoApplication

```java
package com.tqh.demo;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.tqh.demo.mapper")
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

7.运行DemoApplication，打开浏览器访问http://localhost:9090/showUser/1，成功查询到数据库的数据

![img](https://images2017.cnblogs.com/blog/1252282/201712/1252282-20171208173012202-2126121598.png)

##### 常见Q&A

Q: Mybatis报错Parameter not found

A: 对参数加标签@Param("xxx")



Q： Mybatis使用动态表名？

A： 重点是用$和@Param	

```java
@Update("CREATE TABLE ${para}（.....）")

Boolean createTable(@Param(value="para") String data);
```



Q:  如何用Mybatis完成SQL拼接来实现多条件查询？

A：1.建一个SqlProvider类来辅助完成拼接

```java
public class SqlProvider {
    public String selectPerson(@Param("id") String id, @Param("name")String name, @Param("age")String age){
        String sql="SELECT * FROM person ps WHERE 1=1";
        if(id!=null&&!id.trim().equals("")){
            sql+=" AND ps.id=#{id}";
        }
        if(name!=null&&!name.trim().equals("")){
            sql+=" AND ps.name=#{name}";
        }
        if(age!=null&&!age.trim().equals("")){
            sql+=" AND ps.age=#{age}";
        }
        return sql;
    }
```

2.修改mapper里的接口

```java
package com.tqh.demo.mapper;
import com.tqh.demo.model.Person;

@Repository
public interface UserMapper { 
　　@SelectProvider(type = SqlProvider.class, method = "selectPerson")
　　List<Person> selectPerson(@Param("id") String id, @Param("name")String name, @Param("age")String age); 
}
```

