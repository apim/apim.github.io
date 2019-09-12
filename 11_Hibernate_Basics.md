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



**Complete source code for this tutorial:** [GitHub](https://github.com/apim/hibernate-tutorial)