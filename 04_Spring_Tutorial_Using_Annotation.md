# Spring Tutorial - Using Annotation

This tutorial will practice all examples form the earlier tutorial, however, this time it will use **annotation** instead of XML context configuration / bean definition. Spring version 2.5 onward all bean configurations can be done with the aid of annotation and the whole XML work has become redundant. Annotation mechanism in Spring container is not enabled by default hence to do that the a line like this has to be added at the top of any context configuration xml - `<context:component-scan base-package="apim.github.tutorial" />`.

Here **base-package** is the package where we are instructing Spring to look for beans. This parameter is multivalued as several packages can be mentioned using comma. It is to be noted that Spring supports at the same time both annotation based and XML based configurations and the latter overrides the earlier one. There are many annotations defined at Spring and also there are numerous annotations at JDK which are used while developing Spring based applications. For this tutorial, we will discuss the below annotations.

1. **`@Component`** - A generic annotation to define Spring beans
2. **`@Service`** - Specialized case of @Component for the use case of presentation layer
3. **`@Repository`** - Specialized case of @Component for the use case of persistence layer
4. **`@Controller`** - Specialized case of @Component for the use case of service layer
5. **`@Autowired`** - As name suggests, for bean autowiring / dependency injection, works as byType
6. **`@Qualifier`** - To resolve ambiguity while using @Autowired
7. **`@Resource`** - Another way of dependency injection, works as byName
8. **`@Scope`** - To define the scope of a bean

