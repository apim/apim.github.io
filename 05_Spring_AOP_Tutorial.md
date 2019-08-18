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

Now create actual bean definitions for spring aspect. Bean **instrumentalist** is the target object against which aspects are performed in this tutorial. Bean **audience** is the actual advice implementation. Tag `<aop:pointcut>` has expression parameter by which we can specify what parts of the application the concerned aspect shall apply to. The example expression points to a specific method `sing()` in the class `Instrumentalist`, however, we can use regular expression here like `(* <package-name>.*.*(..))` to cover all methods in all classes under a package. Tag `<aop:aspect>` defines one actual aspect and its *ref* parameter points to the bean which has the code implementing the aspect. Tag `<aop:advice-type>` specifies what all actions are to be taken. This advice-type can be of any 5 values as mentioned earlier. Moreover, as the example suggests, any of the advices can be used more than once.

```xml
<bean id="instrumentalist" class="apim.github.tutorial.Instrumentalist" />
<bean id="audience" class="apim.github.tutorial.Audience" />

<aop:config>
	<aop:pointcut id="testPointCut" expression="execution(* apim.github.tutorial.Instrumentalist.sing())" />
	<aop:aspect ref="audience">
		<aop:before method="takeSeat" pointcut-ref="testPointCut" />
		<aop:before method="switchOffPhone" pointcut-ref="testPointCut" />
		<aop:after method="lightsOn" pointcut-ref="testPointCut" />
		<aop:after-returning method="applaud" pointcut-ref="testPointCut" />
		<aop:after-throwing method="demandRefund" pointcut-ref="testPointCut" />
	</aop:aspect>
</aop:config>
```

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

Now let's re-implement this using annotation. The main annotation is `@Aspect` and to use this there should be a line added into the bean configuration XML as `<aop:aspectj-autoproxy/>`. All the tags for advices discussed above have almost one-to-one mapped annotations defined e.g. **@Before**, **@AfterReturning** etc. These annotations directly accept the pointcut reference as input like `@After("execution(* apim.github.tutorial.Instrumentalist.sing())")` or `@Before(pointcut="execution(* apim.github.tutorial.Instrumentalist.sing())")`. However, there can be cases where same pointcut expression is used at multiple places. To aid this, pointcuts can also be explicitly defined using **@Pointcut** annotation on an empty method (e.g. `@Pointcut("execution(* apim.github.tutorial.Instrumentalist.sing())") private void pointCutService() {})`.

There are more features to the Spring's AOP than just method interception. For example, it is easy to interpret the values for the method at pointcut. This is doable by adding an argument of **JoinPoint** to the methods implementing the advices. In addition, *@AfterReturning* advice method can have another argument (type may be *Object*) holding the value being returned from the intercepted method and for this within the annotation a second parameter has to be added as `returning=<argument name>`. Similarly, *@AfterThrowing* can have another argument of type *Throwable* holding the exception being thrown and for this also a second parameter is required within the annotation as `throwing=<argument name>`. Lastly, the missing advice from the above example - `around` (annotation form `@Around`) - here instead of *JoinPoint*, the advice implementation accepts `ProceedingJoinPoint`. Main reason for this is that the advice method, once intercepted pointcut method, shall allow continuation of normal operation by invoking `proceed()` and then again the advice can resume its post processing.