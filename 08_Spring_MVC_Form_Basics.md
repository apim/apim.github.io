# Spring MVC Form Basics

Form handling in Spring, along with special JSP tags provided by Spring, are both rich in features and easily programmable. However, before actually starting with form related operations, familiarity with few basic Spring annotations are essential. These annotations are `@RequestParam`, `@ModelAttribute` and `@PathVariable` and all these are primarily applied on method arguments.

While working with traditional form input elements in HTML or JSP, *@RequestParam* marked argument holds value for one *input* element. This works for both HTTP Get and POST requests. The construct for this annotation is *@RequestParam("field-name") data-type argument* and a sample will look like this - `@RequestParam("name") String customerName`.

On the other hand, for form submission using POST only, *@ModelAttribute* holds values for the complete form and it is necessary to have a bean defined with matching fields as of the HTML form. The construct for this annotation is *@ModelAttribute class-name argument* and a sample will look like this - `@ModelAttribute Customer customerObject`.

Lastly, the *@PathVariable* allows developers to create custom URI. This, clubbed with *@RequestMapping*, allows developers to create Spring MVC with good REST design as this approach supports identification of unique resources clearly. For example, with method annotation as `@RequestMapping("/customer.form/{cid}")` and argument annotation as `@PathVariable("cid") int customerId`, developers create a scenario where customer details are specifically retrieved by their id value.

