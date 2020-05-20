---
layout: post
title: "SpringCloud-Ribbon"
date: 2020-05-18 
description: "SpringCloud-Ribbon"
tag: SpringCloud 
---

------

# 负载均衡 Ribbon

​		如果为同一个的提供者在Eureka中注册了多个服务，那么客户端该如何选择服务呢？这时，就需要在客户端实现服务的负载均衡。在Spring Cloud中推荐使用Ribbon来实现负载均衡。

## Ribbon简介

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/Ribbon简介.png" />  
</div> 

## Ribbon架构

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/Ribbon架构.png" />  
</div> 

## 实例

实例从上次Eureka项目上进行整合。

### 添加依赖

为消费者服务添加Ribbon依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-ribbon -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

​		该依赖可以省略，因为spring-cloud-starter-netflix-eureka-client中已经包含了spring-cloud-starter-netflix-ribbon。

### 添加注解

为Main方法中，RestTemplate上添加@LoadBalanced注解。

```java
//加入RestTemplate用于调用其他服务
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

该注解在整合Eureka时就已经添加，当时是因为消费者服务找不到目标服务。添加这个注解后，RestTemplate就具备了负载均衡的功能。

### 新增服务

修改EurekaUser的端口号、instance-id和Controller返回数据，多次启动服务。

修改application.yml中的端口号和instance-id

```yaml
server:
  port: 8010 #指定端口号 00、01、02、03、04

  instance:
      instance-id: ${spring.application.name}00 #指定实例id 00、01、02、03、04
```

修改UserController.java返回数据

```java
@RequestMapping("/get")
public String getUserInfo() {
    return "获取成功！-----00";// 00、01、02、03、04
}
```

修改一次后发布，注册中心出现多个app-user服务。

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/多个app-user.png" />  
</div> 

使用consumer服务多次获取数据，返回的数据并并不一定相同，也没有固定顺序。其内部是通过Ribbon自己的负载均衡策略，去调用用户服务。

消费者服务的Controller

```java
@RequestMapping("/gettest")
	public String getUserInfoTest() {
		// 该方法走eureka注册中心调用(去注册中心根据app-item查找服务，这种方式必须先开启负载均衡在Main方法中RestTemplate上加注解@LoadBalanced)
		return restTemplate.getForObject("http://app-user/user/get", String.class);
	}
```

以下6条运行结果是多次运行结果拼接起来的，每次运行结果实际只有一条数据。

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/consumer获取数据.png" />  
</div> 

修改消费者服务的Controller。

```java
@RestController
public class ConsumerController {
	
	@Autowired
	private RestTemplate restTemplate;
	
	@Autowired
    private DiscoveryClient discoveryClient;
	
	@RequestMapping("/gettest")
	public String getUserInfoTest() {
		List<ServiceInstance> instances = discoveryClient.getInstances("app-user");
		StringBuilder sb = new StringBuilder();
		for (ServiceInstance serviceInstance : instances) {
			String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort();
			System.out.println(url + " " + serviceInstance.getInstanceId());
			sb.append(restTemplate.getForObject(url + "/user/get", String.class) + "\n");
		}
		return sb.toString();
	}
}
```

访问消费者服务接口

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/获取所有app-user.png" />  
</div> 

控制台打印

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/获取所有app-user控制台打印.png" />  
</div> 

## 其他策略

只需要在配置文件中添加配置。

serviceId.ribbon.NFLoadBalancerRuleClassName=自定义的负载均衡策略类

```yaml
app-user:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/AbstractLoadBalancerRule.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/RandomRule.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/RetryRule.png" />  
</div> 

<div align="center">
	<img src="/images/posts/SpringCloud-Ribbon/RoundRobinRule.png" />  
</div> 


