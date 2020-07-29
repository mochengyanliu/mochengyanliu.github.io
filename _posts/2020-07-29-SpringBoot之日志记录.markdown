---
layout:     post
title:      "SpringBoot 日志记录"
subtitle:   " \"Hello SpringBoot, Hello Log\""
date:       2020-07-29 10:00:00
author:     "墨城烟柳（Mcyl）"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - SpringBoot
---

> “Hello everyone! ”


## 需求介绍

以前老项目接口请求越来越慢，维护越来越困难，要在所有接口中记录请求日志监控，因为项目比较老了，所以接口还是挺多的，
公司新招的毕业生预估需要三天才能完成，我说还是让我来吧！

AOP使用场景：

今天我们就来讲讲切点的一种配置方式：@annotation，通过@annotation配置切点，我们可以灵活的控制切到哪个方法，
同时可以进行一些个性化的设置，今天我们就用它来实现一个记录所有接口请求功能吧。

## AOP实现自定义注解

#### 1、不管三七二十一，先看效果：
	
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : ======================请求开始========================
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求连接：http://localhost:8080/hello
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 接口描述：hello接口
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求类型：GET
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求方法：com.example.aoplog.controller.HelloControllerhello
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求IP：0:0:0:0:0:0:0:1
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求参数：[{"name":"Hello World"}]
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求耗时：77ms
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : 请求返回：hello,Hello World
	  INFO 1110 --- [nio-8080-exec-2] com.example.aoplog.aop.AopLog            : ======================请求结束========================


通过结果我们可以看到，我们的自定义注解EagleEye做到了统一的记录下了请求链接、接口描述、请求类型、请求方法、请求IP、请求参数、请求耗时、请求返回等信息。

是不是感觉还不错呢？下面我们就来一起动手实现它吧！

#### 2、添加jar依赖

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<parent>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-parent</artifactId>
			<version>2.3.0.RELEASE</version>
			<relativePath/> <!-- lookup parent from repository -->
		</parent>
		<groupId>com.example</groupId>
		<artifactId>aoplog</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<name>aoplog</name>
		<description>Demo project for Spring Boot</description>

		<properties>
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
				<exclusions>
					<exclusion>
						<groupId>org.junit.vintage</groupId>
						<artifactId>junit-vintage-engine</artifactId>
					</exclusion>
				</exclusions>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-aop</artifactId>
			</dependency>

			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>fastjson</artifactId>
				<version>1.2.46</version>
			</dependency>
			<dependency>
				<groupId>org.projectlombok</groupId>
				<artifactId>lombok</artifactId>
				<version>1.16.10</version>
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


#### 3、自定义注解

	@Retention(RetentionPolicy.RUNTIME)    // 注解的生命周期
	@Target(ElementType.METHOD)			   // 注解修饰的方法
	@Documented							   // 注解被javadoc记录
	public @interface EagleEye {

	    /**
	     * 接口描述
	     * @return
	     */
	    String desc() default "";


	}

1、定义了注解的生命周期为运行时

2、定义了注解的作用域为方法

3、标识该注解可以被JavaDoc记录

4、定义注解名称为EagleEye（自称为鹰眼）

5、定义一个元素desc，用来描述被修饰的方法


注解虽然定义好了，但是还用不了，因为没有具体的实现逻辑，接下来我们用AOP实现它。

#### 4、配置AOP切面

	@Slf4j
	@Aspect
	@Component
	public class AopLog {

	    @Pointcut("@annotation(com.example.aoplog.aop.EagleEye)")
	    public void eagleEye() {

	    }

	    @Around("eagleEye()")
	    public Object around(ProceedingJoinPoint pjp) throws Throwable {
	        // 系统时间
	        long begin = System.currentTimeMillis();
	        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
	        HttpServletRequest request = attributes.getRequest();

	        Signature signature = pjp.getSignature();
	        MethodSignature methodSignature = (MethodSignature) signature;
	        Method method = methodSignature.getMethod();
	        EagleEye eagleEye = method.getAnnotation(EagleEye.class);

	        //接口信息
	        String desc = eagleEye.desc();

	        log.info("======================请求开始========================");

	        log.info("请求连接：{}", request.getRequestURL().toString());

	        log.info("接口描述：{}", desc);

	        log.info("请求类型：{}", request.getMethod());

	        log.info("请求方法：{}{}", signature.getDeclaringTypeName(), signature.getName());

	        log.info("请求IP：{}", request.getRemoteAddr());

	        log.info("请求参数：{}", JSON.toJSONString(pjp.getArgs()));

	        Object result = pjp.proceed();
	        long end = System.currentTimeMillis();
	        log.info("请求耗时：{}ms", end - begin);

	        log.info("请求返回：{}", result);

	        log.info("======================请求结束========================");
	        return result;
	    }
	}

好了，到大功告成，我们就完成了利用AOP自定义注解的所有步骤。

#### 5、使用配置

	@RestController
	public class HelloController {

	    @RequestMapping("/hello")
	    @EagleEye(desc = "hello接口")
	    public String hello(@RequestBody HelloVO helloVO){
	        return "hello," + helloVO.getName();
	    }
	}

对于需要AOP增强的方法，我们只需要：

1、在方法上加上@EagleEye注解

2、通过desc元素设置方法的描述

接下来启动应用，请求接口看一下控制台输出是不是像我们开头贴出的那样吧。

## 总结

当然，这仅仅是自定义注解的一种小用法而已，用来实现这个日志记录的小功能显得有些大材小用，但是确实挺方便的。

 
####  源码下载：

[https://github.com/mochengyanliu/Study/tree/master/aoplog](https://github.com/mochengyanliu/Study/tree/master/aoplog)


