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