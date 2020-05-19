---
layout: post
title: "SpringCloud-Eureka"
date: 2020-05-19 
description: "SpringCloud-Eureka"
tag: SpringCloud 
---

------

[TOC]

------

# Eureka简介

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/Eureka简介.png" />  
</div> 

Eureka包含两个组件：`Eureka Server`和`Eureka Client`。

- Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。

- Eureka Client是一个java客户端，用于简化与Eureka Server的交互，客户端同时也就别一个内置的、使用轮询(round-robin)负载算法的负载均衡器。

- 在应用启动后，将会向Eureka Server发送心跳,默认周期为30秒，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90秒)。

- Eureka Server之间通过复制的方式完成数据的同步，Eureka还提供了客户端缓存机制，即使所有的Eureka Server都挂掉，客户端依然可以利用缓存中的信息消费其他服务的API。综上，**Eureka通过心跳检查、客户端缓存等机制，确保了系统的高可用性、灵活性和可伸缩性。**

# 基本实例

## 服务中心EurekaServer

### pom文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hu</groupId>
	<artifactId>EurekaServer</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.7.RELEASE</version>
	</parent>

	<dependencyManagement>
		<dependencies>
			<!-- 导入Spring Cloud的依赖管理 -->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Hoxton.SR4</version>
				<type>pom</type>
				<scope>runtime</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!--springboot 整合eureka服务端 -->
		<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-server -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
			<version>2.2.2.RELEASE</version>
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

### Main方法

```java
@SpringBootApplication
@EnableEurekaServer //声明服务器
public class ServerApp {

	public static void main(String[] args) {
		SpringApplication.run(ServerApp.class, args);
	}

}
```

### application.xml配置文件

```yaml
###服务端口号
server:
  port: 8000

###服务名称
spring:
  application:
    name: app-eureka-server

eureka:
  instance:
    #注册中心地址
    hostname: 127.0.0.1
###客户端调用地址
  client:
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8000/eureka/
###是否将自己注册到Eureka服务中,因为该应用本身就是注册中心，不需要再注册自己（集群的时候为true）
    register-with-eureka: false
###是否从Eureka中获取注册信息,因为自己为注册中心,不会在该应用中的检索服务信息
    fetch-registry: false
```

### 运行结果

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/初始EurekaServer主界面.png" />  
</div> 

注册中心出现的这个红色警告，主要是因为当前Eureka进入了自我保护模式。(先开启Eureka server端和client端，然后再断开client端，此时刷新Eureka界面，就会看到红色字样)

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/Eureka的自我保护.png" />  
</div> 

在短时间内丢失了服务实例的心跳，不会剔除该服务，这是eurekaserver的自我保护机制的宗旨。主要是为了防止由于短暂的网络故障误删除可用的服务。

所以，一般进入自我保护模式，无需处理。如果，需要禁用自我保护模式，只需要在配置文件中添加配置即可：
(测试环境、开发环境可以关闭自我保护机制，保证服务不可用时及时剔除)

```yaml
eureka:
  instance:
    #注册中心地址
    hostname: 127.0.0.1

###客户端调用地址
  client:
    serviceUrl:
      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:8000/eureka/
    register-with-eureka: false
    fetch-registry: false

  server:
      enable-self-preservation: false #禁用自我保护模式
```

## 客户端服务EurekaUser

### pom文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.hu</groupId>
  <artifactId>EurekaUser</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.7.RELEASE</version>
	</parent>

	<dependencyManagement>
		<dependencies>
			<!-- 导入Spring Cloud的依赖管理 -->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Hoxton.SR4</version>
				<type>pom</type>
				<scope>runtime</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!--springboot 整合eureka客户端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.2.2.RELEASE</version>
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

### Main方法

```java
@EnableEurekaClient
@SpringBootApplication
public class UserApp {

	public static void main(String[] args) {
		SpringApplication.run(UserApp.class, args);

	}

}
```

### application.xml配置文件

```yaml
###服务端口号(本身是一个web项目)
server:
  port: 8010
  
###起个名字作为服务名称(该服务注册到eureka注册中心的名称)
spring:
    application:
        name: app-user

###服务注册到eureka注册中心的地址
eureka:
  client:
    service-url:
           defaultZone: http://hu:12340@127.0.0.1:8000/eureka
###因为该应用为服务提供者，是eureka的一个客户端，需要注册到注册中心
    register-with-eureka: true
###是否需要从eureka上检索服务
    fetch-registry: true
```

### 运行结果

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/App-User注册成功.png" />  
</div> 

### 添加对外接口

添加对外接口UserController，用于其它服务调用。

```java
@RestController
public class UserController {

	@RequestMapping("/get")
	public String getUserInfo() {
		return "获取成功！";
	}
}
```

## 客户端服务EurekaConsumer

### pom文件

pom文件与EurekaUser类似。

### Main方法

```java
@EnableEurekaClient
@SpringBootApplication
public class ConsumerApp {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApp.class, args);

	}
	
    //加入RestTemplate用于调用其他服务
	@Bean
    @LoadBalanced
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}

}
```

### application.xml配置文件

```yaml
###服务端口号(本身是一个web项目)
server:
  port: 8020
  
###起个名字作为服务名称(该服务注册到eureka注册中心的名称)
spring:
    application:
        name: app-consumer

###服务注册到eureka注册中心的地址
eureka:
  client:
    service-url:
           defaultZone: http://hu:12340@127.0.0.1:8000/eureka
###因为该应用为服务提供者，是eureka的一个客户端，需要注册到注册中心
    register-with-eureka: true
###是否需要从eureka上检索服务
    fetch-registry: true
```

### 添加对外接口

```java
@RestController
public class ConsumerController {
	
	@Autowired
	private RestTemplate restTemplate;
	
	@RequestMapping("/gettest")
	public String getUserInfoTest() {
		// 该方法走eureka注册中心调用去注册中心根据app-item查找服务
        //这种方式必须先开启负载均衡在Main方法中RestTemplate上加注解@LoadBalanced
        //否则会造成找不到服务的错误
		return restTemplate.getForObject("http://app-user/user/get", String.class);
	}
}
```

### 运行结果

运行EurekaConsumer，Eureka注册中心出现新的实例

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/EurekaServer显示两个实例.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/运行访问EurekaConsumer.png" />  
</div> 

# 用户认证

## 添加依赖

为Eureka服务端（eureka-server）添加安全认证依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 修改配置文件

```yaml
###服务端口号
server:
  port: 8000

###服务名称
spring:
  application:
    name: app-eureka-server
  security:
    basic:
      enable: true #开启基于HTTP basic的认证
    user: #配置用户的账号信息
      name: hu
      password: 12340

eureka:
  instance:
    #注册中心地址
    hostname: 127.0.0.1
###客户端调用地址
  client:
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8000/eureka/
###是否将自己注册到Eureka服务中,因为该应用本身就是注册中心，不需要再注册自己（集群的时候为true）
    register-with-eureka: false
###是否从Eureka中获取注册信息,因为自己为注册中心,不会在该应用中的检索服务信息
    fetch-registry: false
```

## 添加安全认证类

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// Configure HttpSecurity as needed (e.g. enable http basic).
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.NEVER);
        http.csrf().disable();
        //注意：为了可以使用 http://${user}:${password}@${host}:${port}/eureka/ 这种方式登录,所以必须是httpBasic,
        // 如果是form方式,不能使用url格式登录
        http.authorizeRequests().anyRequest().authenticated().and().httpBasic();
	}

}
```

## 重启EurekaServer

EurekaServer重启后，进入注册中心需登录。

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/EurekaServer需登录、.png" />  
</div> 

由于Eureka客户端并没有使用账号密码进行服务注册，所以此时注册中心中的实例为空。

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/注册中心为空.png" />  
</div> 

## 客户端认证

修改客户端的配置：`http://USER:PASSWORD@127.0.0.1:端口号/eureka/`

```yaml
eureka:
  client:
    service-url:
           defaultZone: http://hu:12340@127.0.0.1:8000/eureka
```

重启服务

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/EurekaServer显示两个实例.png" />  
</div> 

# Eureka集群

前面介绍的Eureka服务是一个单点服务，在生产环境就会出现单点故障。为了确保Eureka服务的高可用，我需要搭建Eureka服务的集群。

搭建Eureka集群非常简单，只要启动多个Eureka Server服务并且让这些Server端之间彼此进行注册即可实现。

## 添加EurekaServer

pom文件和Main方法与上一个EurekaServer一致。

### application.xml配置文件

```yaml
###服务端口号
server:
  port: 9000

###服务名称
spring:
  application:
    name: app-eureka-server
  security:
    basic:
      enable: true #开启基于HTTP basic的认证
    user: #配置用户的账号信息
      name: hu
      password: 12340

eureka:
  instance:
    #注册中心地址
    hostname: 127.0.0.1
###客户端调用地址
  client:
    serviceUrl:
      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:8000/eureka/
###是否将自己注册到Eureka服务中,因为该应用本身就是注册中心，不需要再注册自己（集群的时候为true）
    register-with-eureka: true
###是否从Eureka中获取注册信息,因为自己为注册中心,不会在该应用中的检索服务信息
    fetch-registry: true
```

集群的EurekaServer需使用同一个applicationname，并需要将自身注册到另外一个注册中心中。

## 修改原有EurekaServer

### application.xml配置文件

```
###服务端口号
server:
  port: 8000

###服务名称
spring:
  application:
    name: app-eureka-server
  security:
    basic:
      enable: true #开启基于HTTP basic的认证
    user: #配置用户的账号信息
      name: hu
      password: 12340

eureka:
  instance:
    #注册中心地址
    hostname: 127.0.0.1
###客户端调用地址
  client:
    serviceUrl:
      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:9000/eureka/
###是否将自己注册到Eureka服务中,因为该应用本身就是注册中心，不需要再注册自己（集群的时候为true）
    register-with-eureka: true
###是否从Eureka中获取注册信息,因为自己为注册中心,不会在该应用中的检索服务信息
    fetch-registry: true
```

## 修改客户端

### application.xml配置文件

将自身注册到两个注册中心中。

```yaml
###服务注册到eureka注册中心的地址
eureka:
  client:
    service-url:
           defaultZone: http://hu:12340@127.0.0.1:8000/eureka,http://hu:12340@127.0.0.1:9000/eureka
```

## 运行结果

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/添加注册中心.png" />  
</div> 

此时若将原有那个EurekaServer关掉，客户端仍能使用另外一个注册中心

# 指定服务的IP地址

在服务的提供者配置文件中可以指定ip地址。若将服务放到云服务器上，在服务器内部的服务之间可通过内部地址（如私有地址或127.0.0.1）进行服务的调用。而其他服务器的服务却不能使用对方服务器内部地址（如私有地址或127.0.0.1）进行服务的调用。这时需要给服务指定一个IP地址（公有地址），使得其他服务器的服务也能调用该服务。

```yaml
eureka:
  instance:
    #注册中心地址
    hostname: 127.0.0.1
    prefer-ip-address: true #将自己的ip地址注册到Eureka服务中
    ip-address: 172.19.160.76
```

# 指定服务实例ID

实例名也就是InstanceInfo类中的instanceId属性，它是区分同一服务中不同实例的唯一标识。通过instance-id 参数指定服务注册到Eureka中的服务实例id。

修改EurekaUser的配置文件，再次启动（不关掉原有服务）。修改端口号，相当于启动了另外一个EurekaUser服务。

```yaml
###服务端口号(本身是一个web项目)
server:
  port: 8011
  
###起个名字作为服务名称(该服务注册到eureka注册中心的名称，比如商品服务)
spring:
    application:
      name: app-user

###服务注册到eureka注册中心的地址
eureka:
  client:
    service-url:
           defaultZone: http://hu:12340@127.0.0.1:8000/eureka,http://hu:12340@127.0.0.1:9000/eureka
###因为该应用为服务提供者，是eureka的一个客户端，需要注册到注册中心
    register-with-eureka: true
###是否需要从eureka上检索服务
    fetch-registry: true
  instance:
      instance-id: ${spring.application.name}###${server.port} #指定实例id
```

运行结果：

<div align="center">
	<img src="/images/posts/SpringCloud-Eureka/修改服务instanceId.png" />  
</div> 


