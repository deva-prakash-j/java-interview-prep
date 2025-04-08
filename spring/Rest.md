# Comprehensive Guide to REST API Development with Spring Boot

This document covers **REST API development with Spring Boot** from basics to advanced topics.

---

## 1. Introduction to REST & Spring Boot

### What is REST?
- **Representational State Transfer** – architectural style
- Stateless communication, uses HTTP methods (GET, POST, PUT, DELETE)
- Resources are represented by URIs and usually served in JSON

### Why Spring Boot?
- Simplifies Spring configuration
- Embedded server (Tomcat/Jetty/Undertow)
- Production-ready features (actuator, metrics, etc.)

### Spring Boot Starter Dependencies
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

## 2. HTTP Status Codes

Understanding HTTP status codes is crucial for REST API design. Here's a breakdown of commonly used status codes:

| Code | Status Text             | Description                                                 |
|------|--------------------------|-------------------------------------------------------------|
| 200  | OK                       | Request has succeeded.                                      |
| 201  | Created                  | Resource has been created successfully.                    |
| 202  | Accepted                 | Request accepted for processing, but not completed.        |
| 204  | No Content               | Successful request, but no content to return.              |
| 301  | Moved Permanently        | Resource has moved permanently to a new URI.               |
| 302  | Found                    | Temporary redirect to a different URI.                     |
| 400  | Bad Request              | Malformed request or invalid parameters.                   |
| 401  | Unauthorized             | Authentication is required and has failed.                 |
| 403  | Forbidden                | Authenticated but not allowed to access the resource.      |
| 404  | Not Found                | The requested resource could not be found.                 |
| 405  | Method Not Allowed       | HTTP method not supported for this resource.               |
| 409  | Conflict                 | Conflict in request, such as duplicate resource.           |
| 415  | Unsupported Media Type   | Content type not supported by the server.                  |
| 422  | Unprocessable Entity     | Semantic error in request payload.                         |
| 429  | Too Many Requests        | Rate limiting has been triggered.                          |
| 500  | Internal Server Error    | Generic server error.                                      |
| 502  | Bad Gateway              | Invalid response from upstream server.                     |
| 503  | Service Unavailable      | Server is down or overloaded.                              |
| 504  | Gateway Timeout          | Upstream server failed to send response in time.           |

---

## 3. HTTP Methods

HTTP methods are the core verbs used in RESTful services. Here's a summary:

| Method | Description                                  | Common Usage                             |
|--------|----------------------------------------------|------------------------------------------|
| GET    | Retrieve a resource or collection             | Fetch employee by ID, list all employees |
| POST   | Create a new resource                         | Add a new employee                       |
| PUT    | Replace an existing resource                  | Update entire employee details           |
| PATCH  | Partially update an existing resource         | Update employee's role                   |
| DELETE | Remove a resource                             | Delete employee by ID                    |
| HEAD   | Same as GET but only returns headers          | Check resource metadata                  |
| OPTIONS| Describe allowed methods for a resource       | CORS and capability checks               |
| TRACE  | Echo back received request                    | Debugging HTTP request                   |
| CONNECT| Establish tunnel to the server (e.g., HTTPS)  | Used for secure communication setup      |

---

## 4. Basic REST API Setup

### Sample Entity
```java
@Entity
public class Employee {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private String role;
    // Getters, Setters, Constructors
}
```

### Repository
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {}
```

### REST Controller
```java
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {
    @Autowired
    private EmployeeRepository repo;

    @GetMapping
    public List<Employee> getAll() {
        return repo.findAll();
    }

    @PostMapping
    public Employee create(@RequestBody Employee employee) {
        return repo.save(employee);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Employee> getById(@PathVariable Long id) {
        return repo.findById(id)
                   .map(ResponseEntity::ok)
                   .orElse(ResponseEntity.notFound().build());
    }
}
```

...

...

---

## 5. Data Validation

### Using `javax.validation`
```java
public class Employee {
    @NotBlank(message = "Name is mandatory")
    private String name;

    @NotNull(message = "Role is required")
    private String role;
}
```

### Enable Validation
```java
@PostMapping
public ResponseEntity<Employee> create(@Valid @RequestBody Employee emp) {
    return ResponseEntity.ok(repo.save(emp));
}
```

---

## 6. Exception Handling

### Global Exception Handler
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(error -> errors.put(error.getField(), error.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<String> handleNotFound(EntityNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

## 7. Advanced REST Features

### Pagination & Sorting
```java
@GetMapping("/paged")
public Page<Employee> getPaged(Pageable pageable) {
    return repo.findAll(pageable);
}

// Example: /api/employees/paged?page=0&size=5&sort=name,asc
```

### HATEOAS (Hypermedia)
```java
@GetMapping("/{id}")
public EntityModel<Employee> one(@PathVariable Long id) {
    Employee emp = repo.findById(id).orElseThrow();
    return EntityModel.of(emp,
        linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel(),
        linkTo(methodOn(EmployeeController.class).getAll()).withRel("employees"));
}
```

---

## 8. Security

### Spring Security Basic Auth
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf().disable()
        .authorizeHttpRequests().anyRequest().authenticated()
        .and().httpBasic();
    return http.build();
}
```

### JWT Authentication (Advanced)
- Use `spring-security-oauth2-jose` for JWT decoding
- Implement `OncePerRequestFilter` to extract and verify token
- Store user roles in token claims

---

## 9. Testing

### Unit Test Controller
```java
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {
    @Autowired
    private MockMvc mvc;

    @MockBean
    private EmployeeRepository repo;

    @Test
    void testGetAll() throws Exception {
        given(repo.findAll()).willReturn(List.of(new Employee(1L, "John", "Dev")));
        mvc.perform(get("/api/employees"))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$[0].name").value("John"));
    }
}
```

---

## 10. Documentation with Swagger / OpenAPI

### Add Dependency
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.4</version>
</dependency>
```

### Access UI
Visit `http://localhost:8080/swagger-ui.html`

---

## 11. CI/CD & Deployment

### Dockerfile
```dockerfile
FROM openjdk:17-jdk-alpine
COPY target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Kubernetes / GKE / Cloud Run
- Use Spring Profiles for environment-specific configs
- Configure health checks via `/actuator/health`

---

## 12. Performance & Monitoring

### Actuator
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Metrics
- Integrate with Prometheus/Grafana
- Expose `/actuator/metrics`

---

## 13. Versioning APIs

### URI Versioning
```java
@RequestMapping("/api/v1/employees")
```

### Header Versioning
```java
@GetMapping
public ResponseEntity<List<Employee>> getAll(@RequestHeader("API-VERSION") String version) {
    if (version.equals("v1")) return ResponseEntity.ok(repo.findAll());
    // handle v2
}
```

---

## 14. Database Migration

### Why Database Migration?
- Keep schema changes versioned and consistent across environments
- Avoid manual errors while applying DB scripts
- Enable rollback and tracking of historical changes

### Common Problems It Solves
- Uncontrolled schema drift
- Broken deployments due to missing migrations
- Team conflicts when altering DB structure

### How to Do It with Flyway

#### Add Dependency:
```xml
<dependency>
  <groupId>org.flywaydb</groupId>
  <artifactId>flyway-core</artifactId>
</dependency>
```

#### Create SQL Migration File:
Place in `src/main/resources/db/migration` with naming pattern:
```
V1__init_schema.sql
```

#### Example SQL:
```sql
CREATE TABLE employee (
  id BIGINT PRIMARY KEY,
  name VARCHAR(255),
  role VARCHAR(255)
);
```

#### Configuration (application.yml):
```yaml
spring:
  flyway:
    enabled: true
    baseline-on-migrate: true
```

---

## 15. Best Practices

- Use DTOs to avoid exposing entities directly
- Enable CORS for frontend integration
- Secure sensitive endpoints with roles
- Avoid business logic in controllers
- Log request/response using filters/interceptors
- Use caching for expensive operations (Spring Cache, Redis)
- Rate limiting (Bucket4j / API Gateway)

---

## 16. Useful Tools
- **Postman**: API testing
- **MapStruct**: DTO mapping
- **Lombok**: Boilerplate reduction
- **Flyway**: DB migrations
- **Testcontainers**: Integration testing with real DBs

---

## 17. Resources
- [Spring Boot Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Baeldung Spring Tutorials](https://www.baeldung.com/)
- [Spring Guides](https://spring.io/guides)
- [OpenAPI](https://swagger.io/specification/)

---

Happy Coding!
