# Spring AOP Tutorial

AOP - Aspect Oriented Programming is one of the biggest gifts of Spring. Aspects are referred as cross-cutting concerns like logging, caching etc. which are essential for every application but not closely related to any business logic. Therefore, support for AOP makes Spring an excellent framework where these aspects / cross-cutting concerns are automatically taken care of at the background. Obviously, there will be some sort of instrumented code which actually implements the aspects, nevertheless, amount of boiler-plate coding is very minimal thus incorporating Spring AOP has become a straightforward job.

Before we dive into some hands-on for AOP, let's clarify few terminologies related to Spring AOP.

* **aspect** - a particular set of code providing AOP functionality in the application
* **advice** - actual piece of code being executed by framework as part of AOP
* **pointcut** - marks the places within application where framework shall invoke AOP

Now there are 5 types of advice supported in Spring. A brief description is listed below.

1. **before** - executes before the pointcut invocation
2. **after** - executes after the pointcut invocation
3. **after-returning** - executes if pointcut returns successfully
4. **after-throwing** - executes if pointcut throws exception
5. **around** - executes before and after pointcut invocation

Begin by creating a new Maven Java project in Eclipse as *spring-aop-tutorial*.