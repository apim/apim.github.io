# Spring AOP Tutorial

AOP - Aspect Oriented Programming is one of the biggest gifts of Spring. Aspects are referred as cross-cutting concerns like logging, caching etc. which are essential for every application but not closely related to any business logic. Therefore, support for AOP makes Spring an excellent framework where these aspects / cross-cutting concerns are automatically taken care of at the background. Obviously, there will be some sort of instrumented code which actually implements the aspects, nevertheless, amount of boiler-plate coding is very minimal thus incorporating Spring AOP has become a straightforward job.

Before we dive into some hands-on for AOP, let's clarify few terminologies related to Spring AOP.

* **aspect** - a particular set of code providing AOP functionality in the application
* **advice** - actual piece of code being executed by framework as part of AOP
* **pointcut** - marks the places within application where framework shall invoke AOP

Now there are 5 types of advice supported in Spring. A brief description is listed below.

1. **before** - executes before the pointcut invocation
2. **after** - executes after the pointcut invocation
3. **after-returning** - executes if pointcut returns successfully
4. **after-throwing** - executes if pointcut throws exception
5. **around** - executes before and after pointcut invocation

Begin by creating a new Maven Java project in Eclipse as *spring-aop-tutorial*. You can copy the base code like *pom.xml* from previous tutorial's project. Change artifact details there as shown below - all other dependencies shall remain same and new dependency for aspect shall be added.

```xml
<groupId>apim.github.tutorial</groupId>
<artifactId>spring-aop</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>Spring AOP Tutorial</name>
```

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aop</artifactId>
	<version>5.1.8.RELEASE</version>
</dependency>
```

Next create spring configuration XML. Note that aspect programming requires a new namespace `spring-aop`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
```