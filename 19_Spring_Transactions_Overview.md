# Spring Transactions Overview

Transaction Management in Spring is available in two flavours - **Declarative** and **Programmatic**. Declarative transaction, which practically almost requires zero coding for transaction management, is what has been used in all earlier tutorials. This is done by primarily by use of annotation `@Transactional`. Programmatic transaction, which gives better control and flexibility than its counterpart but requires more coding, is going to be discussed in this tutorial. However, the customisation we do for programmatic transaction are also doable for declarative transaction by using the various attributes of @Transactional like `readOnly`, `isolation`, `propagation`, `rollbackFor`, `noRollbackFor` etc.

Programmatic transaction is achieved by two means - by use of `PlatformTransactionManager` or by using `TransactionTemplate` (very similar to JdbcTemplate). In this tutorial we will discuss the former. Let's begin with some properties of Spring Transaction Management. Using PlatformTransactionManager implementation, our aim is to derive some implementation of `TransactionDefinition` interface. There are some important aspects of this interface which can be configured in the end-user application.

In short, there are 4 important methods in TransactionDefinition and different combination of values being set there (in the concrete class implementing the interface TransactionDefinition) creates different transactional use cases.

* `getTimeout()` - returns transaction's timeout value (-1 refers to never)
* `isReadOnly()` - tells whether a transaction can make database changes or not (likely set to 'true' for read operations)
* `getIsolationLevel()` - has all the ANSI/ISO transaction isolation level values plus one spring specific one. Below diagram gives a brief overview of all these

![](/images/tx_isolations.jpg)

* `getPropagationBehavior()` - is used when there are multiple operations / methods are included and the value set here dictates how the case of transaction initiated / not initiated in the first method is handled in the second method. Below diagram gives a brief overview of all these

![](/images/tx_propagations.jpg)

In user application if we want to use default implementation of TransactionDefinition interface, then the class `DefaultTransactionDefinition` is instantiated. It comes with values set as *PROPAGATION_REQUIRED*, *ISOLATION_DEFAULT*, *TIMEOUT_DEFAULT*, *readOnly=false*. On the other hand, for a fully customised transaction management, abstract class `DelegatingTransactionDefinition` (implements TransactionDefinition) shall be extended and used.