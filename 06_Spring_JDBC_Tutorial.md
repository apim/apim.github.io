# Spring JDBC Tutorial

Spring provides JDBC support via **JdbcTemplate** class. Main benefit is that while it brings down boiler-plate coding to a minimum, at the same time, there is ample scope of DB query customisation. Use of this template requires a *datasource* configuration and the example here uses *Apache DBCP*. This datasource is usually configured in the spring context file along with other bean definitions (if not using annotation).

Next important part of Spring JDBC is writing a *DAO class* (Data Access Object class). This DAO class is the actual bean (**@Repository** can be used for this) where the *JdbcTemplate* is used (this or the datasource has to be injected hence **@Resource** can be used here). Likewise the DAO pattern, all database related methods are defined here. *JdbcTemplate* supports full *CRUD* operations. There are different methods for this purpose and a brief overview of them is given below.

- **execute()** - To run a DDL command like create or drop table
- **update()** - To run a DML command like insert or update table
- **queryFor<XXX>()** - To read data from table for various primitive data types (e,g. XXX has forms like Int, Double etc.)
- **queryForObject()** - To read data from table for an object (**RowMapper** implementation is used)
- **query()** - To read multiple records from table matching one object (**RowMapper** implementation is used)

For reading table records directly into an object, some implementation of `RowMapper` has to be used. This class will actually do the JDBC *ResultSet* to object/bean transformation. The class will be having only one method `(mapRow(resultSet, rowId))` overridden for all usual purposes. To use all these, developer simply has to obtain the bean of DAO via Spring and then invoke the necessary methods.