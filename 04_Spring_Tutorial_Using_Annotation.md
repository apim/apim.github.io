# Spring Tutorial - Using Annotation

This tutorial will practice all examples form the earlier tutorial, however, this time it will use **annotation** instead of XML context configuration / bean definition. Spring version 2.5 onward all bean configurations can be done with the aid of annotation and the whole XML work has become redundant. Annotation mechanism in Spring container is not enabled by default hence to do that the a line like this has to be added at the top of any context configuration xml - `<context:component-scan base-package="apim.github.tutorial" />`.

Here **base-package** is the package where we are instructing Spring to look for beans. This parameter is multivalued as several packages can be mentioned using comma. It is to be noted that Spring supports at the same time both annotation based and XML based configurations and the latter overrides the earlier one. There are many annotations defined at Spring and also there are numerous annotations at JDK which are used while developing Spring based applications. For this tutorial, we will discuss the below annotations.

1. **`@Component`** - A generic annotation to define Spring beans
2. **`@Service`** - Specialised case of @Component for the use case of presentation layer
3. **`@Repository`** - Specialised case of @Component for the use case of persistence layer
4. **`@Controller`** - Specialised case of @Component for the use case of service layer
5. **`@Autowired`** - As name suggests, for bean autowiring / dependency injection, works as byType
6. **`@Qualifier`** - To resolve ambiguity while using @Autowired
7. **`@Resource`** - Another way of dependency injection, works as byName
8. **`@Scope`** - To define the scope of a bean

There is another annotation `@Required` which works similar to @Autowired, but functionality is restricted only to setter methods while @Autowired can also work for constructors and fields. @Required checks if a particular property has been set or not and in failure of which Spring throws *BeanInitializationException*. However, @Autowired works basically like @Required unless it is explicitly set to false like *@Autowired(required=false)*.

To test annotations in action, eclipse project developed in the earlier tutorial will be re-used. Create a new context configuration XML there like below.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="apim.github.tutorial" />
</beans>
```

Create a new POJO with Spring Annotation like this.

```java
package apim.github.tutorial;

import org.springframework.stereotype.Component;

@Component
public class CustomerAnnot {

	private int id;
	private String name;
	private int balance;

	// getters and setters
}
```

Create a new test method like below. Execution shall print blank values which tells that Spring has been successfully able to find and instantiate the bean. As expected, it's values are not set. In ideal/production cases that should come from some datasource. Note that Spring creates bean names following *camelCase* standard (CustomerAnnot -> customerAnnot).

```java
private static void test7() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-6.xml");
	CustomerAnnot c = (CustomerAnnot) ctx.getBean("customerAnnot");
	System.out.println(c.getId() + ", " + c.getName() + ", " + c.getBalance());
	ctx.close();
}
```

```
0, null, 0
```

Now these 3 annotations - *@Service*, *@Repository* and *@Controller* are just special cases for *@Component* hence by characteristics they are all same. Next comes the useful *@Autowired* by which Spring dependency injection happens in annotation. To verify this, create *AddressAnot* class likewise *CustomerAnnot* and then add @Autowired to either 1) a new address field at CustomerAnnot.java, 2) the address setter method there or 3) a single argument CustomerAnnot constructor. Now test all combinations with the below `test8()` method to see how Spring discovers the wiring of properties and instantiates & injects them.

```java
package apim.github.tutorial;

import org.springframework.stereotype.Component;

@Component
public class AddressAnnot {

	private String location;
	private String city;

	// getters and setters
}
```

```java
package apim.github.tutorial;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CustomerAnnot {

	@Autowired // either this
	private AddressAnnot address;

	@Autowired // or this
	public CustomerAnnot(AddressAnnot address) {
		this.address = address;
	}

	@Autowired // or this
	public void setAddress(AddressAnnot address) {
		this.address = address;
	}
}
```

```java
private static void test8() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-6.xml");
	CustomerAnnot c = (CustomerAnnot) ctx.getBean("customerAnnot");
	System.out.println(c.getId() + ", " + c.getName() + ", " + c.getBalance());
	System.out.println(c.getAddress().getLocation() + ", " + c.getAddress().getCity());
	ctx.close();
}
```

These will show that the bean initialisation is done properly but obviously values are not set as we were not doing anything such via the XML.

```
0, null, 0
null, null
```

To illustrate the next annotation *Qualifier*, below sample XML shows creation of 2 beans with distinct id values. As @Autowired works by default as byType autowiring, @Qualified is used to resolve conflicts when there are multiple matches of same type of bean. For the associated Java code, only change required is the `@Qualifier(value = "address1")` line.

```xml	
<bean id="address1" class="apim.github.tutorial.AddressAnnot">
	<property name="location" value="Torre de Alba" />
	<property name="city" value="Panama City" />
</bean>

<bean id="address2" class="apim.github.tutorial.AddressAnnot">
	<property name="location" value="City Center" />
	<property name="city" value="Athlone" />
</bean>
```

```java
@Autowired
@Qualifier(value = "address1")
private AddressAnnot address;

// rest remains same
}
```

For custom bean initialisation, likewise *init-method*, counterpart annotation is **PostConstruct**. However, although Spring framework makes use of this annotation, it is not defined within Spring rather defined within **common-annotations** package. Therefore, to test it at first add the below Maven dependency.

```xml
<dependency>
	<groupId>javax.annotation</groupId>
	<artifactId>javax.annotation-api</artifactId>
	<version>1.3.2</version>
</dependency>
```

Then create a POJO **CustomerPC.java** by copying the old *Customer.java* and adding the below snippet there.

```java
package apim.github.tutorial;

import javax.annotation.PostConstruct;

import org.springframework.stereotype.Component;

@Component // add this
public class CustomerPC {

	private int id;
	private String name;
	private int balance;

	@PostConstruct // add this
	public void customInit() {
		System.out.println("Custom Bean Initialisation via annotation...");
	}
	
	// rest remains same
}
```

To test, create a new test method as below as you shall see the above print statement.

```java
private static void test9() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context-6.xml");
	ctx.getBean("customerPC");
	ctx.close();
}
```