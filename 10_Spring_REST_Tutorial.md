# Spring REST Tutorial

This tutorial covers the aspect of implementing REST based web services using Spring framework. Basic understanding of Web Services and REST design are prerequisites here. Spring's gift to REST server side programming - two annotations `@RequestBody` and `@ResponseBody` will be used in the examples practiced here. On the other hand, for client side REST call, we have `RestTemplate`. These annotations are smart implementations by Spring which can do serialisation and de-serialisation of various request/response types e.g. JSON, XML etc.

Begin by creating a new Maven Java Web project in Eclipse as *spring-rest-tutorial*. The **pom.xml** shall need only *webmvc* dependency of Spring in special, rest are standard.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring-rest</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring REST Tutorial</name>
	<packaging>war</packaging>

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
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
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

Next, the **web.xml** file. It's very similar to other web projects covered in these tutorials and only important thing to note is the servlet mapping for REST calls.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<display-name>Spring REST Test</display-name>

	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/services/*</url-pattern>
	</servlet-mapping>

</web-app>
```

Following this define the *dispatcher servlet*. Likewise previous tutorials, defining *mvc:annotation-driven* & *mvc:default-servlet-handler* does most of the REST call processing automatically behind the scene by Spring framework.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<context:component-scan base-package="apim.github.tutorial" />
	
	<mvc:annotation-driven />
	<mvc:default-servlet-handler />
	
</beans>
```

Now define the Java classes. Start with a POJO **Employee.java**. This class contains annotations `@XmlRootElement` and `@XmlElement`, which tell Spring that a web service using this POJO shall transmit (serialise / deserialise) contents as XML.

```java
package apim.github.tutorial;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
@XmlAccessorType(XmlAccessType.NONE)
public class Employee {

	@XmlElement
	private int eid;

	@XmlElement
	private String ename;

	@XmlElement
	private double esal;
	
	// getters and setters
	
}
```

Finally, the controller class **EmployeeRestController.java** is developed. For keeping it simple, the GET request returns data from an internal data map loaded as static. The `@ResponseBody` annotation makes it possible for *JSON / XML* object to be transmitted back for the GET request. Similarly, `@RequestBody` annotation does the behind-the-scene work for *JSON / XML* de-serialization into *Employee* object. For the response in the POST method, a simple String value is returned which will be displayed in the REST client. The *headers* attribute with *Accept* value in *@RequestMapping* makes sure that this method is only serviced by JSON input request. For any other request types, Spring framework will throw proper HTTP error response message.

```java
package apim.github.tutorial;

import java.util.HashMap;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/emp")
public class EmployeeRestController {

	private static HashMap<Integer, Employee> empMap;

	static {
		empMap = new HashMap<>();
		Employee emp1 = new Employee();
		emp1.setEid(1);
		emp1.setEname("John");
		emp1.setEsal(125.50);
		Employee emp2 = new Employee();
		emp2.setEid(2);
		emp2.setEname("Cathy");
		emp2.setEsal(75.75);
		Employee emp3 = new Employee();
		emp3.setEid(3);
		emp3.setEname("Peter");
		emp3.setEsal(100);
		empMap.put(1, emp1);
		empMap.put(2, emp2);
		empMap.put(3, emp3);
	}

	@RequestMapping(method = RequestMethod.GET, value = "/{eid}")
	public @ResponseBody Employee findEmployee(@PathVariable("eid") int id) {
		System.out.println("Request received for id: " + id);
		return empMap.get(id);
	}

	@RequestMapping(method = RequestMethod.POST, value = "/save", headers = "Accept=application/json")
	public @ResponseBody String saveEmployee(@RequestBody Employee emp) {
		System.out.println(emp.getEid() + ", " + emp.getEname() + ", " + emp.getEsal());
		return "Employee saved";
	}

}
```

Any standard browser based or standalone REST client can be used to test the application. Build and deploy in your preferred way. Just accessing the GET request url as http://localhost:8080/spring-rest/services/emp/3 shall produce a output in browser as `3Peter100.0`. Upon inspecting the source of the page in browser, you shall see raw XML data as below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?><employee><eid>3</eid><ename>Peter</ename><esal>100.0</esal></employee>
```

Now let's develop a simple Spring based REST client. At first, create a new application context XML **spring-context.xml** for use within the client. Content of this file is straighforward - just loading beans through Spring's component scanning.

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

Lastly, create a Java class to test the REST call. The object **RestTemplate** can be defined through Spring or can simply be direct instantiated within code. The method `getForObject()` invokes a GET REST API and there is similar `postForObject()` method for POST calls.

```java
package apim.github.tutorial;

import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class TestCode {

	public Employee findEmployeeById(int id) {
		return new RestTemplate().getForObject("http://localhost:8080/spring-rest/services/emp/" + id, Employee.class);
	}

	public static void main(String[] args) {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		TestCode service = (TestCode) ctx.getBean("testCode");
		Employee emp = service.findEmployeeById(2);
		System.out.println(emp.getEid() + ", " + emp.getEname() + ", " + emp.getEsal());
		ctx.close();
	}

}
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-rest-tutorial)