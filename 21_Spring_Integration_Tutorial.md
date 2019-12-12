# Spring Integration Tutorial

This tutorial will cover basic concepts of Spring Integration, which were shown in the previous overview tutorial, in a series of hands-on examples. Begin by creating a new Maven Java project in Eclipse as *spring-integration-tutorial*. Our **pom.xml** shall be like below where main new addition is the *spring-integration-core* artifact.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring-integration</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring Integration Tutorial</name>

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
		<dependency>
			<groupId>org.springframework.integration</groupId>
			<artifactId>spring-integration-core</artifactId>
			<version>5.1.8.RELEASE</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

## Transformer

Begin with the Spring context configuration - **spring-context.xml**. This file will gradually grow as the tutorial will bring in more examples in the following sections. In the XML namespace, Spring Integration XSD has been added. 2 messaging channels - one subscribable and the other one pollable, have been configured. Finally, the transformer is defined with the above said input and output channels. The *ref* attribute points to a bean which contains application logic for data translation.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:int="http://www.springframework.org/schema/integration"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/integration http://www.springframework.org/schema/integration/spring-integration.xsd">

	<context:component-scan base-package="apim.github.tutorial" />

	<int:channel id="inputChannel" />
	
	<int:channel id="outputChannel">
		<int:queue capacity="5" />
	</int:channel>

	<int:transformer input-channel="inputChannel" output-channel="outputChannel" ref="dataTranslator" />
	
</beans>
```

Next create a simple POJO as **Person.java** which will be the input to the transformer.

```java
package apim.github.tutorial;

public class Person {

	private String firstName;

	private String lastName;

	private int age;

	// getters and setters

}
```

Next create another POJO **Employee.java** which will be the output from the transformer.

```java
package apim.github.tutorial;

public class Employee {

	private String fullName;

	private LevelEnum band;

	// getters and setters

	public enum LevelEnum {
		HIGH, MEDIUM, LOW
	}

}
```

Now develop the actual transformer implementation **DataTranslator.java**. A method annotated with `@Transformer` does the job.

```java
package apim.github.tutorial;

import org.springframework.integration.annotation.Transformer;
import org.springframework.stereotype.Component;

import apim.github.tutorial.Employee.LevelEnum;

@Component
public class DataTranslator {

	@Transformer
	public Employee convertData(Person p) {
		Employee e = new Employee();
		e.setFullName(p.getFirstName() + " " + p.getLastName());
		if (p.getAge() <= 30) {
			e.setBand(LevelEnum.LOW);
		} else if (p.getAge() > 30 && p.getAge() < 60) {
			e.setBand(LevelEnum.MEDIUM);
		} else {
			e.setBand(LevelEnum.HIGH);
		}
		return e;
	}

}
```

Finally, prepare the test code as **TestCode.java** where there will be multiple methods to demonstrate each spring integration patterns in this tutorial. The test method at first retrieves the input and output channels from context. Then it creates the input object, passes the same as part of payload to the channel and subsequently retrieves the translated output from response channel.

```java
package apim.github.tutorial;

import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.PollableChannel;

public class TestCode {

	public static void main(String args[]) {
		testTransformer();
	}

	private static void testTransformer() {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		MessageChannel channel = (MessageChannel) ctx.getBean("inputChannel");
		PollableChannel outputChannel = (PollableChannel) ctx.getBean("outputChannel");

		Person p = new Person();
		p.setFirstName("Joe");
		p.setLastName("Walsh");
		p.setAge(65);
		Message<Person> msg = MessageBuilder.withPayload(p).build();

		channel.send(msg);

		Message<?> msgRcvd = outputChannel.receive();
		Employee e = (Employee) msgRcvd.getPayload();
		System.out.println("Message received : " + e.getFullName() + ", " + e.getBand());
		ctx.close();
	}
	
}
```