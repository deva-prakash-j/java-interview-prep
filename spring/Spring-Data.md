# Spring Data in Spring Boot: A Comprehensive Guide

Spring Data is a part of the larger Spring ecosystem that aims to simplify data access and persistence. It provides a consistent and robust way to interact with various data stores, whether relational (like MySQL, PostgreSQL) or non-relational (like MongoDB, Redis). When integrated with Spring Boot, it offers auto-configuration and starter dependencies to minimize boilerplate code.

---

## 🔰 1. Core Modules of Spring Data

- **Spring Data JPA**: Provides JPA-based repository support.
- **Spring Data MongoDB**: Support for MongoDB.
- **Spring Data Redis**: Support for Redis.
- **Spring Data JDBC**: Lightweight alternative to JPA.
- **Spring Data REST**: Exposes repositories as REST endpoints automatically.

---

## 🛠️ 2. Basic Setup with Spring Boot

### Maven Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

### Application Properties
```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

## 🧩 3. Entity Mapping

### Entity Class
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String product;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

### Common JPA Annotations for Entity Classes

| Annotation | Description | Example |
|------------|-------------|---------|
| `@Entity` | Specifies that the class is an entity and is mapped to a database table. | `@Entity public class User {}` |
| `@Table` | Specifies the table name. | `@Table(name = "users")` |
| `@Id` | Specifies the primary key. | `@Id private Long id;` |
| `@GeneratedValue` | Specifies the primary key generation strategy. | `@GeneratedValue(strategy = GenerationType.IDENTITY)` |
| `@Column` | Defines column mapping and constraints. | `@Column(name = "username", nullable = false)` |
| `@OneToOne` | Defines a one-to-one relationship. | `@OneToOne @JoinColumn(name = "profile_id") private Profile profile;` |
| `@OneToMany` | Defines a one-to-many relationship. | `@OneToMany(mappedBy = "user")` |
| `@ManyToOne` | Defines a many-to-one relationship. | `@ManyToOne @JoinColumn(name = "user_id")` |
| `@ManyToMany` | Defines a many-to-many relationship. | `@ManyToMany @JoinTable(...)` |
| `@JoinColumn` | Specifies the foreign key column. | `@JoinColumn(name = "user_id")` |
| `@JoinTable` | Defines the join table for many-to-many. | `@JoinTable(name = "user_role", joinColumns = ..., inverseJoinColumns = ...)` |
| `@Lob` | Maps a large object (e.g., BLOB or CLOB). | `@Lob private String description;` |
| `@Enumerated` | Maps enums to the database. | `@Enumerated(EnumType.STRING)` |
| `@Temporal` | Specifies temporal data types for `Date`. | `@Temporal(TemporalType.TIMESTAMP)` |
| `@Transient` | Marks a field as non-persistent. | `@Transient private int temp;` |
| `@Version` | For optimistic locking. | `@Version private int version;` |
| `@Embedded` | Embeds a value type object. | `@Embedded private Address address;` |
| `@Embeddable` | Marks a class to be embedded. | `@Embeddable public class Address {}` |

---

## 📦 4. Repository Layer

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameContaining(String keyword);
}

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByUserId(Long userId);
}
```

### Default Repository Method Naming Conventions

| Method Name | Description | Example |
|-------------|-------------|---------|
| `findBy` | Fetches data by a field | `findByUsername(String username)` |
| `findBy...And...` | Combines multiple conditions with AND | `findByFirstNameAndLastName(String fn, String ln)` |
| `findBy...Or...` | Combines multiple conditions with OR | `findByFirstNameOrLastName(String fn, String ln)` |
| `findBy...Between` | Range query | `findByAgeBetween(int start, int end)` |
| `findBy...LessThan` | Less than comparison | `findBySalaryLessThan(double maxSalary)` |
| `findBy...Like` | Pattern matching | `findByNameLike(String pattern)` |
| `findBy...OrderBy...Asc/Desc` | Sorting results | `findByCityOrderByAgeDesc(String city)` |
| `existsBy...` | Checks existence | `existsByEmail(String email)` |
| `countBy...` | Returns count of records | `countByStatus(String status)` |
| `deleteBy...` | Deletes by condition | `deleteByUsername(String username)` |

### Custom Query Methods

#### JPQL Example
```java
@Query("SELECT u FROM User u WHERE u.email = ?1")
User findByEmail(String email);
```

#### Native Query Example
```java
@Query(value = "SELECT * FROM users WHERE status = ?1", nativeQuery = true)
List<User> findByStatusNative(String status);
```

#### Using SpEL in Queries
```java
@Query("SELECT u FROM User u WHERE u.name = :#{#user.name}")
List<User> findUsersWithName(@Param("user") User user);
```

#### Dynamic Sorting and Paging
```java
Page<User> findByStatus(String status, Pageable pageable);
List<User> findByAgeGreaterThan(int age, Sort sort);
```

---

## 🔄 5. Transaction Management

### Declarative Transactions
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
        // other transactional logic
    }
}
```

### Programmatic Transactions
```java
@Autowired
private PlatformTransactionManager transactionManager;

public void manualTransaction() {
    TransactionTemplate template = new TransactionTemplate(transactionManager);
    template.execute(status -> {
        // transactional logic here
        return null;
    });
}
```

## 🔀 6. Handling Multiple Transactions/DataSources

### Basic Setup for Multiple DataSources

#### application.properties
```properties
spring.datasource1.url=jdbc:mysql://localhost:3306/db1
spring.datasource1.username=root
spring.datasource1.password=secret
spring.datasource1.driver-class-name=com.mysql.cj.jdbc.Driver

spring.datasource2.url=jdbc:mysql://localhost:3306/db2
spring.datasource2.username=root
spring.datasource2.password=secret
spring.datasource2.driver-class-name=com.mysql.cj.jdbc.Driver
```

### Multiple DataSources Configuration
```java
@Configuration
public class DataSourceConfig {

    @Bean(name = "dataSource1")
    @Primary
    @ConfigurationProperties("spring.datasource1")
    public DataSource dataSource1() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "dataSource2")
    @ConfigurationProperties("spring.datasource2")
    public DataSource dataSource2() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "transactionManager1")
    public PlatformTransactionManager transactionManager1(
            @Qualifier("dataSource1") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "transactionManager2")
    public PlatformTransactionManager transactionManager2(
            @Qualifier("dataSource2") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

### Using Specific Transaction Manager
```java
@Transactional("transactionManager1")
public void serviceMethod() {
    // logic here
}
```

---

## 📊 7. Projections and DTOs

### Interface-Based Projection
```java
public interface UserNameOnly {
    String getName();
}

List<UserNameOnly> findBy();
```

### DTO Projection
```java
public class UserDTO {
    private String name;
    private int orderCount;

    public UserDTO(String name, int orderCount) {
        this.name = name;
        this.orderCount = orderCount;
    }
}

@Query("SELECT new com.example.UserDTO(u.name, size(u.orders)) FROM User u")
List<UserDTO> fetchUserStats();
```

---

## 🚀 8. Advanced JPA Features

- **Entity Graphs**: Fetch strategies
- **Query By Example (QBE)**
- **Specifications**: For dynamic queries

### Example: Specifications
```java
public class UserSpecifications {
    public static Specification<User> hasName(String name) {
        return (root, query, cb) -> cb.equal(root.get("name"), name);
    }
}

userRepository.findAll(UserSpecifications.hasName("Alice"));
```

---

## 📡 9. Spring Data REST

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

```java
@RepositoryRestResource
public interface UserRepository extends JpaRepository<User, Long> {}
```

This exposes endpoints like:
- `GET /users`
- `POST /users`

---

## 🔒 10. Best Practices

- Use DTOs to avoid exposing entities.
- Prefer constructor injection over field injection.
- Use `@Transactional` at the service layer.
- Split read/write transactions when using CQRS.
- Consider Spring Data JDBC for lightweight scenarios.

---

## 💼 11. Programmatic Data Management in a Banking System

In addition to declarative transactions, Spring allows fine-grained control over transactions using programmatic approaches. This is useful for complex business logic, conditional rollbacks, or nested transactions.

### Example Domain: Banking System
We'll define a `BankAccount` entity and demonstrate transactional money transfer logic.

### Entity Class
```java
@Entity
public class BankAccount {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String accountNumber;

    private double balance;
}
```

### Repository Interface
```java
public interface BankAccountRepository extends JpaRepository<BankAccount, Long> {
    Optional<BankAccount> findByAccountNumber(String accountNumber);
}
```

### Service Layer with Programmatic Transaction Management
```java
@Service
public class BankingService {

    private final BankAccountRepository repository;
    private final PlatformTransactionManager transactionManager;

    public BankingService(BankAccountRepository repository, PlatformTransactionManager transactionManager) {
        this.repository = repository;
        this.transactionManager = transactionManager;
    }

    public void transfer(String fromAccount, String toAccount, double amount) {
        TransactionTemplate template = new TransactionTemplate(transactionManager);
        template.execute(status -> {
            BankAccount sender = repository.findByAccountNumber(fromAccount)
                    .orElseThrow(() -> new RuntimeException("Sender not found"));

            BankAccount receiver = repository.findByAccountNumber(toAccount)
                    .orElseThrow(() -> new RuntimeException("Receiver not found"));

            if (sender.getBalance() < amount) {
                throw new RuntimeException("Insufficient funds");
            }

            sender.setBalance(sender.getBalance() - amount);
            receiver.setBalance(receiver.getBalance() + amount);

            repository.save(sender);
            repository.save(receiver);

            return null;
        });
    }
}
```

### Available TransactionTemplate Methods

| Method | Description |
|--------|-------------|
| `execute(TransactionCallback<T>)` | Executes the transactional code and commits or rolls back depending on exceptions. |
| `setIsolationLevel(int)` | Sets isolation level for the transaction. |
| `setTimeout(int)` | Timeout in seconds. |
| `setReadOnly(boolean)` | Marks the transaction as read-only. |

### Advanced Transaction Configurations

- **Nested Transactions**: Supported only if the underlying platform supports it (e.g., with savepoints).
- **Rollback Rules**:
```java
@Transactional(rollbackFor = {CustomException.class}, noRollbackFor = {IgnoredException.class})
```
- **Propagation Options**: Like `REQUIRES_NEW`, `NESTED`, `SUPPORTS`, etc., can be configured using `@Transactional(propagation = Propagation.REQUIRES_NEW)`

