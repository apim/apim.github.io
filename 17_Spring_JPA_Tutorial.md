# Spring JPA Tutorial

This tutorial will cover the aspect on how Spring integrates with JPA and Hibernate. Core benefits of using JPA and Hibernate are ripped, alongside, further boiler-plate code reduction happens with Spring (as it replaces the need of JPA's *persistence.xml*). From Spring 3 onwards, Spring's earlier approach to JPA (in a more traditional way) as *JpaTemplate* has been removed and all DB access is now recommended through the annotation `@PersistenceContext` on an `EntityManager` instance.

Start with creating a new Maven Java project in Eclipse as *spring-jpa-tutorial*. First element is the standard **pom.xml** file which contains the now usual Spring, Hibernate and MySql dependencies.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring-jpa</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring JPA Tutorial</name>

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
			<artifactId>spring-orm</artifactId>
			<version>5.1.8.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>5.4.4.Final</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-dbcp2</artifactId>
			<version>2.7.0</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.17</version>
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

Next is the crucial spring configuration - **spring-context.xml**. The *datasource* configuration is self-explanatory. Following that, `jpaVendorAdapter` is configured which will be used down the line to tell Spring who will actually provide persistence functionality (MySql through Hibernate in this case). Then the important element is `entityManagerFactory` which encapsulates datasource, jpavendoradapter and any other ORM specific settings (defined under `jpaProperties`). The remaining configuration for transaction management is same as earlier JPA tutorial.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

	<context:component-scan base-package="apim.github.tutorial" />

	<tx:annotation-driven />

	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
		<property name="username" value="root" />
		<property name="password" value="root" />
		<property name="url" value="jdbc:mysql://localhost:3306/testdb" />
	</bean>

	<bean id="jpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" />

	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
		<property name="dataSource" ref="dataSource" />
		<property name="packagesToScan" value="apim.github.tutorial" />
		<property name="jpaProperties">
	        <props>
	            <prop key="hibernate.hbm2ddl.auto">update</prop>
	            <prop key="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</prop>
	        </props>
	    </property>
	</bean>

	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>

</beans>
```

For the Java code part, at first the same **Customer.java** entity bean is defined.

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

Next, we develop the DAO class **CustomerDAO.java**. It is very much similar to the DAO in Spring-Hibernate example and the *@Transactional* annotation is used for the automatic transaction management by Spring. Core DB calls happen through JPA provided `EntityManager` bean which gets populated in runtime by our spring context.

```java
package apim.github.tutorial;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.transaction.Transactional;

import org.springframework.stereotype.Repository;

@Repository
@SuppressWarnings("unchecked")
public class CustomerDAO {

	@PersistenceContext
	private EntityManager entityManager;

	@Transactional
	public void addCustomer(Customer c) {
		entityManager.persist(c);
	}

	@Transactional
	public List<Customer> getAllCustomers() {
		return entityManager.createQuery("from Customer").getResultList();
	}

	@Transactional
	public List<Customer> findCustomerByBalance(double amt) {
		return (List<Customer>) entityManager.createQuery("select c from Customer as c where c.balance >= " + amt)
				.getResultList();
	}

}
```

Lastly, **TestCode.java** to see all the features in action.

```java
package apim.github.tutorial;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestCode {

	public static void main(String args[]) {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		CustomerDAO dao = (CustomerDAO) ctx.getBean("customerDAO");
		dao.addCustomer(new Customer(50, "Sam", 5000));
		dao.addCustomer(new Customer(60, "Peter", 4000));
		List<Customer> list = dao.findCustomerByBalance(2000);
		for (Customer c : list) {
			System.out.println(c.getId() + ": " + c.getName());
		}
		ctx.close();
	}

}
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-jpa-tutorial)