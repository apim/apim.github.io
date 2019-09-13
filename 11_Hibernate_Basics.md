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



**Complete source code for this tutorial:** [GitHub](https://github.com/apim/hibernate-tutorial)