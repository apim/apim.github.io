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

Start by creating a new Maven Java Web project in Eclipse as *spring-mvc-form-handling-tutorial*. The *pom.xml* shall be identical to *spring-mvc-form-tutorial* project, except for below two additions which are required for validation feature. `validation-api` defines the specs for bean validation and `hibernate-validator` is one of the most popular implementation of the same. The *web.xml* can also be lifted from there as it is.

```xml
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>2.0.1.Final</version>
</dependency>
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>6.0.17.Final</version>
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

Following the above, another small file is to be developed (at the same place with *register.jsp*) just to show the registration success message. This is **result.jsp** and this file does not contain any new ideas other than what has already been discussed.

```html
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title><spring:message code="title.result.label" /></title>
</head>
<body>
	<div align="center">
		<h3>${serverMsg}</h3>
	</div>
</body>
</html>
```

Now create the Java classes. First up is the bean - **RegistrationData.java**. This class is the model and also takes part in validation by *JSR 303 Bean Validation*. Spring is aware of JSR 303 and any implementation (like *hibernate-validator*) will go along. This validation provides few annotations like *@NotNull* and *@Size* which are to be declared on member variables. They work as their name suggests and can be parameterised as shown below. Validation failure error messages also can be directly supplied in the *message* attribute. Although, to use i10n here, it is bit different and the same is explained in the next section.

```java
package apim.github.tutorial;

import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class RegistrationData {

	@Size(min = 10, max = 80, message = "{Size.registrationData.fullName}")
	private String fullName;

	@NotNull(message = "{NotNull.registrationData.age}")
	@Min(value = 18, message = "{Min.registrationData.age}")
	@Max(value = 30, message = "{Max.registrationData.age}")
	private Integer age;

	private String address;

	private String sex;

	private String[] classChoices;

	private String campus;
	
	// getters and setters
	
}
```

For all *JSR 303* validation errors, there are predefined error messages. Customisation is easy if it is done along with the annotation. However, to enable internationalisation here, one has to define the messages in a file named exactly as **ValidationMessages.properties** and the file should be kept under classpath directly i.e. for Maven projects at *src/main/resources*. The message keys within Java can be accessed as variables like `${Max.registrationData.age}`.

```
Size.registrationData.fullName=Name shall be within 10 to 80 characters
NotNull.registrationData.age=Please enter your age
Min.registrationData.age=Should be at least 18
Max.registrationData.age=Maximum 30 years old
```

Next comes the main controller class **RegistrationController.java**. Few part of this class is commented, initially, as at the first step validation is being tested via Bean Validation API only. Combining both JSR 303 and Classic Spring Form Validation is out of scope of this tutorial. Hence in the next section spring validation will be shown by uncommenting those lines.

At first within the controller the `MessageSource` bean is autowired. Next there is a short handy private method to read the resource bundle. Following that, few methods are defined which work as reference data provider for the form (i.e. methods which supply the master data displayed in UI like contents of comboboxes etc.). They are meant to work like that by the usage of `@ModelAttribute`. Then the method `initialize()` works as *form-backing object* i.e. a method which returns values those are to be pre-selected in the UI. It is obvious that this method uses the *RegistrationData* bean and populates that as necessary. This feature is achievable via annotating the method as `@RequestMapping` and specifying method as `GET`. Spring will use all methods annotated with *@ModelAttribute* before and then the *@RequestMapping (GET)* will be used to prepare the final model for the view.

Next comes the *POST* method `registerStudent()` which will actually handle the form submission. It takes the bean as input and the same is annotated with `@Valid`, followed by `@ModelAttribute`. Purpose of the second annotation is already explained in the previous tutorial and the `@Valid` annotation does the behind-the-scene work for complete bean validation. If there are any validation errors, `BindingResult` parameter is populated automatically. Within the method this parameter is checked at first and the control is returned to the form view if there are any errors.

However, control is not returned directly to the success page. Rather it is done via adding `redirect:/`. And another `@RequestMapping GET` method is defined to present the model for the same. This is done to avoid **dual form submission problem**. Without this, refreshing the result page will *re-submit the form* which is an error case. This is a classic implementation from Spring for the *Post/Get/Redirect* pattern. Along with this, `RedirectAttributes` is present in the POST method signature and no real attribute is added here to bypass the issue of appearance of model values at the URL. Without the use of *RedirectAttributes*, any element of model will be by default displayed in the URL which is an unwanted feature.

```java
package apim.github.tutorial;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@Controller
public class RegistrationController {

	@Autowired
	private MessageSource messageSource;

	/*private RegistrationValidator validator;

	@Autowired
	public void setValidator(RegistrationValidator validator) {
		this.validator = validator;
	}

	@InitBinder
	private void initBinder(WebDataBinder binder) {
		binder.setValidator(validator);
	}*/

	private String label(String key) {
		return messageSource.getMessage(key, null, null);
	}

	@ModelAttribute("classList")
	public List<String> populateClasses() {
		List<String> list = new ArrayList<>();
		list.add(label("reference.morning"));
		list.add(label("reference.afternoon"));
		list.add(label("reference.evening"));
		return list;
	}

	@ModelAttribute("campusList")
	public List<String> populateCampuses() {
		List<String> list = new ArrayList<>();
		list.add(label("reference.campus1"));
		list.add(label("reference.campus2"));
		list.add(label("reference.campus3"));
		list.add(label("reference.campus4"));
		return list;
	}

	@RequestMapping(value = "/login", method = RequestMethod.GET)
	public String initialize(Map<String, Object> model) {
		RegistrationData regData = new RegistrationData();
		regData.setSex(label("sex.male.label"));
		regData.setCampus(label("reference.campus2"));
		model.put("regData", regData);
		return "register";
	}

	@RequestMapping(value = "/login", method = RequestMethod.POST)
	public String registerStudent(Map<String, Object> model, @Valid @ModelAttribute("regData") RegistrationData regData,
			BindingResult result, RedirectAttributes ra) {
		if (result.hasErrors()) {
			return "register";
		}
		return "redirect:/result";
	}

	@RequestMapping(value = "/result", method = RequestMethod.GET)
	public String view(Map<String, Object> model) {
		model.put("serverMsg", "Registration Successful");
		return "result";
	}

}
```

It's now time to test. Build and deploy as you prefer. Accessing the url of GET method of the controller http://localhost:8080/spring-mvc-form-handling/login shall load login page as below.

![](/images/ex_ss_07.jpg)

Hitting submit without any input shall demonstrate bean validation in work.

![](/images/ex_ss_08.jpg)

Now fill up some sample valid value and result page shall come up.

![](/images/ex_ss_09.jpg)

Now to see classic spring validation in action. Uncomment the shown lines within controller class. This will introduce the requirement of the last file - **RegistrationValidator.java**. This is a class which implements Spring `Validator interface` and defines methods `supports()` and `validate()`. First one checks whether target class is eligible for this validator implementation or not. Second one contains the logic for programmatic validation and may use Spring provided utility class like `ValidationUtils` etc. Error messages are not usually directly hard-coded here rather their resource bundle keys are used.

To use this validator within the controller, two things are to be done. At first, the validator has to be marked as a bean and the same need to be autowired in the controller. Secondly, a new method needs to be defined, annotated with `@InitBinder`, which will bind this validator instance with the target bean during request processing (thus invocation by Spring context becomes guaranteed). *Nevertheless*, it is to note that using this validator *automatically turns off @Valid* validation.

```java
package apim.github.tutorial;

import org.springframework.stereotype.Component;
import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

@Component
public class RegistrationValidator implements Validator {

	@Override
	public boolean supports(Class<?> clazz) {
		return RegistrationData.class.isAssignableFrom(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		RegistrationData data = (RegistrationData) target;
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "address", "registrationData.address.empty");
		if (data.getClassChoices().length == 0) {
			errors.rejectValue("classChoices", "registrationData.classChoices.empty");
		}
	}

}
```

Rebuild and deploy. Now submitting without valid values shall engage spring validation and sample output shall be like below.

![](/images/ex_ss_10.jpg)

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-mvc-form-handling-tutorial)