# Spring Cloud Gateway - The Ultimate Guide

Spring Cloud Gateway is a non-blocking API gateway built on Spring WebFlux. It provides a simple, yet effective way to route requests, apply filters, and manage cross-cutting concerns such as authentication, logging, rate limiting, etc.

This guide covers everything from basics to expert-level use cases, using a real-world microservices-based architecture as context.

---

## Table of Contents

1. [What is Spring Cloud Gateway?](#what-is-spring-cloud-gateway)
2. [Key Concepts](#key-concepts)
3. [Getting Started](#getting-started)
4. [Real-World Use Case](#real-world-use-case)
5. [Route Configuration](#route-configuration)
6. [Filters (Pre, Post, Global)](#filters)
7. [Predicate Factories](#predicate-factories)
8. [Built-in Filter Factories](#built-in-filter-factories)
9. [Custom Filters](#custom-filters)
10. [Load Balancing with Eureka/Consul](#load-balancing)
11. [Circuit Breaker with Resilience4j](#circuit-breaker)
12. [Rate Limiting](#rate-limiting)
13. [Security and Authentication](#security)
14. [Path Rewriting and Header Manipulation](#path-rewriting)
15. [Actuator Integration](#actuator-integration)
16. [Performance and Scalability Tips](#performance)
17. [Testing Strategies](#testing)
18. [Deployment Considerations](#deployment)

---

## What is Spring Cloud Gateway?
Spring Cloud Gateway is the reactive, non-blocking API gateway built on Spring WebFlux. It's the preferred gateway solution in the Spring Cloud ecosystem, replacing Zuul 1.x.

## Key Concepts
- **Route**: The basic building block, which matches an incoming request and routes it to a backend service.
- **Predicate**: Conditions to match requests.
- **Filter**: Applied before or after the request is routed.
- **Global Filter**: Runs for every request.
- **Custom Filter**: Tailored business logic.

---

## Getting Started
```xml
<!-- pom.xml -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/users/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

---

## Real-World Use Case
Assume an e-commerce system:
- **API Gateway**
- **Auth Service**
- **User Service**
- **Order Service**
- **Product Service**

Spring Cloud Gateway serves as the front door, handling:
- Authentication via JWT
- Load balancing with Eureka
- Rate limiting
- Path rewriting
- Logging and monitoring

---

## Route Configuration
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
          filters:
            - RewritePath=/orders/(?<segment>.*), /${segment}
```

---

## Filters

### Pre Filters (before route):
- Logging
- Authentication
- Validation

### Post Filters (after route):
- Response transformation
- Metrics collection

### Global Filters
```java
@Component
public class LoggingFilter implements GlobalFilter {
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("Request: " + exchange.getRequest().getURI());
        return chain.filter(exchange);
    }
}
```

---

## Predicate Factories
- Path
- Host
- Method
- Header
- Cookie
- Query

```yaml
predicates:
  - Method=GET
  - Header=X-Request-Id, ^[A-Z].*
```

---

## Built-in Filter Factories
- AddRequestHeader
- AddResponseHeader
- RemoveRequestParameter
- RewritePath
- SetPath
- Retry
- CircuitBreaker
- RequestRateLimiter

---

## Custom Filters
```java
public class MyCustomFilter implements GatewayFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // custom logic
        return chain.filter(exchange);
    }
}
```

---

## Load Balancing
Using Spring Cloud LoadBalancer or Netflix Eureka:
```yaml
uri: lb://SERVICE-NAME
```

Register services in Eureka and enable discovery:
```java
@EnableDiscoveryClient
```

---

## Circuit Breaker
```yaml
filters:
  - name: CircuitBreaker
    args:
      name: myCircuitBreaker
      fallbackUri: forward:/fallback
```

---

## Rate Limiting
Backed by Redis:
```yaml
filters:
  - name: RequestRateLimiter
    args:
      redis-rate-limiter.replenishRate: 10
      redis-rate-limiter.burstCapacity: 20
```

---

## Security and Authentication
Use Spring Security with JWT:
```java
@Bean
SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    http
      .authorizeExchange()
        .pathMatchers("/auth/**").permitAll()
        .anyExchange().authenticated()
      .and()
        .oauth2ResourceServer().jwt();
    return http.build();
}
```

---

## Path Rewriting and Header Manipulation
```yaml
filters:
  - AddRequestHeader=X-Request-Foo, Bar
  - RewritePath=/foo/(?<segment>.*), /${segment}
```

---

## Actuator Integration
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Access routes: `/actuator/gateway/routes`

---

## Performance and Scalability Tips
- Use non-blocking WebFlux clients in downstream services
- Offload heavy computation outside the gateway
- Enable async logging
- Tune Redis and use persistent rate limiter storage

---

## Testing Strategies
- Unit test filters using `MockServerWebExchange`
- Integration test using `@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)`
- Use Postman/RestAssured for manual testing

---

## Deployment Considerations
- Deploy behind a load balancer
- Configure health checks
- Use distributed tracing (Zipkin/Jaeger)
- Use centralized logging (ELK/GCP Stackdriver)

---

## Summary
Spring Cloud Gateway is a powerful, flexible gateway solution for microservices. Whether you're doing simple routing or complex edge functions like security, rate limiting, and circuit breaking, it's designed to handle production-grade traffic and integrates smoothly with the Spring ecosystem.

> A well-designed gateway improves both security and scalability of microservices. Make it count.

