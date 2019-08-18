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

Begin by creating a new Maven Java project in Eclipse as *spring-aop-tutorial*. You can copy the base code like *pom.xml* from previous tutorial's project. Change artifact details there as shown below - all other dependencies shall remain same and new dependency for aspect shall be added. Note that spring aspect functionality depends on core aspect libraries from *aspectj* which are added as followup Maven dependencies.

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
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjrt</artifactId>
	<version>1.9.4</version>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjweaver</artifactId>
	<version>1.9.4</version>
</dependency>
```

Next create spring configuration XML **spring-context**. Note that aspect programming requires a new namespace `spring-aop`.

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

Now create actual bean definitions for spring aspect. Bean **instrumentalist** is the target object against which aspects are performed in this tutorial. Bean **audience** is the actual advice implementation. Tag `<aop:pointcut>` has expression parameter by which we can specify what parts of the application the concerned aspect shall apply to. The example expression points to a specific method `sing()` in the class `Instrumentalist`, however, we can use regular expression here like (* <package-name>.*.*(..)) to cover all methods in all classes under a package. Tag `<aop:aspect>` defines one actual aspect and its *ref* parameter points to the bean which has the code implementing the aspect. Tag `<aop:advice-type>` specifies what all actions are to be taken. This advice-type can be of any 5 values as mentioned earlier. Moreover, as the example suggests, any of the advices can be used more than once.

Next develop the Java bean classes.

```java
package apim.github.tutorial;

public class Instrumentalist {

	public void play() {
		System.out.println("Playing the instrument");
	}

	public void sing() {
		System.out.println("Singing with instrument");
	}
	
}
```

```java
package apim.github.tutorial;

public class Audience {

	public void takeSeat() {
		System.out.println("Taking seats");
	}

	public void switchOffPhone() {
		System.out.println("Switching off mobile phones");
	}

	public void lightsOn() {
		System.out.println("Lights are on for audience");
	}

	public void applaud() {
		System.out.println("Praising");
	}

	public void demandRefund() {
		System.out.println("Demanding refund");
	}

}
```

Finally, define the test class **TestCode.java**. The only thing we test is the invocation of the target pointcut method i.e. *sing()* and how Spring AOP functions for it.

```java
package apim.github.tutorial;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestCode {

	public static void main(String args[]) {
		test1();
	}

	private static void test1() {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		Instrumentalist inst = (Instrumentalist) ctx.getBean("instrumentalist");
		inst.sing();
		ctx.close();
	}

}
```

You now shall see the desired output as below.

```
Taking seats
Switching off mobile phones
Singing with instrument
Lights are on for audience
Praising
```

## AOP using Annotation