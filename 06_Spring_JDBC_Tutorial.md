# Spring JDBC Tutorial

Spring provides JDBC support via **JdbcTemplate** class. Main benefit is that while it brings down boiler-plate coding to a minimum, at the same time, there is ample scope of DB query customisation. Use of this template requires a *datasource* configuration and the example here uses *Apache DBCP* library. An installation of *MySql* is required to test out DB operations. This datasource is usually configured in the spring context file along with other bean definitions (if not using annotation).

Next important part of Spring JDBC is writing a *DAO class* (Data Access Object class). This DAO class is the actual bean (**@Repository** can be used for this) where the *JdbcTemplate* is used (this or the datasource has to be injected hence **@Resource** can be used here). Likewise the DAO pattern, all database related methods are defined here. *JdbcTemplate* supports full *CRUD* operations. There are different methods for this purpose and a brief overview of them is given below.

- **execute()** - To run a DDL command like create or drop table
- **update()** - To run a DML command like insert or update table
- **queryForXXX()** - To read data from table for various primitive data types (e,g. XXX has forms like Int, Double etc.)
- **queryForObject()** - To read data from table for an object (**RowMapper** implementation is used)
- **query()** - To read multiple records from table matching one object (**RowMapper** implementation is used)

For reading table records directly into an object, some implementation of `RowMapper` has to be used. This class will actually do the JDBC *ResultSet* to object/bean transformation. The class will be having only one method `(mapRow(resultSet, rowId))` overridden for all usual purposes. To use all these, developer simply has to obtain the bean of DAO via Spring and then invoke the necessary methods.

Let's try the ideas with example. Start with creating a new Maven Java project in Eclipse as *spring-jdbc-tutorial*. You can copy base code like *pom.xml* from earlier project *spring-tutorial*. Add new dependencies there as *spring-jdbc* for core functionality and *commons-dbcp2* & *mysql-connector-java* for providing datasource implementation & JDBC driver.

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>5.1.8.RELEASE</version>
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
```

Now create the important spring configuration **spring-context.xml**. Important point to note in this file is the inclusion of `spring-jdbc` namespace. Next follows usual component scanning for annotation based bean definitions and *datasource* definition which uses MySql driver. Note that you have to edit the credentials & database name as per your environment.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/jdbc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/aop/spring-jdbc.xsd">

	<context:component-scan base-package="apim.github.tutorial" />

	<bean id="datasource" class="org.apache.commons.dbcp2.BasicDataSource">
		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
		<property name="username" value="root" />
		<property name="password" value="root" />
		<property name="url" value="jdbc:mysql://localhost:3306/testdb" />
	</bean>

</beans>
```