---
layout: post
title: "SpringCloud-Feign"
date: 2020-05-20 
description: "SpringCloud-Feign"
tag: SpringCloud 
---

------

# Feign客户端

## 前言

​		之前在使用消费服务调用用户服务时，都是通过RestTemplate.getForObject方法进行调用的。虽然使用了Ribbon和Hystrix可以实现负载均衡和容错处理，但是这个编码在实现大量业务时会显得太过于冗余（如，多参数的URL拼接）。

## Feign简介

<div align="center">
	<img src="/images/posts/SpringCloud-Feign/Feign简介.png" />  
</div> 

## 实例

在之前的Eureka项目上进行Feign整合。

### 添加依赖

给他EurekaConsumer添加feign依赖。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

### Main方法

给Main方法添加注解，开启Feign支持。

```java
@EnableEurekaClient
@SpringBootApplication
@EnableHystrix
@EnableFeignClients
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

#### Controller层

添加对外接口

```java
@RestController
public class ConsumerController {
	@Autowired
	private ConsumerService consumerService;
	
	@RequestMapping("feigntest")
	public String feignTest() {
		String data = consumerService.getUserInfoByFeign();
		return data;
	}
}
```

#### Service

```java
@Service
public class ConsumerServiceImpl implements ConsumerService {
	@Autowired
	private UserClient userClient;

	public String getUserInfoByFeign() {
		return userClient.getUserInfo();
	}
}
```

#### Client

```java
@FeignClient(value = "app-user")
public interface UserClient {

    //服务方参数列表使用注解@RequestParam、@PathVariable修饰 可达到多参数构造
	@RequestMapping("/user/get")
	String getUserInfo();
}
```

运行访问消费服务。

<div align="center">
	<img src="/images/posts/SpringCloud-Feign/访问FeignTest.png" />  
</div> 

流程分析：

1. 由于在入口程序使用了@EnableFeignClients注解，Spring启动后会扫描标注了@FeignClient注解的接口，然后生成代理类。
2. 在@FeignClient接口中指定了value，其实就是指定了在Eureka中的服务名称。
3. 在FeignClient中的定义方法以及使用了SpringMVC的注解，Feign就会根据注解中的内容生成对应的URL，然后基于Ribbon的负载均衡去调用REST服务。

### 统一hystrix fallback接口

一般在实际开发中fallback 方法不会直接写在接口方法所在类里，那样太杂乱。将fallback方法放到类中。

#### Fallback类

新建统一Fallback类

```java
@Component //需要被扫描到
public class UserFallback implements UserClient{
	public String getUserInfo() {
		return "getUserInfo执行失败";
	}
	public String getWrongData() {
		return "getWrongData执行失败";
	}
}
```

#### Controller

```java
@RestController
public class ConsumerController {
	@Autowired
	private ConsumerService consumerService;
	
	@RequestMapping("feigntest")
	public String feignTest() {
		String data = consumerService.getUserInfoByFeign();
		return data;
	}
    @RequestMapping("feignwrong")
	public String feignWrong() {
		String data = consumerService.getWrongByFeign();
		return data;
	}
}
```

#### Service

```java
@Service
public class ConsumerServiceImpl implements ConsumerService {
	@Autowired
	private UserClient userClient;
	public String getUserInfoByFeign() {
		return userClient.getUserInfo();
	}
	public String getWrongByFeign() {
		return userClient.getWrongData();
	}
}
```

#### Client

```java
@FeignClient(value = "app-user",fallback = UserFallback.class) //指定fallback类
public interface UserClient {

	@RequestMapping("/user/get")
	String getUserInfo();
	
	@RequestMapping("/user/wrong")
	String getWrongData();
}
```

#### 配置文件

配置文件中开启hystrix

```yaml
#开启hystrix断路器
feign:
  hystrix:
    enabled: true
```

运行结果如下

<div align="center">
	<img src="/images/posts/SpringCloud-Feign/访问FeignTest.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Feign/getWrongData执行失败.png" />  
</div> 


