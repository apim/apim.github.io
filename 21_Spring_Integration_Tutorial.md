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

## Filter

Just add this line in the context XML, configuration is very much alike to the Transformer.

```xml
<int:filter input-channel="inputChannel" output-channel="outputChannel" ref="dataFilter" />
```

Next make a POJO **Order.java** to use with this example.

```java
package apim.github.tutorial;

public class Order {

	private int id;

	private double amount;

	// getters and setters

	public String toString() {
		return id + "-" + amount;
	}

}
```

Now create the actual filter implementation **DataFilter.java** which will do the job of selective passing of messages. Method annotated with `@Filter` shall return a boolean value to decide whether the input message shall be copied to the output channel.

```java
package apim.github.tutorial;

import org.springframework.integration.annotation.Filter;
import org.springframework.stereotype.Component;

@Component
public class DataFilter {

	@Filter
	public boolean isValidOrder(Order msg) {
		return msg.getAmount() > 1000;
	}

}
```

Then prepare the test method. Notice that here two different Order objects are made and sent through the channel. The first one shall not pass through, therefore, the `receive()` is invoked with a timeout value and for the second one, we are correctly expecting a response.

```java
private static void testFilter() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
	MessageChannel channel = (MessageChannel) ctx.getBean("inputChannel");
	PollableChannel outputChannel = (PollableChannel) ctx.getBean("outputChannel");

	Order o = new Order();
	o.setId(1);
	o.setAmount(800);
	Message<Order> msg = MessageBuilder.withPayload(o).build();

	channel.send(msg);
	Message<?> msgRcvd = outputChannel.receive(1000);
	System.out.println("Message received : " + msgRcvd);

	o.setId(2);
	o.setAmount(1200);
	msg = MessageBuilder.withPayload(o).build();

	channel.send(msg);
	msgRcvd = outputChannel.receive();
	System.out.println("Message received : " + (Order) msgRcvd.getPayload());

	ctx.close();
}
```

## Router

Add below router configurations in the context XML. The alternate channel is added to show that router shall produce output to different messaging channels based on business logic.

```xml
<int:channel id="alternateChannel">
	<int:queue capacity="5" />
</int:channel>

<int:router input-channel="inputChannel" ref="customRouter" />
```

Create the POJO **Item.java** for the example.

```java
package apim.github.tutorial;

public class Item {

	private String name;

	private int dimension;

	// getters and setters

	public String toString() {
		return name + "-" + dimension;
	}

}
```

Now develop the router implementation **CustomRouter.java**. Method annotated with `@Router` shall return a collection of *String* or *MessageChannel*. In case of String, they shall match exactly to the channel names.

```java
package apim.github.tutorial;

import java.util.ArrayList;
import java.util.List;

import org.springframework.integration.annotation.Router;
import org.springframework.stereotype.Component;

@Component
public class CustomRouter {

	@Router
	public List<String> routeMessage(Item msg) {
		List<String> rsp = new ArrayList<>();
		if (msg.getDimension() > 100) {
			rsp.add("alternateChannel");
		} else {
			rsp.add("outputChannel");
			rsp.add("alternateChannel");
		}
		return rsp;
	}

}
```

Lastly, create the test method where at first scenario output is available only in the alternate channel; and in the second case it appears in both the channels.

```java
private static void testRouter() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
	MessageChannel channel = (MessageChannel) ctx.getBean("inputChannel");
	PollableChannel outputChannel = (PollableChannel) ctx.getBean("outputChannel");
	PollableChannel alternateChannel = (PollableChannel) ctx.getBean("alternateChannel");

	Item it = new Item();
	it.setName("item-1");
	it.setDimension(150);
	Message<Item> msg = MessageBuilder.withPayload(it).build();

	channel.send(msg);
	Message<?> msgRcvd1 = outputChannel.receive(1000);
	Message<?> msgRcvd2 = alternateChannel.receive();
	System.out.println("Message received at outputChannel: " + msgRcvd1);
	System.out.println("Message received at alternateChannel: " + (Item) msgRcvd2.getPayload());

	it.setName("item-2");
	it.setDimension(50);
	msg = MessageBuilder.withPayload(it).build();

	channel.send(msg);
	msgRcvd1 = outputChannel.receive();
	msgRcvd2 = alternateChannel.receive();
	System.out.println("Message received at outputChannel: " + (Item) msgRcvd2.getPayload());
	System.out.println("Message received at alternateChannel: " + (Item) msgRcvd2.getPayload());

	ctx.close();
}
```

## Splitter

Add the below line in context XML for splitter configuration.

```xml
<int:splitter input-channel="inputChannel" output-channel="outputChannel" ref="customSplitter" />
```

Create a POJO **Customer.java**.

```java
package apim.github.tutorial;

public class Customer {

	private String name;

	private String address;

	private double balance;

	// getters and setters

}
```

Next create a POJO **Field.java**, which is basically the splitted output from the end-point. This class shall implement `Serializable`.

```java
package apim.github.tutorial;

import java.io.Serializable;

public class Field implements Serializable {

	private static final long serialVersionUID = 1L;

	private FieldDescriptor desc;

	private Object value;
	
	public Field(} {
	}

	public Field(FieldDescriptor desc, Object value) {
		this.desc = desc;
		this.value = value;
	}

	// getters and setters

	public String toString() {
		return desc + "-" + value;
	}

	public enum FieldDescriptor {

		NAME(1), ADDRESS(2), BALANCE(3);

		private int fieldId;

		private FieldDescriptor(int fieldId) {
			this.fieldId = fieldId;
		}

		public int getFieldId() {
			return fieldId;
		}
	}

}
```

Now develop the splitter implementation **CustomSplitter.java**. Method annotated with `@Splitter` shall return a collection of objects.

```java
package apim.github.tutorial;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

import org.springframework.integration.annotation.Splitter;
import org.springframework.stereotype.Component;

import apim.github.tutorial.Field.FieldDescriptor;

@Component
public class CustomSplitter {

	@Splitter
	public Collection<Field> splitMessage(Customer c) {
		List<Field> messages = new ArrayList<>();
		Field field = new Field(FieldDescriptor.NAME, c.getName());
		messages.add(field);
		field = new Field(FieldDescriptor.ADDRESS, c.getAddress());
		messages.add(field);
		field = new Field(FieldDescriptor.BALANCE, c.getBalance());
		messages.add(field);
		return messages;
	}

}
```

Next create the test method. Note that we are receiving output from same response channel multiple times as the single input message has been splitted in that way.

```java
private static void testSplitter() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
	MessageChannel channel = (MessageChannel) ctx.getBean("inputChannel");
	PollableChannel outputChannel = (PollableChannel) ctx.getBean("outputChannel");

	Customer cust = new Customer();
	cust.setName("Robert");
	cust.setAddress("City Quarter Apartments, Athlone");
	cust.setBalance(1350.75);
	Message<Customer> msg = MessageBuilder.withPayload(cust).build();

	channel.send(msg);
	Message<?> msgRcvd1 = outputChannel.receive();
	Message<?> msgRcvd2 = outputChannel.receive();
	Message<?> msgRcvd3 = outputChannel.receive();
	System.out.println("Splitted message at outputChannel: " + msgRcvd1.getPayload());
	System.out.println("Splitted message at outputChannel: " + msgRcvd2.getPayload());
	System.out.println("Splitted message at outputChannel: " + msgRcvd3.getPayload());

	ctx.close();
}
```

## Aggregator

Add the below configuration for aggregator example. For this example to work, the earlier splitter configuration needs to be commented in the context XML.

```xml
<int:channel id="aggregatorInputChannel" />
	
<int:channel id="aggregatorOutputChannel">
	<int:queue capacity="5" />
</int:channel>

<int:splitter input-channel="inputChannel" output-channel="aggregatorInputChannel" ref="customSplitter" />

<int:aggregator input-channel="aggregatorInputChannel" output-channel="aggregatorOutputChannel" ref="customAggregator" />
```

This example uses few classes like *Customer* and *Field* from previous splitter example. Here in the aggregator implementation, **CustomAggregator.java**, a method annotated with `@Aggregator` shall accept a collection of messages or message paylods. Note that how the same *Customer* object is being recreated here which was splitted in the earlier example.

```java
package apim.github.tutorial;

import java.util.Collection;

import org.springframework.integration.annotation.Aggregator;
import org.springframework.stereotype.Component;

import apim.github.tutorial.Field.FieldDescriptor;

@Component
public class CustomAggregator {

	@Aggregator
	public Customer aggregateMessages(Collection<Field> fields) {
		Customer c = new Customer();
		for (Field field : fields) {
			if (field.getDesc() == FieldDescriptor.NAME) {
				c.setName(field.getValue().toString());
			}
			if (field.getDesc() == FieldDescriptor.ADDRESS) {
				c.setAddress(field.getValue().toString());
			}
			if (field.getDesc() == FieldDescriptor.BALANCE) {
				c.setBalance(Double.valueOf(field.getValue().toString()));
			}
		}
		return c;
	}

}
```

Lastly, create the final test method. Here the *Customer* object is first sent to a splitter and then collected via `aggregatorInputChannel`, after which aggregated and final response is sent to `aggregatorOutputChannel` (all these are done by Spring behind the scene).

```java
private static void testAggregator() {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
	MessageChannel channel = (MessageChannel) ctx.getBean("inputChannel");
	PollableChannel aggregatorOutputChannel = (PollableChannel) ctx.getBean("aggregatorOutputChannel");

	Customer cust = new Customer();
	cust.setName("Robert");
	cust.setAddress("City Quarter Apartments, Athlone");
	cust.setBalance(1350.75);
	Message<Customer> msg = MessageBuilder.withPayload(cust).build();

	channel.send(msg);
	Message<?> msgRcvd = aggregatorOutputChannel.receive();
	Customer c = (Customer) msgRcvd.getPayload();
	System.out.println("Aggregated response : " + c.getName() + "-" + c.getAddress() + "-" + c.getBalance());

	ctx.close();
}
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-integration-tutorial)