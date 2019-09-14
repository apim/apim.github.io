# Hibernate - One to One

This tutorial will demonstrate Hibernate one-to-one relationship using annotation. The example contains an *Employee (employee_one_to_one) table* and a *Laptop table* where each employee has one laptop. Table creation scripts are shown at first to illustrate the foreign-key relationships. Moreover, we will reuse the sample Maven project created in earlier tutorial.

```sql
CREATE TABLE `employee_one_to_one` (
	`ID` int(11) NOT NULL,
	`NAME` varchar(255) DEFAULT NULL,
	`DESIGNATION` varchar(255) DEFAULT NULL,
	`SALARY` int(11) DEFAULT NULL,
	PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `laptop` (
	`EID` int(11) NOT NULL,
	`BRAND` varchar(255) DEFAULT NULL,
	PRIMARY KEY (`EID`),
	CONSTRAINT `FK_1` FOREIGN KEY (`EID`) REFERENCES `employee_one_to_one` (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

For Hibernate one to one association, there is always one *owner* and one *inverse* side. Inverse side points to owner side by *foreign key*. Let's start with inverse side first. The class **EmployeeO2O** is in inverse side here and it contains a laptop field that is annotated by `@OneToOne` and contains the attribute `mappedBy`. This attribute points to the foreign key of the joined relation. The `cascade` attribute tells Hibernate how to update the child entity in a relation when the parent entity is being modified. Value as `all` indicates to propagate all changes. Other generally used values are `save`, `save-update` etc.

```java
package apim.github.tutorial;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name = "employee_one_to_one")
public class EmployeeO2O {

	@Id
	private int id;

	@Column
	private String name;

	@Column
	private String designation;

	@Column
	private int salary;

	@OneToOne(cascade = CascadeType.ALL, mappedBy = "employee")
	private Laptop laptop;

	public EmployeeO2O() {
	}

	public EmployeeO2O(int id, String name, String designation, int salary, Laptop laptop) {
		this.id = id;
		this.name = name;
		this.designation = designation;
		this.salary = salary;
		this.laptop = laptop;
	}

	// getters and setters

}
```

Next, the owner side i.e. **Laptop**. Here the identifier column has been decorated with two additional annotations - `@GenericGenerator` and `@GeneratedValue`. The first one defines a *generator* which will indicate Hibernate how to use the foreign key, it's attribute `@Parameter` tells which class member variable actually points to the joined table. The second annotation takes in use of the defined generator. Following this, the main annotation `@OneToOne` is defined on the field (employee) which is the joined relation. Finally, the `@PrimaryKeyJoinColumn` tells that the primary key of the used relation to be used as joining column pointed by foreign key.

```java
package apim.github.tutorial;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.PrimaryKeyJoinColumn;

import org.hibernate.annotations.GenericGenerator;
import org.hibernate.annotations.Parameter;

@Entity
public class Laptop {

	@Id
	@GenericGenerator(name = "generator", strategy = "foreign", parameters = @Parameter(name = "property", value = "employee"))
	@GeneratedValue(generator = "generator")
	@Column(name = "eid")
	private int employeeId;

	@Column
	private String brand;

	@OneToOne
	@PrimaryKeyJoinColumn
	private EmployeeO2O employee;
	
	public Laptop() {
	}

	public Laptop(String brand) {
		this.brand = brand;
	}

	// getters and setters

}
```

Now write the test class **TestCodeO2O** to verify these features. Construct of this class is straight-forward. Hibernate loading is also same as previous tutorial.

```java
package apim.github.tutorial;

import java.util.List;
import java.util.Properties;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class TestCodeO2O {

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
				.addAnnotatedClass(EmployeeO2O.class)
				.addAnnotatedClass(Laptop.class)
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
			Laptop lp1 = new Laptop("Sony");
			Laptop lp2 = new Laptop("Dell");
			EmployeeO2O emp1 = new EmployeeO2O(1, "Rashid", "Developer", 100, lp1);
			EmployeeO2O emp2 = new EmployeeO2O(2, "Abdul", "Senior Developer", 140, lp2);
			lp1.setEmployee(emp1);
			lp2.setEmployee(emp2);
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
			List result = session.createQuery("from EmployeeO2O").list();
			for (Object obj : result) {
				EmployeeO2O e = (EmployeeO2O) obj;
				System.out.print("Name: " + e.getName() + ", Designation: " + e.getDesignation() + ", Salary: " + e.getSalary());
				System.out.println(", Laptop: " + e.getLaptop().getBrand());
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

Upon successful execution, above tables shall have content like below.

```
mysql> select * from testdb.employee_one_to_one;
+----+--------+------------------+--------+
| ID | NAME   | DESIGNATION      | SALARY |
+----+--------+------------------+--------+
|  1 | Rashid | Developer        |    100 |
|  2 | Abdul  | Senior Developer |    140 |
+----+--------+------------------+--------+
2 rows in set (0.00 sec)

mysql> select * from testdb.laptop;
+-----+-------+
| EID | BRAND |
+-----+-------+
|   1 | Sony  |
|   2 | Dell  |
+-----+-------+
2 rows in set (0.00 sec)
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/hibernate-tutorial)