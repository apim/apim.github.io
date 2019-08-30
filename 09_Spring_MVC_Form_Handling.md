# Spring MVC Form Handling

In this tutorial all areas of Spring MVC based form handling will be discussed. Below is a listing of points which are under the scope of this tutorial.

1. How to develop view based on **Spring provided JSP tag libraries** e.g. http://www.springframework.org/tags and http://www.springframework.org/tags/form
2. How to implement **internationalization (i10n)** in Spring based view e.g. *\<spring:message\>*
3. How to make input form based on Spring tags e.g. **\<form:form\>**, <form:input> and <form:radiobutton> etc.
4. How to display form input validation error messages inline e.g. **\<form:errors\>**
5. How to use *JSR 303* **Bean Validation** in Spring e.g. *@NotNull* etc. annotations
6. How to use Spring Classic **Form Validation** (with *@InitBinder* & *Validator interface*)
7. How to incorporate i10n with Spring classic Form Validation
8. How to build **reference data for View** at Controller (with *@ModelAttribute*)
9. How to prepare **form-backing object for View** at Controller (with *@RequestMapping GET* method)
10. How to process form submission
11. How to use resource bundle within Spring Controller class
12. How to avoid form double submission at Spring with **Post/Redirect/Get**

Start by creating a new Maven Java Web project in Eclipse as *spring-mvc-form-handling-tutorial*. The *pom.xml* shall be identical to *spring-mvc-form-tutorial* project, except for below two additions which are required for validation feature. The *web.xml* can be lifted from there as it is.

```xml
<dependency>
	<groupId>javax.servlet.jsp.jstl</groupId>
	<artifactId>jstl-api</artifactId>
	<version>1.2</version>
</dependency>
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>2.0.1.Final</version>
</dependency>
```

Next, the sprint context file i.e. *dispatcher-servlet.xml*. This is also identical to previous project expect for the addition of `messageSource` bean whose job is to load the properties file containing i10n messages used within the web application.

```xml
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basename" value="/WEB-INF/messages" />
</bean>
```

