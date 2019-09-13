# Hibernate Basics

In this tutorial basics of Hibernate will be covered as warm-up for the next tutorials. Architecture and/or design of an ORM is not under the purview of this tutorial rather, here it is shown how to use all the primary features of Hibernate. In the following tutorials, Spring with Hibernate, JPA, Spring-Data and their combinations etc. will be discussed therefore, it is primarily a knowledge refresher for Hibernate. This tutorial will concentrate on latest release of Hibernate 5.x. Furthermore, an installation of *MySql* is required to test out DB operations in this tutorial.

Start with creating a new Maven Java project in Eclipse as *hibernate-tutorial*. First element is the standard *pom.xml* file - here we only need two dependencies: hibernate core and mysql connector.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>hibernate</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Hibernate Tutorial</name>

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

We need a backend table to test the application. So create a DB table at first in MySql.

```sql
CREATE TABLE `employee` (
	`ID` int(11) NOT NULL,
	`NAME` varchar(255) DEFAULT NULL,
	`DESIGNATION` varchar(255) DEFAULT NULL,
	`SALARY` int(11) DEFAULT NULL,
	PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Next create the Java Entity bean **Employee.java**. This is a POJO, doing object relational mapping (*ORM*) for table *employee*. Class is defined through annotation `@Entity` which sets the base for ORM candidacy. Primary key column is annotated with `@Id` and all other fields are annotated as `@Column`. If DB column names are not matching with POJO field names, then *@Column* annotation can be customised with *name* parameter.

```java
package apim.github.tutorial;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Employee {

	@Id
	private int id;

	@Column
	private String name;

	@Column
	private String designation;

	@Column
	private int salary;

	public Employee() {
	}

	public Employee(int id, String name, String designation, int salary) {
		this.id = id;
		this.name = name;
		this.designation = designation;
		this.salary = salary;
	}

	// getters and setters

}
```

Finally, develop the test class which will load Hibernate and will perform basic CRUD operations. Note that we are not using any xml based mappings anywhere as annotation based practice for Hibernate is almost de facto standard now. At the start of the class, standard database configuration is loaded into Java *Properties* object. Then at *main* method, Hibernate is loaded in its `SessionFactory`. To do so, `addPackage` is configured to provide any package specific meta data and the `addAnnotatedClass` is added where entity bean definition is passed. Now to test, there are 4 methods pertaining to all 4 operations of CRUD. Each method at first obtains *session* from *factory* and then initiates *transaction* through that session object. Rest of the code is pretty self-explanatory.

```java
package apim.github.tutorial;

import java.util.List;
import java.util.Properties;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class TestCode {

	private static final Properties prop;

	static {
		prop = new Properties();
		prop.setProperty("hibernate.connection.url", "jdbc:mysql://localhost:3306/testdb");
		prop.setProperty("hibernate.connection.username", "root");
		prop.setProperty("hibernate.connection.password", "root");
		prop.setProperty("dialect", "org.hibernate.dialect.MySQL8Dialect");
	}

	public static void main(String... args) {
		SessionFactory factory = new Configuration()
				.addPackage("apim.github.tutorial")
				.addProperties(prop)
				.addAnnotatedClass(Employee.class)
				.buildSessionFactory();

		createRecord(factory);
		readRecord(factory);
		updateRecord(factory);
		deleteRecord(factory);

		factory.close();
	}

	private static void createRecord(SessionFactory factory) {
		Session session = null;
		Transaction tx = null;
		try {
			session = factory.openSession();
			tx = session.beginTransaction();
			Employee emp1 = new Employee(1, "John", "Programmer", 100);
			Employee emp2 = new Employee(2, "Sally", "Painter", 90);
			session.save(emp1);
			session.save(emp2);
			tx.commit();
		} catch (Exception ex) {
			if (tx != null) {
				tx.rollback();
			}
			System.err.println("Exception occurred: " + ex.getMessage());
		} finally {
			session.close();
		}
	}

	private static void readRecord(SessionFactory factory) {
		Session session = null;
		Transaction tx = null;
		try {
			session = factory.openSession();
			tx = session.beginTransaction();
			@SuppressWarnings("rawtypes")
			List result = session.createQuery("from Employee").list();
			for (Object obj : result) {
				Employee e = (Employee) obj;
				System.out.println("Name: " + e.getName() + ", Designation: " + e.getDesignation() + ", Salary: " + e.getSalary());
			}
			tx.commit();
		} catch (Exception ex) {
			if (tx != null) {
				tx.rollback();
			}
			System.err.println("Exception occurred: " + ex.getMessage());
		} finally {
			session.close();
		}
	}

	private static void updateRecord(SessionFactory factory) {
		Session session = null;
		Transaction tx = null;
		try {
			session = factory.openSession();
			tx = session.beginTransaction();
			Employee e = (Employee) session.get(Employee.class, 1);
			e.setSalary(105);
			session.update(e);
			tx.commit();
		} catch (Exception ex) {
			if (tx != null) {
				tx.rollback();
			}
			System.err.println("Exception occurred: " + ex.getMessage());
		} finally {
			session.close();
		}
	}

	private static void deleteRecord(SessionFactory factory) {
		Session session = null;
		Transaction tx = null;
		try {
			session = factory.openSession();
			tx = session.beginTransaction();
			Employee e = (Employee) session.get(Employee.class, 2);
			session.delete(e);
			tx.commit();
		} catch (Exception ex) {
			if (tx != null) {
				tx.rollback();
			}
			System.err.println("Exception occurred: " + ex.getMessage());
		} finally {
			session.close();
		}
	}

}
```

Content for the DB table after all executions shall look like below.

```
mysql> select * from testdb.employee;
+----+------+-------------+--------+
| ID | NAME | DESIGNATION | SALARY |
+----+------+-------------+--------+
|  1 | John | Programmer  |    105 |
+----+------+-------------+--------+
1 row in set (0.00 sec)
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/hibernate-tutorial)