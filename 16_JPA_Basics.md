# JPA Basics

JPA - Java Persistence API is an evolved solution after developers started working with many ORM tools like Hibernate, TopLink, iBatis etc. JPA classes are already in use here and there within previous tutorials but in this tutorial we are going to illustrate the role JPA actually plays and how to develop a simple JPA application with an ORM of choice (Hibernate). JPA integration with Spring will come in next tutorials.

In summary, JPA is a layer between your application and ORM, such that application code remains neutral about any specific logic about ORM tool (no ORM specific import within application classes) and you can replace the ORM without touching the application code. JPA is an amalgamation of best practices followed by many existing tools and in present days it is always recommended to use JPA with your application rather to do direct integration with any ORM tool. Below picture illustrates the place of JPA within an application.

![](/images/jpa_basics.jpg)

Start with creating a new Maven Java project in Eclipse as *jpa-tutorial*. First element is the standard **pom.xml** file - only required dependencies are the Hibernate (our selected ORM) and MySql usuals.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>jpa</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>JPA Tutorial</name>

	<dependencies>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>5.4.4.Final</version>
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

Now under *src/main/resources/*, create a new directory as **META-INF**. Here will reside the important JPA configuration file viz. **persistence.xml**. Main unit within this file is a `persistence-unit` element. One such field defines configuration to one particular data source. The whole configuration will be referenced from Java code by the user given name in the *name* attribute.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.2"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

	<persistence-unit name="TestUnit">
		<provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
		<properties>
			<property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver" />
			<property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/testdb" />
			<property name="javax.persistence.jdbc.user" value="root" />
			<property name="javax.persistence.jdbc.password" value="root" />
			<property name="hibernate.dialect" value="org.hibernate.dialect.MySQL8Dialect" />
			<property name="hibernate.hbm2ddl.auto" value="update" />
		</properties>
	</persistence-unit>

</persistence>
```

Next is an entity bean - a simple POJO as **Customer.java** where all annotations are JPA annotations.

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

Lastly, the class to test the features - **TestCode.java**. This is how JPA is used - at first an `EntityManager` object is created via one `EntityManagerFactory` instance. Then from this manager object other DB operation related objects are instantiated on the fly. For inserting data, `EntityTransaction` is created from manager. For reading data, `createQuery` API is used from manager object. If the complete table is selected, then `query.getResultList()` output can be directly casted to a generic type (e.g. `List<Customer>`). Otherwise, when there is a particular select clause then the same output shall be consumed as `List<Object[]>`.

```java
package apim.github.tutorial;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import javax.persistence.Query;

public class TestCode {

	@SuppressWarnings("unchecked")
	public static void main(String[] args) {
		EntityManagerFactory factory = Persistence.createEntityManagerFactory("TestUnit");
		EntityManager manager = factory.createEntityManager();

		Customer c1 = new Customer(30, "Jerry", 2000);
		Customer c2 = new Customer(40, "Holly", 1800);

		EntityTransaction tx = manager.getTransaction();
		tx.begin();
		manager.persist(c1);
		manager.persist(c2);
		tx.commit();

		Query query = manager.createQuery("select c from Customer as c");
		List<Customer> list = query.getResultList();
		for (Customer c : list) {
			System.out.println(c.getId() + ", " + c.getName() + ", " + c.getBalance());
		}

		Query querySelected = manager.createQuery("select c.id, c.name from Customer as c");
		List<Object[]> listSelected = querySelected.getResultList();
		for (Object[] arr : listSelected) {
			System.out.println(arr[0] + ", " + arr[1]);
		}

		manager.close();
		factory.close();
	}

}
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/jpa-tutorial)