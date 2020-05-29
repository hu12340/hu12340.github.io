---
layout: post
title: "SpringCloud-Config"
date: 2020-05-22 
description: "SpringCloud-Config"
tag: SpringCloud 
---

------

# 分布式配置中心

- 我们开发项目时，需要有很多的配置项需要写在配置文件中，如：数据库的连接信息等。
- 这样看似没有问题，但是我们想一想，如果我们的项目已经启动运行，那么数据库服务器的ip地址发生了改变，我们该怎么办？
- 如果真是这样，我们的应用需要重新修改配置文件，然后重新启动，如果应用数量庞大，那么这个维护成本就太大了！
- 有没有好的办法解决呢？当然是有的，Spring Cloud Config提供了这样的功能，可以让我们统一管理配置文件，以及实时同步更新，并不需要重新启动应用程序。

## SpringCloudConfig简介

<div align="center">
	<img src="/images/posts/SpringCloud-Config/SpringCloudConfig简介.png" />  
</div> 

- Config Server是一个可横向扩展、集中式的配置服务器，它用于集中管理应用程序各个环境下的配置，默认使用Git存储配置文件内容，也可以使用SVN存储，或者是本地文件存储。
- Config Client是Config Server的客户端，用于操作存储在Config Server中的配置内容。微服务在启动时会请求Config Server获取配置文件的内容，请求到后再启动容器。

## SpringCloudConfig架构

<div align="center">
	<img src="/images/posts/SpringCloud-Config/SpringCloudConfig架构.png" />  
</div> 

## 实例

继续使用之前的Eureka项目。

### 新建远程仓库

​		推送文件到git服务器，这里使用的是码云，当然也可以使用github或者使用svn。使用码云创建一个项目(私有项目需要账号密码)。该实例使用的码云仓库为：`https://gitee.com/hu12340/springcloudconfigserver.git`

### 准备配置文件

这里使用的本地推送到远程仓库（本地新建-add-commit-push）。嫌麻烦可以直接在远程仓库中添加配置文件。

新建一个文件夹，将Eureka项目的所有配置文件复制到该文件夹并重命名。

客户端配置文件引用的默认命名规则为：`服务名称`-`后缀`.`文件类型`。如：`app-consumer-dev.yml` 。

文件名可自义定，可以在客户端配置文件中指定。使用 `前缀`-`后缀`.`文件类型`。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/配置文件目录.png" />  
</div> 

​		本次实例中使用的文件名命名规则为默认命名规则（`服务名称`-`后缀`.`文件类型`）。由于注册中心有两个，第二个注册中心的配置文件名使用的自定义文件名（需要在它自己的配置文件中指定该名字）。

​		将目录中的配置文件同步到远程仓库（add-commit-push）。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/远程仓库目录.png" />  
</div> 

### 新建配置中心项目

#### pom文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hu</groupId>
	<artifactId>EurekaConfig</artifactId>
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
		<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-config-server -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
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

#### Main方法

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigApp {
	public static void main(String[] args) {
		SpringApplication.run(ConfigApp.class, args);
	}
}
```

#### application.xml配置文件

```yaml
server:
  port: 7788 #服务端口
spring:
  application:
    name:  app-config-server #指定服务名
  cloud:
    config:
      server:
        git: #配置git仓库地址
          uri: https://gitee.com/hu12340/springcloudconfigserver.git
          search-paths: /  #配置文件目录地址
          #username: hu12340 #码云账号（公有项目不需要设置）
          #password: 123456  #码云密码（公有项目不需要设置）
      label: master #分支名称
```

#### 启动测试

启动并通过配置中心地址访问配置文件。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/测试访问配置文件.png" />  
</div> 

请求配置文件的规则为（其中{label}是指分支，默认是master）：

/{application}/{profile}/[label]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties

## 整合项目

把所有的Eureka项目都设置为Config客户端，使用配置中心获取配置文件。

### 添加依赖

给之前所有的Eureka项目添加config依赖。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

### 修改原有配置文件

修改之前所有的Eureka项目的配置文件 application.yml ，将原有配置全部注释掉。直接删除该配置文件也可。

### 添加配置文件

给之前所有的Eureka项目添加配置文件 bootstrap.yml 。由于原有配置文件被全部清楚，该项目失去了服务名称。所以需要在新的配置文件中填写服务名称，若原有配置文件的 `spring.application.name` 值还在，可以不指定 `spring.cloud.config.name` 。

```yaml
spring:
  cloud:
    config:
      name: app-consumer #这里写对应配置文件的名称前缀
      uri: http://127.0.0.1:7788/  #配置中心的地址
      profile: dev #对应配置服务中的{profile} 名称后缀
      label: master #对应的分支
```

### 启动测试

重启所有之前所有的Eureka项目。注意：须先启动配置中心项目，因为其他所有项目都须依赖它提供配置文件。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/注册中心.png" />  
</div> 

## 配置文件更新

如果git服务器中的配置文件更新了怎么办？正在运行的应用中的配置内容如何更新？

### 测试

修改远程仓库中  `app-user-dev.yml` 的端口和实例id。

```yaml
server:
  port: 8011
eureka:
  instance:
      instance-id: ${spring.application.name}01 #指定实例id
```

访问测试：

<div align="center">
	<img src="/images/posts/SpringCloud-Config/文件更新成功.png" />  
</div> 

可以看到这里查询到的是新的数据，端口已经更新为8011，实例id也修改成功。

查看注册中心，发现用户服务的实例id并未改变。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/注册中心未改变.png" />  
</div> 

重启用户服务，再次查看注册中心。用户实例id成功更新。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/重启后更新.png" />  
</div> 

## 整合Actuator监控中心

​		服务部署在微服务上之后，重启服务会比较麻烦。如果需要重启服务才能更新配置文件，那么配置中心的作用就无关紧要了。

​		为了能不重启项目就更新配置文件，需要为Config客户端添加Actuator支持。使用actuator 监控中心完成刷新功能。

### 添加依赖

​		在添加了actuator依赖后，直接使用POST的请求方式访问该服务的 `/actuator/refresh` 地址。该服务自己就会自动更新配置文件。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 修改消费服务

​		由于必须使用POST请求访问该地址，浏览器输入的地址是GET方式的。所以这里使用Consumer服务发出
POST请求去访问消费服务的刷新接口。

修改ConsumerController.java，添加对外接口用于发出POST请求。

```java
@RequestMapping("/refresh")
public void refresh() {
    consumerService.refresh();
}
```

修改ConsumerServiceImpl.java，添加service方法实现。

```java
public void refresh() {
    userClient.refresh();
}
```

修改UserClient.java，添加refresh接口。

```java
@FeignClient(value = "app-user",fallback = UserFallback.class)
public interface UserClient {

	@RequestMapping("/user/get")
	String getUserInfo();
	
	@RequestMapping("/user/wrong")
	String getWrongData();
	
    //使用请求方式设置为POST
	@RequestMapping(value = "/actuator/refresh", method = RequestMethod.POST)
	void refresh();
}
```

### 测试

重启服务

<div align="center">
	<img src="/images/posts/SpringCloud-Config/注册中心未变.png" />  
</div> 

修改gitee上 `app-user-dev.yml` 中的实例id。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/修改gitee配置文件.png" />  
</div> 

多次刷新注册中心，用户服务的实例id没有改变。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/注册中心未变.png" />  
</div> 

访问 `http://localhost:8020/refresh` 

<div align="center">
	<img src="/images/posts/SpringCloud-Config/访问刷新接口.png" />  
</div> 

多次刷新注册中心，用户服务的实例id改变成功。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/用户服务实例id改变.png" />  
</div> 



## WebHooks

​		码云、github等git服务器提供了web hook功能，意思是，在仓库中的资源发生更新时会通知给谁，这里的谁是一个url地址。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/WebHook.png" />  
</div> 

​		设置了webhook之后，每当选择的事件触发时，就会向URL发出请求。可以使用这个功能来向指定服务的 `/actuator/refresh` 接口发出请求，进行配置文件更新。

​		我自己使用的是gitee作为配置文件仓库，但gitee的webhook在发送请求时，会带上Payload。

<div align="center">
	<img src="/images/posts/SpringCloud-Config/payload.png" />  
</div> 

​		如果不对该请求做任何处理，直接用该请求访问对应服务的刷新接口，会造成JSON转换异常。因为actuator并不能成功转换请求中的payload，这时就需要对该请求进行过滤。

​		在网上找了很多资料后，就只发现了一种解决方法，大多博客都是转载抄袭或使用的这一种方法。它的主要处理思想是使用拦截器拦截该请求，然后将Payload清空（使用一个空数据替换原有数据）。这样actuator就不会出现JSON转化异常了。

新建拦截器WebhooksFilter

```java
@Component
public class WebhooksFilter implements Filter {

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest httpServletRequest = (HttpServletRequest) request;
		String url = new String(httpServletRequest.getRequestURI());
		// 只过滤/actuator/refresh
		if (!url.endsWith("/actuator/refresh")) {
			chain.doFilter(request, response);
			return;
		}
        //使用HttpServletRequest包装原始请求达到修改post请求中body内容的目的
		MyHttpServletRequestWrapper requestWrapper = new MyHttpServletRequestWrapper(httpServletRequest);
		chain.doFilter(requestWrapper, response);
		return;
	}

}
```

新建MyHttpServletRequestWrapper类

```java
public class MyHttpServletRequestWrapper extends HttpServletRequestWrapper {
	
	private ByteArrayInputStream byteArrayInputStream;

	public MyHttpServletRequestWrapper(HttpServletRequest request) {
		super(request);
	}
	
	@Override
	public ServletInputStream getInputStream() throws IOException {
        byte[] bytes = new byte[0];
        byteArrayInputStream = new ByteArrayInputStream(bytes);
		return new ServletInputStream() {

			@Override
			public boolean isFinished() {
				// TODO Auto-generated method stub
                return byteArrayInputStream.read() == -1 ? true : false;
			}

			@Override
			public boolean isReady() {
				// TODO Auto-generated method stub
				return false;
			}

			@Override
			public void setReadListener(ReadListener listener) {
				// TODO Auto-generated method stub
			}

			@Override
			public int read() throws IOException {
				// TODO Auto-generated method stub
                return byteArrayInputStream.read();
			}
		};
	}
}
```

​		这样就是将请求中的数据用一个空数组代替，接口拿到的时候就不会造成JSON转化异常了（因为什么都没有）。但是我在测试的时候，发现这种方法虽然不会造成JSON转化异常。但是会造成一个别的异常： `Content-length different from byte array length!` 。

​		在http的协议中Content-Length首部告诉浏览器报文中实体主体的大小。这个大小是包含了内容编码的，比如对文件进行了gzip压缩，Content-Length就是压缩后的大小（这点对我们编写服务器非常重要）。除非使用了分块编码，否则Content-Length首部就是带有实体主体的报文必须使用的。使用Content-Length首部是为了能够检测出服务器崩溃而导致的报文截尾，并对共享持久连接的多个报文进行正确分段。

​		由于Content-Length记录了本次请求中报文的大小。在拦截器中将报文大小替换为空之后，Content-Length并没有被改变。这就导致接口在收到请求时，发现本次请求中的报文大小与Content-Length不符，造成请求错误。

​		既然在网上找不到有用的解决方法，那就只能自己想办法解决了。既然webhook发送请求会带参数，造成JSON转化异常。那么我自己手动发请求不就可以了。主要思想就是：在拦截到webhook发送的请求时不放行（即不让该次请求发送出去），调用另外一个接口发送POST请求给相应服务的刷新接口。

修改消费服务，添加一个对外接口

```java
@RequestMapping("/refresh")
public void refresh() {
    consumerService.refresh();
}
```

添加消费服务的service实现

```java
public void refresh() {
    userClient.refresh();
}
```

修改消费服务中用户的Feign客户端

```java
@FeignClient(value = "app-user",fallback = UserFallback.class)
public interface UserClient {

	@RequestMapping("/user/get")
	String getUserInfo();
	
	@RequestMapping("/user/wrong")
	String getWrongData();
	
    //向/actuator/refresh接口发送POST请求
	@RequestMapping(value = "/actuator/refresh", method = RequestMethod.POST)
	void refresh();
}
```

修改WebhooksFilter

```java
@Component
public class WebhooksFilter implements Filter {

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest httpServletRequest = (HttpServletRequest) request;
		String url = new String(httpServletRequest.getRequestURI());
		// 只过滤/actuator/refresh
		if (!url.endsWith("/actuator/refresh")) {
			chain.doFilter(request, response);
			return;
		}
        restTemplate.getForObject("http://app-consumer/refresh", String.class);
		return;
	}

}
```

运行，修改gitee仓库中 `app-user-dev.yml` 文件的instanceid

<div align="center">
	<img src="/images/posts/SpringCloud-Config/instanceid01.png" />  
</div> 

刷新注册中心，用户服务实例id未改变

<div align="center">
	<img src="/images/posts/SpringCloud-Config/app-user00.png" />  
</div> 

访问 `http://localhost:8050/refreshconfig`

<div align="center">
	<img src="/images/posts/SpringCloud-Config/8050refresh.png" />  
</div> 

刷新注册中心，实例id改变

<div align="center">
	<img src="/images/posts/SpringCloud-Config/注册中心01.png" />  
</div> 



​		上述方法是在拦截器中调用消费服务，然后让消费服务去调用用户服务的配置刷新接口。那么可不可以直接在拦截器中调用用户服务的配置刷新接口呢？

修改拦截器

```java
@Component
public class WebhooksFilter implements Filter {

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest httpServletRequest = (HttpServletRequest) request;
		String url = new String(httpServletRequest.getRequestURI());
		// 只过滤/actuator/refresh
		if (!url.endsWith("/actuator/refresh")) {
			chain.doFilter(request, response);
			return;
		}
        restTemplate.postForObject("http://app-user/actuator/refresh", httpServletRequest, String.class);
		return;
	}

}
```

测试运行，访问 `http://localhost:8050/refreshconfig`

<div align="center">
	<img src="/images/posts/SpringCloud-Config/访问报错.png" />  
</div> 

