# Spring Boot REST API Testing: From Basics to Expert Level

Testing is an integral part of software development, and with Spring Boot's powerful support for testing, you can ensure your REST APIs are reliable, maintainable, and production-ready. This guide walks you through everything you need to know about testing Spring Boot REST APIs, from basic unit tests to advanced integration and end-to-end testing strategies, with real-world examples.

---

## 🧩 Table of Contents
1. [Introduction](#-introduction)
2. [Types of Tests](#types-of-tests)
3. [Setting up Testing Dependencies](#setting-up-testing-dependencies)
4. [JUnit 5 vs JUnit 4 Annotations and Compatibility](#️junit-5-vs-junit-4-annotations-and-compatibility)
5. [Testing Configurations](#testing-configurations)
6. [Unit Testing](#unit-testing)
7. [Integration Testing](#integration-testing)
8. [End-to-End Testing](#end-to-end-testing)
9. [TestContainers](#testcontainers)
10. [Parameterized Tests](#parameterized-tests)
11. [Repository Testing](#repository-testing)
12. [Mocking with Mockito](#mocking-with-mockito)
13. [Data Setup and Cleanup](#data-setup-and-cleanup)
14. [Security Testing](#security-testing)
15. [Testing Exception Handling](#testing-exception-handling)
16. [Transactional Testing](#transactional-testing)
17. [Test Annotations Reference](#test-annotations-reference)
18. [Best Practices](#best-practices)
19. [Real-world Example: Employee Management API](#real-world-example-employee-management-api)
20. [Summary](#summary)
    
---

## 📌 Introduction

Spring Boot simplifies REST API development and brings powerful tools for testing APIs. Testing verifies correctness, ensures regressions are caught early, and gives confidence during deployment.

---

## 🧪 Types of Tests

| Test Type          | Scope                             | Tools Used           |
|--------------------|-----------------------------------|----------------------|
| Unit Tests         | Individual components (controllers, services) | JUnit, Mockito        |
| Integration Tests  | Multiple layers working together | Spring Test, MockMvc |
| End-to-End (E2E)   | Full application flow             | TestRestTemplate, RestAssured |
| Database Tests     | Validate repository, data         | H2 DB, TestContainers|

---

## ⚙️ Setting up Testing Dependencies

Add these to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

---

## ⚖️ JUnit 5 vs JUnit 4 Annotations and Compatibility

Spring Boot supports both JUnit 4 and JUnit 5, but JUnit 5 (also known as Jupiter) is the preferred choice for modern applications.

| Purpose | JUnit 4 | JUnit 5 (Preferred) | Notes |
|--------|---------|----------------------|-------|
| Mark a test method | `@Test` | `@Test` | Same usage, but JUnit 5 comes from `org.junit.jupiter.api` |
| Run before each test | `@Before` | `@BeforeEach` | Renamed and more descriptive in JUnit 5 |
| Run after each test | `@After` | `@AfterEach` |  |
| Run once before all tests | `@BeforeClass` (must be static) | `@BeforeAll` (can be non-static with `@TestInstance`) | Better flexibility in JUnit 5 |
| Run once after all tests | `@AfterClass` | `@AfterAll` |  |
| Ignore a test | `@Ignore` | `@Disabled` |  |
| Set test display name | ✗ | `@DisplayName("Custom Name")` | New in JUnit 5 |
| Parameterized tests | `@RunWith(Parameterized.class)` | `@ParameterizedTest` | Much more powerful and built-in in JUnit 5 |
| Exception testing | `@Test(expected = ...)` | `assertThrows(Exception.class, ...)` | Better assertions in JUnit 5 |

> ✅ Spring Boot 2.2+ supports JUnit 5 out of the box. Use JUnit 5 unless legacy code or libraries require JUnit 4.

---

## 🧪 Testing Configurations

### 🎯 Objective:
Customize Spring Boot's behavior for test environments without affecting production or development profiles.

### 🛠️ Techniques:

#### 1. Isolating environment with `application-test.yml`
- Place `application-test.yml` under `src/test/resources`.
- Use `@ActiveProfiles("test")` to activate it during testing.

```java
@SpringBootTest
@ActiveProfiles("test")
class MyTestClass {
    // will load application-test.yml
}
```

#### 2. Injecting test-specific properties with `@TestPropertySource`
- Override properties directly from test class.

```java
@SpringBootTest
@TestPropertySource(properties = {
    "app.feature-x.enabled=false",
    "custom.timeout=1000"
})
class PropertyOverrideTest {
    // custom values applied
}
```

#### 3. Disabling specific auto-configurations in test
- Prevents loading unnecessary or disruptive beans like security, mail, etc.

```java
@SpringBootTest
@EnableAutoConfiguration(exclude = SecurityAutoConfiguration.class)
class NoSecurityConfigTest {
    // disables default security config
}
```

---

## 🧪 Unit Testing

### 🎯 Objective:
Test a single unit in isolation (e.g., service method, controller method).

### 📦 Tools: JUnit, Mockito

```java
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private EmployeeService employeeService;

    @Test
    void shouldReturnEmployeeById() throws Exception {
        Employee emp = new Employee(1L, "John Doe", "HR");
        Mockito.when(employeeService.getById(1L)).thenReturn(emp);

        mockMvc.perform(get("/employees/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John Doe"));
    }
}
```

---

## 🔗 Integration Testing

### 🎯 Objective:
Test multiple layers (controller + service + repository).

### 📦 Tools: @SpringBootTest, @AutoConfigureMockMvc

```java
@SpringBootTest
@AutoConfigureMockMvc
class EmployeeIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private EmployeeRepository employeeRepository;

    @BeforeEach
    void setup() {
        employeeRepository.save(new Employee(null, "Jane Smith", "Finance"));
    }

    @Test
    void shouldReturnAllEmployees() throws Exception {
        mockMvc.perform(get("/employees"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$[0].name").value("Jane Smith"));
    }
}
```

---

## 🌐 End-to-End Testing

### 🎯 Objective:
Simulate real user scenarios by hitting endpoints over HTTP.

### 📦 Tools: TestRestTemplate, RestAssured

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class EmployeeE2ETest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void shouldCreateNewEmployee() {
        Employee emp = new Employee(null, "Alice", "Engineering");
        ResponseEntity<Employee> response = restTemplate.postForEntity("/employees", emp, Employee.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody().getId());
    }
}
```

---

## 🐳 TestContainers

Run a real PostgreSQL or MySQL container for DB testing.

```java
@Testcontainers
@SpringBootTest
class EmployeeContainerTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("user")
        .withPassword("pass");

    @DynamicPropertySource
    static void setProps(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    // Your integration test methods here...
}
```

---

## 🎯 Parameterized Tests

JUnit 5 provides built-in support for parameterized tests using the `@ParameterizedTest` annotation. This allows a test to run multiple times with different inputs.

### ✨ Benefits:
- Reduces duplication in test cases
- Ensures better test coverage with input variations

### 📦 Common Parameter Sources
| Annotation | Description |
|------------|-------------|
| `@ValueSource` | Pass primitive types and Strings |
| `@CsvSource` | Pass multiple parameters per test in CSV format |
| `@CsvFileSource` | Load data from a CSV file |
| `@EnumSource` | Use enum constants as parameters |
| `@MethodSource` | Supply data using a static method |
| `@ArgumentsSource` | Custom provider implementation |

### 🔍 Examples

#### 1. Using `@ValueSource`
```java
@ParameterizedTest
@ValueSource(strings = {"HR", "IT", "Finance"})
void testDepartmentName(String dept) {
    assertTrue(dept.length() > 1);
}
```

#### 2. Using `@CsvSource`
```java
@ParameterizedTest
@CsvSource({
    "John, HR",
    "Alice, IT"
})
void testEmployeeFields(String name, String dept) {
    assertNotNull(name);
    assertNotNull(dept);
}
```

#### 3. Using `@MethodSource`
```java
static Stream<Arguments> nameAndDeptProvider() {
    return Stream.of(
        Arguments.of("Mark", "HR"),
        Arguments.of("Sophia", "Finance")
    );
}

@ParameterizedTest
@MethodSource("nameAndDeptProvider")
void testWithMethodSource(String name, String dept) {
    assertTrue(name.length() > 2);
    assertTrue(dept.length() > 2);
}
```

#### 4. Using `@CsvFileSource`
```java
@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
void testFromCsvFile(String name, String dept) {
    assertNotNull(name);
}
```

---

## 🧪 Repository Testing

### 🎯 Objective:
Verify JPA repository interfaces and custom queries work correctly with the database.

### 📦 Tools: `@DataJpaTest`, H2/TestContainers, JPA

- Uses an in-memory or containerized database.
- Automatically configures Spring Data JPA components.
- Excludes service and controller layers.

### ✅ Example:
```java
@DataJpaTest
class EmployeeRepositoryTest {

    @Autowired
    private EmployeeRepository repository;

    @Test
    void shouldFindByDepartment() {
        repository.save(new Employee(null, "Tom", "Sales"));
        repository.save(new Employee(null, "Jerry", "Marketing"));

        List<Employee> result = repository.findByDepartment("Sales");
        assertEquals(1, result.size());
        assertEquals("Tom", result.get(0).getName());
    }
}
```

---

## 🧪 Mocking with Mockito

### 🎯 Objective:
To isolate the unit under test by mocking its dependencies, ensuring tests are focused, fast, and reliable.

### 🛠️ Key Annotations:
- `@Mock`: Creates a mock instance of a class or interface.
- `@InjectMocks`: Injects the mocks into the class under test automatically.

### ✅ Example:
In the example below, we mock the repository and inject it into the service. This allows us to test the service logic without depending on the actual repository.

```java
@ExtendWith(MockitoExtension.class)
class EmployeeServiceTest {

    @Mock
    private EmployeeRepository repo; // Mocked dependency

    @InjectMocks
    private EmployeeService service; // Class under test

    @Test
    void testGetById() {
        Employee emp = new Employee(1L, "Mark", "IT");
        when(repo.findById(1L)).thenReturn(Optional.of(emp));

        Employee found = service.getById(1L);
        assertEquals("Mark", found.getName());
    }
}
```

---

## 🔄 Data Setup and Cleanup

### 🎯 Objective:
To prepare the test environment before each test and clean it up afterward to maintain test independence and consistency.

### 🛠️ Common Annotations:
- `@BeforeEach`: Executed before every test method. Used to initialize data or state.
- `@AfterEach`: Executed after every test method. Used for cleanup.
- `@Sql`: Used to run SQL scripts before or after a test method or class.

### ✅ Example:
```java
@BeforeEach
void init() {
    employeeRepository.deleteAll();
    employeeRepository.save(new Employee(null, "Laura", "Marketing"));
}

@AfterEach
void cleanUp() {
    employeeRepository.deleteAll();
}

@Sql(scripts = "/test-schema.sql", executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
@Sql(scripts = "/test-cleanup.sql", executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
class EmployeeRepositorySqlTest {
    // tests here...
}
```

These annotations help in creating a consistent and reproducible test environment, which is essential for reliable test results.

---

## 🔐 Security Testing

### 🎯 Objective:
To ensure protected endpoints behave correctly based on user roles, authentication status, and access control policies.

### 🔧 Key Concepts:
- **Authentication**: Verifying the user's identity.
- **Authorization**: Granting access based on roles/permissions.
- **Role-based Access Control (RBAC)**: Common in Spring Security via annotations.

### 🛠️ Common Annotations:
- `@WithMockUser`: Mocks an authenticated user with roles.
- `@WithUserDetails`: Loads a user from a configured `UserDetailsService`.
- `@WithAnonymousUser`: Simulates anonymous access (unauthenticated).

### ✅ Example:
```java
@WebMvcTest(EmployeeController.class)
class EmployeeSecurityTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    void shouldAllowAccessToAdminEndpoint() throws Exception {
        mockMvc.perform(get("/admin/employees"))
               .andExpect(status().isOk());
    }

    @Test
    @WithAnonymousUser
    void shouldDenyAccessForAnonymousUser() throws Exception {
        mockMvc.perform(get("/admin/employees"))
               .andExpect(status().isUnauthorized());
    }
}
```

### ✅ Best Practices:
- Cover both access and denial cases.
- Test with multiple roles and permission combinations.
- Validate behavior when token or session is missing or expired.
- Use `@SpringBootTest` for broader security filters and `@WebMvcTest` for controller-level checks.

---

## ❗ Testing Exception Handling

### 🎯 Objective:
Ensure that custom exceptions and global exception handlers (like `@ControllerAdvice`) return the correct HTTP status codes and response bodies.

### 🔧 Key Concepts:
- `@ControllerAdvice`: Used to define global exception handling logic.
- Custom exceptions: Domain-specific exceptions such as `ResourceNotFoundException`, `ValidationException`, etc.
- Use `MockMvc` to validate the actual response status and body.

### ✅ Example:
**Custom Exception:**
```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Global Exception Handler:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

**Test Case Using MockMvc:**
```java
@WebMvcTest(EmployeeController.class)
class EmployeeExceptionTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private EmployeeService service;

    @Test
    void shouldReturn404ForMissingEmployee() throws Exception {
        when(service.getById(99L)).thenThrow(new ResourceNotFoundException("Employee not found"));

        mockMvc.perform(get("/employees/99"))
               .andExpect(status().isNotFound())
               .andExpect(content().string("Employee not found"));
    }
}
```

### ✅ Best Practices:
- Cover all defined exception scenarios with tests.
- Verify both status codes and error messages.
- Ensure `@RestControllerAdvice` returns consistent, informative responses.

---

## 🔁 Transactional Testing

### 🎯 Objective:
Use Spring’s `@Transactional` annotation in tests to automatically roll back database changes after each test execution, maintaining a clean state for every test.

### 🛠️ Key Annotation:
- `@Transactional`: Applies a transaction around a test method, automatically rolling back changes after the method finishes.

### ✅ Example:
```java
@SpringBootTest
@Transactional
class TransactionalEmployeeTest {

    @Autowired
    private EmployeeRepository repository;

    @Test
    void testInsertEmployee() {
        repository.save(new Employee(null, "Eve", "Support"));
        assertEquals(1, repository.count());
    }

    // Changes rolled back after this test
}
```

### ✅ Pros:
- Keeps database state clean between tests
- No need for manual cleanup
- Ensures test isolation and consistency

### ⚠️ Cons:
- Some side effects (like event publishing, file writes) may not be rolled back
- Cannot use in combination with `@Testcontainers` if persistence across tests is required

Use `@Transactional` wisely depending on whether you need test data to persist or be reset between runs.

---

## 🏷️ Test Annotations Reference

| Annotation | Description | Example |
|------------|-------------|---------|
| `@Test` | Marks a method as a test case. | `@Test void testName() {}` |
| `@BeforeEach` | Executes before each test method. | `@BeforeEach void setup() {}` |
| `@AfterEach` | Executes after each test method. | `@AfterEach void tearDown() {}` |
| `@BeforeAll` | Executes once before all tests in the class. | `@BeforeAll static void initAll() {}` |
| `@AfterAll` | Executes once after all tests in the class. | `@AfterAll static void cleanAll() {}` |
| `@SpringBootTest` | Loads the complete Spring context for integration testing. | `@SpringBootTest` |
| `@WebMvcTest` | Loads only web layer (controllers). | `@WebMvcTest(MyController.class)` |
| `@DataJpaTest` | Loads only JPA components. | `@DataJpaTest` |
| `@MockBean` | Adds a mock to the Spring context. | `@MockBean MyService service;` |
| `@Mock` | Creates a mock object (used with Mockito). | `@Mock MyRepository repo;` |
| `@InjectMocks` | Injects mock dependencies. | `@InjectMocks MyService service;` |
| `@Sql` | Executes SQL scripts before or after tests. | `@Sql("classpath:setup.sql")` |
| `@Testcontainers` | Enables TestContainers support. | `@Testcontainers` |
| `@Container` | Declares a container instance. | `@Container PostgreSQLContainer<?> db = ...` |
| `@DynamicPropertySource` | Dynamically overrides Spring properties. | `@DynamicPropertySource static void overrideProps(...)` |
| `@AutoConfigureMockMvc` | Auto-configures MockMvc. | `@AutoConfigureMockMvc` |
| `@Nested` | Groups inner test classes. | `@Nested class SubTests {}` |
| `@DisplayName` | Provides a custom display name for the test. | `@DisplayName("Test user creation")` |

---

## ✅ Best Practices

- Use `@Testcontainers` for real DB scenarios.
- Avoid hardcoded values; use dynamic test data.
- Isolate unit tests using mocks.
- Use `@WebMvcTest` for controller-only tests.
- Test happy paths AND edge cases.
- Organize tests with clear naming: `ClassNameTest`, `TestMethodNameShouldExpectedBehavior()`.
- Keep tests independent and repeatable.
- Integrate tests in CI/CD pipeline.

---

## 🏗️ Real-world Example: Employee Management API

### Scenario:
- REST Endpoints: `/employees`, `/employees/{id}`
- Actions: GET, POST, DELETE, PUT
- Test: 
  - Validate creation
  - Validate fetch
  - Validate update
  - Validate deletion
  - Handle not-found

### Sample Test
```java
@Test
void shouldReturn404WhenEmployeeNotFound() throws Exception {
    mockMvc.perform(get("/employees/99"))
           .andExpect(status().isNotFound());
}
```

---

## 🧠 Summary

Testing REST APIs in Spring Boot involves a mix of strategies, tools, and environments. From mocking in unit tests to full-blown TestContainers for integration, each level ensures your APIs behave as expected. Mastering these techniques makes you a confident, skilled backend developer capable of delivering reliable services.

---

Feel free to adapt this structure for your projects and evolve your test suite as your application grows.

Happy Testing! 🚀


