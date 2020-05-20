---
layout: post
title: "SpringCloud-Hystrix"
date: 2020-05-20 
description: "SpringCloud-Hystrix"
tag: SpringCloud 
---

------

# 容错保护：Hystrix

## 前言

<div align="center">
	<img src="/images/posts/SpringCloud-Hystrix/容错保护例子.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Hystrix/容错保护作用.png" />  
</div> 

## 雪崩效应

- 在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种因“服务提供者”的不可用导致“服务消费者”的不可用,并将不可用逐渐放大的过程。

- 如果下图所示：A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。
<div align="center">
	<img src="/images/posts/SpringCloud-Hystrix/雪崩效应.png" />  
</div> 

## Hystrix简介

<div align="center">
	<img src="/images/posts/SpringCloud-Hystrix/Hystrix简介.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Hystrix/Hystrix简介2.png" />  
</div> 

​		当对特定服务的呼叫达到一定阈值时（Hystrix中的默认值为5秒内的20次故障），电路打开，不进行通讯。并且是一个隔离的线程中进行的。

## 实例

直接咋填之前的Eureka项目上进行Hystrix整合。

### 添加依赖

给消费服务EurekaConsumer添加依赖。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

### Main方法

Main方法添加注解@EnableHystrix，启用Hystrix。

```java
@EnableEurekaClient
@SpringBootApplication
@EnableHystrix
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

### 对外接口

添加对外接口

```java
@RestController
public class ConsumerController {
	
	@Autowired
	private RestTemplate restTemplate;
	
	@Autowired
	private ConsumerService consumerService;
	
	@RequestMapping("/gettest")
	public String getUserInfoTest() {
		return restTemplate.getForObject("http://app-user/user/get", String.class);
	}
	
	@RequestMapping("hystrixtest")
	public String hystrixTest() {
		String data = consumerService.getUserInfoByHystrix();
		return data;
	}
}
```

添加Service层及其实现类

```java
public interface ConsumerService {
	String getUserInfoByHystrix();
}
```

```java
@Service
public class ConsumerServiceImpl implements ConsumerService {
	
	@Autowired
	private RestTemplate restTemplate;

	@HystrixCommand(fallbackMethod = "userInfoFallback")//指定方法失败时调用的方法
	public String getUserInfoByHystrix() {
		//模拟方法内部出错
		int i = 1/0;
		return restTemplate.getForObject("http://app-user/user/get", String.class);
	}
	
	public String userInfoFallback() {
		return "获取失败！-----userInfoFallback";
	}
}
```

启动消费服务。

### 运行结果

<div align="center">
	<img src="/images/posts/SpringCloud-Hystrix/获取失败.png" />  
</div> 

​		从运行结果可以看出，访问该接口获得的数据是@HystrixCommand指定的fallbackMechod方法的。在执行方法getUserInfoByHystrix时，由于 `int i = 1/0` 语句执行出错，导致整个方法出错。方法执行可能会因为其调用的服务出错（如用户服务不可用）等，在执行出错时，并不会因为方法执行失败而导致请求失败。这样就达到了容错的效果。


