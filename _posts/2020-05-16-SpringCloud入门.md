---
layout: post
title: "SpringCloud入门"
date: 2020-05-16 
description: "SpringCloud入门"
tag: SpringCloud 
---

------

------

# 微服务架构

目前微服务是非常火的架构或者说概念，也是在构建大型互联网项目时采用的架构方式。

## 单体架构

单体架构，是指将开发好的项目打成war包，然后发布到tomcat等容器中的应用。

假设你正准备开发一款与Uber和滴滴竞争的出租车调度软件，经过初步会议和需求分析，你可能会手动或者使用基于Spring Boot、Play或者Maven的生成器开始这个新项目，它的六边形架构是模块化的 ，架构图如下：
<div align="center">
	<img src="/images/posts/SpringCloud入门/单体架构图.png" />  
</div> 

- 应用核心是业务逻辑，由定义服务、领域对象和事件的模块完成。围绕着核心的是与外界打交道的适配器。适配器包括数据库访问组件、生产和处理消息的消息组件，以及提供API或者UI访问支持的web模块等。

- 尽管也是模块化逻辑，但是最终它还是会打包并部署为单体式应用。具体的格式依赖于应用语言和框架。例如，许多Java应用会被打包为WAR格式，部署在Tomcat或者Jetty上，而另外一些Java应用会被打包成自包含的JAR格式，类似的，Rails和Node.js会被打包成层级目录。

- 这种应用开发风格很常见，因为IDE和其它工具都擅长开发一个简单应用，这类应用也很易于调试，只需要简单运行此应用，用Selenium链接UI就可以完成端到端测试。单体式应用也易于部署，只需要把打包应用拷贝到服务器端，通过在负载均衡器后端运行多个拷贝就可以轻松实现应用扩展。在早期这类应用运行的很好。

## 单体架构存在的问题

<div align="center">
	<img src="/images/posts/SpringCloud入门/单体架构存在的问题1.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud入门/单体架构存在的问题2.png" />  
</div> 

## 什么是微服务

<div align="center">
	<img src="/images/posts/SpringCloud入门/什么是微服务.png" />  
</div> 

## 微服务架构的特征

<div align="center">
	<img src="/images/posts/SpringCloud入门/微服务架构的特征.png" />  
</div> 



# SpringCloud

## SpringCloud简介

<div align="center">
	<img src="/images/posts/SpringCloud入门/SpringCloud简介.png" />  
</div> 

## 框架特点

<div align="center">
	<img src="/images/posts/SpringCloud入门/SpringCloud框架特点.png" />  
</div> 



## 实例

### 用户服务

#### pom文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.hu</groupId>
  <artifactId>RestTemplateUser</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.7.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- 资源文件拷贝插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!-- java编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Main方法

```java
@SpringBootApplication
public class UserApp {

	public static void main(String[] args) {
		SpringApplication.run(UserApp.class, args);
	}

}
```

#### application.xml配置文件

```yaml
server:
  port: 8010 #服务端口
```

#### 对外接口

```java
@RestController
@RequestMapping("user")
public class UserController {

	@RequestMapping("/get")
	public String getUserInfo() {
		return "获取成功！";
	}
}
```

启动服务，等待消费服务访问。

### 消费服务

pom文件与用户服务一样

#### application.xml配置文件

```yaml
server:
  port: 8020 #服务端口
```

#### Main方法

```java
@SpringBootApplication
public class ConsumerApp {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApp.class, args);
	}
	
    //加入RestTemplate用于调用其他服务
	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}

}
```

#### 对外接口

```java
@RestController
public class ConsumerController {

	@Resource
	private RestTemplate restTemplate;
	
	@RequestMapping("gettest")
	public String getUserInfo() {
        //使用restTemplate调用用户服务模块
		return restTemplate.getForObject("http://127.0.0.1:8010/user/get", String.class);
	}
}
```

#### 运行结果

<div align="center">
	<img src="/images/posts/SpringCloud入门/消费服务调用用户服务.png" />  
</div> 

## OkHttp

### 简介

HTTP是现代应用常用的一种交换数据和媒体的网络方式，高效地使用HTTP能让资源加载更快，节省带宽。

### 特性

OkHttp是一个高效的HTTP客户端，它有以下默认特性：

- 支持HTTP/2，允许所有同一个主机地址的请求共享同一个socket连接
- 连接池减少请求延时
- 透明的GZIP压缩减少响应数据的大小
- 缓存响应内容，避免一些完全重复的请求

当网络出现问题的时候OkHttp依然坚守自己的职责，它会自动恢复一般的连接问题，如果你的服务有多个IP地址，当第一个IP请求失败时，OkHttp会交替尝试你配置的其他IP，OkHttp使用现代TLS技术(SNI, ALPN)初始化新的连接，当握手失败时会回退到TLS 1.0。

### 实例

#### 添加依赖

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
```

#### 修改Main方法

```java
@SpringBootApplication
public class ConsumerApp {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApp.class, args);
	}
	
	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate(new OkHttp3ClientHttpRequestFactory());
	}

}
```

### 硬编码问题

将url地址写入到application.yml配置文件中。

```yaml
server:
  port: 8082 #服务端口
resttemplate:
  item:
    url: http://127.0.0.1:8010/user/get
```

修改对外接口

```java
@RestController
public class ConsumerController {

    @Value("${resttemplate.user.url}")
    private String url;
    
	@Resource
	private RestTemplate restTemplate;
	
	@RequestMapping("gettest")
	public String getUserInfo() {
        //使用restTemplate调用用户服务模块
		return restTemplate.getForObject(url, String.class);
	}
}
```

运行

<div align="center">
	<img src="/images/posts/SpringCloud入门/消费服务调用用户服务.png" />  
</div> 

