# Spring with Hibernate

Spring provides easy integration with Hibernate via inbuilt **HibernateTemplate** class which behaves much alike *JdbcTemplate*. In this tutorial Spring 5.x and Hibernate 5.x integration will be discussed. The code snippet would be little different while working with older Hibernate versions (not covered here).

Start with creating a new Maven Java project in Eclipse as *spring-hibernate-tutorial*. First element is the standard **pom.xml** file - noteworthy dependency here is *spring-orm*.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>apim.github.tutorial</groupId>
	<artifactId>spring-hibernate</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring Hibernate Tutorial</name>

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
			<artifactId>spring-jdbc</artifactId>
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

The most important piece of code for Spring Hibernate integration is the Spring context configuration. Therefore, we will start with **spring-context.xml**. Namespaces included here are the standard context & beans, in addition, for automatic spring transaction support - *spring-tx* has been included. The elements `context:component-scan` and `dataSource` bean are self-explanatory, similar of them has already been used in other examples. The important new one is the `sessionFactory` - it configures Hibernate session factory using the *dataSource*. The next important element is the `transactionManager` - this is to use Hibernate in-built transaction management. Spring support various transaction modes and details of that will be covered in a future tutorial. The tag `tx:annotation-driven` is used to enable the `@Transactional` annotation within Java code viz. the DAO classes.

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

	<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.hbm2ddl.auto">update</prop>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</prop>
			</props>
		</property>
		<property name="packagesToScan" value="apim.github.tutorial" />
	</bean>

	<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

</beans>
```

Next, the **Customer.java** which is a simple entity bean. 

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

Now we code the crucial Java piece of our example - the DAO class **CustomerDAO.java**. Here at first the sessionFactory is autowired by Spring. Then within the setter method we initialise the HibernateTemplate instance which will do the actual database operations. Each DB method is annotated with *@Transactional*, use of this annotation is very important to notify Spring to do the underlying session opening, transaction opening and then further commit/rollback and finally session closing tasks. The API *loadAll* selects all rows from a table. The API *findByCriteria* uses a selection clause (defined in Java by *DetachedCriteria* class) to retrieve selective records.

```java
package apim.github.tutorial;

import java.util.List;

import javax.annotation.Resource;
import javax.transaction.Transactional;

import org.hibernate.SessionFactory;
import org.hibernate.criterion.DetachedCriteria;
import org.hibernate.criterion.Restrictions;
import org.springframework.orm.hibernate5.HibernateTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class CustomerDAO {

	private SessionFactory sessionFactory;

	private HibernateTemplate hibernateTemplate;

	public SessionFactory getSessionFactory() {
		return sessionFactory;
	}

	@Resource
	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
		hibernateTemplate = new HibernateTemplate(sessionFactory);
	}

	@Transactional
	public void addCustomer(Customer c) {
		hibernateTemplate.save(c);
	}

	@Transactional
	public List<Customer> getAllCustomers() {
		return hibernateTemplate.loadAll(Customer.class);
	}

	@SuppressWarnings("unchecked")
	@Transactional
	public List<Customer> findCustomerByBalance(int amt) {
		DetachedCriteria criteria = DetachedCriteria.forClass(Customer.class);
		criteria.add(Restrictions.gt("balance", amt));
		return (List<Customer>) hibernateTemplate.findByCriteria(criteria);
	}

}
```

Finally, the main class to do testing - *TestCode.java*. Code here is almost same as the test code with Spring JDBC tutorial and the class itself is straight-forward.

```java
package apim.github.tutorial;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestCode {

	public static void main(String args[]) {
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/spring-context.xml");
		CustomerDAO dao = (CustomerDAO) ctx.getBean("customerDAO");
		dao.addCustomer(new Customer(10, "Noah", 10000));
		dao.addCustomer(new Customer(20, "Alley", 8500));
		List<Customer> list = dao.findCustomerByBalance(9000);
		for (Customer c : list) {
			System.out.println(c.getId() + ": " + c.getName());
		}
		ctx.close();
	}

}
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/spring-hibernate-tutorial)