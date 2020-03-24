---
layout: post
title: "Spring-控制反转(IOC)"
date: 2020-03-23 
description: "Spring-控制反转(IOC)"
tag: spring 
---

------

# **Spring框架概述**

​		Spring是一个开源框架，Spring是于2003 年兴起的一个轻量级的Java 开发框架，由Rod Johnson 在其著作Expert One-On-One J2EE Development and Design中阐述的部分理念和原型衍生而来。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为 J2EE 应用程序开发提供集成的框架。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是**控制反转（IoC）**和**面向切面（AOP）**。简单来说，Spring是一个分层的JavaSE/EE full-stack(一站式) 轻量级开源框架。

# **Spring的优点**

- **方便解耦，简化开发（高内聚低耦合）**

    Spring就是一个大工厂（容器），管理着所有对象的创建和依赖关系。Spring工厂是用于生成bean。

- **AOP编程的支持**

    Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能

- **声明式事务的支持**

    只需要通过配置就可以完成对事务的管理，而无需手动编程

- **方便程序的测试**

    Spring对Junit4支持，可以通过注解方便的测试Spring程序

- **方便集成各种优秀框架**

    Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、MyBatis、Quartz等）的直接支持

- **降低JavaEE API的使用难度**

    Spring 对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低

# **Spring的使用**

​		使用Spring框架需要在项目中导入四个核心jar包和一个依赖。4个核心（[beans](https://repo1.maven.org/maven2/org/springframework/spring-beans/5.2.4.RELEASE/spring-beans-5.2.4.RELEASE.jar)、[core](https://repo1.maven.org/maven2/org/springframework/spring-core/5.2.4.RELEASE/spring-core-5.2.4.RELEASE.jar)、[context](https://repo1.maven.org/maven2/org/springframework/spring-context/5.2.4.RELEASE/spring-context-5.2.4.RELEASE.jar)、[expression](https://repo1.maven.org/maven2/org/springframework/spring-expression/5.2.4.RELEASE/spring-expression-5.2.4.RELEASE.jar)） + 1个依赖（[commons-logging](https://repo1.maven.org/maven2/commons-logging/commons-logging/1.2/commons-logging-1.2.jar)）。

在src目录下创建配置文件 `applicationContext.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

新建一个实体类 `Person.java`

```java
public class Person {

	private String name;
	private int age;
	private char sex;
    
    //构造方法Person()、Person(String name)、Person(String name, int age, char sex) 
    //在构造方法中添加控制台输入，用于分辨
	public Person() {
		System.out.println("Person()");
	}

	public Person(String name) {
		System.out.println("Person(String name)");
		this.name = name;
	}

	public Person(String name, int age, char sex) {
		System.out.println("Person(String name, int age, char sex)");
		this.name = name;
		this.age = age;
		this.sex = sex;
	}
    
    //Getter()和Setter()
    //toString()
    ...
}
```

在配置文件`applicationContext.xml`中，添加类`Person`的bean。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- schema约束规则 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- name:该bean的名字(可使用该名字进行bean的获取) class:该bean指向的类(设置该bean对象的Java类型) -->
	<bean name="person" class="com.hu.pojo.Person">
        <!-- property:属性 name:属性名 value:属性值 -->
		<property name="name" value="hu"></property>
		<property name="age" value="22"></property>
		<property name="sex" value="男"></property>
	</bean>
</beans>
```

测试类`Demo.java`：

```java
public class Demo {
	public static void main(String[] args) {
		ClassPathXmlApplicationContext context = 
            new ClassPathXmlApplicationContext("applicationContext.xml");
		//通过context获取Person对象
		Person person = (Person) context.getBean("person");
		System.out.println(person);
		context.close();
	}
}
```

结果分析：

<div align="center">
	<img src="/images/posts/Spring-控制反转(IOC)/property注入参数.png" />  
</div> 

​		从结果可以看出，Person对象被成功创建，并且属性已被赋值。这里并没有使用new Person()创建对象，而是使用了控制反转的方式（**控制反转：类依赖于接口的实现类，但是自己不去构造它，构造的选择权交给第三方**）。spring容器构建对象，默认使用的是无参构造。若想使用有参构创建对象，则需要使用`constructor-arg`标签。

修改配置文件`applicationContext.xml`中的`<bean>`标签。

```xml
<!-- name:该bean的名字 可使用该名字进行bean的获取 class:该bean指向的类 -->
<bean name="person" class="com.hu.pojo.Person">
    <!-- constructor-arg:构造函数的参数 value:值 index:该参数的位置坐标 -->
    <constructor-arg value="hu" index="0"></constructor-arg>
    <constructor-arg value="22" index="1"></constructor-arg>
    <constructor-arg value="男" index="2"></constructor-arg>
</bean>
```

再次执行测试类：

<div align="center">
	<img src="/images/posts/Spring-控制反转(IOC)/constructor-arg注入参数.png" />  
</div> 

​		从这次的结果可以看出，Person对象被成功创建，并且属性已被赋值。而这次使用的构造方法却是Person(String name, int age, char sex) 。其中如果constructor-arg标签设置值的顺序与构造方法中参数的值参数类型顺序一致(即类型匹配)，则可将index属性省略。index属性用于设置该参数在构造方法参数中的位置(0,1...)。

# Spring的依赖注入

​		**类依赖于接口的实现类，该实现类由第三方去注入。**

​		在日常开发中，只是设置这些基本数据类型值并不能满足需求。如一个bean里面包含了另一对象(如另一个bean或集合)，则可通过spring将该对象注入到这个bean中。

创建实体类`CollectionDemo.java`

```java
public class CollectionDemo {

    //属性
	private String name;
	private List<String> list;
	private Set<String> set;
	private Map<String,String> map;
	private Person person;
    
    //与Person类类似创建构造方法
    //Getter()和Setter()
    ...
}
```

在配置文件applicationContext.xml中定义该类的bean。

```xml
<bean name="collectionDemo" class="com.hu.pojo.CollectionDemo">
    <property name="name" value="hu"></property>
    <property name="list">
        <list>
            <value>hu</value>
            <value>liu</value>
        </list>
    </property>
    <property name="set">
        <set>
            <value>xie</value>
            <value>li</value>
        </set>
    </property>
    <property name="map">
        <map>
            <entry key="name" value="hu"></entry>
            <entry key="age" value="22"></entry>
        </map>
    </property>
    <property name="person" ref="person"></property>
</bean>
```

测试：

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext context = 
        new ClassPathXmlApplicationContext("applicationContext.xml");
    //通过context获取Person对象
    CollectionDemo demo = (CollectionDemo) context.getBean("collectionDemo");
    //String
    System.out.println("name:" + demo.getName());
    //List<String>
    System.out.print("list:");
    for (String string : demo.getList()) {
        System.out.print(string + " ");
    }
    //Set<String>
    System.out.print("\nset:");
    for (String string : demo.getSet()) {
        System.out.print(string + " ");
    }
    //Map<String,String>
    System.out.println("\nmap:");
    for (Entry<String, String> entry : demo.getMap().entrySet()) {
        System.out.println("key:" + entry.getKey() + " value:" + entry.getValue());
    }
    //Person
    System.out.println(demo.getPerson());
    context.close();
}
```

运行结果：

<div align="center">
	<img src="/images/posts/Spring-控制反转(IOC)/依赖注入.png" />  
</div> 

​		从运行结果可以看出，CollectionDemo类中的各个属性都已被赋值。它的Persong属性也已被赋值，该属性在配置文件中，通过使用`ref`指向了前面定义的bean-person。即通过`property`标签的`ref`属性，可将别的bean设置到该属性上(即该属性值引用了别的bean)。

# Spring的Bean属性

​		这里贴一篇大佬的博客： [Spring配置文件解析--bean属性](https://www.cnblogs.com/wzz1020/p/4753067.html)，以下内容主要参考该博客。

```xml
<bean

id="beanId"（1）

name="beanName"（2）

class="beanClass"（3）

parent="parentBean"（4）

abstract="true | false"（5）

singleton="true | false"（6）

lazy-init="true | false | default"（7）

autowire="no | byName | byType | constructor | autodetect | default"（8）

dependency-check = "none | objects | simple | all | default"（9）

depends-on="dependsOnBean"（10）

init-method="method"（11）

destroy-method="method"（12）

factory-method="method"（13）

factory-bean="bean">（14）

</bean>
```



（1）`id`: Bean的**唯一**标识名。它必须是合法的XML ID，在整个XML文档中唯一。

（2）`name`: 用来为id创建**一个或多个**别名。它可以是任意的字母符合。多个别名之间用逗号或空格分开。

（3）`class`: 用来定义类的全限定名（包名＋类名）。只有子类Bean不用定义该属性。

（4）`parent`: 子类Bean定义它所引用它的父类Bean。这时前面的class属性失效。子类Bean会继承父类Bean的所有属性，子类Bean也可以覆盖父类Bean的属性。注意：子类Bean和父类Bean是同一个Java类。

（5）`abstract`（默认为`false`）：用来定义Bean是否为抽象Bean。它表示这个Bean将不会被实例化，一般用于父类Bean，因为父类Bean主要是供子类Bean继承使用。

（6）`singleton`（默认为`true`）：定义Bean是否是Singleton（单例）。如果设为“true”,则在BeanFactory作用范围内，只维护此Bean的一个实例。如果设为“flase”，Bean将是`Prototype`（原型）状态，BeanFactory将为每次Bean请求创建一个新的Bean实例。

（7）`lazy-init`（默认为`default`）：用来定义这个Bean是否实现懒初始化。如果为“true”，它将在BeanFactory启动时初始化所有的Singleton Bean。反之，如果为“false”,它只在Bean请求时才开始创建Singleton Bean。

（8）`autowire`（自动装配，默认为`default`）：它定义了Bean的自动装载方式。

​		a.`no`:不使用自动装配功能。

​		b.`byName`:通过Bean的属性名实现自动装配。

​		c.`byType`:通过Bean的类型实现自动装配。

​		d.`constructor`:类似于byType，但它是用于构造函数的参数的自动组装。

​		e.`autodetect`:通过Bean类的反省机制（introspection）决定是使用“constructor”还是使用“byType”。

（9）`dependency-check`（依赖检查，默认为`default`）：它用来确保Bean组件通过JavaBean描述的所以依赖关系都得到满足。在与自动装配功能一起使用时，它特别有用。

​		a. `none`：不进行依赖检查。

​		b. `objects`：只做对象间依赖的检查。

​		c. `simple`：只做原始类型和String类型依赖的检查

​		d. `all`：对所有类型的依赖进行检查。它包括了前面的objects和simple。

（10）`depends-on`（依赖对象）：这个Bean在初始化时依赖的对象，这个对象会在这个Bean初始化之前创建。

（11）`init-method`:用来定义Bean的初始化方法，它会在Bean组装之后调用。它必须是一个无参数的方法。

（12）`destroy-method`：用来定义Bean的销毁方法，它在BeanFactory关闭时调用。同样，它也必须是一个无参数的方法。它只能应用于singleton Bean。

（13）`factory-method`：定义创建该Bean对象的工厂方法。它用于下面的`factory-bean`，表示这个Bean是通过工厂方法创建。此时，`class`属性失效。

（14）`factory-bean`:定义创建该Bean对象的工厂类。如果使用了`factory-bean`则`class`属性失效。

------

下面列出`<ref>`元素的所有可用的指定方式：

- `bean`：可以在当前文件中查找依赖对象，也可以在应用上下文（ApplicationContext）中查找其它配置文件的对象。
- `local`：只在当前文件中查找依赖对象。这个属性是一个XML IDREF，所以它指定的对象必须存在，否则它的验证检查会报错。
- `external`：在其它文件中查找依赖对象，而不在当前文件中查找。

总的来说，`<ref bean="..."/>`和`<ref local="..."/>`大部分的时候可以通用。“bean”是最灵活的方式，它允许你在多个文件之间共享Bean。而“local”则提供了便利的XML验证。

------

如何使用spring的作用域：

```xml
<bean id="person" class="com.hu.pojo.Person" scope="singleton"/>
```

​		这里的`scope`就是用来配置spring bean的作用域，它标识bean的作用域。在spring2.0之前bean只有2种作用域即：`singleton`(单例)、`non-singleton`（也称 `prototype`）, Spring2.0以后，增加了`session`、`request`、`global session`三种专用于Web应用程序上下文的Bean。因此，默认情况下Spring2.0现在有五种类型的Bean。当然，Spring2.0对 Bean的类型的设计进行了重构，并设计出灵活的Bean类型支持，理论上可以有无数多种类型的Bean，用户可以根据自己的需要，增加新的Bean类 型，满足实际应用需求。

1.`singleton`

```xml
<bean id="person" class="com.hu.pojo.Person" scope="singleton"/>
<!--或者-->
<bean id="person" class="com.hu.pojo.Person" singleton="true"/>
```

​		当一个bean的作用域设置为`singleton`, 那么Spring IOC容器中**只会存在一个共享的bean实例**，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的**同一实例**。换言之，当把 一个bean定义设置为`singleton`作用域时，Spring IOC容器只会创建该bean定义的唯一实例。这个单一实例会被存储到单例缓存（singleton cache）中，并且所有针对该bean的后续请求和引用都将返回被缓存的对象实例，这里要注意的是`singleton`作用域和GOF设计模式中的单例是完全不同的，单例设计模式表示一个ClassLoader中 只有一个class存在，而这里的`singleton`则表示一个容器对应一个bean，也就是说当一个bean被标识为`singleton`时 候，spring的IOC容器中只会存在一个该bean。

2.`prototype`

```xml
<bean id="person" class="com.hu.pojo.Person" scope="prototype"/>
<!--或者-->
<bean id="person" class="com.hu.pojo.Person" singleton="false"/>
```

​		`prototype`作用域部署的bean，每一次请求（将其注入到另一个bean中，或者以程序的方式调用容器的`getBean()`方法）**都会产生一个新的bean实例，相当与一个new的操作**，对于`prototype`作用域的bean，有一点非常重要，那就是Spring不能对一个prototype bean的整个生命周期负责，容器在初始化、配置、装饰或者是装配完一个`prototype`实例后，将它交给客户端，随后就对该`prototype`实例不闻不问了。不管何种作用域，容器都会调用所有对象的初始化生命周期回调方法，而对`prototype`而言，任何配置好的析构生命周期回调方法都将不会被调用。 清除`prototype`作用域的对象并释放任何prototype bean所持有的昂贵资源，都是客户端代码的职责。（让Spring容器释放被`singleton`作用域bean占用资源的一种可行方式是，通过使用 bean的后置处理器，该处理器持有要被清除的bean的引用。）

3.`request`

​		`request`表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效。

`request`、`session`、`global session`使用的时候首先要在初始化web的web.xml中做如下配置：

如果你使用的是Servlet 2.4及以上的web容器，那么你仅需要在web应用的XML声明文件web.xml中增加下述ContextListener即可：

```xml
<web-app>
   ...
  <listener>
<listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
  </listener>
   ...
</web-app>
```

如果是Servlet2.4以前的web容器,那么你要使用一个javax.servlet.Filter的实现：

```xml
<web-app>
 ..
 <filter>
    <filter-name>requestContextFilter</filter-name>
    <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
 </filter>
 <filter-mapping>
    <filter-name>requestContextFilter</filter-name>
    <url-pattern>/*</url-pattern>
 </filter-mapping>
   ...
</web-app>
```

接着既可以配置bean的作用域了：

```xml
<bean id="person" class="com.hu.pojo.Person" scope="request"/>
```

4.`session`

```xml
<bean id="person" class="com.hu.pojo.Person" scope="session"/>
```

​		`session`作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效。和`request`配置实例的前提一样，需要配置好web启动文件。

5.`global session`

```xml
<bean id="person" class="com.hu.pojo.Person" scope="global session"/>
```

​		`global session`作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个 portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。如果你在web中使用global session作用域来标识bean，那么web会自动当成session类型来使用。

​		和`request`配置实例的前提一样，需要配置好web启动文件。

6.自定义bean装配作用域

​		在spring2.0中作用域是可以任意扩展的，你可以自定义作用域，甚至你也可以重新定义已有的作用域（但是你不能覆盖`singleton`和 `prototype`），spring的作用域由接口org.springframework.beans.factory.config.Scope来定 义，自定义自己的作用域只要实现该接口即可。此次暂不做过多介绍。

