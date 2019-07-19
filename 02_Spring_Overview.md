# Spring Overview

Spring is an open source Jakarta EE container framework and a popular alternative to legacy EJB framework. Spring has many added advantages over EJB like Dependency Injection and Aspect Oriented Programming etc. The only slight disadvantage it has from EJB that Spring does not support distributed transaction well, which nontheless, is getting itself out of fashion from modern architecture which concentrates more on Micro Services.

## Characteristics of Spring

1. **Module based** - Spring is essentially a collection of modules. Hence you use only what you need. E.g., if you need basic spring core, use that module only - no need to mention any separate modules like spring-aop, spring-mvc etc.
2. **Lightweight** - Very small library size, initial distribution ranged about 2 MB only.
3. **POJO based** - Spring is all of POJO, hence good old easy Java coding.
4. **Non-intrusive** - Application code does not need to be dependent on Spring specific classes.
5. **Dependency Injection** - a.k.a. Inversion of Control (IoC), is one of the best proven way of reducing coupling. This means, simply, if object X is composed of object Y - then X does not construct Y rather someone else gives constructed Y to X.
6. **Aspect Oriented Programming (AOP)** - A controversial feature, few till opposes it as it does not obey the virtue of OOPS but once mastered - it brings a huge advantage in development. The idea of AOP is to separate *cross-cutting concerns*. E.g., logging, transaction etc. To achieve this, say logging using AOP, one simply skips adding log statements like method entry and method exit within application code; rather uses one defined annotation and Spring framework automatically does the logging as and when the method is invoked.
7. **Easy to integrate** - Unlike EJB, other standard third party software like Hibernate, Quartz etc are very easy to integrate with Spring.
8. **Quick testable** - One of the pioneering reasons of Spring's popularity - code written in Spring, as they are POJO based, are quickly deployable and testable.
9. **Reduces boiler-plate code** - A very much disliked point in EJB, Spring gives the remedy as blessing that it requires extremely less boiler plate codes.

> If an object depends on having an instance of another object, the needed object is injected into dependent object automatically.
> - Martin Fowler's definition of Dependency Injection

## Spring Framework

![](/images/spring_framework.jpg)

Spring contains about 20+ modules. They are kind of grouped in 4 parts - Core Container, Data Access / Integration, Web and AOP / Instrumentation.

* **Core Container:**
  * Core - Dependency Injection
  * Beans - BeanFactory
  * Context - ApplicationContext
  * Expression Language - Spring EL like Unified EL (JSP 2.x)
* **Data Access / Integration:**
  * JDBC - Support for easy writing of JDBC code
  * ORM - Support for almost all popular ORM tools
  * OXM - Support for object-xml mapping with tools like JAXB/Jackson
  * JMS - Support for J2EE JMS specification
  * Transaction - Support for both Declarative and Programmatic transaction management
* **Web:**
  * Web - IoC container with servlet listener
  * Servlet - Spring MVC framework
  * Struts - Support for Struts 1.x (deprecated as of Spring 3.x.x)
  * Portlet - Support for MVC in portlet environment
* **AOP / Instrumentation:**
  * AOP - Support for Aspects
  * Instrumentation - Support for class instrumentation and class loader