# Spring Core & Spring Boot Interview Questions for Senior Engineers

## Question 1: Dependency Injection Deep Dive

**Question:** Explain the concept of Dependency Injection (DI) in Spring. Discuss the different types of DI you've used (Constructor, Setter, Field) and the pros and cons of each. Describe a scenario in one of your projects (e.g., at Intralinks, Herc Rentals, or TOPS) where choosing a specific type of DI was important for code clarity, testability, or managing complex dependencies in your Java/J2EE microservices.

**Discussion Points / Sample Answer Outline:**

*   **Explain IoC and DI:** The core principle of Spring.
*   **Types of DI:**
    *   **Constructor Injection:**
        *   Pros: Enforces mandatory dependencies, immutability, clear contracts, better testability.
        *   Cons: Can lead to constructors with many parameters (code smell, consider refactoring).
    *   **Setter Injection:**
        *   Pros: Good for optional dependencies, more readable if many dependencies.
        *   Cons: Object can be in an incomplete state before all setters are called, less immutable.
    *   **Field Injection (`@Autowired` on fields):**
        *   Pros: Concise code.
        *   Cons: Harder to test (requires reflection or Spring context), hides dependencies, can violate single responsibility principle if too many injected fields.
*   **Scenario from Projects:**
    *   *Recall a specific service or component from Intralinks VDRPro, Herc Admin Tool, or TOPS.*
    *   Why was a particular DI type chosen? (e.g., constructor injection for critical repository/service dependencies to ensure the bean is always in a valid state).
    *   Were there challenges with one type that made you prefer another? (e.g., circular dependencies, test setup complexity).
    *   How did it impact testability using Mockito or Spring Test?
*   **Best Practices:** When to use which type. Generally, constructor injection for mandatory dependencies, setter for optional. Avoid field injection where possible, especially in library code or complex components.
*   **Circular Dependencies:** How Spring handles them (or fails to with constructor injection).

## Question 2: Spring Bean Scopes and Lifecycle

**Question:** Discuss various Spring bean scopes (singleton, prototype, request, session). Provide a real-world example from your experience where you used a scope other than singleton and explain why it was necessary. Also, describe key phases in the Spring bean lifecycle and how you might use lifecycle callbacks like `@PostConstruct` or `@PreDestroy`.

**Discussion Points / Sample Answer Outline:**

*   **Explain Bean Scopes:**
    *   `singleton`: Default, one instance per container.
    *   `prototype`: New instance each time requested.
    *   `request`: One instance per HTTP request (Spring MVC).
    *   `session`: One instance per HTTP session (Spring MVC).
    *   `application`: One instance per ServletContext (Spring MVC).
    *   `websocket`: One instance per WebSocket session.
*   **Real-world Example for Non-Singleton Scope:**
    *   *Think about stateful beans. E.g., a shopping cart (`session` scope), a request-specific cache (`request` scope), or a complex, configurable object that shouldn't be shared (`prototype` scope).*
    *   *Perhaps in a web application at Herc Rentals or Intralinks.*
    *   Why was singleton not appropriate? What issues would it cause?
*   **Bean Lifecycle Phases:**
    *   Instantiation, Property Population (DI), `Aware` interfaces (`BeanNameAware`, `ApplicationContextAware`), `BeanPostProcessor` (before/after init), Initialization (`@PostConstruct`, `afterPropertiesSet`, init-method), Destruction (`@PreDestroy`, `destroy`, destroy-method).
*   **Lifecycle Callbacks Example:**
    *   `@PostConstruct`: Initializing resources, pre-calculating data, starting background tasks.
        *   *E.g., warming up a cache, establishing connections in a custom connection pool manager (as per your sample answer in theory.md).*
    *   `@PreDestroy`: Releasing resources, graceful shutdown.
        *   *E.g., closing file handles, stopping thread pools, deregistering from a service.*
    *   *Connect this to your experience in projects at BOFA, Wipro, or Herc Rentals.*

## Question 3: Spring AOP for Cross-Cutting Concerns

**Question:** Explain Aspect-Oriented Programming (AOP) in Spring. What are cross-cutting concerns? Describe a specific instance where you implemented (or would have implemented) a custom Spring AOP aspect to address a concern like logging, security checks, performance monitoring, or custom transaction management in your Java microservices. Detail the advice type, pointcut expression, and the benefits achieved.

**Discussion Points / Sample Answer Outline:**

*   **Explain AOP Concepts:** Aspect, Join Point, Advice (`@Before`, `@AfterReturning`, `@AfterThrowing`, `@After`, `@Around`), Pointcut, Target Object, Proxy (JDK Dynamic vs. CGLIB).
*   **Cross-Cutting Concerns:** Logging, security, transactions, caching, exception handling, monitoring.
*   **Specific AOP Implementation Example:**
    *   *Refer to your experience: performance monitoring at TOPS, custom logging for specific API calls, or a custom security check before method execution.*
    *   **Concern Addressed:** Clearly define the problem.
    *   **Advice Type:** Why was `@Around` chosen (e.g., for timing) or `@Before` (for pre-condition checks)?
    *   **Pointcut Expression:** How did you target the specific methods? (e.g., `execution(* com.yourproject.service.*.*(..))`, `@annotation(com.yourproject.annotation.MyCustomAnnotation)`).
    *   **Implementation Details:** Briefly describe what the advice method did.
    *   **Benefits:** Reduced code duplication, improved modularity, cleaner business logic.
*   **Spring's Declarative Services using AOP:**
    *   Mention `@Transactional` as a prime example of AOP in action.
    *   Briefly touch upon Spring Security's method security if relevant.

## Question 4: Spring Boot Auto-Configuration and Starters

**Question:** How does Spring Boot's auto-configuration mechanism work? Explain the role of `@EnableAutoConfiguration` and conditional annotations (e.g., `@ConditionalOnClass`, `@ConditionalOnMissingBean`). Then, describe how Spring Boot Starters simplify dependency management. Provide an example from one of your projects where auto-configuration and a specific starter (e.g., `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-actuator`) significantly reduced your setup and configuration effort.

**Discussion Points / Sample Answer Outline:**

*   **Auto-Configuration Mechanism:**
    *   Scans `META-INF/spring.factories` (or `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in Spring Boot 2.7+) for auto-configuration classes.
    *   `@EnableAutoConfiguration` (part of `@SpringBootApplication`) triggers this.
    *   **Conditional Annotations:**
        *   `@ConditionalOnClass`: Configures if a specific class is on the classpath.
        *   `@ConditionalOnMissingBean`: Configures only if a bean of that type isn't already defined by the user.
        *   `@ConditionalOnProperty`: Configures based on a property value.
        *   `@ConditionalOnWebApplication`, etc.
    *   How it "backs off" if user provides their own configuration.
*   **Spring Boot Starters:**
    *   "Opinionated" sets of transitive dependencies.
    *   Simplify build configuration (Maven POM / Gradle build file).
    *   Ensure compatibility between libraries.
*   **Project Example (Herc Admin Tool, Intralinks microservices):**
    *   Choose a starter (e.g., `spring-boot-starter-web`).
    *   What did it bring in? (Tomcat, Spring MVC, Jackson).
    *   What auto-configuration was triggered? (DispatcherServlet, HttpMessageConverters, Error Page handling).
    *   What manual configuration did it save you compared to a traditional Spring MVC setup?
    *   *Similarly for `spring-boot-starter-data-jpa`: auto-configures DataSource, EntityManagerFactory, transaction management if a DB is on classpath.*
    *   *For `spring-boot-starter-actuator`: auto-configures endpoints like `/health`, `/metrics`.*

## Question 5: Spring Boot Actuator for Production Monitoring

**Question:** You are deploying a Spring Boot microservice (e.g., part of the Herc Rentals or Intralinks platforms) into a production AWS environment. Which Spring Boot Actuator endpoints would you enable and consider essential for monitoring and operational insight? How would you secure these endpoints? Discuss how these Actuator endpoints integrate with monitoring systems like Prometheus/Grafana or CloudWatch.

**Discussion Points / Sample Answer Outline:**

*   **Essential Actuator Endpoints & Use Cases:**
    *   `/health`: Critical for load balancer health checks (AWS ALB), container orchestrator (ECS, Kubernetes) liveness/readiness probes. Custom health indicators for downstream dependencies (databases, other services).
    *   `/info`: Application metadata (version, build info, git commit).
    *   `/metrics`:
        *   JVM metrics (heap, threads, GC).
        *   System metrics (CPU).
        *   HTTP request metrics (latency, count, error rates).
        *   DataSource metrics (pool usage).
        *   Custom application-specific metrics via Micrometer.
    *   `/loggers`: View and change log levels at runtime for debugging.
    *   `/env`: Inspect environment properties (be cautious with sensitive data).
    *   `/threaddump`, `/heapdump`: For deeper diagnostics (usually accessed carefully).
*   **Securing Actuator Endpoints:**
    *   Default: Most web endpoints are disabled except `/health` and `/info` (in newer Spring Boot versions).
    *   Enable specific endpoints via `management.endpoints.web.exposure.include`.
    *   Security:
        *   Integrate with Spring Security to require authentication.
        *   Deploy behind a reverse proxy or API Gateway that handles auth.
        *   Use a separate management port (`management.server.port`) and restrict network access to it.
*   **Integration with Monitoring Systems:**
    *   **Prometheus/Grafana:**
        *   Include `micrometer-registry-prometheus`. Actuator's `/actuator/prometheus` endpoint exposes metrics in Prometheus format.
        *   Prometheus scrapes this endpoint. Grafana visualizes.
        *   *Your experience with this setup.*
    *   **CloudWatch:**
        *   `micrometer-registry-cloudwatch` can push metrics to CloudWatch.
        *   Logs (via `awslogs` driver or Fluentd) from Actuator (if configured to log to console) can go to CloudWatch Logs.
        *   CloudWatch Alarms can be set on metrics exposed by Actuator.
*   **Customization:** Adding custom metrics, health indicators, or info contributors.

## Question 6: Externalized Configuration and Profiles in Spring Boot

**Question:** Describe how Spring Boot supports externalized configuration. What are the common ways to provide configuration to a Spring Boot application? Explain the concept of Spring Profiles and how you've used them to manage environment-specific configurations in a CI/CD pipeline for projects like TOPS or Herc Rentals deployed on AWS.

**Discussion Points / Sample Answer Outline:**

*   **Sources of Externalized Configuration:**
    *   `application.properties` / `application.yml` (packaged and external).
    *   Command-line arguments.
    *   Environment variables.
    *   System properties.
    *   Profile-specific properties/YAML files.
    *   `SPRING_APPLICATION_JSON`.
    *   Order of precedence.
*   **Spring Profiles (`spring.profiles.active`):**
    *   Purpose: To segregate parts of your application configuration and make it only available in certain environments.
    *   Defining profile-specific properties: `application-{profile}.properties` or multi-document YAML files.
    *   Activating profiles: via `spring.profiles.active` property, environment variable `SPRING_PROFILES_ACTIVE`, or programmatically.
*   **Real-world Scenario (TOPS, Herc Rentals on AWS):**
    *   **Problem:** Managing different database URLs, Kafka brokers, API keys, feature flags across dev, QA, UAT, prod.
    *   **Solution:**
        *   Use `application-{profile}.yml` for each environment.
        *   Common properties in `application.yml`.
        *   Profile-specific overrides for DB URLs, Kafka brokers, etc.
        *   How `spring.profiles.active` was set in different environments (e.g., environment variables in ECS Task Definitions, Jenkins pipeline parameters).
        *   Use of `@Profile` annotation on beans or configuration classes.
    *   **Benefits:** Single build artifact deployable to multiple environments, clear separation of concerns, reduced risk of misconfiguration.
    *   **Sensitive Data:** How to handle secrets (e.g., DB passwords, API keys) – environment variables, AWS Secrets Manager, HashiCorp Vault, Spring Cloud Config Server (if used).
        *   *Mention your experience with AWS Secrets Manager or Parameter Store if applicable.*

## Question 7: Building RESTful APIs with Spring Boot

**Question:** You need to design and implement a RESTful API endpoint using Spring Boot for a new feature in the Herc Admin Tool or an Intralinks microservice. Describe the key annotations and components you would use. Discuss considerations for request validation, error handling, and versioning.

**Discussion Points / Sample Answer Outline:**

*   **Core Components & Annotations:**
    *   `@RestController` on the class.
    *   `@RequestMapping` (at class level for base path) and specific HTTP method annotations (`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`).
    *   `@PathVariable` for path parameters.
    *   `@RequestParam` for query parameters.
    *   `@RequestBody` for request payload, automatically converted from JSON/XML by HttpMessageConverters (e.g., Jackson).
    *   `ResponseEntity<T>` for full control over response (status, headers, body).
    *   Service layer (`@Service`) and Repository layer (`@Repository`) separation.
*   **Request Validation:**
    *   Bean Validation API (JSR 380) with annotations like `@Valid`, `@NotNull`, `@Size`, `@Pattern` on request DTOs.
    *   Spring Boot automatically validates `@Valid` annotated parameters.
    *   Custom validators if needed.
    *   How to handle validation errors (e.g., `MethodArgumentNotValidException` leading to a 400 Bad Request).
*   **Error Handling:**
    *   `@ExceptionHandler` methods within the controller or a `@ControllerAdvice` class for global error handling.
    *   Mapping specific exceptions to HTTP status codes (e.g., `ResourceNotFoundException` to 404).
    *   Consistent error response format (e.g., JSON with `timestamp`, `status`, `error`, `message`).
    *   Spring Boot's default error handling (`BasicErrorController`).
*   **API Versioning Strategies:**
    *   URI Versioning (e.g., `/v1/resource`, `/v2/resource`). Simplest, most common.
    *   Header Versioning (e.g., `Accept: application/vnd.myapi.v1+json`).
    *   Query Parameter Versioning (e.g., `/resource?version=1`).
    *   Discuss pros and cons of each.
*   **DTOs (Data Transfer Objects):** Use DTOs for request and response payloads to decouple API contract from internal domain models.
*   **Security:** Mention Spring Security for securing the endpoints (e.g., JWT-based auth).
*   **Testing:**
    *   `MockMvc` for testing the controller layer.
    *   `@SpringBootTest` with `TestRestTemplate` for integration testing.
*   **Example:** Walk through a simple POST endpoint that creates a resource, including validation and a success/error response.

## Question 8: Spring Boot vs. Traditional Spring Framework

**Question:** What are the main advantages of using Spring Boot over the traditional Spring Framework for developing new applications, especially microservices? When might you still consider using the traditional Spring Framework without Spring Boot, or what are some potential (though rare) downsides of Spring Boot's "opinionated" nature?

**Discussion Points / Sample Answer Outline:**

*   **Advantages of Spring Boot:**
    *   **Rapid Application Development:** Auto-configuration, starters, embedded servers drastically reduce setup time.
    *   **Simplified Dependency Management:** Starters manage versions and transitive dependencies.
    *   **Opinionated Defaults:** Sensible defaults for common use cases ("convention over configuration").
    *   **Embedded Servers (Tomcat, Jetty, Undertow):** No need to deploy WAR files to external servers; applications are self-contained JARs. Ideal for microservices and cloud deployments (Docker).
    *   **Production-Ready Features:** Actuator (health, metrics, etc.) out of the box.
    *   **No XML Configuration (mostly):** Focus on Java-based configuration.
    *   **Spring Boot CLI:** For quick prototyping.
    *   **Large and Active Community:** Extensive documentation and support.
*   **How these advantages help with Microservices:**
    *   Quick to bootstrap new services.
    *   Self-contained JARs are easy to containerize (Docker) and manage with orchestrators (ECS, Kubernetes).
    *   Actuator helps with monitoring in a distributed environment.
*   **When Traditional Spring (without Boot) Might Be Considered (Rare for new projects):**
    *   Existing large, legacy Spring applications not yet ready for a Boot migration.
    *   Situations requiring extremely fine-grained control over every aspect of Spring configuration where Boot's opinions might conflict (though Boot is highly customizable and allows overriding defaults).
    *   Very resource-constrained environments where even the minimal overhead of Boot's auto-configuration might be an issue (highly unlikely for most server-side apps).
    *   If you need to build a very specific type of application that Spring Boot doesn't have strong opinions or direct support for (e.g., certain types of desktop apps, though Spring Framework itself is also less common there now).
*   **Potential Downsides of Spring Boot's Opinions (and how to mitigate):**
    *   **"Magic" / Lack of Understanding:** Developers might not understand what's happening under the hood due to auto-configuration.
        *   Mitigation: Encourage learning, use Actuator's `/conditions` endpoint, read documentation.
    *   **Larger JAR size (initially):** Due to bundled dependencies, though this is usually not a major issue with modern deployment practices. Tree-shaking/optimization can help.
    *   **Overriding Defaults:** Sometimes requires understanding how to correctly override an auto-configured bean or property.
*   **Your Preference and Rationale:** Based on your experience building Java applications at Intralinks, Herc Rentals, etc., why has Spring Boot become the standard for new Spring projects?
---
These questions aim to cover a range of Spring Core and Spring Boot topics relevant to a senior Java engineer, prompting discussion of both theoretical understanding and practical application from Sridhar's project experiences.
