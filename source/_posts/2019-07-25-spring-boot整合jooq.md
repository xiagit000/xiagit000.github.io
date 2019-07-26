---
title: Spring boot整合jooq
date: 2019-07-25 15:59:11
tags:
- java
- spring
- jooq
categories:
- java
- spring
---

## 前言

最近使用spring cloud做分布式应用的开发，底层准备采用jooq做持久层，因为都是单应用，相互之间没有代码上的耦合，所以没有严格按照三层的设计模式来开发，这个时候jooq的代码生成工具就能极大地提升应用的开发效率，整合的过程中发现网上的帖子要么太过简单，要么言语不详，并没有讲清楚具体的开发细节，所以自己完整的记录一下使用过程。
<!-- more -->

>本教程依赖于spring boot框架，spring boot的配置与搭建在这里不做赘述


## 依赖&配置
### pom.xml
引入jooq的依赖
``` bash
...

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jooq</artifactId>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-meta</artifactId>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>

<build>
    <plugins>
        /*用于读取yml配置属性的插件*/
        <plugin>
            <groupId>it.ozimov</groupId>
            <artifactId>yaml-properties-maven-plugin</artifactId>
            <version>1.1.3</version>
            <executions>
                <execution>
                    <phase>initialize</phase>
                    <goals>
                        <goal>read-project-properties</goal>
                    </goals>
                    <configuration>
                        <files>
                            <file>src/main/resources/application.yml</file>
                        </files>
                    </configuration>
                </execution>
            </executions>
        </plugin>
	
        /*jooq的配置插件*/
        <plugin>
            <groupId>org.jooq</groupId>
            <artifactId>jooq-codegen-maven</artifactId>
            <version>${jooq.version}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <jdbc>
                    <driver>${spring.datasource.driver-class-name}</driver>
                    <url>${spring.datasource.url}</url>
                    <user>${spring.datasource.username}</user>
                    <password>${spring.datasource.password}</password>
                </jdbc>
                <generator>
                    <name>org.jooq.util.DefaultGenerator</name>
                    <database>
                        <name>org.jooq.util.mysql.MySQLDatabase</name>
                        <includes />
                        <inputSchema>${my.datasource.schema}</inputSchema>
                    </database>
                    <generate>
                        <daos>true</daos>
                        <pojos>true</pojos>
                        <javaTimeTypes>true</javaTimeTypes>
                    </generate>
                    <target>
                        <packageName>your package name</packageName>
                        <directory>target dir for your code generated</directory>
                    </target>
                </generator>
            </configuration>
        </plugin>
    </plugins>
</build>

...
```

### application.yml
相关的配置如下：
``` bash
spring:
  datasource:
    url: {{your database url}}
    username: {{your username}}
    password: {{your password}}
    driver-class-name: com.mysql.jdbc.Driver
  jooq:
    sql-dialect: mysql

my:
  datasource:
    schema: {{your database schema}}
```
至此jooq的基本配置已经完成。

## 生成代码
这个时候就需要我们在数据库里面创建业务表，完成以后执行mvn compile命令jooq自动进行逆向操作，根据刚创建的数据表生成java代码。
生成完成的代码结构如图：
{% asset_image 1.png %}
生成的代码包含数据库对应的实体，查询映射，还有dao层工具。

## 使用方法
### 实体
按照jooq的基本用法即可，省去了自己手动创建标准对象类的功夫，但是有个弊端就是jooq生成的代码，你是无法改动的，因为下一次使用maven编译的时候会将所有生成的代码进行覆盖，因此如果需要对jooq生成的对象做改动，尽量灵活使用继承，组合的方式进行增强。
### dao层的用法
jooq生成的dao层提供了很多查询方法，但这些方法对于我们某些业务来说肯定是远远不够的，这就要求我们必须在此基础上添加自定的dao方法，既可以享受jooq已经生成的方法的便利，又可以自定义特定的数据库操作，一举多得。
首先我们要自定义一个自己的dao类继承jooq生成的dao类，作为增强类，同时还要把spring已经实例化的jooq configuration注入到父类里面，这样通过增强类去调用继承的方法时，才能正常查询，否则jooq生成的dao类会因为无法获取正确的配置实例而报空指针错误，示例：
``` bash
@Repository
public class UserDaoExt extends UsersDao implements BaseDao {

    @Autowired
    private DSLContext create;

    /*讲spring托管的jooq配置注入到继承的父类当中*/
    @Autowired
    public UserDaoExt(Configuration configuration) {
        super(configuration);
    }

    ...
    根据业务需要添加自定义的数据库操作函数
    ...
}
```
至此jooq的整合就已经基本完成，再往上层调用就跟传统的使用方式一样，不需要再关注持久层的实现。