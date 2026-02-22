# Spring Boot Interview Questions

---

## 1. Spring Boot Fundamentals & Internal Architecture

- Explain Spring Boot architecture and how it works internally (auto-configuration, starter dependencies, embedded server, SpringApplication.run)
- What is @SpringBootApplication and what annotations does it combine?
- What is auto-configuration and how Spring Boot decides which beans to create?
- What are starter dependencies and why are they used?
- What is embedded Tomcat and how Spring Boot runs without external server deployment?

---

## 2. Dependency Injection, Beans, and IOC

- Explain IOC and Dependency Injection and how ApplicationContext manages beans
- Difference between @Component, @Service, @Repository, @Controller, and @Bean
- Explain @Autowired, constructor injection, and why constructor injection is recommended
- Explain Spring Bean lifecycle and how Spring creates and initializes beans
- How Spring handles multiple beans of same type using @Qualifier and @Primary

---

## 3. Configuration and Profiles

- How Spring Boot reads configuration from application.properties / application.yml
- Difference between @Value and @ConfigurationProperties
- What are Spring Profiles and how they are used in dev, test, and production

---

## 4. REST API and Request Flow (VERY IMPORTANT)

- How REST APIs are created using @RestController, @RequestMapping, @RequestBody, @PathVariable, and @RequestParam
- Explain complete request flow: Client → DispatcherServlet → Controller → Service → Repository → Database → Response
- What is DispatcherServlet and its role in Spring Boot
- What is ResponseEntity and why it is used

---

## 5. Database and Spring Data JPA / Hibernate

- How Spring Boot interacts with database using Spring Data JPA and JpaRepository
- Explain Entity mapping annotations: @Entity, @Id, @GeneratedValue, @Column
- Explain relationships: OneToOne, OneToMany, ManyToOne, ManyToMany
- Difference between LAZY and EAGER loading in Hibernate
- How query methods work (derived queries, @Query, pagination)

---

## 6. Exception Handling and Validation

- How exception handling works using @ExceptionHandler and @ControllerAdvice
- How validation works using @Valid, @NotNull, @NotBlank

---

## 7. Microservices Core Concepts (VERY FREQUENT)

- How Spring Boot is used to build microservices
- How microservices communicate using RestTemplate, WebClient, or Feign Client
- What is service discovery (Eureka Server and Client) and how load balancing works
- What is API Gateway and its role in microservices

---

## 8. Transactions, Caching, and Performance

- How transaction management works using @Transactional
- How caching works using Redis and @Cacheable
- How Spring Boot handles multiple concurrent requests (threading model)
- How database connection pooling works using HikariCP

---

## 9. Security

- How Spring Boot APIs are secured using Spring Security and JWT

---

## 10. Production and Deployment (MOST IMPORTANT FOR 4 YOE)

- How Spring Boot application is deployed using jar, Docker, and Kubernetes
- How Spring Boot works behind Nginx or load balancer
- How Spring Boot integrates with Kafka and Redis in production

---

## 11. Scenario-Based (ALWAYS ASKED)

- Explain your project architecture and how microservices communicate
- How you handled performance issues or high traffic
- How you implemented exception handling, logging, and monitoring

---