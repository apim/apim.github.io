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

Now create the main spring configuration **spring-context.xml**. Important point to note in this file is the inclusion of `spring-jdbc` namespace. Next follows usual component scanning for annotation based bean definitions and *datasource* definition which uses MySql driver. Give attention to the properties as you have to edit the credentials & database name as per your environment.

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

Next, a simple POJO class **Book.java**. Note that this is not marked with annotations to work as a Spring bean.

```java
package apim.github.tutorial;

public class Book {

	private int isbn;
	private String title;
	private String author;

	// getters and setters

}
```

After this the RowMapper implementation: **BookRowMapper.java**

```java
package apim.github.tutorial;

import java.sql.ResultSet;
import java.sql.SQLException;

import org.springframework.jdbc.core.RowMapper;

public class BookRowMapper implements RowMapper<Book> {

	@Override
	public Book mapRow(ResultSet rs, int rowId) throws SQLException {
		Book book = new Book();
		book.setIsbn(rs.getInt(1));
		book.setTitle(rs.getString(2));
		book.setAuthor(rs.getString(3));
		return book;
	}

}
```

Now comes the crucial piece of code - the DAO class. Here **BookDAO.java** is developed which is marked as spring bean and also datasource is autowired within. *JdbcTemplate* is instantiated from the injected datasource. Several methods are defined to cover all sorts of database operations in practice and they use above-mentioned APIs from JdbcTemplate.

```java
package apim.github.tutorial;

import java.util.List;

import javax.annotation.Resource;
import javax.sql.DataSource;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class BookDAO {

	private DataSource dataSource;

	private JdbcTemplate jdbcTemplate;

	public DataSource getDataSource() {
		return dataSource;
	}

	@Resource
	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
		jdbcTemplate = new JdbcTemplate(dataSource);
	}

	public void createBookTable() {
		String query = "create table book (isbn integer not null primary key, title varchar(45) not null, author varchar(45) not null)";
		jdbcTemplate.execute(query);
	}

	public void addBook(int isbn, String title, String author) {
		String query = "insert into book values(?,?,?)";
		jdbcTemplate.update(query, isbn, title, author);
	}

	public String getBookTitleByIsbn(int isbn) {
		String query = "select title from book where isbn=?";
		return jdbcTemplate.queryForObject(query, String.class, isbn);
	}

	public Book getBookByAuthor(String author) {
		String query = "select * from book where author=?";
		return jdbcTemplate.queryForObject(query, new Object[] { author }, new BookRowMapper());
	}

	public List<Book> getAllBooks() {
		String query = "select * from book";
		return jdbcTemplate.query(query, new BookRowMapper());
	}

}
```

Finally, the main class **TestCode.java** to test the code.

```java
package apim.github.tutorial;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestCode {

	public static void main(String args[]) {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		BookDAO dao = (BookDAO) ctx.getBean("bookDAO");
		dao.createBookTable();
		dao.addBook(101, "The Target", "David Baldacci");
		dao.addBook(202, "Hunger Games", "Suzanne Collins");
		System.out.println(dao.getBookTitleByIsbn(101));
		Book book = dao.getBookByAuthor("Suzanne Collins");
		System.out.println(book.getIsbn() + ", " + book.getTitle() + ", " + book.getAuthor());
		List<Book> books = dao.getAllBooks();
		for (Book bk : books) {
			System.out.println(bk.getIsbn() + ", " + bk.getTitle() + ", " + bk.getAuthor());
		}
		ctx.close();
	}

}
```

Now executing the above shall produce output like below. Also verify (by MySql CLI or your preferred way) that database table is created and populated accordingly.

```
The Target
202, Hunger Games, Suzanne Collins
101, The Target, David Baldacci
202, Hunger Games, Suzanne Collins
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-jdbc-tutorial)