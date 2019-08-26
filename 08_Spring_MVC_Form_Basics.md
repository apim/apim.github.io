# Spring MVC Form Basics

Form handling in Spring, along with special JSP tags provided by Spring, are both rich in features and easily programmable. However, before actually starting with form related operations, familiarity with few basic Spring annotations are essential. These annotations are `@RequestParam`, `@ModelAttribute` and `@PathVariable` and all these are primarily applied on method arguments.

While working with traditional form input elements in HTML or JSP, *@RequestParam* marked argument holds value for one *input* element. This works for both HTTP Get and POST requests. The construct for this annotation is *@RequestParam("field-name") data-type argument* and a sample will look like this - `@RequestParam("name") String customerName`.

On the other hand, for form submission using POST only, *@ModelAttribute* holds values for the complete form and it is necessary to have a bean defined with matching fields as of the HTML form. The construct for this annotation is *@ModelAttribute class-name argument* and a sample will look like this - `@ModelAttribute Customer customerObject`.

Lastly, the *@PathVariable* allows developers to create custom URI. This, clubbed with *@RequestMapping*, allows developers to create Spring MVC with good REST design as this approach supports identification of unique resources clearly. For example, with method annotation as `@RequestMapping("/customer.form/{cid}")` and argument annotation as `@PathVariable("cid") int customerId`, developers create a scenario where customer details are specifically retrieved by their id value.

Begin by creating a new Maven Java Web project in Eclipse as *spring-mvc-form-tutorial*. Create the **web.xml** copying from the *spring-mvc-tutorial* project and making only change at `url-pattern` tag from `*.form` to `/`. This is required for the application to work with different URIs. Copy the *dispatcher-servlet.xml (annotation version)* as it is from the previous project. Only change is to add two important lines: `<mvc:annotation-driven />` & `<mvc:default-servlet-handler />`, which will enable Spring to auto discover *Controller* beans. Make a new input HTML file as **index.html** under *webapp* directory as shown below. Note that its `form.action` attribute points to a particular controller method. This will be changed and re-deployed to test the other methods in this example. Next, lift the *show_message.jsp* from the previous project. Following this, the Java classes only remain to be developed.

New web.xml file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<display-name>Spring MVC Form Test</display-name>

	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

</web-app>
```

New dispatcher-servlet.xml file:

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

	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

</beans>
```

New index.html file:

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>Basic Spring MVC Form</title>
</head>
<body>
	<div align="center">
		<form action="get.form" method="get">
			<table>
				<tr>
					<td>Number One:</td>
					<td><input type="text" name="ip1" /></td>
				</tr>
				<tr>
					<td>Number Two:</td>
					<td><input type="text" name="ip2" /></td>
				</tr>
				<tr>
					<td colspan="2" align="center"><input type="submit" value="Do Sum"></td>
				</tr>
			</table>
		</form>
	</div>
</body>
</html>
```

Now start with the Java coding by creating a small bean class **Numbers.java**. This will be used with the *@ModelAttribute* and will match the input tags present in the input html file. However, it is to note that there is another use of *@ModelAttribute* where it works as view backing object and in that case the annotation is applied at the method level. This feature will be discussed in the following Spring MVC Form Handling Tutorial. Finally, create the controller class **FormController.java** where 3 different methods will be developed. Each method in this class performs as use case for the 3 annotations discussed above.

New Numbers.java file:

```java
package apim.github.tutorial;

public class Numbers {

	private int ip1;
	private int ip2;

	// getters and setters
}
```

New FormController.java file:

```java
package apim.github.tutorial;

import java.util.Map;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class FormController {

	@RequestMapping("/get.form")
	public String processGet(@RequestParam int ip1, @RequestParam int ip2, Map<String, Object> model) {
		model.put("message", ip1 + ip2);
		return "show_message";
	}

	@RequestMapping("/post.form")
	public String processPost(@ModelAttribute Numbers ip, Map<String, Object> model) {
		model.put("message", ip.getIp1() + ip.getIp2());
		return "show_message";
	}

	@RequestMapping("/pvtest.form/{val}")
	public String processPV(@PathVariable int val, Map<String, Object> model) {
		String result = val % 2 == 0 ? "Even Number" : "Odd Number";
		model.put("message", result);
		return "show_message";
	}

}
```

For testing, deploy the project as was done in the earlier tutorial. Accessing the application via URL http://localhost:8080/spring-mvc-form/ shall show something like the first image below. Giving some input shall display expected result page similar to the second image below. Note the modified URL where values are being passed as request parameters through *@RequestMapping*.

![](/images/ex_ss_03.jpg)

![](/images/ex_ss_04.jpg)

Next, edit the form fields in HTML to change the `form.action` to `post.form` and `method` to `post`. Testing like before shall product same output however, this time modified URL shall not contain anything as input is going via HTTP Post (sample output is the first image below). Point to note is that input tag *ip1* and *ip2* are mapped by Spring's *@ModelAttribute* to *Numbers* bean's *ip1* and *ip2* fields. Lastly, test the `@PathVariable` via URL http://localhost:8080/spring-mvc-form/pvtest.form/889. A sample response for the given input is shown in the second image below.

![](/images/ex_ss_05.jpg)

![](/images/ex_ss_06.jpg)

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-mvc-form-tutorial)