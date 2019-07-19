# Spring Tutorial

This tutorial will cover all basic concepts of Spring and all tutorials here will consist of full hands-on practices. Platform used is Windows 10, with all *latest versions* of *Eclipse*, *Java* & *Maven* installed. Complete source for all tutorials is given at the end of the pages as a link to the GitHub repository. Begin by creating a new Maven Java project in Eclipse as spring-tutorial.

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

Now create the first actual element - a spring bean definition XML file. Name it anything e.g. spring-context-1.xml. Using this file Spring instantiates and populates the classes defined within it i.e. the Beans. In brief, it contains bean definitions where each bean corresponds to a POJO. Bean definitions shall have unique Id and FQN as class.

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

Finally, create a test class as TestCode.java. Here these things will happen:

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

Now in this example ApplicationContext has been used as container. There is another container viz. BeanFactory although it is no longer being used in present days' applications as ApplicationContext already has all the features of the other one. There are two main implementations of ApplicationContext: ClassPathXmlApplicationContext and FileSystemXmlApplicationContext. The differences are straightforward - first one allows loading of context definition only from classpath where the second one allows loading of the same from filesystem. There is also a third implementation - WebXmlApplicationContext. This one functions when Spring is used in a web application.