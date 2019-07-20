# Spring Tutorial

This tutorial will cover all basic concepts of Spring and all tutorials here will consist of full hands-on practices. Platform used is Windows 10, with all *latest versions* of *Eclipse*, *Java* & *Maven* installed. Complete source for all tutorials is given at the end of the pages as a link to the GitHub repository. Finally, for the sake of brevity, standard auto generated getter and setter methods are ommitted in below code snippets.

Begin by creating a new Maven Java project in Eclipse as *spring-tutorial*.

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

	// getters and setters
}
```

Finally, create a new class for testing as **TestCode.java**. Here these things will happen:

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

You shall get output like this.

```
1, APIM, 500
```

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

## Custom Bean Initialisation

Spring allows you to fine tune an object creation by providing a callback method. This is simply a hook for developers to do some customisation once the bean is instantiated by the container. To do such, XML configuration requires the attribute **init-method** and the bean class shall have the corresponding method definition. Below modification of previous XML file and a new bean class viz. *CustomerCBI.java* will illustrate this.

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

	// getters and setters
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

```
Custom Bean Initialisation...
10, Test User
```

## Bean Inheritance

By XML definition we can make one bean inherited from another bean. This is not exactly Java inheritance although the core concept remains same. Child bean will have all the properties from parent bean and can also override them. Following example will show this. Start by creating a new spring context configuration as *spring-context-2.xml*.

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

	<bean id="spCustomer" class="apim.github.tutorial.SpecialCustomer" parent="customer">
		<property name="balance" value="2000" />
		<property name="loan" value="1000" />
	</bean>
</beans>
```

Customer class remains same and a new *SpecialCustomer.java* is defined.

```java
package apim.github.tutorial;

public class SpecialCustomer {

	private int id;
	private String name;
	private int balance;
	private int loan;

	// getters and setters
}
```

Write a test method to verify this.

```java
private static void test4() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-2.xml");
	Customer c = (Customer) ctx.getBean("customer");
	SpecialCustomer spc = (SpecialCustomer) ctx.getBean("spCustomer");
	System.out.println(c.getId() + ", " + c.getName() + ", " + c.getBalance());
	System.out.println(spc.getId() + ", " + spc.getName() + ", " + spc.getBalance() + ", " + spc.getLoan());
	ctx.close();
}
```

This shall print like this.

```
1, APIM, 500
1, APIM, 2000, 1000
```

## Dependency Injection

Probably the most popular feature of Spring is DI / IoC. In spring context it means bean A shall not instantiate bean B rather Spring shall instantiate both bean A & bean B and then pass bean B to bean A. Now these dependencies can be injected in two ways: via *Constructor* and via *Setter Method*.

At first, let's explore dependency injection by constructor. Create a new bean definition for *Address* and make a new POJO class for the same (prepare a new spring context as **spring-context-3.xml**). Make a new customerDI bean there to accept Address being injected via constructor - using **constructor-arg** parameter. Develop a new constructor and new address field at *CustomerDI.java*, copying the rest from Customer.java file. Finally, develop a new test method to print the complete result.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="customer" class="apim.github.tutorial.CustomerDI">
		<property name="id" value="1" />
		<property name="name" value="APIM" />
		<property name="balance" value="500" />
		<constructor-arg ref="address" />
	</bean>

	<bean id="address" class="apim.github.tutorial.Address">
		<property name="location" value="Torre de Alba" />
		<property name="city" value="Panama City" />
	</bean>
</beans>
```

```java
package apim.github.tutorial;

public class Address {

	private String location;
	private String city;

	// getters and setters
}
```

```java
package apim.github.tutorial;

public class CustomerDI {

	private int id;
	private String name;
	private int balance;
	private Address address;

	public CustomerDI(Address address) {
		this.address = address;
	}

	public void setAddress(Address address) {
		this.address = address;
	}

	// getters and setters
}
```

```java
private static void test5() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-3.xml");
	CustomerDI c = (CustomerDI) ctx.getBean("customer");
	System.out.println(c.getId() + ", " + c.getName() + ", " + c.getBalance());
	System.out.println(c.getAddress().getLocation() + ", " + c.getAddress().getCity());
	ctx.close();
}
```

Output shall look like this.

```
1, APIM, 500
Torre de Alba, Panama City
```

There can be cases of **ambiguity** when there are *multiple constructor arguments*. To counter that, Spring provides more than one solution.

* Ordering of parameters 

```xml
<bean id="customerDI" class="apim.github.tutorial.CustomerDI">
	<constructor-arg ref="address1"/>
	<constructor-arg ref="address2"/>
</bean>
```

* Defining type of parameters 

```xml
<bean id="customerDI" class="apim.github.tutorial.CustomerDI">
	<constructor-arg type="Integer" value="75"/>
	<constructor-arg type="String" value="ABCXYZ"/>
</bean>
```

* Setting the index of parameters 

```xml
<bean id="customerDI" class="apim.github.tutorial.CustomerDI">
	<constructor-arg index="0" ref="address1"/>
	<constructor-arg index="1" ref="address2"/>
</bean>
```

Secondly, there is the case of dependency injection by setter methods. Here the process is straightforward and dependency is just like another bean property. If the dependency is on another bean/object then **ref** attribute is used where if it is on some primitive data type then **value** attribute is used. The following sample bean definition will illustrate this.

```xml
<bean id="customer" class="apim.github.tutorial.CustomerDI">
	<property name="id" value="1" />
	<property name="name" value="APIM" />
	<property name="balance" value="500" />
	<property name="address" ref="address"/>
</bean>
```

## Injecting Java Collection

Spring provides 4 options to work with collections viz. *list*, *set*, *map* and *properties*. Except *properties*, all of them accept both *value* (i.e. collection containing primitives) and *ref* arguments (containing objects/other beans). Below example will elaborate the behavior. At first, let's prepare a new context XML **spring-context-4.xml** which will consist of all supported types.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="dissertation" class="apim.github.tutorial.Dissertation">
		<property name="mentorList">
			<list>
				<value>Niambh Connar</value>
				<ref bean="designation" />
			</list>
		</property>
		<property name="approverSet">
			<set>
				<value>Joe Walsh</value>
				<ref bean="designation" />
			</set>
		</property>
		<property name="referenceMap">
			<map>
				<entry key="book1" value="Let Us C"></entry>
				<entry key="book2" value-ref="book"></entry>
			</map>
		</property>
		<property name="chapterProperties">
			<props>
				<prop key="part1">Introduction</prop>
			</props>
		</property>
	</bean>

	<bean id="designation" class="apim.github.tutorial.Designation">
		<property name="post" value="Professor" />
		<property name="unit" value="Computer Science" />
	</bean>

	<bean id="book" class="apim.github.tutorial.Book">
		<property name="isbn" value="123" />
		<property name="title" value="Mastering Programming" />
		<property name="author" value="Oscar Martin" />
	</bean>
</beans>
```

Next create the associated Java POJO classes viz. *Dissertation.java*, *Designation.java* & *Book.java*. Few of them have *toString()* overridden just for easy testing through console.

```java
package apim.github.tutorial;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

@SuppressWarnings("rawtypes")
public class Dissertation {

	private List mentorList;
	private Set approverSet;
	private Map referenceMap;
	private Properties chapterProperties;

	// getters and setters
}
```

```java
package apim.github.tutorial;

public class Designation {

	private String post;
	private String unit;

	// getters and setters

	public String toString() {
		return post + ": " + unit;
	}
}
```

```java
package apim.github.tutorial;

public class Book {

	private int isbn;
	private String title;
	private String author;

	// getters and setters

	public String toString() {
		return isbn + "-" + title + "-" + author;
	}
}
```

Write a new test method to validate the concept.

```java
private static void test6() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-4.xml");
	Dissertation obj = (Dissertation) ctx.getBean("dissertation");
	System.out.println(obj.getMentorList());
	System.out.println(obj.getApproverSet());
	System.out.println(obj.getReferenceMap());
	System.out.println(obj.getChapterProperties());
	ctx.close();
}
```

This shall print like below.

```
[Niambh Connar, Professor: Computer Science]
[Joe Walsh, Professor: Computer Science]
{book1=Let Us C, book2=123-Mastering Programming-Oscar Martin}
{part1=Introduction}
```

## Bean Autowiring

Spring manages dependency injection by constructor-arg and property elements. However, Spring also has *autowiring* feature which can do this job automatically and thus it reduces a lot of XML configuration. There are in general 5 types of autowiring and a brief overview of them is listed below.

1. **no**: default behavior is to have no automatic bean wiring
2. **byName**: if some bean is defined with autowire as byName, then Spring tries matching and autowiring of bean property by the names
3. **byType**: if some bean is defined with autowire as byType, then Spring tries matching and autowiring of bean property by the data types
4. **constructor**: same as byType but matching is done for constructor arguments
5. **auto**: here Spring at first attempts autowiring by constructor, if that fails then by byType

There will be *runtime exception* when byName / byType / constructor autowiring processes are used and there is not exactly one match found.

To test the *byName* autowiring, create a new bean configuration **spring-context-5.xml** like below.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="customer" class="apim.github.tutorial.CustomerDI" autowire="byName">
		<property name="id" value="1" />
		<property name="name" value="APIM" />
		<property name="balance" value="500" />
	</bean>

	<bean id="address" class="apim.github.tutorial.Address">
		<property name="location" value="City Center" />
		<property name="city" value="Athlone" />
	</bean>
</beans>
```

*CustomerDI.java* has one field as address hence Spring will automatically look for a bean named address. As Spring creates object using default constructor only, hence CustomerDI.java needs to be updated with a default constructor. Next, use the previously created `test5()` method to verify the functionality, only by changing the xml file name being loaded. If the second bean definition is not named as *address* then there will be **NullPointerException** from the container.

Now to test the *byType* autowiring simply just change the `byName` text to `byType` in the XML file. Here Spring will look for a bean whose type/class attribute is Address. In case there are multiple such addresses defined, **NoUniqueBeanDefinitionException** exception will be thrown. To test autowiring by *constructor*, at first update the CustomerDI.java to have a new (now third actually) constructor like below.

```java
public CustomerDI(int id, String name, int balance, Address address) {
	this.id = id;
	this.name = name;
	this.balance = balance;
	this.address = address;
}
```

Then update the bean configuration file to have a new bean definition. Here the index attribute is used because the constructor is multi-valued. To test this, update the earlier `test5()` method to use latest bean id `customerAW`. Testing shall produce output matched with earlier example.

```xml
<bean id="customerAW" class="apim.github.tutorial.CustomerDI" autowire="constructor">
	<constructor-arg index="0" value="1" />
	<constructor-arg index="1" value="APIM" />
	<constructor-arg index="2" value="500" />
</bean>
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-tutorial)