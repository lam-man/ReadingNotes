# Spring Core: The IoC Container

[TOC]

IoC Container: Inversion of Control container. 

## 1. Introduction to the Spring IoC Container and Beans 

IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) **only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.** The container then injects those dependencies when it creates the bean. his process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Frameworks Ioc container. The `BeanFactory` interface provides an advanced configuration mechanism capable of managing any type of object. `ApplicationContext` is a sub-interface of `BeanFactory`. 

In short, the `BeanFactory` provides the configuration framework and basic functionality, and the `ApplicationContext` adds more enterprise-specific functionality. The `ApplicationContext` is a complete superset of the `BeanFactory` and is used exclusively in tis chapter in descriptions of Spring's IoC container. 

### 1.1 Definition of Bean

A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. **Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.**



## 2. Container Overview 

The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. **The configuration metadata is represented in XML, Java annotations, or Java code.** It lets you express the objects that compose your application and the rich interdependencies between those objects.

The following diagram shows a high-level view of how Spring works. Your application classes are combined with configuration metadata so that, after the `ApplicationContext` is created and initialized, you have a fully configured and executable system or application.

![container-magic](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/container-magic.png)

### 2.1 Configuration MetaData

**What is configuration MetaData**

As the preceding diagram shows, the Spring IoC container consumes a form of configuration metadata. This configuration metadata represents how you, as an application developer, tell the Spring container to instantiate, configure, and assemble the objects in your application.

This chapter will use **XML format configration metadata** to convery key concepts and features of the Spring IoC container. 

Spring configuration consists of at least one and typically more than one bean definition that the container must manage. XML-based configuration metadata configures these beans as `<bean/>` elements inside a top-level `<beans/>` element. Java configuration typically uses `@Bean`-annotated methods within a `@Configuration` class.

The following example shows the basic structure of XML-based configuration metadata:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanOne" class="x.y.ThingOne">   
        <!-- collaborators and configuration for this bean go here -->
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo">
        <!-- collaborators and configuration for this bean go here -->
    </bean>
  
    <bean id="beanThree" class="x.y.ThingThree"/>
    <!-- more bean definitions go here -->

</beans>
```



## 2.2 Intantiating a Container 











































