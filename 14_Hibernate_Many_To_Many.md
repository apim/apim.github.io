# Hibernate - Many to Many

This tutorial will demonstrate Hibernate many-to-many relationship. The example contains an *Employee (employee_many_to_many) table* and an *Project table*. For all M:N entity relationships, there is a separate table holding the relation. Here we have *EMP_PRJ table* for the same. For our use case, many employees can belong to one project and at the same time one employee can work for multiple projects. Table creation scripts are shown at first to illustrate the foreign-key relationships. Moreover, we will reuse the sample Maven project created in the first Hibernate tutorial.

```sql
CREATE TABLE `employee_many_to_many` (
	`ID` int(11) NOT NULL,
	`NAME` varchar(255) DEFAULT NULL,
	`DESIGNATION` varchar(255) DEFAULT NULL,
	`SALARY` int(11) DEFAULT NULL,
	PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `project` (
	`PROJECTID` int(11) NOT NULL,
	`NAME` varchar(255) DEFAULT NULL,
	`CLIENT` varchar(255) DEFAULT NULL,
	PRIMARY KEY (`PROJECTID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `emp_prj` (
	`EID` int(11) NOT NULL,
	`PID` int(11) NOT NULL,
	PRIMARY KEY (`PID`,`EID`),
	KEY `FK_3` (`EID`),
	CONSTRAINT `FK_3` FOREIGN KEY (`EID`) REFERENCES `employee_many_to_many` (`ID`),
	CONSTRAINT `FK_4` FOREIGN KEY (`PID`) REFERENCES `project` (`PROJECTID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Likewise earlier case, many-to-many association too has one *owner* side and one *inverse* side. **EmployeeM2M.java** has the annotation `@JoinTable`, which in turn contains `@JoinColumn`, that indicates to Hibernate that this entity is the relation *owner*. The annotation `@JoinColumn` tells which column in the mapping table *(EMP_PRJ)* is the key from this entity and the `inverseJoinColumn` attribute tells which column in the mapping table maps to the other entity *(Project)*. On the other hand, **Project.java** contains inverse side. This is determined by the `mappedBy` field within the `@ManyToMany` annotation. This tells that the said attribute (*projects* in this case) owns the relation.

```java
package apim.github.tutorial;

import java.util.Set;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;
import javax.persistence.Table;

@Entity
@Table(name = "employee_many_to_many")
public class EmployeeM2M {

	@Id
	private int id;

	@Column
	private String name;

	@Column
	private String designation;

	@Column
	private int salary;

	@ManyToMany(cascade = CascadeType.ALL)
	@JoinTable(name = "emp_prj", joinColumns = @JoinColumn(name = "eid"), inverseJoinColumns = @JoinColumn(name = "pid"))
	private Set<Project> projects;

	public EmployeeM2M() {
	}

	public EmployeeM2M(int id, String name, String designation, int salary) {
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

import java.util.Set;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.ManyToMany;

@Entity
public class Project {

	@Id
	@Column
	private int projectId;

	@Column
	private String name;

	@Column
	private String client;

	@ManyToMany(mappedBy = "projects")
	private Set<EmployeeM2M> employees;

	public Project() {
	}

	public Project(int projectId, String name, String client) {
		this.projectId = projectId;
		this.name = name;
		this.client = client;
	}

	// getters and setters

}
```

Now write the test class **TestCodeM2M** to verify these features. Construct of this class is straight-forward. Hibernate loading is also same as previous tutorial.

```java
package apim.github.tutorial;

import java.util.HashSet;
import java.util.List;
import java.util.Properties;
import java.util.Set;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class TestCodeM2M {

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
				.addAnnotatedClass(EmployeeM2M.class)
				.addAnnotatedClass(Project.class)
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
			EmployeeM2M emp1 = new EmployeeM2M(1, "John", "Coder", 50);
			EmployeeM2M emp2 = new EmployeeM2M(2, "Jenny", "Coder", 50);
			EmployeeM2M emp3 = new EmployeeM2M(3, "Jim", "Coder", 50);
			Project prj1 = new Project(1, "Development", "ATT");
			Project prj2 = new Project(2, "Testing", "OSP");
			Set<Project> projects1 = new HashSet<>();
			projects1.add(prj1);
			emp1.setProjects(projects1);
			Set<Project> projects2 = new HashSet<>();
			projects2.add(prj1);
			projects2.add(prj2);
			emp2.setProjects(projects2);
			Set<Project> projects3 = new HashSet<>();
			projects3.add(prj1);
			emp3.setProjects(projects3);
			session.save(emp1);
			session.save(emp2);
			session.save(emp3);
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
			List result = session.createQuery("from EmployeeM2M").list();
			for (Object obj : result) {
				EmployeeM2M e = (EmployeeM2M) obj;
				System.out.println("Name: " + e.getName() + ", Designation: " + e.getDesignation() + ", Salary: " + e.getSalary());
				for (Project prj : e.getProjects()) {
					System.out.println("Project of " + e.getName() + ": " + prj.getName() + ", " + prj.getClient());
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
mysql> select * from testdb.employee_many_to_many;
+----+-------+-------------+--------+
| ID | NAME  | DESIGNATION | SALARY |
+----+-------+-------------+--------+
|  1 | John  | Coder       |     50 |
|  2 | Jenny | Coder       |     50 |
|  3 | Jim   | Coder       |     50 |
+----+-------+-------------+--------+
3 rows in set (0.00 sec)

mysql> select * from testdb.project;
+-----------+-------------+--------+
| PROJECTID | NAME        | CLIENT |
+-----------+-------------+--------+
|         1 | Development | ATT    |
|         2 | Testing     | OSP    |
+-----------+-------------+--------+
2 rows in set (0.00 sec)

mysql> select * from testdb.emp_prj;
+-----+-----+
| EID | PID |
+-----+-----+
|   1 |   1 |
|   2 |   1 |
|   2 |   2 |
|   3 |   1 |
+-----+-----+
4 rows in set (0.00 sec)
```

**Complete source code for this tutorial:** [GitHub](https://github.com/apim/hibernate-tutorial)