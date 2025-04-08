# Spring Core Concepts - A Comprehensive Guide

Spring Framework is one of the most powerful and widely used frameworks in the Java ecosystem. It provides comprehensive infrastructure support for developing Java applications. Let's walk through the core concepts of Spring from the ground up to an expert level, tailored for a developer with 20+ years of experience.

---

## 1. **Inversion of Control (IoC)**

### Definition
IoC is a design principle where the control of object creation and lifecycle is delegated to the container.

### Implementation in Spring
Spring achieves IoC through **Dependency Injection (DI)**.

### Types of DI:
- **Constructor Injection**
- **Setter Injection**
- **Field Injection (via reflection - not recommended)**

### Examples:
#### Constructor Injection
```java
@Component
public class Car {
    private final Engine engine;

    @Autowired
    public Car(Engine engine) {
        this.engine = engine;
    }
}

@Component
public class Engine {}
```

#### Setter Injection
```java
@Component
public class Car {
    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

#### Field Injection (not recommended for testing and immutability reasons)
```java
@Component
public class Car {
    @Autowired
    private Engine engine;
}

@Component
public class Engine {}
```

---

## 2. **Dependency Injection (DI)**

Dependency Injection is the core mechanism of Inversion of Control in Spring. It allows objects to declare their dependencies without creating them.

### Injection Types:
- **Manual (XML or JavaConfig)**
- **Annotation-based (@Autowired, @Inject)**
- **Interface-based (less common, used with ApplicationContextAware, etc.)**

### Examples:
#### Manual Injection via JavaConfig
```java
@Configuration
public class AppConfig {
    @Bean
    public Engine engine() {
        return new Engine();
    }

    @Bean
    public Car car() {
        return new Car(engine());
    }
}
```

#### Annotation-based Injection
```java
@Component
public class Car {
    private final Engine engine;

    @Autowired
    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

#### Interface-based Injection (less common)
```java
@Component
public class AwareBean implements ApplicationContextAware {
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {
        this.context = context;
    }
}
```

### Scopes:
- `singleton` (default)
- `prototype`
- `request`, `session`, `application` (for web-aware contexts)

### Life Cycle:
- **Instantiation -> Dependency Injection -> Custom init -> Ready to use -> Custom destroy**

---

## 3. **Bean Life Cycle**

### Bean Creation Phases:
1. Instantiation
2. Populate properties (DI)
3. `BeanNameAware`, `BeanFactoryAware`, etc.
4. `@PostConstruct` / `InitializingBean#afterPropertiesSet()`
5. Custom init methods
6. In use
7. `@PreDestroy` / `DisposableBean#destroy()`
8. Custom destroy methods

### Example:
```java
@Component
public class ResourceManager implements InitializingBean, DisposableBean {
    public void afterPropertiesSet() { // init }
    public void destroy() { // cleanup }
}
```

---

## 4. **ApplicationContext vs BeanFactory**

- **BeanFactory**: Basic container, lazy loading.
- **ApplicationContext**: Superset with internationalization, event propagation, AOP, etc.

### Types of ApplicationContext:
- `AnnotationConfigApplicationContext`
- `ClassPathXmlApplicationContext`
- `GenericWebApplicationContext`

---

## 5. **Spring Configuration Styles**

### XML Configuration:
```xml
<bean id="car" class="com.example.Car"/>
```

### Java-based Configuration:
```java
@Configuration
public class AppConfig {
    @Bean
    public Car car() {
        return new Car(engine());
    }
    @Bean
    public Engine engine() {
        return new Engine();
    }
}
```

### Annotation-based:
```java
@Component
public class Engine {}
```

---

## 6. **Component Scanning and Stereotypes**
- `@Component`: Generic bean
- `@Service`: Service layer
- `@Repository`: DAO layer (adds exception translation)
- `@Controller`: MVC controller

### Example:
```java
@ComponentScan(basePackages = "com.example")
```

---

## 7. **Spring Expression Language (SpEL)**

### Use Cases:
- Conditional bean creation
- Injecting computed values

### Example:
```java
@Value("#{2 * 3}")
private int six;
```

---

## 8. **Profiles and Environment Abstraction**

### Purpose:
To define beans for different environments.

### Example:
```java
@Configuration
@Profile("dev")
public class DevConfig {}
```

Activate using:
```properties
spring.profiles.active=dev
```

---

## 9. **Event Handling in Spring**

### Usage:
Loose coupling between components.

### Example:
```java
public class CustomEvent extends ApplicationEvent {
    public CustomEvent(Object source) { super(source); }
}

@Component
public class CustomListener {
    @EventListener
    public void handle(CustomEvent event) {}
}
```

---

## 10. **AOP (Aspect-Oriented Programming)**

### Concepts:
- **Aspect**: Modularization of cross-cutting concern
- **Join Point**: Point in execution
- **Advice**: Code to run at join point
- **Pointcut**: Predicate to match join points
- **Weaving**: Applying aspects to target object

### Example:
```java
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.*.*(..))")
    public void logBefore() {
        System.out.println("Method is called");
    }
}
```

---

## 11. **Transaction Management**

### Types:
- Programmatic
- Declarative (`@Transactional`)

### Propagation Types:
- `REQUIRED`, `REQUIRES_NEW`, `MANDATORY`, etc.

### Example:
```java
@Service
public class OrderService {
    @Transactional
    public void placeOrder() {}
}
```

---

## 12. **BeanPostProcessor and BeanFactoryPostProcessor**

### BeanPostProcessor:
Intercepts bean creation and allows modification.

### BeanFactoryPostProcessor:
Modifies the bean definition before instantiation.

### Example:
```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {
    public Object postProcessBeforeInitialization(Object bean, String name) {
        return bean;
    }
}
```

---

## 13. **Custom Annotations and Meta-Annotations**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface MyService {}
```

---

## 14. **FactoryBeans**

Allows you to create complex bean logic.

### Example:
```java
public class MyFactoryBean implements FactoryBean<MyBean> {
    public MyBean getObject() { return new MyBean(); }
    public Class<?> getObjectType() { return MyBean.class; }
}
```

---

## 15. **Spring Modules Interconnection**
- **Spring Core**: Core features and DI
- **Spring AOP**: Aspect-oriented programming
- **Spring Context**: ApplicationContext and lifecycle
- **Spring ORM**: Integration with ORM tools like Hibernate
- **Spring JDBC**: Simplified JDBC handling
- **Spring Web**: Web applications and REST

---

## 16. **Best Practices**
- Prefer constructor injection for immutability and testability
- Use profiles for environment-specific beans
- Avoid field injection in favor of constructor/setter
- Keep configuration externalized
- Use `@Value` and `@ConfigurationProperties` wisely
- Limit AOP to cross-cutting concerns
- Properly handle transactions and rollback scenarios

---

## Conclusion
Spring Core is the backbone of the entire Spring ecosystem. Mastering these concepts allows developers to build modular, maintainable, and testable applications. With 20+ years of experience, understanding how Spring orchestrates all these components under the hood gives a strategic edge in architecting modern Java applications.

For advanced Spring topics, one should delve into:
- Spring Boot auto-configuration internals
- Conditional bean registration
- Advanced AOP use cases
- Context hierarchy in web apps
- Spring integration with Kafka, Redis, etc.

---

Happy coding with Spring! 🌱

