
# Spring Security: From Basics to Expert-Level

Spring Security is a powerful and highly customizable authentication and access-control framework for the Java Spring ecosystem. It is the de facto standard for securing Spring-based applications.

---

## 🌱 Basics of Spring Security

### What is Spring Security?
A framework that provides authentication, authorization, and protection against common attacks such as CSRF, XSS, and session fixation.

### Why Use Spring Security?
- Built-in security mechanisms
- Seamless integration with Spring applications
- Highly customizable
- Support for OAuth2, JWT, LDAP, SAML, and more

---

## 🧱 Core Concepts

### Default Security Filters
Spring Security uses a chain of filters that intercept HTTP requests to apply various security checks. These filters are executed in a specific order:

1. **`WebAsyncManagerIntegrationFilter`** – Integrates Spring Security with Spring Web’s `AsyncManager`.
2. **`SecurityContextPersistenceFilter`** – Manages `SecurityContext` persistence between requests (loads and saves it).
3. **`HeaderWriterFilter`** – Adds security headers to the response (e.g., X-Content-Type-Options).
4. **`CsrfFilter`** – Protects against Cross-Site Request Forgery attacks.
5. **`LogoutFilter`** – Handles logout requests.
6. **`OAuth2AuthorizationRequestRedirectFilter`** – Used in OAuth2 login flows.
7. **`Saml2WebSsoAuthenticationRequestFilter`** – Used in SAML2 login flows.
8. **`UsernamePasswordAuthenticationFilter`** – Handles form login.
9. **`DefaultLoginPageGeneratingFilter`** – Generates a default login page if none is provided.
10. **`BasicAuthenticationFilter`** – Handles HTTP Basic authentication.
11. **`RequestCacheAwareFilter`** – Restores request after authentication.
12. **`SecurityContextHolderAwareRequestFilter`** – Wraps requests to provide additional security-related methods.
13. **`AnonymousAuthenticationFilter`** – Provides anonymous authentication if no authentication is present.
14. **`SessionManagementFilter`** – Manages session-related security (e.g., session fixation).
15. **`ExceptionTranslationFilter`** – Handles access denied and authentication entry point exceptions.
16. **`FilterSecurityInterceptor`** – Authorizes web requests.

These filters form the security filter chain, and understanding their purpose helps in customizing Spring Security behavior effectively.

### Disabling Default Security
Spring Security secures all endpoints by default when added to your classpath. You can disable this behavior to allow unrestricted access or customize it as needed.

- **Spring Boot 2.x (using WebSecurityConfigurerAdapter):**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().permitAll()
            .and().csrf().disable();
    }
}
```

- **Spring Boot 3.x+ (using SecurityFilterChain):**
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
        .csrf(csrf -> csrf.disable());
    return http.build();
}
```


### 1. **Authentication**
The process of verifying the identity of a user.
- **Username/Password-based login**
- **Token-based (JWT)**
- **Social login (OAuth2)**

_Example:_
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user")
            .password(passwordEncoder().encode("password"))
            .roles("USER");
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 2. **Authorization**
Deciding whether a user has permission to perform an action.

#### Annotations:
- `@PreAuthorize("hasRole('ADMIN')")`
- `@Secured("ROLE_USER")`

_Example:_
```java
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/admin")
public String adminPage() {
    return "Admin content";
}
```

### 3. **Filters and Filter Chain**
Spring Security uses a filter chain to apply security.
- `UsernamePasswordAuthenticationFilter`
- `BasicAuthenticationFilter`
- `OncePerRequestFilter`

### 4. **SecurityContext and SecurityContextHolder**
Holds security-related information of the current execution thread.
```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```

---

## 🛡️ Advanced Topics

### 1. **Custom Authentication Providers**
Spring Security allows you to define custom authentication logic by implementing the `AuthenticationProvider` interface. This is useful when you need to integrate with external systems or apply non-standard credential checks.

#### Step-by-Step Example:

**1. Implement `AuthenticationProvider`:**
```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();

        if ("customuser".equals(username) && "secret".equals(password)) {
            return new UsernamePasswordAuthenticationToken(username, password, Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER")));
        } else {
            throw new BadCredentialsException("Authentication failed");
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

**2. Register the custom provider in the security config:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomAuthenticationProvider customAuthenticationProvider;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customAuthenticationProvider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin();
    }
}
```

With this setup, your application will use the custom provider to handle authentication logic instead of the default in-memory or JDBC mechanisms.

### 2. **JWT (JSON Web Tokens)**
JWT is a compact, URL-safe token format used for stateless authentication. It allows clients to prove their identity by presenting a signed token instead of managing sessions.

#### Structure of a JWT
A JWT has three parts, separated by dots (`.`):
```
<Header>.<Payload>.<Signature>
```

- **Header**: Specifies the signing algorithm and token type.
  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

- **Payload**: Contains claims (user data and metadata).
  ```json
  {
    "sub": "user123",
    "role": "ADMIN",
    "exp": 1712345678
  }
  ```

- **Signature**: Used to verify the token’s authenticity. It's created by signing the header and payload with a secret.

#### How JWT Authentication Works in Spring Boot
1. User logs in with username/password.
2. Server validates credentials and issues a JWT.
3. The client stores the JWT (e.g., in localStorage).
4. The JWT is sent in the Authorization header with each request.
5. Server verifies and extracts the user identity and roles from the token.

#### Enable JWT Authentication in Spring Boot

1. **Add a Filter to Validate JWT**
```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            // Validate and parse token
            String username = JwtUtil.extractUsername(token);
            List<GrantedAuthority> authorities = JwtUtil.extractAuthorities(token);

            UsernamePasswordAuthenticationToken auth = new UsernamePasswordAuthenticationToken(
                    username, null, authorities);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        filterChain.doFilter(request, response);
    }
}
```

2. **Register the Filter in Security Configuration**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtFilter;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

3. **Create a Utility Class for JWT Handling** (using jjwt or similar)
```java
public class JwtUtil {
    private static final String SECRET_KEY = "secret";

    public static String extractUsername(String token) {
        return Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody().getSubject();
    }

    public static List<GrantedAuthority> extractAuthorities(String token) {
        Claims claims = Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody();
        String roles = (String) claims.get("role");
        return List.of(new SimpleGrantedAuthority("ROLE_" + roles));
    }
}
```

_Example:_ Using JWT in REST API authentication.
```java
@GetMapping("/secure")
public ResponseEntity<String> securedEndpoint(@RequestHeader("Authorization") String token) {
    if (isValid(token)) {
        return ResponseEntity.ok("Valid token");
    } else {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
}
```

### 3. **OAuth2 and OIDC**

OAuth2 is an authorization framework that enables applications to obtain limited access to user accounts on an HTTP service. OpenID Connect (OIDC) is an authentication layer built on top of OAuth2.

#### 📌 Basic Concepts of OAuth2
- **Resource Owner**: The user who owns the data.
- **Client**: The application requesting access to the user's data.
- **Authorization Server**: Issues access tokens to the client after authenticating the resource owner.
- **Resource Server**: Hosts the protected resources (APIs).

#### 🔐 OAuth2 Grant Types (Authentication Flows)
1. **Authorization Code** – Used by web apps and confidential clients.
2. **Client Credentials** – Used for machine-to-machine authentication.
3. **Resource Owner Password Credentials** – Deprecated; used when the resource owner provides credentials to the client.
4. **Implicit** – Deprecated; for single-page apps (SPA).

#### 🌍 Integrating External OAuth2 Provider (e.g., Google)
To enable login via Google or other providers, Spring Boot simplifies integration via `spring-security-oauth2-client`.

**application.yml:**
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: your-client-id
            client-secret: your-client-secret
            scope: openid, profile, email
        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/v2/auth
            token-uri: https://www.googleapis.com/oauth2/v4/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
```

**Security Configuration (Spring Boot 3.x):**
```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login**").permitAll()
                .anyRequest().authenticated())
            .oauth2Login(Customizer.withDefaults());

        return http.build();
    }
}
```

#### 👤 Implement Custom OAuth2 Login Success Handler
```java
@Component
public class CustomOAuth2SuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
                                        Authentication authentication) throws IOException, ServletException {
        OAuth2AuthenticationToken token = (OAuth2AuthenticationToken) authentication;
        Map<String, Object> attributes = token.getPrincipal().getAttributes();

        // Access user info here (like email, name)
        String email = (String) attributes.get("email");

        // Perform any post-login action
        response.sendRedirect("/dashboard");
    }
}
```

**Register the success handler:**
```java
.oauth2Login(oauth2 -> oauth2
    .successHandler(customOAuth2SuccessHandler))
```

This setup enables login with external providers while giving you full control over the post-login workflow.
Support for login via Google, Facebook, GitHub, etc.

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: your-client-id
            client-secret: your-client-secret
```

### 4. **CORS and CSRF**

#### 🛑 What is CSRF (Cross-Site Request Forgery)?
CSRF is an attack that tricks the user into submitting unwanted actions on a web application in which they're authenticated. Spring Security enables CSRF protection by default for web applications using sessions.

##### When to Disable CSRF?
You should disable CSRF **only for stateless services** such as REST APIs, where no session is maintained.
```java
http.csrf().disable();
```

---

#### 🌍 What is CORS (Cross-Origin Resource Sharing)?
CORS is a security feature enforced by browsers to restrict resources from being requested from another domain. Spring Security provides built-in support for CORS.

##### Global CORS Configuration (Java-based):
```java
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")
                    .allowedOrigins("http://localhost:3000")
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("*");
        }
    };
}
```

##### Controller-Level CORS Configuration:
```java
@CrossOrigin(origins = "http://localhost:3000")
@RestController
@RequestMapping("/api")
public class ApiController {
    @GetMapping("/data")
    public String getData() {
        return "CORS-enabled response";
    }
}
```

##### Security Filter-Based CORS Configuration:
```java
http.cors().configurationSource(request -> {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:3000"));
    config.setAllowedMethods(List.of("GET", "POST"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    return config;
});
```

### 5. **Custom Filters**

Spring Security allows the creation and registration of custom filters to modify or extend the behavior of the default filter chain. You can intercept requests, add custom headers, or apply any custom logic.

#### 🧱 Steps to Create and Register a Custom Filter

**1. Create a Custom Filter:**
```java
@Component
public class CustomRequestFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        // Example: Log request details
        String path = request.getRequestURI();
        System.out.println("Incoming request to: " + path);

        // Add custom header
        response.addHeader("X-App-Processed", "true");

        filterChain.doFilter(request, response);
    }
}
```

**2. Register the Custom Filter in Security Configuration:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private CustomRequestFilter customRequestFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
            .addFilterBefore(customRequestFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

You can also use `.addFilterAfter(...)` or `.addFilterAt(...)` depending on where you want the filter to sit in the chain. `OncePerRequestFilter` ensures your filter is executed only once per request.

### 6. **Session Management**

Session management in Spring Security defines how and when sessions are created, maintained, and invalidated. It is critical for securing stateful web applications and detecting anomalies like session hijacking or concurrent logins.

#### 🧩 Session Creation Policies

| Policy                        | Description                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| `ALWAYS`                     | Always create a session if one doesn’t exist                                |
| `IF_REQUIRED` (default)      | Create a session only if required                                          |
| `NEVER`                      | Spring Security will never create a session but uses one if it exists      |
| `STATELESS`                  | No session will be created or used; ideal for REST APIs                     |

_Example:_
```java
http.sessionManagement()
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS); // For stateless APIs
```

#### 👥 Concurrent Session Control
Limits the number of active sessions a user can have.
```java
http.sessionManagement()
    .maximumSessions(1) // allow only one session per user
    .maxSessionsPreventsLogin(true); // prevent new logins if limit is reached
```

#### 🔄 Session Fixation Protection
Prevents attackers from hijacking an existing user session.
Spring Security protects against this by default by invalidating the old session and creating a new one after login.
```java
http.sessionManagement()
    .sessionFixation().migrateSession(); // default option
```

Other options:
- `.none()` – No session fixation protection
- `.newSession()` – Create a new session without copying attributes

These controls help ensure secure session handling in both stateful and stateless application contexts.

### 7. **SecurityContext and Thread-Local Propagation**

Spring Security stores authentication details in a thread-local `SecurityContext` object, which is accessible via `SecurityContextHolder`. This design ensures that security data remains tied to the currently executing thread.

#### 🧵 Thread-Local Mechanism
`SecurityContextHolder` by default uses `ThreadLocal` to store `SecurityContext`. This works well in synchronous web applications where each request is handled in a single thread.
```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```

#### 🚧 Problem in Asynchronous or Multi-Threaded Execution
In async tasks or scheduled jobs, the original thread (and thus its thread-local context) may not be available. This can lead to `null` security context unless manually propagated.

#### 🛠️ Manual Propagation Example
```java
SecurityContext context = SecurityContextHolder.getContext();
Executors.newSingleThreadExecutor().submit(() -> {
    SecurityContextHolder.setContext(context);
    // your logic here
});
```

#### 🪄 Recommended: Use `DelegatingSecurityContextRunnable` or `DelegatingSecurityContextExecutor`
Spring provides wrappers that automatically propagate the security context.
```java
Runnable securedTask = new DelegatingSecurityContextRunnable(originalTask);
executorService.submit(securedTask);
```

Or for `Executor`:
```java
Executor securedExecutor = new DelegatingSecurityContextExecutor(executor);
securedExecutor.execute(task);
```

Using these tools ensures the security context is safely and consistently available across thread boundaries.

### 8. **Multi-Tenancy and Security**

Multi-tenancy is a software architecture where a single application serves multiple tenants (clients). In Spring Security, handling multi-tenancy requires isolating authentication and authorization contexts per tenant.

#### 🧠 Key Concepts
- **Tenant Identification**: Determine the tenant from request (e.g., subdomain, header, or path).
- **Per-Tenant User Details**: Load user data based on the current tenant.
- **Authorization Isolation**: Ensure users from one tenant cannot access data of another.

#### 🧭 Implementing Multi-Tenant Security in Spring

**1. Extract Tenant Identifier:**
```java
public class TenantContext {
    private static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();

    public static void setTenant(String tenantId) {
        CURRENT_TENANT.set(tenantId);
    }

    public static String getTenant() {
        return CURRENT_TENANT.get();
    }

    public static void clear() {
        CURRENT_TENANT.remove();
    }
}
```

**2. Create a Filter to Capture Tenant Info:**
```java
@Component
public class TenantFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String tenantId = request.getHeader("X-Tenant-ID");
        TenantContext.setTenant(tenantId);
        try {
            filterChain.doFilter(request, response);
        } finally {
            TenantContext.clear();
        }
    }
}
```

**3. Modify `UserDetailsService` to Use Tenant Info:**
```java
@Service
public class MultiTenantUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        String tenantId = TenantContext.getTenant();
        // Load user for the given tenant from DB or tenant-specific service
        return findUserInTenantDatabase(username, tenantId);
    }
}
```

**4. Register the Filter in Security Config:**
```java
http.addFilterBefore(tenantFilter, UsernamePasswordAuthenticationFilter.class);
```

This setup ensures that authentication and authorization are contextually bound to the tenant, improving data isolation and security across tenants.

### 9. **Role Hierarchy**

Role hierarchy in Spring Security allows you to define inheritance relationships between roles. This is particularly useful in applications with complex role-based access control (RBAC), where higher-level roles naturally include permissions of lower-level roles.

#### 🎯 Why Use Role Hierarchy?
Without a role hierarchy, Spring only checks for explicit roles. If a user has `ROLE_ADMIN` but your method requires `ROLE_USER`, access will be denied unless the user also has `ROLE_USER`. Role hierarchy solves this by declaring that `ROLE_ADMIN` includes `ROLE_USER`.

#### 🔧 Configuration Example:
```java
@Bean
public RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl hierarchy = new RoleHierarchyImpl();
    hierarchy.setHierarchy("ROLE_ADMIN > ROLE_MANAGER 
 ROLE_MANAGER > ROLE_USER");
    return hierarchy;
}
```

#### 🌐 Real-World Scenario:
In a project management system:
- `ROLE_USER` can view projects.
- `ROLE_MANAGER` can also edit them.
- `ROLE_ADMIN` can delete them.

Instead of assigning all three roles to an admin, use role hierarchy:
```
ROLE_ADMIN > ROLE_MANAGER
ROLE_MANAGER > ROLE_USER
```

This simplifies role assignment and ensures maintainability.

### 10. **Security Headers**

Security headers help protect web applications from well-known vulnerabilities by instructing the browser how to behave when handling content.

#### 🔐 Common Security Headers and Their Purpose

| Header                      | Purpose                                                                 |
|-----------------------------|-------------------------------------------------------------------------|
| `X-Content-Type-Options`   | Prevents MIME-sniffing; enforces declared content type (`nosniff`)      |
| `X-Frame-Options`          | Protects against clickjacking (`DENY`, `SAMEORIGIN`)                    |
| `X-XSS-Protection`         | Enables cross-site scripting (XSS) filtering in browsers                |
| `Strict-Transport-Security` (HSTS) | Enforces HTTPS and protects against SSL stripping attacks           |
| `Content-Security-Policy` (CSP)    | Controls resources the user agent is allowed to load (e.g., JS, CSS) |

#### 🛠️ Enabling Security Headers in Spring Security
```java
http.headers()
    .xssProtection().block(true)
    .and()
    .frameOptions().sameOrigin()
    .and()
    .contentTypeOptions()
    .and()
    .httpStrictTransportSecurity().includeSubDomains(true).maxAgeInSeconds(31536000)
    .and()
    .contentSecurityPolicy("script-src 'self'");
```

These headers add another layer of defense, especially important for public-facing and browser-based applications.

### 11. **Password Policies**

Enforcing password policies ensures stronger security hygiene and prevents users from using weak, reused, or compromised passwords. Spring Security does not impose password policies out of the box, but you can implement them using custom validators and logic.

#### 🔐 Common Password Policy Requirements:
- Minimum length (e.g., 8 characters)
- At least one uppercase letter
- At least one lowercase letter
- At least one digit and one special character
- No reuse of last N passwords
- Expiration after a set period

#### 🛠️ Example: Custom Password Validator
```java
public class PasswordPolicyValidator {
    public static boolean isValid(String password) {
        return password.matches("^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$");
    }
}
```

#### ✅ Enforce During Registration or Password Update
```java
@PostMapping("/register")
public ResponseEntity<?> register(@RequestBody UserDto userDto) {
    if (!PasswordPolicyValidator.isValid(userDto.getPassword())) {
        return ResponseEntity.badRequest().body("Password does not meet policy requirements.");
    }
    userService.save(userDto);
    return ResponseEntity.ok("Registered");
}
```

#### 🧠 Best Practices
- Use `BCryptPasswordEncoder` for secure storage.
- Track password history to prevent reuse.
- Integrate with identity providers or password management systems for advanced policies.
- Use Spring Validator and JSR-303/JSR-380 (`@Pattern`, `@Size`, etc.) annotations for additional checks.

Robust password policies reduce the risk of brute-force and credential stuffing attacks.

### 12. **Rate Limiting and Brute Force Protection**

Protecting authentication endpoints from brute-force attacks is a critical aspect of application security. This can be achieved through rate limiting and account lockout strategies.

#### 🚫 Brute Force Attack
Occurs when an attacker attempts to guess a user’s credentials by trying many combinations rapidly.

#### 🛡️ Strategies for Protection
- Rate limiting per IP or username
- Account lockout after repeated failures
- Introduce CAPTCHA on repeated attempts
- Log and alert unusual login patterns

#### 💡 Spring-Based Rate Limiting (Simple In-Memory Example)
```java
@Component
public class LoginAttemptService {
    private final int MAX_ATTEMPT = 5;
    private final LoadingCache<String, Integer> attemptsCache;

    public LoginAttemptService() {
        attemptsCache = CacheBuilder.newBuilder()
            .expireAfterWrite(15, TimeUnit.MINUTES)
            .build(new CacheLoader<>() {
                public Integer load(String key) {
                    return 0;
                }
            });
    }

    public void loginSucceeded(String key) {
        attemptsCache.invalidate(key);
    }

    public void loginFailed(String key) {
        int attempts = attemptsCache.getUnchecked(key);
        attemptsCache.put(key, attempts + 1);
    }

    public boolean isBlocked(String key) {
        return attemptsCache.getUnchecked(key) >= MAX_ATTEMPT;
    }
}
```

#### 🧪 Use in Authentication Flow
```java
if (loginAttemptService.isBlocked(username)) {
    throw new LockedException("Too many failed login attempts");
}
```

#### ✅ Alternative Solutions
- Use Spring Integration with third-party APIs (e.g., Redis, Bucket4j) for distributed rate limiting
- Use API Gateway like Kong or NGINX for HTTP-level rate limiting

Implementing brute-force protection enhances the security posture of your login mechanisms, especially in public-facing applications.

---

## 🔐 Security Features

### CSRF (Cross-Site Request Forgery)
Enabled by default for web apps.
```java
http.csrf().disable(); // for stateless APIs
```

### CORS (Cross-Origin Resource Sharing)
```java
http.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());
```

### XSS Protection
Handled via Spring Boot’s headers:
```java
http.headers().xssProtection().block(true);
```

### Session Management
```java
http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
```

---

## 🔖 Common Security Annotations

### `@EnableWebSecurity`
Enables Spring Security’s web security support in your application.
```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
}
```

### `@PreAuthorize`
Used to apply authorization rules before method execution.
```java
@PreAuthorize("hasRole('ADMIN')")
public void adminMethod() {}
```

### `@PostAuthorize`
Evaluates authorization logic after method execution.
```java
@PostAuthorize("returnObject.owner == authentication.name")
public Document getDocument() {}
```

### `@Secured`
Defines role-based access to a method.
```java
@Secured("ROLE_USER")
public void userMethod() {}
```

### `@RolesAllowed`
JSR-250 annotation similar to `@Secured`, allows role-based access control.
```java
@RolesAllowed("ROLE_ADMIN")
public void adminOnlyMethod() {}
```

### `@WithMockUser`
Used in unit tests to simulate an authenticated user.
```java
@Test
@WithMockUser(username = "test", roles = {"USER"})
public void testWithMockUser() {
    // test logic
}
```

### `@AuthenticationPrincipal`
Injects the currently authenticated principal into controller methods.
```java
@GetMapping("/profile")
public String getProfile(@AuthenticationPrincipal UserDetails user) {
    return user.getUsername();
}
```

---

## 🧪 Testing Security

### MockMvc with Security
```java
@Test
public void testSecureEndpoint() throws Exception {
    mockMvc.perform(get("/secure").with(user("admin").roles("ADMIN")))
           .andExpect(status().isOk());
}
```

---

## 🏗️ Real-World Example

A REST API for managing books:
- **User roles**: USER, ADMIN
- **Endpoints**:
  - `GET /books` — Public
  - `POST /books` — ADMIN only
  - `DELETE /books/{id}` — ADMIN only

### Controller:
```java
@RestController
@RequestMapping("/books")
public class BookController {
    @GetMapping
    public List<Book> getBooks() { return bookService.findAll(); }

    @PreAuthorize("hasRole('ADMIN')")
    @PostMapping
    public Book create(@RequestBody Book book) { return bookService.save(book); }

    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) { bookService.delete(id); }
}
```

---

## 🧰 Tools and Libraries
- Spring Security
- Spring Boot
- JWT Libraries (e.g., jjwt)
- OAuth2 Clients (e.g., Keycloak, Okta)

---

## 🧠 Tips for Experts

- Create layered security (e.g., endpoint + method)
- Use `SecurityFilterChain` over extending `WebSecurityConfigurerAdapter` (modern approach)
- Use `PasswordEncoder` (e.g., `BCryptPasswordEncoder`)
- Monitor sessions and login attempts
- Integrate with centralized identity providers (IdPs)

---

## 📚 References
- [Spring Security Docs](https://docs.spring.io/spring-security/site/docs/current/reference/html5/)
- [Baeldung Spring Security](https://www.baeldung.com/security-spring)
- [Spring Authorization Server](https://spring.io/projects/spring-authorization-server)

---

This comprehensive guide covers everything from the basics to advanced, production-ready concepts of Spring Security.

