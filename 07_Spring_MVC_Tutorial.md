# Spring MVC Tutorial

Spring MVC is a popular framework implementing *Front Controller* design pattern. Below diagram gives an overview of how Spring MVC functions.

![](/images/spring_mvc.jpg)

All actions here surround the front controller i.e. **DispatcherServlet**, which along with all other above mentioned components are part of the **WebApplicationContext** of Spring. Following steps illustrate the process depicted above.

- Client sends HTTPRequest to `DispatcherServlet` (via web container)
- DispatcherServlet requests `HandlerMapping` for a proper controller against this client request
- HandlerMapping returns appropriate controller information
- DispatcherServlet requests that `Controller` for action
- Controller (business logic is either implemented here or service is invoked from here) processes the request and returns *MovelAndView* object with logical view name
- DispathcerServlet requests `ViewResolver` for the actual view implementation
- ViewResolver returns view information
- DispatcherServlet requests `View` to prepare the response with the input *Model*
- View returns rendered response
- DipatcherServlet returns HTTPResponse to the Client (via web container)

This tutorial will cover core concepts of Spring MVC and it will include both XML based and annotation based approaches. Prerequisite is to have installation of some web servers like Tomcat or JBoss EAP or WildFly. This tutorial uses latest Tomcat 9.x and runs the application through Eclipse Server configurtion.

Begin by creating a new Maven Java Web project in Eclipse as *spring-mvc-tutorial*. Being this a web project, create additional necessary files like *web.xml* etc. Complete project structure snapshot in eclipse for this tutorial is shown below.

![](/images/proj_struct.jpg)

## XML based Example

First element is the **pom.xml** file. Dependencies of importance here are *spring-webmvc* and *javax.servlet-api*.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring-mvc</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring MVC Tutorial</name>
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
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
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

Next define the **web.xml**. Here the dispatcher servlet is configured like any standard Java servlet. It is important to note that whatever be the servlet-name chosen here; at the next step, Spring framework by default will look for a file named *servlet-name-servlet.xml* to locate the application context configuration.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<display-name>Spring MVC Test</display-name>

	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>*.form</url-pattern>
	</servlet-mapping>

</web-app>
```

Now define the front controller for Spring MVC. Here it would be a file named **dispatcher-servlet.xml**. This file works like normal bean definitions, plus, the handler mapping and view resolver configuration is also defined. The mapping shows that for request *test.form* controller *testController* has to be invoked. Furthermore, using Spring's in built view resolver *InternalResourceViewResolver* it is configured that all concrete view implementations are stored at *views* directory and their file extension is *jsp*. Also note the inclusion of namespace *spring-mvc*.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/mvc http://www.springframework.org/schema/aop/spring-mvc.xsd">

	<bean id="testController" class="apim.github.tutorial.TestController" />
	<bean id="urlMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<property name="mappings">
			<props>
				<prop key="/test.form">testController</prop>
			</props>
		</property>
	</bean>
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

</beans>
```

Following this, create the only Java file for this tutorial which will have the controller implementation - **TestController.java**. This file must implement Spring's *Controller* interface and provide definition for the `handleRequest()` method. This is the place where actual business logic is implemented or service layer is invoked. Return type for this method is `ModelAndView` and this object will contain concrete model and logical view name.

```java
package apim.github.tutorial;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class TestController implements Controller {

	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		ModelAndView mv = new ModelAndView("show_message");
		mv.addObject("message", "Hello World to Spring MVC");
		return mv;
	}

}
```

Finally, the view file - **showMessage.jsp**. Here simply the hello world message as returned from the controller is printed.

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>Testing Spring MVC</title>
</head>
<body>
	<div align="center">
		<h3>${message}</h3>
	</div>
</body>
</html>
```

To test all these together, configure and start the web server through eclipse. Add the project into server's runtime (right click on server instance at eclipse server view > Add and Remove... > select your project and add). When the deployment is happening, monitor the console for an error free deployment. If all goes well, testing through browser with the URL http://localhost:8080/spring-mvc/test.form shall show that Spring MVC works.

![](/images/ex_ss_01.jpg)

### Annotation based Example

