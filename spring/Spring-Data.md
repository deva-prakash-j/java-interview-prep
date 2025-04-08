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

### 3.1 Entity Class

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

| Annotation        | Description                                                              | Example                                                                       |
| ----------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| `@Entity`         | Specifies that the class is an entity and is mapped to a database table. | `@Entity public class User {}`                                                |
| `@Table`          | Specifies the table name.                                                | `@Table(name = "users")`                                                      |
| `@Id`             | Specifies the primary key.                                               | `@Id private Long id;`                                                        |
| `@GeneratedValue` | Specifies the primary key generation strategy.                           | `@GeneratedValue(strategy = GenerationType.IDENTITY)`                         |
| `@Column`         | Defines column mapping and constraints.                                  | `@Column(name = "username", nullable = false)`                                |
| `@OneToOne`       | Defines a one-to-one relationship.                                       | `@OneToOne @JoinColumn(name = "profile_id") private Profile profile;`         |
| `@OneToMany`      | Defines a one-to-many relationship.                                      | `@OneToMany(mappedBy = "user")`                                               |
| `@ManyToOne`      | Defines a many-to-one relationship.                                      | `@ManyToOne @JoinColumn(name = "user_id")`                                    |
| `@ManyToMany`     | Defines a many-to-many relationship.                                     | `@ManyToMany @JoinTable(...)`                                                 |
| `@JoinColumn`     | Specifies the foreign key column.                                        | `@JoinColumn(name = "user_id")`                                               |
| `@JoinTable`      | Defines the join table for many-to-many.                                 | `@JoinTable(name = "user_role", joinColumns = ..., inverseJoinColumns = ...)` |
| `@Lob`            | Maps a large object (e.g., BLOB or CLOB).                                | `@Lob private String description;`                                            |
| `@Enumerated`     | Maps enums to the database.                                              | `@Enumerated(EnumType.STRING)`                                                |
| `@Temporal`       | Specifies temporal data types for `Date`.                                | `@Temporal(TemporalType.TIMESTAMP)`                                           |
| `@Transient`      | Marks a field as non-persistent.                                         | `@Transient private int temp;`                                                |
| `@Version`        | For optimistic locking.                                                  | `@Version private int version;`                                               |
| `@Embedded`       | Embeds a value type object.                                              | `@Embedded private Address address;`                                          |
| `@Embeddable`     | Marks a class to be embedded.                                            | `@Embeddable public class Address {}`                                         |

### 3.2 Understanding Entity Relationships in JPA

JPA provides annotations to define relationships between entities. Understanding these relationships is crucial for designing normalized and connected data models.

### 🔁 One-to-One
Each entity instance is related to only one instance of another entity.

```java
@Entity
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;

    @OneToOne(mappedBy = "profile")
    private User user;
}

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "profile_id")
    private Profile profile;
}
```

### 🔁 One-to-Many and Many-to-One
A single entity is related to many entities, and those entities point back to the parent.

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```

### 🔁 Many-to-Many
Both entities reference many instances of each other using a join table.

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses;
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students;
}
```

These annotations manage foreign key constraints and help Hibernate manage joins effectively. Always consider the cardinality and direction when designing relationships.

### ⚙️ Fetch Types in JPA

JPA provides two fetch types:

| Fetch Type | Description                                                                 |
|------------|-----------------------------------------------------------------------------|
| `LAZY`     | Loads related entities on demand (default for `@OneToMany`, `@ManyToMany`) |
| `EAGER`    | Loads related entities immediately (default for `@OneToOne`, `@ManyToOne`) |

#### Example:
```java
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private List<Order> orders;

@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "user_id")
private User user;
```

> **Best Practice**: Prefer `LAZY` loading by default and use fetch joins when eager loading is required to avoid N+1 problems.


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

| Method Name                   | Description                           | Example                                            |
| ----------------------------- | ------------------------------------- | -------------------------------------------------- |
| `findBy`                      | Fetches data by a field               | `findByUsername(String username)`                  |
| `findBy...And...`             | Combines multiple conditions with AND | `findByFirstNameAndLastName(String fn, String ln)` |
| `findBy...Or...`              | Combines multiple conditions with OR  | `findByFirstNameOrLastName(String fn, String ln)`  |
| `findBy...Between`            | Range query                           | `findByAgeBetween(int start, int end)`             |
| `findBy...LessThan`           | Less than comparison                  | `findBySalaryLessThan(double maxSalary)`           |
| `findBy...Like`               | Pattern matching                      | `findByNameLike(String pattern)`                   |
| `findBy...OrderBy...Asc/Desc` | Sorting results                       | `findByCityOrderByAgeDesc(String city)`            |
| `existsBy...`                 | Checks existence                      | `existsByEmail(String email)`                      |
| `countBy...`                  | Returns count of records              | `countByStatus(String status)`                     |
| `deleteBy...`                 | Deletes by condition                  | `deleteByUsername(String username)`                |

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

---

## 🧾 Transaction Concepts and Isolation Levels

Spring supports a wide variety of transaction configurations through annotations and programmatic APIs.

### 🔐 Transaction Isolation Levels

Isolation levels define the visibility of changes made inside one transaction to others. Spring uses JDBC isolation constants:

| Level                 | Constant                                  | Description                                                                 |
| --------------------- | ----------------------------------------- | --------------------------------------------------------------------------- |
| **READ\_UNCOMMITTED** | `Connection.TRANSACTION_READ_UNCOMMITTED` | Allows dirty reads (i.e., uncommitted changes).                             |
| **READ\_COMMITTED**   | `Connection.TRANSACTION_READ_COMMITTED`   | Prevents dirty reads, but non-repeatable reads can occur.                   |
| **REPEATABLE\_READ**  | `Connection.TRANSACTION_REPEATABLE_READ`  | Prevents dirty and non-repeatable reads. Phantom reads can still occur.     |
| **SERIALIZABLE**      | `Connection.TRANSACTION_SERIALIZABLE`     | Most strict. Prevents dirty reads, non-repeatable reads, and phantom reads. |

### 🔄 Propagation Behaviors

Defines how methods behave when called within an existing transaction:

| Propagation     | Description                                                          |
| --------------- | -------------------------------------------------------------------- |
| `REQUIRED`      | Join existing or create new transaction (default).                   |
| `REQUIRES_NEW`  | Always create a new transaction, suspending any current one.         |
| `SUPPORTS`      | Join if a transaction exists, else execute non-transactionally.      |
| `MANDATORY`     | Must run within a transaction. Throws exception if none.             |
| `NEVER`         | Must not run within a transaction. Throws exception if there is one. |
| `NOT_SUPPORTED` | Suspend current transaction if one exists.                           |
| `NESTED`        | Executes within a nested transaction using savepoints.               |

### 🧪 Common Transaction Properties in `@Transactional`

| Property        | Purpose                                     | Example                                                        |
| --------------- | ------------------------------------------- | -------------------------------------------------------------- |
| `readOnly`      | Optimizes for read operations               | `@Transactional(readOnly = true)`                              |
| `timeout`       | Timeout in seconds                          | `@Transactional(timeout = 30)`                                 |
| `rollbackFor`   | Specify exception types to trigger rollback | `@Transactional(rollbackFor = IOException.class)`              |
| `noRollbackFor` | Specify exception types to ignore rollback  | `@Transactional(noRollbackFor = CustomIgnoredException.class)` |
| `isolation`     | Set transaction isolation level             | `@Transactional(isolation = Isolation.SERIALIZABLE)`           |
| `propagation`   | Control nested and concurrent transactions  | `@Transactional(propagation = Propagation.REQUIRES_NEW)`       |

### 🧠 Summary Tips

- Use `REQUIRES_NEW` cautiously—it's costly and not always necessary.
- Use `readOnly = true` for queries to allow optimization by some databases.
- Isolation level choice depends on consistency needs vs performance.
- Use nested transactions only with supported databases and JDBC drivers.

---

- Use DTOs to avoid exposing entities.
- Prefer constructor injection over field injection.
- Use `@Transactional` at the service layer.
- Split read/write transactions when using CQRS.
- Consider Spring Data JDBC for lightweight scenarios.

---

## 🕵️ 12. Auditing, Caching, DDL and ID Strategies

### 🔍 Auditing in Spring Data JPA

Spring Data JPA provides a way to track creation and modification details with the help of auditing annotations.

#### Common Annotations

| Annotation          | Description                                     |
| ------------------- | ----------------------------------------------- |
| `@CreatedDate`      | Marks the field to store the creation timestamp |
| `@LastModifiedDate` | Stores the timestamp of the last update         |
| `@CreatedBy`        | Stores the user who created the entity          |
| `@LastModifiedBy`   | Stores the user who last modified the entity    |

#### Setup

```java
@Configuration
@EnableJpaAuditing
public class AuditConfig {
}
```

```java
@EntityListeners(AuditingEntityListener.class)
@Entity
public class AuditableEntity {

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    private String createdBy;

    @LastModifiedBy
    private String modifiedBy;
}
```

---

### 🚀 Caching in the Data Layer

Spring supports caching at various levels using `@EnableCaching` and cache abstraction APIs.

#### Setup

```java
@Configuration
@EnableCaching
public class CacheConfig {
}
```

#### Usage

```java
@Cacheable("users")
public User getUserById(Long id) {
    return userRepository.findById(id).orElse(null);
}

@CacheEvict("users")
public void deleteUser(Long id) {
    userRepository.deleteById(id);
}
```

---

### 🧱 Hibernate DDL Auto Strategies

The `spring.jpa.hibernate.ddl-auto` property controls schema generation:

| Value         | Description                           |
| ------------- | ------------------------------------- |
| `none`        | No changes to DB                      |
| `validate`    | Validate schema against entities      |
| `update`      | Update schema automatically           |
| `create`      | Drops and re-creates schema each time |
| `create-drop` | Drops schema on shutdown              |

Use `validate` or `none` in production.

---

### 🧮 ID Generation Strategies

JPA provides multiple strategies for ID generation via `@GeneratedValue`:

| Strategy   | Description                                         |
| ---------- | --------------------------------------------------- |
| `AUTO`     | Default; uses DB-specific strategy                  |
| `IDENTITY` | Uses DB auto-increment                              |
| `SEQUENCE` | Uses sequence object (for PostgreSQL, Oracle, etc.) |
| `TABLE`    | Uses a table to simulate sequence generation        |

#### Example

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
private Long id;
```

---

## 🧰 13. Additional Essential Concepts

### 📚 Pagination with Annotation and Programmatic Approach

#### Using Spring Data Method Signature

```java
Page<User> findByActive(boolean active, Pageable pageable);
```

#### Controller Example

```java
@GetMapping("/users")
public Page<User> getUsers(@RequestParam int page, @RequestParam int size) {
    Pageable pageable = PageRequest.of(page, size);
    return userRepository.findByActive(true, pageable);
}
```

### 🧬 Custom Repository Implementation using `JpaEntityInformation`

Create a custom interface and implementation:

#### Interface

```java
public interface CustomUserRepository {
    User findCustomById(Long id);
}
```

#### Implementation

```java
public class CustomUserRepositoryImpl implements CustomUserRepository {

    @PersistenceContext
    private EntityManager em;

    @Autowired
    private JpaEntityInformation<User, ?> entityInfo;

    @Override
    public User findCustomById(Long id) {
        return em.find(entityInfo.getJavaType(), id);
    }
}
```

Register it by extending in your repository:

```java
public interface UserRepository extends JpaRepository<User, Long>, CustomUserRepository {}
```

### 🧪 Lifecycle Events (`@PrePersist`, `@PostLoad`, etc.)

```java
@Entity
public class AuditEntity {

    @PrePersist
    public void prePersist() {
        System.out.println("Pre-persist event triggered");
    }

    @PostLoad
    public void postLoad() {
        System.out.println("Post-load event triggered");
    }
}
```

### ✅ Schema Validation Using Bean Validation

```java
@Entity
public class Product {

    @NotNull
    private String name;

    @Size(min = 3, max = 100)
    private String description;
}
```

Add dependency:

```xml
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
</dependency>
```

### 🧹 Soft Delete Handling

```java
@Entity
public class Document {

    @Id
    private Long id;

    private boolean deleted = false;
}

@Query("SELECT d FROM Document d WHERE d.deleted = false")
List<Document> findAllActive();
```

Or use Hibernate filters for more automation.

### 🚀 Performance Tuning Tips

- **Batching**: Set batch size in `application.properties`

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=30
```

- **Fetch Joins**: Use `JOIN FETCH` to reduce N+1 problem.

```java
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> fetchUsersWithOrders();
```

- **Query Hints**: Optimize queries

```java
@QueryHints(@QueryHint(name = "org.hibernate.readOnly", value = "true"))
@Query("FROM User")
List<User> findAllReadOnly();
```

---

## 🛠️ 14. EntityManager, Lifecycle Events, Bean Validation & Hibernate Filters

### 🧠 Understanding EntityManager

`EntityManager` is the central interface in JPA used to manage the persistence lifecycle of entities.

#### Commonly Used Methods

### 📌 EntityManager Query Examples

#### JPQL Query with `createQuery`
```java
public List<User> getUsersWithOrders() {
    return entityManager.createQuery(
        "SELECT u FROM User u JOIN FETCH u.orders", User.class
    ).getResultList();
}
```

#### Named Query with `createNamedQuery`

Define in the entity:
```java
@Entity
@NamedQuery(
    name = "User.findByStatus",
    query = "SELECT u FROM User u WHERE u.status = :status"
)
public class User {
    // fields...
}
```

Execute the named query:
```java
public List<User> findByStatus(String status) {
    return entityManager.createNamedQuery("User.findByStatus", User.class)
                        .setParameter("status", status)
                        .getResultList();
}
```

#### Native SQL Query with `createNativeQuery`
```java
public List<Object[]> getRawUserData() {
    return entityManager.createNativeQuery("SELECT id, name FROM users")
                        .getResultList();
}
```

You can also map to an entity:
```java
public List<User> getUsersNative() {
    return entityManager.createNativeQuery("SELECT * FROM users", User.class)
                        .getResultList();
}
```

| Method                       | Description                                     |
| ---------------------------- | ----------------------------------------------- |
| `persist(Object entity)`     | Makes an entity managed and persistent.         |
| `merge(Object entity)`       | Updates an entity in the database.              |
| `remove(Object entity)`      | Deletes the entity from the database.           |
| `find(Class<T>, Object id)`  | Finds an entity by its primary key.             |
| `flush()`                    | Synchronizes persistence context with database. |
| `clear()`                    | Clears the persistence context.                 |
| `detach(Object entity)`      | Detaches the entity from persistence context.   |
| `createQuery(String)`        | Creates JPQL query.                             |
| `createNamedQuery``(String)` | Executes a pre-defined named query.             |
| `createNativeQuery(String)`  | Executes a native SQL query.                    |

#### Example

```java
@PersistenceContext
private EntityManager entityManager;

public User getUser(Long id) {
    return entityManager.find(User.class, id);
}
```

---

### 🔁 JPA Lifecycle Event Annotations

| Annotation     | Trigger Time              | Usage                      |
| -------------- | ------------------------- | -------------------------- |
| `@PrePersist`  | Before entity is inserted | Logging, defaulting fields |
| `@PostPersist` | After entity is inserted  | Notifications              |
| `@PreUpdate`   | Before an update occurs   | Validation, audit          |
| `@PostUpdate`  | After update completes    | Triggers, sync             |
| `@PreRemove`   | Before deletion           | Validation, logging        |
| `@PostRemove`  | After deletion            | Cleanup                    |
| `@PostLoad`    | After loading from DB     | Formatting, caching        |

#### Example

```java
@Entity
public class Product {

    @PrePersist
    public void beforeInsert() {
        System.out.println("Saving new product");
    }

    @PostLoad
    public void afterLoad() {
        System.out.println("Product loaded from DB");
    }
}
```

---

### ✅ Bean Validation Annotations

| Annotation                | Purpose                   | Example                     |
| ------------------------- | ------------------------- | --------------------------- |
| `@NotNull`                | Field must not be null    | `@NotNull String name;`     |
| `@Size(min, max)`         | Field length constraints  | `@Size(min=3, max=20)`      |
| `@Min` / `@Max`           | Min/Max numeric values    | `@Min(18)` / `@Max(100)`    |
| `@Email`                  | Validates email format    | `@Email String email;`      |
| `@Pattern`                | Regex match               | `@Pattern(regexp="[A-Z]+")` |
| `@Positive` / `@Negative` | Positive/negative numbers | `@Positive int age;`        |
| `@Past` / `@Future`       | Date must be past/future  | `@Past LocalDate dob;`      |

Validation can be triggered by Spring controllers or JPA events.

#### Example

```java
@PostMapping("/register")
public ResponseEntity<String> register(@Valid @RequestBody User user) {
    // Validated automatically
    return ResponseEntity.ok("Registered");
}
```

---

### 🧽 Hibernate Filters for Soft Deletion

1. **Enable Filter on Entity**

```java
@Entity
@FilterDef(name = "softDeleteFilter", parameters = @ParamDef(name = "isDeleted", type = "boolean"))
@Filter(name = "softDeleteFilter", condition = "deleted = :isDeleted")
public class Employee {

    @Id
    private Long id;

    private String name;

    private boolean deleted;
}
```

2. **Activate Filter via EntityManager**

```java
Session session = entityManager.unwrap(Session.class);
session.enableFilter("softDeleteFilter").setParameter("isDeleted", false);
```

This approach dynamically includes or excludes soft-deleted records in queries.

---

## 📐 15. Criteria API in JPA

The Criteria API allows the construction of type-safe and dynamic queries at runtime using Java code instead of JPQL or SQL strings.

### 🔍 Why Use Criteria API?
- Compile-time safety
- Easy to build dynamic queries
- Avoids syntax errors in JPQL/SQL
- Good for complex queries based on user input

### 🧩 Core Interfaces

| Interface/Class | Purpose |
|-----------------|---------|
| `CriteriaBuilder` | Used to construct criteria queries, expressions, predicates. |
| `CriteriaQuery<T>` | Represents a query object. |
| `Root<T>` | Represents query root for entity. |
| `Predicate` | Represents conditions (like WHERE clauses). |

### ✨ Example: Find Users by Name and Email

```java
@PersistenceContext
private EntityManager entityManager;

public List<User> findUsers(String name, String email) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> root = query.from(User.class);

    List<Predicate> predicates = new ArrayList<>();
    if (name != null) {
        predicates.add(cb.equal(root.get("name"), name));
    }
    if (email != null) {
        predicates.add(cb.equal(root.get("email"), email));
    }

    query.select(root).where(cb.and(predicates.toArray(new Predicate[0])));

    return entityManager.createQuery(query).getResultList();
}
```

### 🛠️ Common CriteriaBuilder Methods

| Method | Description |
|--------|-------------|
| `cb.equal(path, value)` | Creates equality condition |
| `cb.like(path, pattern)` | LIKE condition |
| `cb.and(pred1, pred2)` | Logical AND |
| `cb.or(pred1, pred2)` | Logical OR |
| `cb.between(path, start, end)` | Range check |
| `cb.greaterThan(path, value)` | Greater than check |
| `cb.lessThan(path, value)` | Less than check |
| `cb.isNull(path)` | IS NULL check |
| `cb.isNotNull(path)` | IS NOT NULL check |
| `cb.orderBy(cb.asc(path))` | Sorting ascending |
| `cb.orderBy(cb.desc(path))` | Sorting descending |

---

## 💼 16. Programmatic Data Management in a Banking System

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

| Method                            | Description                                                                        |
| --------------------------------- | ---------------------------------------------------------------------------------- |
| `execute(TransactionCallback<T>)` | Executes the transactional code and commits or rolls back depending on exceptions. |
| `setIsolationLevel(int)`          | Sets isolation level for the transaction.                                          |
| `setTimeout(int)`                 | Timeout in seconds.                                                                |
| `setReadOnly(boolean)`            | Marks the transaction as read-only.                                                |

### Advanced Transaction Configurations

- **Nested Transactions**: Supported only if the underlying platform supports it (e.g., with savepoints).
- **Rollback Rules**:

```java
@Transactional(rollbackFor = {CustomException.class}, noRollbackFor = {IgnoredException.class})
```

- **Propagation Options**: Like `REQUIRES_NEW`, `NESTED`, `SUPPORTS`, etc., can be configured using `@Transactional(propagation = Propagation.REQUIRES_NEW)`

