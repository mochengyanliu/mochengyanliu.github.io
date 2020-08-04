---
layout:     post
title:      "SpringBoot Swagger-knife4j"
subtitle:   " \"Hello SpringBoot, Hello knife4j\""
date:       2020-08-04 17:30:00
author:     "墨城烟柳（Mcyl）"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - SpringBoot
---

> “Hello everyone! ”


## 简介

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案,前身是swagger-bootstrap-ui,取名kni4j是希望它能像一把匕首一样小巧,轻量,并且功能强悍!

## 主要模块

1、knife4j    为Java MVC框架集成Swagger的增强解决方案

2、knife4j-admin   云端Swagger接口文档注册管理中心，集成gateway网关对任意微服务文档进行组合集成

3、knife4j-extension  chrome浏览器的增强Swagger接口文档UI，快速渲染Swagger资源

4、knife4j-service 为Swagger服务的一系列接口服务程序

5、knife4j-front  knife4j-spring-ui的纯前端静态版本，用于集成非Java语言使用

6、swagger-bootstrao-ui  knife4j的前身，最后发布的版本是1.9.6

## 核心功能

该UI增强包主要包括两大核心功能：文档说明 和 在线调试

1、文档说明：根据Swagger的规范说明，详细列出接口文档的说明，包括接口地址、类型、请求示例、请求参数、响应示例、响应参数、响应码等信息，
使用swagger-bootstrap-ui能根据该文档说明，对该接口的使用情况一目了然。

2、在线调试：提供在线接口联调的强大功能，自动解析当前接口参数,同时包含表单验证，调用参数可返回接口响应内容、headers、Curl请求命令实例、
响应时间、响应状态码等信息，帮助开发者在线调试，而不必通过其他测试工具测试接口是否正确,简介、强大。


## 文档生成支持

 1、markdwon

 2、html

 3、word(开发中)

 4、pdf(开发中)


## 文档截图

![OTP](/img/kinfe4j/1.jpg)


## 文档截图

#### 1、添加jar依赖

	 	<!--整合Knife4j-->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>2.0.4</version>
        </dependency>

        <!--swagger-ui-->
         <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>


#### 2、配置类

相对于之前的Swagger增加@EnableKnife4j注解

	package com.example.swagger2;

	import com.github.xiaoymin.knife4j.spring.annotations.EnableKnife4j;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import springfox.documentation.builders.ApiInfoBuilder;
	import springfox.documentation.builders.PathSelectors;
	import springfox.documentation.builders.RequestHandlerSelectors;
	import springfox.documentation.service.ApiInfo;
	import springfox.documentation.service.Contact;
	import springfox.documentation.spi.DocumentationType;
	import springfox.documentation.spring.web.plugins.Docket;
	import springfox.documentation.swagger2.annotations.EnableSwagger2;

	/**
	 * @Author lijun
	 * @Description
	 * @Date 2020-07-31 2:39 下午
	 **/

	@Configuration
	@EnableSwagger2
	@EnableKnife4j
	public class Swagger2 {
	    /**
	     * 创建API应用
	     * apiInfo() 增加API相关信息
	     * 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现，
	     * 本例采用指定扫描的包路径来定义指定要建立API的目录。
	     *
	     * @return
	     */
	    @Bean
	    public Docket createRestApi() {
	        return new Docket(DocumentationType.SWAGGER_2)
	                .apiInfo(apiInfo())
	                .select()
	                //为当前包路径
	                .apis(RequestHandlerSelectors.basePackage("com.example.swagger2.controller"))
	                .paths(PathSelectors.any())
	                .build();
	    }

	    /**
	     * 创建该API的基本信息（这些基本信息会展现在文档页面中）
	     * 访问地址：http://项目实际地址/swagger-ui.html
	     * @return
	     */
	    private ApiInfo apiInfo() {
	        return new ApiInfoBuilder()
	                //页面标题
	                .title("XX权限系统")
	                //描述
	                .description("XX权限系统中使用Swagger2构建RESTful APIs")
	                //服务地址
	                .termsOfServiceUrl("http://localhost:8080")
	                //创建人
	                .contact(new Contact("lijun", "http://localhost:8080/doc.html", "1256615689@qq.com"))
	                //版本号
	                .version("1.0")
	                .build();
	    }
	}

这样knife4j的所有配置都完成了，启动项目可以访问地址：
[http://localhost:8080/doc.html](http://localhost:8080/doc.html)

#### 3、Controller类

	package com.example.swagger2.controller;

	import com.example.swagger2.base.RequestModel;
	import com.example.swagger2.base.ResponseModel;
	import com.example.swagger2.vo.UpdatePasswordVO;
	import com.example.swagger2.vo.UserVO;
	import io.swagger.annotations.Api;
	import io.swagger.annotations.ApiImplicitParam;
	import io.swagger.annotations.ApiOperation;
	import io.swagger.annotations.ApiResponse;
	import io.swagger.annotations.ApiResponses;
	import org.springframework.util.StringUtils;
	import org.springframework.web.bind.annotation.RequestBody;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;

	/**
	 * @Author lijun
	 * @Description
	 * @Date 2020-07-31 2:42 下午
	 **/

	@RestController
	@RequestMapping("/Hello")
	@Api(tags = "用户管理模块")
	public class UserController {

	    @RequestMapping(value = "/getUserInfo", method = RequestMethod.GET)
	    @ApiOperation(value = "根据用户编号获取用户信息", notes = "test: 仅1和2有正确返回")
	    @ApiResponses({
	            @ApiResponse(code = 200, message = "成功"),
	            @ApiResponse(code = 500, message = "联系开发人员调试")
	    })
	    @ApiImplicitParam(paramType = "query", name = "id", value = "用户编号", required = true, dataType = "Integer")
	    public ResponseModel<UserVO> getUserInfo(@RequestParam Integer id) {
	        ResponseModel<UserVO> response = new ResponseModel<>();
	        UserVO userInfo = new UserVO();
	        if (id == 1) {
	            userInfo.setId(1);
	            userInfo.setName("张三");
	            userInfo.setPhone("17789891212");
	            userInfo.setEmail("123456@qq.com");
	            userInfo.setSex("男");
	            response.setCode("0000");
	            response.setMsg("成功");
	            response.setData(userInfo);
	            return response;
	        } else if (id == 2) {
	            userInfo.setId(1);
	            userInfo.setName("李四");
	            userInfo.setPhone("17789891234");
	            userInfo.setEmail("123478@qq.com");
	            userInfo.setSex("女");
	            response.setCode("0000");
	            response.setMsg("成功");
	            response.setData(userInfo);
	            return response;
	        } else {
	            response.setCode("9999");
	            response.setMsg("失败");
	            return response;
	        }
	    }

	    @RequestMapping(value = "/updatePassword", method = RequestMethod.POST)
	    @ApiOperation(value = "修改用户密码", notes = "根据用户id修改密码")
	    @ApiResponses({
	            @ApiResponse(code = 200, message = "成功"),
	            @ApiResponse(code = 500, message = "联系开发人员调试")
	    })
	    public ResponseModel<Object> updatePassword(@RequestBody RequestModel<UpdatePasswordVO> request) {
	        ResponseModel<Object> response = new ResponseModel<>();
	        if (request.getData().getUserId() <= 0 || request.getData().getUserId() > 2) {
	            response.setCode("0001");
	            response.setMsg("请求参数没填好");
	            return response;
	        }
	        if (StringUtils.isEmpty(request.getData().getPassword()) || StringUtils.isEmpty(request.getData().getPassword())) {
	            response.setCode("0002");
	            response.setMsg("密码不能为空");
	            return response;
	        }
	        if (request.getData().getPassword().equals(request.getData().getNewPassword())) {
	            response.setCode("0003");
	            response.setMsg("新旧密码不能相同");
	            return response;
	        }
	        response.setCode("0000");
	        response.setMsg("成功");
	        return response;
	    }
	}

#### 4、接口统一接收类

	package com.example.swagger2.base;

	import io.swagger.annotations.ApiModel;
	import io.swagger.annotations.ApiModelProperty;
	import lombok.Data;

	/**
	 * @Author lijun
	 * @Description
	 * @Date 2020-08-03 1:45 下午
	 **/

	@Data
	@ApiModel("请求接口通用接收对象")
	public class RequestModel<T> {
	    @ApiModelProperty(required = true, notes = "token", example = "4a7d1ed414474e4033ac29ccb8653d9b")
	    private String token;

	    @ApiModelProperty(required = true, notes = "用户ID", example = "1")
	    private Integer userId;

	    @ApiModelProperty(required = true, notes = "签名", example = "dcddb75469b4b4875094e14561e573d8")
	    private String sign;

	    @ApiModelProperty(required = true, notes = "时间戳", example = "1596433801")
	    private String time;

	    @ApiModelProperty(required = true, notes = "请求参数", example = "{\"name\":\"admin\"}")
	    private T data;
	}

#### 5、接口统一返回类

	@Data
	@ApiModel("通用接口返回对象")
	public class ResponseModel<T> {
	    @ApiModelProperty(required = true, notes = "结果码", example = "0000")
	    private String code;

	    @ApiModelProperty(required = true, notes = "结果信息", example = "成功")
	    private String msg;

	    @ApiModelProperty(required = true, notes = "返回数据", example = "{\"name\":\"admin\"}")
	    private T data;
	}

#### 5、数据封装类

	package com.example.swagger2.vo;

	import io.swagger.annotations.ApiModel;
	import io.swagger.annotations.ApiModelProperty;
	import lombok.Data;

	/**
	 * @Author lijun
	 * @Description
	 * @Date 2020-08-03 2:11 下午
	 **/

	@Data
	@ApiModel("用户对象")
	public class UserVO {

	    @ApiModelProperty(required = true, notes = "用户ID", example = "1")
	    private Integer id;

	    @ApiModelProperty(required = true, notes = "姓名", example = "张三")
	    private String name;

	    @ApiModelProperty(required = true, notes = "手机号", example = "17700000000")
	    private String phone;

	    @ApiModelProperty(required = true, notes = "性别", example = "男")
	    private String sex;

	    @ApiModelProperty(required = true, notes = "邮箱", example = "123@163.com")
	    private String email;
	}

重启项目，查看文档的效果：
![OTP](/img/kinfe4j/2.jpg)

当然，原来的swagger-ui也能访问：[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)

## 总结

knife4j有很多注解，可以使文档展示的信息更加完善和友好，大家可以自行尝试和学习，如果还不会用swagger的同学可以看之前我的一篇文章[Swagger](https://mochengyanliu.github.io/2018/10/24/微服务之SpringBoot集成Swagger-2018/)。

 
####  源码下载：

[https://github.com/mochengyanliu/Study/tree/master/swagger2](https://github.com/mochengyanliu/Study/tree/master/swagger2)


