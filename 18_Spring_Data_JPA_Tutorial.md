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

Begin by creating a new Maven Java project in Eclipse as *spring-data-jpa-tutorial*. You can copy the base code like *pom.xml* from previous tutorial's project. Change artifact details there as shown below - all other dependencies shall remain same and new dependency for *spring-data-jpa* shall be added.

```xml
<groupId>apim.github.tutorial</groupId>
<artifactId>spring-data-jpa</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>Spring Data JPA Tutorial</name>
```

```xml
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-jpa</artifactId>
	<version>2.2.3.RELEASE</version>
</dependency>
```

Next is the spring context file which is really identical to Spring JPA Tutorial barring single change of `jap-repositories` tag, which tells Spring where to scan for application defined repositories.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jpa="http://www.springframework.org/schema/data/jpa"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

	<context:component-scan base-package="apim.github.tutorial" />
	
	<jpa:repositories base-package="apim.github.tutorial" />
```

Next, develop the entity bean *Customer*, exactly same as previous examples.

```java
package apim.github.tutorial;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Customer {

	@Id
	private int id;
	private String name;
	private int balance;

	public Customer() {
	}

	public Customer(int id, String name, int balance) {
		this.id = id;
		this.name = name;
		this.balance = balance;
	}

	// getters and setters
	
}
```

Now we define the important part - the repository class **CustomerRepository.java**. As expected, it extends *JpaRepository*. The type definitions here are the entity's type and it's identifier field type. A single method is declared here which is a specialised data access method for user application and does not fall into the kitty of standard CRUD. To code that, `@Query` annotation from JPA, along with `@Param` have been used. However, no implementation of this interface is required anywhere and still all DB level operations will be managed by Spring Data.

```java
package apim.github.tutorial;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface CustomerRepository extends JpaRepository<Customer, Integer> {

	@Query("select c from Customer c where c.balance > :balance")
	public List<Customer> findSelectedBalances(@Param("balance") int balance);

}
```

Next, the DAO class **CustomerDAO.java**. Here modification is to use the repository methods only instead of any direct DB related coding.

```java
package apim.github.tutorial;

import java.util.List;

import javax.annotation.Resource;
import javax.transaction.Transactional;

import org.springframework.stereotype.Repository;

@Repository
public class CustomerDAO {

	private CustomerRepository repository;

	public CustomerRepository getRepository() {
		return repository;
	}

	@Resource
	public void setRepository(CustomerRepository repository) {
		this.repository = repository;
	}

	@Transactional
	public void addCustomer(Customer c) {
		repository.save(c);
	}

	@Transactional
	public List<Customer> getAllCustomers() {
		return repository.findAll();
	}

	@Transactional
	public List<Customer> getSelectedCustomers(int balance) {
		return repository.findSelectedBalances(balance);
	}

}
```

Finally, the main class **TestCode.java**. No change here since last tutorial, just new method from DAO has been used.

```java
package apim.github.tutorial;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestCode {

	public static void main(String args[]) {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		CustomerDAO dao = (CustomerDAO) ctx.getBean("customerDAO");
		dao.addCustomer(new Customer(70, "Alex", 3750));
		dao.addCustomer(new Customer(80, "Luis", 4250));
		List<Customer> list = dao.getAllCustomers();
		for (Customer c : list) {
			System.out.println(c.getId() + ": " + c.getName() + " : " + c.getBalance());
		}
		list = dao.getSelectedCustomers(5000);
		for (Customer c : list) {
			System.out.println(c.getId() + ": " + c.getName() + " : " + c.getBalance());
		}
		ctx.close();
	}

}
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-data-jpa-tutorial)