# Spring Data JPA Tutorial

We have gradually evolved in our tutorials that how database connectivity is covered in Spring by various means like Sping-JDBC, Spring-Hibernate and Spring-JPA-Hiberndate. Finally, we have reached to probably the best solution provided by Spring for this - Spring Data JPA. Notwithstanding the fact that Spring Data adds a new layer within the application, it superbly simplifies basic CRUD operations and provides multiple ways to add customised and/or complex queries. Below diagram depicts a top level overview of Spring Data JPA.

![](/images/spring_data_jpa.jpg)

There are numerous features present in Spring Data and in this tutorial only the core concepts will be touched upon. The code snippet will be almost similar to previous Spring-JPA example (except the DAO class where the actual improvement will be shown), however, understanding the concept beforehand of Spring Data and its capability are crucial here.

> Makes it easy to use new data access technologies, such as non-relational databases, map-reduce frameworks, and cloud based data services. Spring Data also provides improved support for relational database technologies. This is an umbrella project which contains many subprojects that are specific to a given database.
> - Spring Data Official Intro

As noted above, Spring Data houses primarily Spring-Data-Common project which is then used by implementation specific projects e.g. Spring-Data-JPA. Spring Data supports many platforms like JPA, MongoDB, Hadoop etc. From the official source of Spring-Data-JPA:

> Implementing a data access layer of an application has been cumbersome for quite a while. Too much boilerplate code has to be written to execute simple queries as well as perform pagination, and auditing. Spring Data JPA aims to significantly improve the implementation of data access layers by reducing the effort to the amount that's actually needed. As a developer you write your repository interfaces, including custom finder methods, and Spring will provide the implementation automatically.

To put these into action, here DAO class will make use of some repositories which are direct child of Spring-Data-JPA provided repositories. However, these repositories are marker/tagged interfaces only. Application code also will not have any implementation for them and the same will be supplied by Spring. Therefore, user will concentrate solely on business logic and total data access layer related codebase will be managed by Spring.

As Spring Data JPA is primarily all about the repositories, here we will delve into them bit deeper before starting writing code. Below diagram gives a brief overview of the main repositories present, along with the important method contracts residing there.

![](/images/spring_data_arch.jpg)

Application code will always refer to JpaRepository and will use methods mentioned there without writing any other DOA related code. In addition, developer may add any method(s) in the child repository (of JpaRepository).