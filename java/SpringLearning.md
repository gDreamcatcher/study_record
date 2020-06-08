# 概述

[TOC]

## Spring Framework

Spring 框架是一个Java平台，它为开发Java应用程序提供全面的基础架构支持。Spring负责基础架构，因此您可以专注于应用程序的开发。简单来讲就是简化开发。具体实现方式是Spring框架**控制反转IOC组件**通过提供一系列的标准化方法把完全不同的组件组合起来， 及**依赖注入(Dependency Injection)**。
### 模块
Spring框架的功能被有组织的分散到20个模块中，这些模块分布在核心容器、数据访问集成，web，AOP，植入(instrumentation)， 消息传输和测试中，如下图所示：
![img](D:\study_record\java\images\spring-overview.png)

#### 核心容器
核心容器是由core，bean，context和expression(Spring表达式)模块组成的， 其中**core**和**bean**提供了框架的基础功能，**context**提供了一个框架式的对象访问方式，类似于JNDI注册表。多用于资源集合，事件传播，资源负载并且创建上下文，告诉缓存和调度。**Spring-expression**模块提供了强大的表达式语言去支持设置和获取属性值。

#### AOP和Instrumentation
**Spring-AOP**模块允许定义方法拦截器和切入点从而实现解耦；
**Spring-Instrumentation**提供了类植入的支持和类加载器的实现，可以用于特定的应用服务器中，例如spring-instrument-tomcat 模块包含了支持Tomcat的植入代理。

#### 消息
**spring-messaging**(消息传递模块)，其中包含来自Spring Integration的项目，例如，Message，MessageChannel，MessageHandler，和其他用来传输消息的基础应用。该模块还包括一组用于将消息映射到方法的注释(annotations)，类似于基于Spring MVC注释的编程模型。

#### 数据访问集成
数据访问/集成层由**JDBC，ORM，OXM，JMS**和**事务模块**组成。
**spring-jdbc**模块提供了一个JDBC –抽象层，消除了需要的繁琐的JDBC编码和数据库厂商特有的错误代码解析。
**spring-tx**模块支持用于实现特殊接口和所有POJO（普通Java对象）的类的编程和声明式事务管理。
**spring-orm**模块为流行的对象关系映射(object-relational mapping )API提供集成层，包括JPA和Hibernate。使用spring-orm模块，您可以将这些O / R映射框架与Spring提供的所有其他功能结合使用，例如前面提到的简单声明性事务管理功能。
**spring-oxm**模块提供了一个支持对象/ XML映射实现的抽象层，如JAXB，Castor，JiBX和XStream。
**spring-jms**模块(Java Messaging Service) 包含用于生产和消费消息的功能。自Spring Framework 4.1以来，它提供了与 spring-messaging模块的集成。

#### Web
Web层由spring-web，spring-webmvc和spring-websocket 模块组成。
spring-web模块提供基本的面向Web的集成功能，例如多部分文件上传功能，以及初始化一个使用了Servlet侦听器和面向Web的应用程序上下文的IoC容器。它还包含一个HTTP客户端和Spring的远程支持的Web相关部分。
spring-webmvc模块（也称为Web-Servlet模块）包含用于Web应用程序的Spring的模型-视图-控制器(MVC)和REST Web Services实现。 Spring的MVC框架提供了领域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成。

#### Test
spring-test模块支持使用JUnit或TestNG对Spring组件进行单元测试和 集成测试。它提供了Spring ApplicationContexts的一致加载和这些上下文的缓存。它还提供可用于独立测试代码的模仿(mock)对象。

#### Executable jars and Java
为了生成可执行的jar包，需要在pom文件中添加spring-boot-maven-plugin插件，示例：
```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```