---
layout:     post
title:      "SpringBoot 数据库文档"
subtitle:   " \"Hello SpringBoot, Hello screw\""
date:       2020-07-30 10:00:00
author:     "墨城烟柳（Mcyl）"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - SpringBoot
---

> “Hello everyone! ”


## 简介

在企业级开发中、我们经常会有编写数据库表结构文档的时间付出，从业以来，待过几家企业，关于数据库表结构文档状态：
要么没有、要么有、但都是手写、后期运维开发，需要手动进行维护到文档中，很是繁琐、如果忘记一次维护、就会给以后
工作造成很多困扰、无形中制造了很多坑留给自己和后人，于是需要一个插件工具 screw 来维护。

## 特点

1、简洁、轻量、设计良好

2、多数据库支持

3、多种格式文档

4、灵活扩展

5、支持自定义模板

## 数据库支持

 1、MySQL

 2、MariaDB

 3、TIDB

 4、Oracle

 5、SqlServer

 6、PostgreSQL

 7、Cache DB

## 文档生成支持

 1、html

 2、word

 3、markdwon

## 文档截图

![OTP](/img/screw/1.png)


## 文档截图

#### 1、添加jar依赖

	 <!--数据库文档核心依赖-->
	  <dependency>
	      <groupId>cn.smallbun.screw</groupId>
	      <artifactId>screw-core</artifactId>
	      <version>1.0.3</version>
	  </dependency>
	  <!-- HikariCP -->
	  <dependency>
	      <groupId>com.zaxxer</groupId>
	      <artifactId>HikariCP</artifactId>
	      <version>3.4.5</version>
	  </dependency>
	  <!--mysql driver-->
	  <dependency>
	      <groupId>mysql</groupId>
	      <artifactId>mysql-connector-java</artifactId>
	      <version>8.0.20</version>
	  </dependency>


#### 2、编写代码方式生成

	package com.example.screw.word;

	import cn.smallbun.screw.core.Configuration;
	import cn.smallbun.screw.core.engine.EngineConfig;
	import cn.smallbun.screw.core.engine.EngineFileType;
	import cn.smallbun.screw.core.engine.EngineTemplateType;
	import cn.smallbun.screw.core.execute.DocumentationExecute;
	import cn.smallbun.screw.core.process.ProcessConfig;
	import com.zaxxer.hikari.HikariConfig;
	import com.zaxxer.hikari.HikariDataSource;

	import javax.sql.DataSource;
	import java.util.ArrayList;

	/**
	 * @Author lijun
	 * @Description
	 * @Date 2020-07-29 4:26 下午
	 **/

	public class Scrrew {
	    public static void main(String[] args) {
	        //数据源
	        HikariConfig hikariConfig = new HikariConfig();
	        hikariConfig.setDriverClassName("com.mysql.cj.jdbc.Driver");
	        hikariConfig.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/db_auth");
	        hikariConfig.setUsername("root");
	        hikariConfig.setPassword("root");
	        //设置可以获取tables remarks信息
	        hikariConfig.addDataSourceProperty("useInformationSchema", "true");
	        hikariConfig.setMinimumIdle(2);
	        hikariConfig.setMaximumPoolSize(5);
	        DataSource dataSource = new HikariDataSource(hikariConfig);
	        //生成配置
	        EngineConfig engineConfig = EngineConfig.builder()
	                //生成文件路径
	                .fileOutputDir("/Users/lijun/doce")
	                //打开目录
	                .openOutputDir(true)
	                //文件类型
	                .fileType(EngineFileType.HTML)
	                //生成模板实现
	                .produceType(EngineTemplateType.freemarker)
	                .build();

	        //忽略表
	        ArrayList<String> ignoreTableName = new ArrayList<>();
	        ignoreTableName.add("test_user");
	        ignoreTableName.add("test_group");
	        //忽略表前缀
	        ArrayList<String> ignorePrefix = new ArrayList<>();
	        ignorePrefix.add("test_");
	        //忽略表后缀
	        ArrayList<String> ignoreSuffix = new ArrayList<>();
	        ignoreSuffix.add("_test");
	        ProcessConfig processConfig = ProcessConfig.builder()
	                //指定生成逻辑、当存在指定表、指定表前缀、指定表后缀时，将生成指定表，其余表不生成、并跳过忽略表配置
	                //根据名称指定表生成
	                .designatedTableName(new ArrayList<>())
	                //根据表前缀生成
	                .designatedTablePrefix(new ArrayList<>())
	                //根据表后缀生成
	                .designatedTableSuffix(new ArrayList<>())
	                //忽略表名
	                .ignoreTableName(ignoreTableName)
	                //忽略表前缀
	                .ignoreTablePrefix(ignorePrefix)
	                //忽略表后缀
	                .ignoreTableSuffix(ignoreSuffix).build();
	        //配置
	        Configuration config = Configuration.builder()
	                //版本
	                .version("1.0.0")
	                //描述
	                .description("数据库设计文档生成")
	                //数据源
	                .dataSource(dataSource)
	                //生成配置
	                .engineConfig(engineConfig)
	                //生成配置
	                .produceConfig(processConfig)
	                .build();
	        //执行生成
	        new DocumentationExecute(config).execute();
	    }
	}


直接run这个main方法就可以直接生成了。

#### 3、Maven 插件方式生成

	<build>
        <plugins>
            <plugin>
                <groupId>cn.smallbun.screw</groupId>
                <artifactId>screw-maven-plugin</artifactId>
                <version>1.0.3</version>
                <dependencies>
                    <!-- HikariCP -->
                    <dependency>
                        <groupId>com.zaxxer</groupId>
                        <artifactId>HikariCP</artifactId>
                        <version>3.4.5</version>
                    </dependency>
                    <!--mysql driver-->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.20</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <!--username-->
                    <username>root</username>
                    <!--password-->
                    <password>root</password>
                    <!--driver-->
                    <driverClassName>com.mysql.cj.jdbc.Driver</driverClassName>
                    <!--jdbc url-->
                    <jdbcUrl>jdbc:mysql://127.0.0.1:3306/db_auth</jdbcUrl>
                    <!--生成文件类型-->
                    <fileType>HTML</fileType>
                    <!--文件输出目录-->
                    <fileOutputDir>/Users/lijun/doce</fileOutputDir>
                    <!--打开文件输出目录-->
                    <openOutputDir>true</openOutputDir>
                    <!--生成模板-->
                    <produceType>freemarker</produceType>
                    <!--描述-->
                    <description>数据库文档生成</description>
                    <!--版本-->
                    <version>${project.version}</version>
                    <!--标题-->
                    <title>数据库文档</title>
                </configuration>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

直接使用maven的install方法一下就可以生成了，是不是很简单呢?

## 总结

这么好用的工具，我在这里贴上screw的[源码](https://gitee.com/leshalv/screw)，后续作者应该会持续完善，我们一起期待变得更好吧！

 
####  源码下载：

[https://github.com/mochengyanliu/Study/tree/master/screw](https://github.com/mochengyanliu/Study/tree/master/screw)


