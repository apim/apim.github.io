# Spring Tutorial

This tutorial will cover all basic concepts of Spring and all tutorials here will consist of full hands-on practices. Platform used is Windows 10, with all *latest versions* of *Eclipse*, *Java* & *Maven* installed. Complete source for all tutorials is given at the end of the pages as a link to the GitHub repository.

To being, create a new Maven Java project in Eclipse as spring-tutorial. Add a pom file like below.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring Tutorial</name>

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
	</dependencies>
</project>
```