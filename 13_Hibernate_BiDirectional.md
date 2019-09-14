# Hibernate - One to Many & Many to One

One to Many relationship in Hibernate can be accomplished in 3 ways - either by use of a *Set* (all unique elements) or by some *List* (may have duplicates) or by some *Map* (key-value pairs). For the sake of brevity, only list variant will be discussed here. Remaining types are quite self explanatory as per their Java types. The example contains an *Employee (re used from first Hibernate tutorial) table* and an *Address table* where each employee can have many addresses. Table creation scripts are shown at first to illustrate the foreign-key relationships. Moreover, we will reuse the sample Maven project created in the first Hibernate tutorial.

```sql
CREATE TABLE `address` (
	`ADDRESSID` int(11) NOT NULL,
	`PLOTNO` varchar(255) DEFAULT NULL,
	`STREET` varchar(255) DEFAULT NULL,
	`LOCATION` varchar(255) DEFAULT NULL,
	`CITY` varchar(255) DEFAULT NULL,
	`EID` int(11) DEFAULT NULL,
	PRIMARY KEY (`ADDRESSID`),
	KEY `FK_2` (`EID`),
	CONSTRAINT `FK_2` FOREIGN KEY (`EID`) REFERENCES `employee` (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

This tutorial shows a *bi-directional* relation where one side is **many-to-one** and the other side is **one-to-many**. Likewise with *one-to-one* relation, here is one *owner* side and one *inverse* side. For the **EmployeeBiDi.java**, the relation is in inverse side because here `@OneToMany` annotation contains the attribute `mappedBy`. For the **Address.java**, the relation is in owner side because along with the `@ManyToOne` annotation, it contains the `@JoinColumn` annotation which tells Hibernate that the corresponding table has a column with a foreign key to the referenced table.

```java
package apim.github.tutorial;

import java.util.List;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Table;

@Entity
@Table(name = "employee")
public class EmployeeBiDi {

	@Id
	private int id;

	@Column
	private String name;

	@Column
	private String designation;

	@Column
	private int salary;

	@OneToMany(cascade = CascadeType.ALL, mappedBy = "employee")
	private List<Address> addresses;

	public EmployeeBiDi() {
	}

	public EmployeeBiDi(int id, String name, String designation, int salary) {
		this.id = id;
		this.name = name;
		this.designation = designation;
		this.salary = salary;
	}

	// getters and setters

}
```

```java
package apim.github.tutorial;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;

@Entity
public class Address {

	@Id
	private int addressId;

	@Column
	private String plotNo;

	@Column
	private String street;

	@Column
	private String location;

	@Column
	private String city;

	@ManyToOne
	@JoinColumn(name = "eid")
	private EmployeeBiDi employee;

	public Address() {
	}

	public Address(int addressId, String plotNo, String street, String location, String city) {
		this.addressId = addressId;
		this.plotNo = plotNo;
		this.street = street;
		this.location = location;
		this.city = city;
	}

	// getters and setters

}
```

Now write the test class **TestCodeBiDi** to verify these features. Construct of this class is straight-forward. Hibernate loading is also same as previous tutorial.

```java
package apim.github.tutorial;

import java.util.LinkedList;
import java.util.List;
import java.util.Properties;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class TestCodeBiDi {

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
				.addAnnotatedClass(EmployeeBiDi.class)
				.addAnnotatedClass(Address.class)
				.buildSessionFactory();

		createRecord(factory);
		readRecord(factory);

		factory.close();
	}

	private static void createRecord(SessionFactory factory) {
		Session session = null;
		Transaction tx = null;
		try {
			session = factory.openSession();
			tx = session.beginTransaction();
			EmployeeBiDi emp = new EmployeeBiDi(10, "Peter", "Analyst", 150);
			Address addr1 = new Address(1, "765A", "Birk Road", "Downtown", "Tokyo");
			addr1.setEmployee(emp);
			Address addr2 = new Address(2, "407/1", "Creek Lane", "Upper Area", "Ohio");
			addr2.setEmployee(emp);
			List<Address> addrs = new LinkedList<>();
			addrs.add(addr1);
			addrs.add(addr2);
			emp.setAddresses(addrs);
			session.save(emp);
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
			List result = session.createQuery("from EmployeeBiDi").list();
			for (Object obj : result) {
				EmployeeBiDi e = (EmployeeBiDi) obj;
				System.out.println("Name: " + e.getName() + ", Designation: " + e.getDesignation() + ", Salary: " + e.getSalary());
				for (Address addr : e.getAddresses()) {
					System.out.println("Address: " + addr.getPlotNo() + ", " + addr.getStreet() + ", " + addr.getLocation());
				}
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

}
```

Upon successful execution, above tables shall have content like below (presuming no previous records present).

```
mysql> select * from testdb.employee;
+----+-------+-------------+--------+
| ID | NAME  | DESIGNATION | SALARY |
+----+-------+-------------+--------+
| 10 | Peter | Analyst     |    150 |
+----+-------+-------------+--------+
2 rows in set (0.00 sec)

mysql> select * from testdb.address;
+-----------+--------+------------+------------+-------+------+
| ADDRESSID | PLOTNO | STREET     | LOCATION   | CITY  | EID  |
+-----------+--------+------------+------------+-------+------+
|         1 | 765A   | Birk Road  | Downtown   | Tokyo |   10 |
|         2 | 407/1  | Creek Lane | Upper Area | Ohio  |   10 |
+-----------+--------+------------+------------+-------+------+
2 rows in set (0.00 sec)
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/hibernate-tutorial)