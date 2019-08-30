# Spring MVC Form Handling

In this tutorial all areas of Spring MVC based form handling will be discussed. Below is a listing of points which are under the scope of this tutorial.

1. How to develop view based on **Spring provided JSP tag libraries** e.g. http://www.springframework.org/tags and http://www.springframework.org/tags/form
2. How to implement **internationalization (i10n)** in Spring based view e.g. *\<spring:message\>*
3. How to make input form based on Spring tags e.g. **\<form:form\>**, \<form:input\> and \<form:radiobutton\> etc.
4. How to display form input validation error messages inline e.g. **\<form:errors\>**
5. How to use *JSR 303* **Bean Validation** in Spring e.g. *@NotNull* etc. annotations
6. How to use Spring Classic **Form Validation** (with *@InitBinder* & *Validator interface*)
7. How to incorporate i10n with Spring classic Form Validation
8. How to build **reference data for View** at Controller (with *@ModelAttribute*)
9. How to prepare **form-backing object for View** at Controller (with *@RequestMapping GET* method)
10. How to process form submission
11. How to use resource bundle within Spring Controller class
12. How to avoid form double submission at Spring with **Post/Redirect/Get**

Start by creating a new Maven Java Web project in Eclipse as *spring-mvc-form-handling-tutorial*. The *pom.xml* shall be identical to *spring-mvc-form-tutorial* project, except for below two additions which are required for validation feature. The *web.xml* can also be lifted from there as it is.

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

As shown in the above file, a resource bundle as **messages.properties** (essentially the default one) is to be prepared and kept at specifically in this location - *src/main/webapp/WEB-INF/*. Content of this file will be used in the following sections.

```
reference.morning=Morning
reference.afternoon=Afternoon
reference.evening=Evening
reference.campus1=City Extension
reference.campus2=City Center
reference.campus3=Greater Boulevard
reference.campus4=University Hall
title.register.label=Spring MVC Form Handling
title.result.label=Form Submitted
fullName.label=Full Name:
age.label=Age:
address.label=Address:
sex.label=Sex:
sex.male.label=Male
sex.female.label=Female
classChoices.label=Class Choices:
campus.label=Preferred Campus:
submit.label=Register
registrationData.address.empty=Please enter your address
registrationData.classChoices.empty=Please select at least one class option
```

Next develop the main form view - **register.jsp**. This is a Spring JSP tag based file. Keep it within the usual *views* sub-directory. Required taglibs for Spring Forms are added at the top. There is a small style section for error messages to be displayed separately. Following that the form is defined. Here any label is read from the resource bundle like this way: `<spring:message code="title.register.label" />`. The form is defined using `<form:form>` tag and the `modelAttribute` parameter refers to the Bean holding form data (as defined in the Controller class). For almost every standard HTML input tags, Spring has its corresponding tag like `<form:input>`, `<form:checkboxes>`, `<form:select>` etc. Important thing to notice here is the `path` attribute which refers to the Java bean property. HTML elements which require reference (or master) data they are referenced as variable e.g. `${classList}`. This variable will be defined in the Controller class by `@ModelAttribute`. Lastly, the error messages are displayed using `<form:errors>` tag and a CSS style reference is there to highlight the same in UI.

```html
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form"%>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<style>
.errMsg {
	color: red;
}
</style>
<title><spring:message code="title.register.label" /></title>
</head>
<body>
	<div align="center">
		<form:form modelAttribute="regData" method="post">
			<table>
				<tr>
					<td><spring:message code="fullName.label" /></td>
					<td><form:input path="fullName" /></td>
					<td><form:errors path="fullName" cssClass="errMsg" /></td>
				</tr>
				<tr>
					<td><spring:message code="age.label" /></td>
					<td><form:input path="age" /></td>
					<td><form:errors path="age" cssClass="errMsg" /></td>
				</tr>
				<tr>
					<td><spring:message code="address.label" /></td>
					<td><form:textarea path="address" /></td>
					<td><form:errors path="address" cssClass="errMsg" /></td>
				</tr>
				<tr>
					<td><spring:message code="sex.label" /></td>
					<td>
						<spring:message code="sex.male.label" var="male" /> 
						<form:radiobutton path="sex" value="Male" label="${male}" /> 
						<spring:message code="sex.female.label" var="female" /> 
						<form:radiobutton path="sex" value="Female" label="${female}" />
					</td>
					<td><form:errors path="sex" cssClass="errMsg" /></td>
				</tr>
				<tr>
					<td><spring:message code="classChoices.label" /></td>
					<td><form:checkboxes path="classChoices" items="${classList}" /></td>
					<td><form:errors path="classChoices" cssClass="errMsg" /></td>
				</tr>
				<tr>
					<td><spring:message code="campus.label" /></td>
					<td><form:select path="campus" items="${campusList}" /></td>
					<td><form:errors path="campus" cssClass="errMsg" /></td>
				</tr>
				<tr>
					<td colspan="3" align="center">
						<spring:message code="submit.label" var="submit" /> 
						<input type="submit" value="${submit}">
					</td>
				</tr>
			</table>
		</form:form>
	</div>
</body>
</html>
```

