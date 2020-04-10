---
layout: post
title: "SpringMVC的基本使用"
date: 2020-04-03 
description: "SpringMVC的基本使用"
tag: springmvc 
---

------

# SpringMVC介绍

​		SpringMVC是spring框架的一个模块，springmvc和spring无需通过中间整合层进行整合。

## 什么是MVC

SpringMVC是一个基于MVC的web框架(MVC是指，C控制层，M模块层，V显示层这样的设计理念)。

1. 最上面的一层，是直接面向最终用户的"视图层"（View）。它是提供给用户的操作界面，是程序的外壳。
2. 最底下的一层，是核心的"数据层"（Model），也就是程序需要操作的数据或信息。
3. 中间的一层，就是"控制层"（Controller），它负责根据用户从"视图层"输入的指令，选取"数据层"中的数据，然后对其进行相应的操作，产生最终结果。

​		这三层是紧密联系在一起的，但又是互相独立的，每一层内部的变化不影响其他层。每一层都对外提供接口，供上面一层调用。这样一来，软件就可以实现模块化，修改外观或者变更数据都不用修改其他层，大大方便了维护和升级。

## 为什么使用SpringMVC

​		Spring Web MVC是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，即使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用**请求-响应**模型，框架的目的就是帮助我们简化开发，Spring Web MVC也是要简化我们日常Web开发的。

## SpringMVC的执行流程

SpringMVC框架的核心是`DispatcherServlet`（前端处理器）。它对用户的请求处理大致如下：

![](/home/android/胡广/hu12340.github.io/images/posts/SpringMVC的基本使用/SpringMVC的执行流程.png)

1. 用户发送请求到`DispatcherServlet`
2. `DispatcherServlet`交由`HandlerMapping`查询`Handler`
3. `HandlerMapping`将查询到的`HandlerExecutionChain`返回给`DispatcherServlet`
4. `DispatcherServlet`将`Handler`交给`HandlerAdapter`处理
5. `HandlerAdapter`交给具体的`Handler`处理该请求
6. `Handler`返回`ModelAndView`给`HandlerAdapter`
7. `HandlerAdapter`将`ModelAndView`返回给`DispatcherServlet`
8. `DispatcherServlet`将`ModelAndView`交由`ViewResolver`解析
9. `ViewResolver`返回视图对象给`DispatcherServlet`
10. `DispatcherServlet`渲染视图
11. `DispatcherServlet`响应用户请求

------

# SpringMVC的使用

## 配置文件

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<welcome-file-list>
		<welcome-file>/html/index.html</welcome-file>
	</welcome-file-list>
	<servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>
```

​		`welcome-file-lis` 标签用来配置项目的启动页（该项目的启动页为html目录下的index.html）。

​		`servlet` 标签用来配置springmvc的前端处理器`DispatcherServlet`；`servlet-mapping`用来配置定义的servlet所处理的请求。`/` 表示使用该处理器处理所有请求。

​		`init-param` 用来设置初始化参数。

springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.0.xsd   
        http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.0.xsd   
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.3.xsd
        http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd  
        ">
	
	<!-- 启用spring mvc 注解 -->
	<mvc:annotation-driven />

	<!-- 注解扫描包 -->
	<context:component-scan base-package="com.hu" />

	<!-- 静态资源 -->
	<mvc:resources location="/html/" mapping="/html/**" />

	<!-- 视图解析器 prefix：前缀 suffix：后缀 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver"
		p:prefix="/html/" p:suffix=".html" />
</beans>
```

springmvc的配置文件主要有几步：

1. 开启springmvc注解
2. 注解扫描包
3. 配置静态资源路径
4. 配置视图解析器

------

## 常见注解

**@Controller**

​		标注在类后，该类就会作为一个Controller，用于接收用户请求。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。

**@RequestMapping**

​		可以使用@RequestMapping 来映射URL 到控制器类，或者是到Controller 控制器的处理方法上。当@RequestMapping 标记在Controller 类上的时候，里面使用@RequestMapping 标记的方法的请求地址都是相对于类上的@RequestMapping 而言的；当Controller 类上没有标记@RequestMapping 注解时，方法上的@RequestMapping 都是绝对路径。这种绝对路径和相对路径所组合成的最终路径都是相对于根路径“/ ”而言的。

**@ResponseBody**

​		作用在方法上的，@ResponseBody 表示该方法的返回结果直接写入 HTTP response body 中，通常用来返回JSON数据或者是XML数据，一般在异步获取数据时使用。

**@Service**

​		标注在类上，一般标注在逻辑层的类上。Controller层调用Service进行逻辑处理。





























​		这个是使用github搭建博客所需使用的文章模板，新建文章都可以在此基础上复制并修改。并且博客文章的名字也必须是 yyyy-mm-dd-文章标题 （如：2019-11-01-博客文章模板）

<div align="center">
	<img src="/images/posts/图片路径" />  
</div> 

