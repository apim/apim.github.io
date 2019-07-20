# Spring Tutorial

This tutorial will cover all basic concepts of Spring and all tutorials here will consist of full hands-on practices. Platform used is Windows 10, with all *latest versions* of *Eclipse*, *Java* & *Maven* installed. Complete source for all tutorials is given at the end of the pages as a link to the GitHub repository. Begin by creating a new Maven Java project in Eclipse as *spring-tutorial*.

## First Example

Add a pom file like below.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring Tutorial</name>

	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>5.1.8.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>5.1.8.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>5.1.8.RELEASE</version>
		</dependency>
	</dependencies>
</project>
```

Now create the first actual element - a spring bean definition XML file. Name it anything e.g. **spring-context-1.xml**. Using this file Spring instantiates and populates the classes defined within it i.e. the Beans. In brief, it contains bean definitions where each bean corresponds to a POJO. Bean definitions shall have unique Id and FQN as class.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="customer" class="apim.github.tutorial.Customer">
		<property name="id" value="1" />
		<property name="name" value="APIM" />
		<property name="balance" value="500" />
	</bean>
</beans>
```

Next step - create the bean Java file following the package and structure defined above.

```java
package apim.github.tutorial;

public class Customer {

	private int id;
	private String name;
	private int balance;

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getBalance() {
		return balance;
	}

	public void setBalance(int balance) {
		this.balance = balance;
	}

}
```

Finally, create a test class as **TestCode.java**. Here these things will happen:

* The spring context will be loaded from classpath
* The bean will be accessed from the context
* It's values will be printed to show that Spring works!

```java
package apim.github.tutorial;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestCode {

	public static void main(String args[]) {
		test1();
	}

	private static void test1() {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-1.xml");
		Customer c = (Customer) ctx.getBean("customer");
		System.out.println(c.getId() + ", " + c.getName() + ", " + c.getBalance());
		ctx.close();
	}
}
```

You shall get output like
> 1, APIM, 500

Now in this example **ClassPathXmlApplicationContext** has been used as spring container. There is another popular container viz. *FileSystemXmlApplicationContext*. The differences are straightforward - first one allows loading of context definition only from classpath where the second one allows loading of the same from filesystem. There is also a third implementation - *WebXmlApplicationContext*, which is used when Spring framework is working in a web application.

## Scopes for a Bean

When defining a bean, it's scope can be customised. How a bean will be instantiated by the Spring Container - that is governed by this scope parameter. There are 5 possible values for scope:

* **singleton** - one instance of the bean always and this is default scope
* **prototype** - each request results into new objects
* **request** - bean instance is valid for one HTTP request
* **session** - bean instance is valid for one HTTP session
* **global-session** - bean instance is valid for global HTTP session

To understand singleton and prototype scopes, let's do a small test. As singleton is the default scope, without changing bean definition create a new test method as below.

```java
private static void test2() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-1.xml");
	Customer c1 = (Customer) ctx.getBean("customer");
	Customer c2 = (Customer) ctx.getBean("customer");
	System.out.println(c1 == c2);
	ctx.close();
}
```

This shall result in *true* being printed as container will have one object only. Now if bean definition is modified as `<bean id="customer" class="apim.github.tutorial.Customer" scope="prototype">` in the xml file, the result will be *false*. This shows that for prototype scope, container always creates new bean instances.

## Custom Bean Initialization

Spring allows you to fine tune an object creation by providing a callback method. This is simply a hook for developers to do some customisation once the bean is instantiated by the container. To do such, XML configuration requires the attribute **init-method** and the bean class shall have the corresponding method definition. Below modification of previous XML file and a new bean class viz. CustomerCBI.java will illustrate this.

```xml
<bean id="customerCBI" class="apim.github.tutorial.CustomerCBI" init-method="customInit">
	<property name="cId" value="10" />
	<property name="cName" value="Test User" />
</bean>
```

```java
package apim.github.tutorial;

public class CustomerCBI {

	private int cId;
	private String cName;

	public void customInit() {
		System.out.println("Custom Bean Initialisation...");
	}

	public int getcId() {
		return cId;
	}

	public void setcId(int cId) {
		this.cId = cId;
	}

	public String getcName() {
		return cName;
	}

	public void setcName(String cName) {
		this.cName = cName;
	}
}
```

Use the below test method to verify the same.

```java
private static void test3() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-1.xml");
	CustomerCBI c = (CustomerCBI) ctx.getBean("customerCBI");
	System.out.println(c.getcId() + ", " + c.getcName());
	ctx.close();
}
```

> Custom Bean Initialisation...
10, Test User

## Bean Inheritance

By XML definition we can make one bean inherited from another bean. This is not exactly Java inheritance although the core concept remains same. Child bean will have all the properties from parent bean and can also override them. Following example will show this.