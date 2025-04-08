
# JPA and Hibernate in Spring Boot: A Comprehensive Guide

This document explores Java Persistence API (JPA) and Hibernate within the context of Spring Boot. It spans beginner to expert-level concepts, including configuration, mappings, transaction management, and advanced topics like multi-datasource management and performance tuning.

---

## 📌 Table of Contents
1. [Introduction to JPA and Hibernate](#1-introduction-to-jpa-and-hibernate)
2. [Spring Boot JPA Configuration](#2-spring-boot-jpa-configuration)
3. [Entity Mappings](#3-entity-mappings)
    - [@Entity, @Table, and @Id](#entity-table-and-id)
    - [@Column, @Embedded, and @Embeddable](#column-embedded-and-embeddable)
    - [Relationships: @OneToOne, @OneToMany, etc.](#relationships-onetoone-onetomany-etc)
4. [Repositories](#4-repositories)
5. [Custom Queries: JPQL and Native](#5-custom-queries-jpql-and-native)
6. [Transaction Management](#6-transaction-management)
7. [ACID and Isolation Levels](#7-acid-and-isolation-levels)
8. [Advanced Topics](#8-advanced-topics)
    - [Multiple Datasource Support](#multiple-datasource-support)
    - [Batch Inserts and Updates](#batch-inserts-and-updates)
    - [Optimistic vs Pessimistic Locking](#optimistic-vs-pessimistic-locking)
    - [Hibernate Caching](#hibernate-caching)
    - [Auditing with Envers or Spring Data](#auditing-with-envers-or-spring-data)
    - [Entity Lifecycle Callbacks](#entity-lifecycle-callbacks)
    - [DDL Auto Strategies](#ddl-auto-strategies)

---

## 1. Introduction to JPA and Hibernate

- **JPA**: A Java specification for ORM. Abstracts persistence logic.
- **Hibernate**: A JPA implementation. Adds features like caching, lazy loading, HQL.

Spring Boot uses **Spring Data JPA** to simplify integration:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

## 2. Spring Boot JPA Configuration

`application.yml` or `application.properties`
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

---

## 3. Entity Mappings

### Entity, Table and Id
```java
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String department;
}
```

### Column, Embedded, and Embeddable
```java
@Embeddable
public class Address {
    private String city;
    private String state;
}

@Entity
public class Office {
    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address address;
}
```

### Relationships: @OneToOne, @OneToMany, etc.
```java
@Entity
public class Department {
    @Id
    private Long id;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees;
}

@Entity
public class Employee {
    @ManyToOne(fetch = FetchType.LAZY)
    private Department department;
}
```

---

## 4. Repositories
```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    List<Employee> findByDepartment(String department);
}
```

---

## 5. Custom Queries: JPQL and Native
```java
@Query("SELECT e FROM Employee e WHERE e.department = :dept")
List<Employee> findByDept(@Param("dept") String dept);

@Query(value = "SELECT * FROM employees WHERE department = ?1", nativeQuery = true)
List<Employee> nativeFindByDept(String dept);
```

---

## 6. Transaction Management

Spring uses `@Transactional` for declarative transaction management.
```java
@Service
public class EmployeeService {
    @Transactional
    public void transferEmployee(Long empId, Long newDeptId) {
        // load employee and department
        // set department
        // save employee
    }
}
```

For programmatic transactions:
```java
@Transactional(propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
public void someMethod() {
    // isolated transaction
}
```

---

## 7. ACID and Isolation Levels

### ACID Properties
- **Atomicity**: All operations in a transaction are treated as a single unit.
- **Consistency**: Data is always in a valid state before and after the transaction.
- **Isolation**: Concurrent transactions do not interfere.
- **Durability**: Once committed, the transaction changes persist.

Spring ensures ACID properties through `@Transactional` and integration with JPA.

### Isolation Levels (from `javax.transaction.Transactional` or `org.springframework.transaction.annotation.Transactional`)
- **DEFAULT**: Database default isolation.
- **READ_UNCOMMITTED**: Dirty reads possible.
- **READ_COMMITTED**: Prevents dirty reads.
- **REPEATABLE_READ**: Prevents non-repeatable reads.
- **SERIALIZABLE**: Highest level, full isolation.

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void updateAccount() {
    // logic here
}
```

Choose based on the use case:
- For analytics: `READ_COMMITTED`
- For financial apps: `REPEATABLE_READ` or `SERIALIZABLE`

---

## 8. Advanced Topics

### Multiple Datasource Support
```yaml
spring:
  datasource:
    primary:
      url: jdbc:mysql://...
    secondary:
      url: jdbc:postgresql://...
```
```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.example.primary",
    entityManagerFactoryRef = "primaryEntityManager",
    transactionManagerRef = "primaryTransactionManager")
public class PrimaryDbConfig { ... }
```

### Batch Inserts and Updates
```yaml
spring.jpa.properties.hibernate.jdbc.batch_size: 30
spring.jpa.properties.hibernate.order_inserts: true
```

### Optimistic vs Pessimistic Locking
```java
@Version
private int version;

@Lock(LockModeType.PESSIMISTIC_WRITE)
Employee findById(Long id);
```

### Hibernate Caching
```xml
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```
```java
@Cacheable("employees")
public Employee findEmployee(Long id);
```

### Auditing with Envers or Spring Data
```java
@EnableJpaAuditing
@Configuration
public class AuditingConfig {}

@EntityListeners(AuditingEntityListener.class)
public class Employee {
    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### Entity Lifecycle Callbacks
```java
@PrePersist
void beforeSave() {
    createdAt = LocalDateTime.now();
}
```

### DDL Auto Strategies
- `none`: Don’t touch DB
- `validate`: Validate schema
- `update`: Update schema (risky in prod)
- `create`: Drop/create tables
- `create-drop`: Same as create, but drop at shutdown

---

## ✅ Conclusion

Spring Boot with JPA and Hibernate provides a powerful abstraction over database operations. Mastery of both basic and advanced features allows you to build robust, scalable applications with transactional integrity, performance tuning, and auditability.

> Best practices:
> - Use `@Transactional` carefully.
> - Avoid `DDL-auto=update` in production.
> - Prefer JPQL for portability, native SQL for performance.
> - Test entity mappings thoroughly.

---

Need an example project with these configurations and patterns? Just ask!
