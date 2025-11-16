# Complete Spring Boot Reference Guide

## Table of Contents
1. [Spring Boot Fundamentals](#spring-boot-fundamentals)
2. [Core Concepts](#core-concepts)
3. [Data Access & Persistence](#data-access--persistence)
4. [RESTful Services](#restful-services)
5. [Security](#security)
6. [Microservices Patterns](#microservices-patterns)
7. [Testing](#testing)
8. [Production & Deployment](#production--deployment)
9. [Performance Optimization](#performance-optimization)
10. [Real-World Use Cases](#real-world-use-cases)

---

## Spring Boot Fundamentals

### What is Spring Boot?

| Aspect | Details |
|--------|---------|
| **Purpose** | Opinionated framework for building production-ready Spring applications |
| **Philosophy** | Convention over configuration |
| **Auto-Configuration** | Automatically configures beans based on classpath |
| **Embedded Servers** | Tomcat, Jetty, Undertow built-in |
| **Production Ready** | Actuator, metrics, health checks out-of-box |
| **Microservices** | First-class support for distributed systems |

**Key Philosophy:**
> "Just run" - No XML configuration, minimal setup, production-ready defaults

---

### Spring Boot vs Spring Framework

| Feature | Spring Framework | Spring Boot |
|---------|-----------------|-------------|
| Configuration | XML or Java Config | Auto-configuration |
| Server Setup | External server needed | Embedded server |
| Starter Dependencies | Manual selection | Pre-configured starters |
| Production Features | Manual setup | Actuator included |
| Setup Time | Hours/Days | Minutes |
| Learning Curve | Steep | Gentler |
| Use Case | Full control needed | Rapid development |

---

### Spring Boot Architecture Layers

```
┌─────────────────────────────────────┐
│     Presentation Layer              │
│  (Controllers, REST APIs)           │
├─────────────────────────────────────┤
│     Business Logic Layer            │
│  (Services, Business Rules)         │
├─────────────────────────────────────┤
│     Data Access Layer               │
│  (Repositories, Entities)           │
├─────────────────────────────────────┤
│     Database / External Services    │
└─────────────────────────────────────┘
```

---

## Core Concepts

### 1. Dependency Injection & IoC Container

**What It Solves:** Loose coupling, testability, maintainability

| Injection Type | Method | Use Case | Example |
|---------------|--------|----------|---------|
| Constructor | @Autowired on constructor | Immutable dependencies, required | `UserService(UserRepository repo)` |
| Setter | @Autowired on setter | Optional dependencies | `setEmailService(EmailService)` |
| Field | @Autowired on field | Quick prototyping (not recommended for prod) | `@Autowired UserRepository repo` |

**Best Practice: Constructor Injection**
```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    
    // Recommended: Constructor injection
    public OrderService(OrderRepository orderRepository, 
                       PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }
}
```

**Why Constructor Injection?**
- Immutable dependencies
- Easy to test (can mock in tests)
- Fail-fast (missing dependencies caught at startup)
- No need for @Autowired (Spring 4.3+)

---

### 2. Spring Boot Annotations Deep-Dive

#### Core Stereotypes

| Annotation | Layer | Purpose | Bean Scope |
|-----------|-------|---------|-----------|
| @Component | Generic | Generic Spring-managed component | Singleton |
| @Service | Business | Business logic layer | Singleton |
| @Repository | Data | Data access layer, exception translation | Singleton |
| @Controller | Web | MVC controller, returns views | Singleton |
| @RestController | API | REST API, returns JSON/XML | Singleton |
| @Configuration | Config | Java-based configuration | Singleton |

**Real-World Usage:**

```java
// Service Layer - Business Logic
@Service
public class PaymentService {
    public PaymentResult processPayment(Order order) {
        // Business logic here
    }
}

// Repository Layer - Data Access
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// REST API Layer
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrder(@PathVariable Long id) {
        return orderService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

---

### 3. Application Properties & Configuration

#### Configuration Hierarchy

| Priority | Source | Use Case |
|----------|--------|----------|
| 1 (Highest) | Command line args | Override for testing |
| 2 | System properties | JVM-level config |
| 3 | OS environment variables | Container/cloud config |
| 4 | application-{profile}.properties | Environment-specific |
| 5 | application.properties | Default config |

**Environment-Specific Configuration:**

```yaml
# application.yml (default)
spring:
  application:
    name: payment-service
  datasource:
    url: jdbc:mysql://localhost:3306/dev_db
    
---
# application-prod.yml
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://prod-db.example.com:3306/prod_db
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
```

**Configuration Properties Class:**

```java
@ConfigurationProperties(prefix = "payment")
@Validated
public class PaymentProperties {
    @NotBlank
    private String apiKey;
    
    @Min(1000)
    @Max(10000)
    private int timeout = 5000;
    
    private boolean sandboxMode = false;
    
    // Getters and setters
}

// Enable in main class
@SpringBootApplication
@EnableConfigurationProperties(PaymentProperties.class)
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

### 4. Bean Scopes & Lifecycle

| Scope | Description | Use Case | Thread-Safe? |
|-------|-------------|----------|--------------|
| Singleton | One instance per container | Stateless services | Must be |
| Prototype | New instance per request | Stateful objects | N/A |
| Request | One per HTTP request | Web request data | Yes |
| Session | One per HTTP session | User session data | Yes |
| Application | One per ServletContext | Global app state | Must be |

**Bean Lifecycle Hooks:**

```java
@Component
public class DatabaseConnectionManager {
    
    @PostConstruct
    public void init() {
        // Called after dependency injection
        System.out.println("Initializing database connection pool");
    }
    
    @PreDestroy
    public void cleanup() {
        // Called before bean destruction
        System.out.println("Closing database connections");
    }
}
```

**Custom Bean Creation:**

```java
@Configuration
public class AppConfig {
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setMaximumPoolSize(20);
        ds.setConnectionTimeout(30000);
        return ds;
    }
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}
```

---

## Data Access & Persistence

### 1. Spring Data JPA

**Repository Hierarchy:**

```
Repository (marker interface)
    ↓
CrudRepository (basic CRUD)
    ↓
PagingAndSortingRepository (pagination + sorting)
    ↓
JpaRepository (JPA-specific, batch operations)
```

**Query Methods Comparison:**

| Method Type | Example | Use Case | Performance |
|------------|---------|----------|-------------|
| Derived Query | `findByEmailAndStatus` | Simple queries | Good |
| @Query (JPQL) | `@Query("SELECT u FROM User u")` | Complex queries | Good |
| @Query (Native) | `@Query(nativeQuery=true)` | Database-specific | Excellent |
| Specification | Dynamic criteria | Runtime query building | Good |
| QueryDSL | Type-safe queries | Complex dynamic queries | Good |

**Real-World Repository:**

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    // Derived query method
    List<Order> findByCustomerIdAndStatus(Long customerId, OrderStatus status);
    
    // JPQL with pagination
    @Query("SELECT o FROM Order o WHERE o.totalAmount > :amount")
    Page<Order> findHighValueOrders(@Param("amount") BigDecimal amount, 
                                    Pageable pageable);
    
    // Native SQL for performance
    @Query(value = "SELECT * FROM orders WHERE created_at > DATE_SUB(NOW(), INTERVAL 30 DAY)", 
           nativeQuery = true)
    List<Order> findRecentOrders();
    
    // Custom method with @Modifying
    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.id = :orderId")
    int updateOrderStatus(@Param("orderId") Long orderId, 
                         @Param("status") OrderStatus status);
    
    // Projection for performance
    @Query("SELECT new com.example.dto.OrderSummary(o.id, o.totalAmount, o.status) " +
           "FROM Order o WHERE o.customerId = :customerId")
    List<OrderSummary> findOrderSummaries(@Param("customerId") Long customerId);
}
```

---

### 2. Entity Relationships

**Relationship Types & Best Practices:**

| Relationship | Annotation | Fetch Type | Cascade | Use Case |
|-------------|-----------|-----------|---------|----------|
| One-to-One | @OneToOne | LAZY | PERSIST, MERGE | User ↔ Profile |
| One-to-Many | @OneToMany | LAZY | ALL | Order → Items |
| Many-to-One | @ManyToOne | EAGER | PERSIST, MERGE | Items → Order |
| Many-to-Many | @ManyToMany | LAZY | PERSIST, MERGE | Students ↔ Courses |

**Entity Example with Best Practices:**

```java
@Entity
@Table(name = "orders", indexes = {
    @Index(name = "idx_customer_id", columnList = "customer_id"),
    @Index(name = "idx_order_date", columnList = "order_date")
})
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private Long customerId;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private OrderStatus status;
    
    // One-to-Many: LAZY loading to avoid N+1
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, 
               orphanRemoval = true, fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();
    
    @Column(name = "order_date")
    private LocalDateTime orderDate;
    
    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
    
    @Version
    private Long version; // Optimistic locking
    
    // Helper methods for bidirectional relationship
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
    
    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
    }
}
```

---

### 3. Transaction Management

**Transaction Propagation:**

| Propagation | Behavior | Use Case |
|------------|----------|----------|
| REQUIRED (default) | Join existing or create new | Most operations |
| REQUIRES_NEW | Always create new transaction | Independent operations |
| NESTED | Nested within existing | Savepoints |
| MANDATORY | Must have existing transaction | Called within transaction |
| SUPPORTS | Optional transaction | Read operations |
| NOT_SUPPORTED | Execute without transaction | External API calls |
| NEVER | Fail if transaction exists | Pure read operations |

**Transaction Isolation:**

| Level | Dirty Read | Non-Repeatable | Phantom | Use Case |
|-------|-----------|---------------|---------|----------|
| READ_UNCOMMITTED | Yes | Yes | Yes | Rare, analytics |
| READ_COMMITTED | No | Yes | Yes | Default (most DBs) |
| REPEATABLE_READ | No | No | Yes | Financial reports |
| SERIALIZABLE | No | No | No | Critical operations |

**Real-World Transaction Example:**

```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    
    @Transactional(
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        rollbackFor = Exception.class
    )
    public Order createOrder(OrderRequest request) {
        // 1. Create order
        Order order = new Order();
        order.setCustomerId(request.getCustomerId());
        order.setStatus(OrderStatus.PENDING);
        
        // 2. Reserve inventory
        inventoryService.reserveItems(request.getItems());
        
        // 3. Process payment
        PaymentResult payment = paymentService.processPayment(
            request.getPaymentDetails()
        );
        
        if (!payment.isSuccessful()) {
            throw new PaymentException("Payment failed"); // Rollback
        }
        
        order.setStatus(OrderStatus.CONFIRMED);
        return orderRepository.save(order);
    }
    
    // Read-only optimization
    @Transactional(readOnly = true)
    public List<Order> getCustomerOrders(Long customerId) {
        return orderRepository.findByCustomerId(customerId);
    }
    
    // New transaction for audit log
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderEvent(Long orderId, String event) {
        // This will commit even if parent transaction fails
        auditRepository.save(new AuditLog(orderId, event));
    }
}
```

---

### 4. Database Connection Pooling

**HikariCP Configuration (Default in Spring Boot):**

```yaml
spring:
  datasource:
    hikari:
      # Pool sizing
      maximum-pool-size: 20        # Max connections
      minimum-idle: 5              # Min idle connections
      
      # Connection lifecycle
      connection-timeout: 30000    # Max wait for connection (ms)
      idle-timeout: 600000         # Max idle time (10 min)
      max-lifetime: 1800000        # Max connection lifetime (30 min)
      
      # Performance
      auto-commit: false           # Manual commit for transactions
      connection-test-query: SELECT 1
      
      # Monitoring
      leak-detection-threshold: 60000  # Detect connection leaks (60s)
      register-mbeans: true            # JMX monitoring
```

**Connection Pool Sizing:**

```
Optimal Pool Size = (Core Count × 2) + Effective Spindle Count

For CPU-bound: Core Count + 1
For I/O-bound: Core Count × 2
For mixed workload: Start with 10-20, tune based on metrics
```

---

## RESTful Services

### 1. REST API Best Practices

**HTTP Methods Mapping:**

| Method | Operation | Idempotent | Safe | Request Body | Response Body |
|--------|-----------|-----------|------|--------------|---------------|
| GET | Read | Yes | Yes | No | Yes |
| POST | Create | No | No | Yes | Yes |
| PUT | Update/Replace | Yes | No | Yes | Yes |
| PATCH | Partial Update | No | No | Yes | Yes |
| DELETE | Delete | Yes | No | Optional | Optional |

**REST Controller Example:**

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {
    
    private final UserService userService;
    
    // GET - List with pagination
    @GetMapping
    public ResponseEntity<Page<UserDTO>> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(required = false) String search
    ) {
        Pageable pageable = PageRequest.of(page, size);
        Page<UserDTO> users = userService.findUsers(search, pageable);
        return ResponseEntity.ok(users);
    }
    
    // GET - Single resource
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // POST - Create
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<UserDTO> createUser(
        @Valid @RequestBody CreateUserRequest request
    ) {
        UserDTO created = userService.createUser(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
    
    // PUT - Full update
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
        @PathVariable Long id,
        @Valid @RequestBody UpdateUserRequest request
    ) {
        return userService.updateUser(id, request)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // PATCH - Partial update
    @PatchMapping("/{id}")
    public ResponseEntity<UserDTO> patchUser(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates
    ) {
        return userService.patchUser(id, updates)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // DELETE
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### 2. Request Validation

**Validation Annotations:**

| Annotation | Purpose | Example |
|-----------|---------|---------|
| @NotNull | Not null | `@NotNull String name` |
| @NotBlank | Not null/empty/whitespace | `@NotBlank String email` |
| @NotEmpty | Not null/empty | `@NotEmpty List<String> items` |
| @Size | String/collection size | `@Size(min=8, max=20) String password` |
| @Min/@Max | Numeric bounds | `@Min(0) @Max(150) int age` |
| @Email | Valid email | `@Email String email` |
| @Pattern | Regex match | `@Pattern(regexp="^[0-9]{10}$") String phone` |
| @Past/@Future | Date validation | `@Past LocalDate birthDate` |

**Request DTO with Validation:**

```java
public class CreateUserRequest {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(
        regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$",
        message = "Password must contain uppercase, lowercase, digit, and special character"
    )
    private String password;
    
    @Min(value = 18, message = "Must be at least 18 years old")
    @Max(value = 150, message = "Age must be realistic")
    private Integer age;
    
    @NotEmpty(message = "At least one role is required")
    private Set<@NotBlank String> roles;
    
    // Custom validator
    @ValidPhoneNumber
    private String phoneNumber;
}

// Custom validator annotation
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneNumberValidator.class)
public @interface ValidPhoneNumber {
    String message() default "Invalid phone number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

---

### 3. Exception Handling

**Global Exception Handler:**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationException(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        return ErrorResponse.builder()
            .status(HttpStatus.BAD_REQUEST.value())
            .message("Validation failed")
            .errors(errors)
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    // Handle resource not found
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFoundException(ResourceNotFoundException ex) {
        return ErrorResponse.builder()
            .status(HttpStatus.NOT_FOUND.value())
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    // Handle business logic errors
    @ExceptionHandler(BusinessException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ErrorResponse handleBusinessException(BusinessException ex) {
        return ErrorResponse.builder()
            .status(HttpStatus.UNPROCESSABLE_ENTITY.value())
            .message(ex.getMessage())
            .errorCode(ex.getErrorCode())
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    // Handle database errors
    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDataIntegrityViolation(
        DataIntegrityViolationException ex
    ) {
        String message = "Database constraint violation";
        if (ex.getMessage().contains("duplicate key")) {
            message = "Resource already exists";
        }
        
        return ErrorResponse.builder()
            .status(HttpStatus.CONFLICT.value())
            .message(message)
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    // Catch-all for unexpected errors
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenericException(Exception ex) {
        log.error("Unexpected error", ex);
        
        return ErrorResponse.builder()
            .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
            .message("An unexpected error occurred")
            .timestamp(LocalDateTime.now())
            .build();
    }
}

@Data
@Builder
public class ErrorResponse {
    private int status;
    private String message;
    private String errorCode;
    private Map<String, String> errors;
    private LocalDateTime timestamp;
}
```

---

### 4. API Versioning Strategies

| Strategy | Implementation | Pros | Cons | Example |
|----------|---------------|------|------|---------|
| URI Versioning | `/api/v1/users` | Clear, cacheable | URL pollution | Most common |
| Header Versioning | `API-Version: 1` | Clean URLs | Less visible | GitHub API |
| Query Parameter | `/api/users?version=1` | Easy to implement | URL pollution | Less common |
| Content Negotiation | `Accept: application/vnd.api.v1+json` | RESTful | Complex | Rare |

**URI Versioning Example:**

```java
// Version 1
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    @GetMapping("/{id}")
    public UserDTOV1 getUser(@PathVariable Long id) {
        return userService.getUserV1(id);
    }
}

// Version 2 with breaking changes
@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    @GetMapping("/{id}")
    public UserDTOV2 getUser(@PathVariable Long id) {
        return userService.getUserV2(id);
    }
}
```

---

## Security

### 1. Spring Security Basics

**Security Architecture:**

```
Request → SecurityFilterChain
    ↓
Authentication Filter
    ↓
Authentication Manager
    ↓
Authentication Provider
    ↓
UserDetailsService
    ↓
Authorization
    ↓
Controller
```

**Basic Security Configuration:**

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            )
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/api/public/**", "/actuator/health").permitAll()
                
                // Swagger/OpenAPI
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                
                // Role-based access
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                
                // Authenticated requests
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authenticationEntryPoint())
                .accessDeniedHandler(accessDeniedHandler())
            )
            .addFilterBefore(jwtAuthenticationFilter(), 
                            UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
        AuthenticationConfiguration config
    ) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

### 2. JWT Authentication

**JWT Token Structure:**

| Part | Content | Purpose |
|------|---------|---------|
| Header | Algorithm & token type | `{"alg":"HS256","typ":"JWT"}` |
| Payload | Claims (user data) | `{"sub":"user@example.com","roles":["USER"]}` |
| Signature | Verification | `HMACSHA256(base64(header)+"."+base64(payload), secret)` |

**JWT Service Implementation:**

```java
@Service
public class JwtService {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration; // milliseconds
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }
    
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
    
    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }
    
    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

**JWT Authentication Filter:**

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {
        
        final String authHeader = request.getHeader("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        
        try {
            final String jwt = authHeader.substring(7);
            final String username = jwtService.extractUsername(jwt);
            
            if (username != null && SecurityContextHolder.getContext()
                .getAuthentication() == null) {
                
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                if (jwtService.isTokenValid(jwt, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                            userDetails,
                            null,
                            userDetails.getAuthorities()
                        );
                    
                    authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                    );
                    
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            log.error("JWT authentication failed", e);
        }
        
        filterChain.doFilter(request, response);
    }
}
```

---

### 3. Method-Level Security

```java
@Service
public class OrderService {
    
    // Only authenticated users
    @PreAuthorize("isAuthenticated()")
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }
    
    // Role-based
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(Long orderId) {
        orderRepository.deleteById(orderId);
    }
    
    // Multiple roles
    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public Order approveOrder(Long orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();
        order.setStatus(OrderStatus.APPROVED);
        return orderRepository.save(order);
    }
    
    // Owner or admin
    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }
    
    // Complex expression
    @PreAuthorize("@orderSecurityService.canAccessOrder(#orderId, authentication)")
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId).orElseThrow();
    }
    
    // Post authorization
    @PostAuthorize("returnObject.userId == authentication.principal.id")
    public Order findOrder(Long orderId) {
        return orderRepository.findById(orderId).orElseThrow();
    }
}

// Custom security service
@Service("orderSecurityService")
public class OrderSecurityService {
    
    public boolean canAccessOrder(Long orderId, Authentication authentication) {
        UserDetails user = (UserDetails) authentication.getPrincipal();
        Order order = orderRepository.findById(orderId).orElse(null);
        
        if (order == null) return false;
        
        // Admin can access all
        if (hasRole(authentication, "ADMIN")) return true;
        
        // User can access their own orders
        return order.getUserEmail().equals(user.getUsername());
    }
    
    private boolean hasRole(Authentication auth, String role) {
        return auth.getAuthorities().stream()
            .anyMatch(a -> a.getAuthority().equals("ROLE_" + role));
    }
}
```

---

### 4. OAuth2 & Social Login

**OAuth2 Configuration:**

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope:
              - email
              - profile
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope:
              - user:email
              - read:user
```

**OAuth2 Security Configuration:**

```java
@Configuration
public class OAuth2SecurityConfig {
    
    @Bean
    public SecurityFilterChain oauth2FilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/error").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .failureUrl("/login?error=true")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(oauth2UserService())
                )
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
            );
        
        return http.build();
    }
    
    @Bean
    public OAuth2UserService<OAuth2UserRequest, OAuth2User> oauth2UserService() {
        return new CustomOAuth2UserService();
    }
}

@Service
public class CustomOAuth2UserService 
    extends DefaultOAuth2UserService {
    
    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) {
        OAuth2User oauth2User = super.loadUser(userRequest);
        
        // Process OAuth2 user
        String registrationId = userRequest.getClientRegistration()
            .getRegistrationId();
        String email = oauth2User.getAttribute("email");
        
        // Save or update user in database
        User user = userRepository.findByEmail(email)
            .orElseGet(() -> createUser(oauth2User, registrationId));
        
        return new CustomOAuth2User(oauth2User, user);
    }
    
    private User createUser(OAuth2User oauth2User, String provider) {
        User user = new User();
        user.setEmail(oauth2User.getAttribute("email"));
        user.setName(oauth2User.getAttribute("name"));
        user.setProvider(provider);
        user.setProviderId(oauth2User.getName());
        return userRepository.save(user);
    }
}
```

---

## Microservices Patterns

### 1. Service Discovery with Eureka

**Eureka Server:**

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
```

**Eureka Client (Microservice):**

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

```yaml
# application.yml
spring:
  application:
    name: order-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 30
```

---

### 2. API Gateway Pattern

**Spring Cloud Gateway Configuration:**

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // Order Service
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    .circuitBreaker(c -> c
                        .setName("orderServiceCircuitBreaker")
                        .setFallbackUri("forward:/fallback/orders")
                    )
                    .retry(config -> config
                        .setRetries(3)
                        .setStatuses(HttpStatus.BAD_GATEWAY)
                    )
                )
                .uri("lb://order-service")
            )
            // Payment Service
            .route("payment-service", r -> r
                .path("/api/payments/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .requestRateLimiter(c -> c
                        .setRateLimiter(redisRateLimiter())
                    )
                )
                .uri("lb://payment-service")
            )
            // User Service with authentication
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .filter(new JwtAuthenticationFilter())
                )
                .uri("lb://user-service")
            )
            .build();
    }
    
    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(10, 20); // 10 requests per second
    }
}
```

**Gateway Global Filters:**

```java
@Component
public class GlobalLoggingFilter implements GlobalFilter, Ordered {
    
    private static final Logger log = LoggerFactory.getLogger(GlobalLoggingFilter.class);
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String requestId = UUID.randomUUID().toString();
        
        log.info("Request ID: {} - Path: {} - Method: {}",
            requestId,
            exchange.getRequest().getPath(),
            exchange.getRequest().getMethod()
        );
        
        exchange.getRequest().mutate()
            .header("X-Request-ID", requestId)
            .build();
        
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            log.info("Request ID: {} - Status: {}",
                requestId,
                exchange.getResponse().getStatusCode()
            );
        }));
    }
    
    @Override
    public int getOrder() {
        return -1; // High priority
    }
}
```

---

### 3. Circuit Breaker with Resilience4j

**Dependencies & Configuration:**

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        wait-duration-in-open-state: 30s
        failure-rate-threshold: 50
        slow-call-rate-threshold: 100
        slow-call-duration-threshold: 2s
        
  retry:
    instances:
      paymentService:
        max-attempts: 3
        wait-duration: 1s
        enable-exponential-backoff: true
        exponential-backoff-multiplier: 2
        
  ratelimiter:
    instances:
      paymentService:
        limit-for-period: 10
        limit-refresh-period: 1s
        timeout-duration: 0s
```

**Service Implementation with Resilience4j:**

```java
@Service
public class OrderService {
    
    private final PaymentServiceClient paymentClient;
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @RateLimiter(name = "paymentService")
    public PaymentResponse processPayment(PaymentRequest request) {
        return paymentClient.processPayment(request);
    }
    
    // Fallback method
    private PaymentResponse paymentFallback(PaymentRequest request, Exception ex) {
        log.error("Payment service unavailable, using fallback", ex);
        
        // Return cached response or default
        return PaymentResponse.builder()
            .status("PENDING")
            .message("Payment is being processed. You'll receive confirmation shortly.")
            .build();
    }
    
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResponse> processPaymentAsync(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> 
            paymentClient.processPayment(request)
        );
    }
}

// Circuit Breaker Event Listener
@Component
public class CircuitBreakerEventListener {
    
    @EventListener
    public void onCircuitBreakerEvent(CircuitBreakerOnStateTransitionEvent event) {
        log.warn("Circuit Breaker {} changed state from {} to {}",
            event.getCircuitBreakerName(),
            event.getStateTransition().getFromState(),
            event.getStateTransition().getToState()
        );
        
        // Send alert to monitoring system
        if (event.getStateTransition().getToState() == CircuitBreaker.State.OPEN) {
            alertingService.sendAlert(
                "Circuit breaker opened for " + event.getCircuitBreakerName()
            );
        }
    }
}
```

---

### 4. Distributed Tracing with Sleuth & Zipkin

**Configuration:**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

```yaml
spring:
  sleuth:
    sampler:
      probability: 1.0  # Sample 100% in dev, reduce in prod (0.1 = 10%)
  zipkin:
    base-url: http://localhost:9411
    enabled: true
```

**Custom Span Creation:**

```java
@Service
public class OrderProcessingService {
    
    private final Tracer tracer;
    
    public Order processOrder(OrderRequest request) {
        // Create custom span
        Span span = tracer.nextSpan().name("process-order").start();
        
        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            // Add tags for filtering
            span.tag("order.id", request.getOrderId().toString());
            span.tag("order.amount", request.getAmount().toString());
            span.tag("customer.id", request.getCustomerId().toString());
            
            // Business logic
            validateOrder(request);
            
            // Create nested span
            Span inventorySpan = tracer.nextSpan().name("check-inventory").start();
            try (Tracer.SpanInScope ws2 = tracer.withSpan(inventorySpan)) {
                checkInventory(request);
            } finally {
                inventorySpan.end();
            }
            
            processPayment(request);
            
            // Add event
            span.event("order-processed");
            
            return createOrder(request);
            
        } catch (Exception e) {
            span.tag("error", e.getMessage());
            span.error(e);
            throw e;
        } finally {
            span.end();
        }
    }
}
```

---

### 5. Feign Client for Service Communication

**Feign Client Definition:**

```java
@FeignClient(
    name = "payment-service",
    url = "${payment-service.url}",
    fallback = PaymentServiceFallback.class,
    configuration = FeignConfig.class
)
public interface PaymentServiceClient {
    
    @GetMapping("/api/payments/{id}")
    PaymentDTO getPayment(@PathVariable("id") Long id);
    
    @PostMapping("/api/payments")
    PaymentResponse processPayment(@RequestBody PaymentRequest request);
    
    @GetMapping("/api/payments/user/{userId}")
    List<PaymentDTO> getUserPayments(
        @PathVariable("userId") Long userId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size
    );
}

// Fallback implementation
@Component
public class PaymentServiceFallback implements PaymentServiceClient {
    
    @Override
    public PaymentDTO getPayment(Long id) {
        return PaymentDTO.builder()
            .id(id)
            .status("UNAVAILABLE")
            .build();
    }
    
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        return PaymentResponse.builder()
            .status("PENDING")
            .message("Service temporarily unavailable")
            .build();
    }
    
    @Override
    public List<PaymentDTO> getUserPayments(Long userId, int page, int size) {
        return Collections.emptyList();
    }
}

// Feign Configuration
@Configuration
public class FeignConfig {
    
    @Bean
    public RequestInterceptor requestInterceptor() {
        return template -> {
            // Add authentication header
            String token = SecurityContextHolder.getContext()
                .getAuthentication()
                .getCredentials()
                .toString();
            template.header("Authorization", "Bearer " + token);
            
            // Add correlation ID
            template.header("X-Correlation-ID", MDC.get("correlationId"));
        };
    }
    
    @Bean
    public ErrorDecoder errorDecoder() {
        return new CustomErrorDecoder();
    }
    
    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}

// Custom Error Decoder
public class CustomErrorDecoder implements ErrorDecoder {
    
    private final ErrorDecoder defaultDecoder = new Default();
    
    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                return new BadRequestException("Bad request to payment service");
            case 404:
                return new ResourceNotFoundException("Payment not found");
            case 503:
                return new ServiceUnavailableException("Payment service unavailable");
            default:
                return defaultDecoder.decode(methodKey, response);
        }
    }
}
```

---

## Testing

### 1. Unit Testing

**Service Layer Testing:**

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock
    private OrderRepository orderRepository;
    
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private InventoryService inventoryService;
    
    @InjectMocks
    private OrderService orderService;
    
    @Test
    @DisplayName("Should create order successfully")
    void shouldCreateOrderSuccessfully() {
        // Given
        OrderRequest request = OrderRequest.builder()
            .customerId(1L)
            .items(List.of(new OrderItem(1L, 2)))
            .build();
        
        Order savedOrder = new Order();
        savedOrder.setId(1L);
        savedOrder.setStatus(OrderStatus.CONFIRMED);
        
        when(inventoryService.reserveItems(any())).thenReturn(true);
        when(paymentService.processPayment(any())).thenReturn(
            new PaymentResult(true, "SUCCESS")
        );
        when(orderRepository.save(any(Order.class))).thenReturn(savedOrder);
        
        // When
        Order result = orderService.createOrder(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
        
        verify(inventoryService).reserveItems(request.getItems());
        verify(paymentService).processPayment(any());
        verify(orderRepository).save(any(Order.class));
    }
    
    @Test
    @DisplayName("Should rollback order when payment fails")
    void shouldRollbackWhenPaymentFails() {
        // Given
        OrderRequest request = OrderRequest.builder()
            .customerId(1L)
            .build();
        
        when(inventoryService.reserveItems(any())).thenReturn(true);
        when(paymentService.processPayment(any()))
            .thenThrow(new PaymentException("Payment failed"));
        
        // When & Then
        assertThatThrownBy(() -> orderService.createOrder(request))
            .isInstanceOf(PaymentException.class)
            .hasMessage("Payment failed");
        
        verify(orderRepository, never()).save(any());
    }
    
    @Test
    @DisplayName("Should find orders by customer ID")
    void shouldFindOrdersByCustomerId() {
        // Given
        Long customerId = 1L;
        List<Order> expectedOrders = Arrays.asList(
            createOrder(1L, customerId),
            createOrder(2L, customerId)
        );
        
        when(orderRepository.findByCustomerId(customerId))
            .thenReturn(expectedOrders);
        
        // When
        List<Order> result = orderService.getCustomerOrders(customerId);
        
        // Then
        assertThat(result)
            .hasSize(2)
            .extracting(Order::getCustomerId)
            .containsOnly(customerId);
    }
    
    private Order createOrder(Long id, Long customerId) {
        Order order = new Order();
        order.setId(id);
        order.setCustomerId(customerId);
        order.setStatus(OrderStatus.PENDING);
        return order;
    }
}
```

---

### 2. Integration Testing

**Repository Integration Tests:**

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderRepositoryIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void shouldFindOrdersByCustomerIdAndStatus() {
        // Given
        Order order1 = createOrder(1L, OrderStatus.PENDING);
        Order order2 = createOrder(1L, OrderStatus.CONFIRMED);
        Order order3 = createOrder(2L, OrderStatus.PENDING);
        
        entityManager.persist(order1);
        entityManager.persist(order2);
        entityManager.persist(order3);
        entityManager.flush();
        
        // When
        List<Order> result = orderRepository
            .findByCustomerIdAndStatus(1L, OrderStatus.PENDING);
        
        // Then
        assertThat(result)
            .hasSize(1)
            .first()
            .satisfies(order -> {
                assertThat(order.getCustomerId()).isEqualTo(1L);
                assertThat(order.getStatus()).isEqualTo(OrderStatus.PENDING);
            });
    }
    
    @Test
    void shouldFindHighValueOrders() {
        // Given
        Order lowValue = createOrder(1L, new BigDecimal("50.00"));
        Order highValue1 = createOrder(1L, new BigDecimal("1500.00"));
        Order highValue2 = createOrder(2L, new BigDecimal("2000.00"));
        
        entityManager.persist(lowValue);
        entityManager.persist(highValue1);
        entityManager.persist(highValue2);
        entityManager.flush();
        
        // When
        Page<Order> result = orderRepository.findHighValueOrders(
            new BigDecimal("1000.00"),
            PageRequest.of(0, 10)
        );
        
        // Then
        assertThat(result.getContent())
            .hasSize(2)
            .extracting(Order::getTotalAmount)
            .allMatch(amount -> amount.compareTo(new BigDecimal("1000.00")) > 0);
    }
    
    private Order createOrder(Long customerId, OrderStatus status) {
        Order order = new Order();
        order.setCustomerId(customerId);
        order.setStatus(status);
        order.setTotalAmount(new BigDecimal("100.00"));
        order.setOrderDate(LocalDateTime.now());
        return order;
    }
}
```

---

### 3. REST API Testing

**Controller Integration Tests:**

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class OrderControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @MockBean
    private OrderService orderService;
    
    @Test
    @WithMockUser(roles = "USER")
    void shouldGetOrderSuccessfully() throws Exception {
        // Given
        Long orderId = 1L;
        OrderDTO orderDTO = OrderDTO.builder()
            .id(orderId)
            .customerId(1L)
            .status("CONFIRMED")
            .totalAmount(new BigDecimal("150.00"))
            .build();
        
        when(orderService.findById(orderId))
            .thenReturn(Optional.of(orderDTO));
        
        // When & Then
        mockMvc.perform(get("/api/v1/orders/{id}", orderId)
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(orderId))
            .andExpect(jsonPath("$.status").value("CONFIRMED"))
            .andExpect(jsonPath("$.totalAmount").value(150.00))
            .andDo(print());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void shouldReturn404WhenOrderNotFound() throws Exception {
        // Given
        Long orderId = 999L;
        when(orderService.findById(orderId))
            .thenReturn(Optional.empty());
        
        // When & Then
        mockMvc.perform(get("/api/v1/orders/{id}", orderId))
            .andExpect(status().isNotFound());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void shouldCreateOrderSuccessfully() throws Exception {
        // Given
        CreateOrderRequest request = CreateOrderRequest.builder()
            .customerId(1L)
            .items(List.of(new OrderItemRequest(1L, 2)))
            .build();
        
        OrderDTO createdOrder = OrderDTO.builder()
            .id(1L)
            .customerId(1L)
            .status("PENDING")
            .build();
        
        when(orderService.createOrder(any()))
            .thenReturn(createdOrder);
        
        // When & Then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.status").value("PENDING"));
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void shouldReturn400WhenValidationFails() throws Exception {
        // Given - Invalid request (missing customerId)
        CreateOrderRequest request = CreateOrderRequest.builder()
            .items(List.of())
            .build();
        
        // When & Then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").exists());
    }
    
    @Test
    void shouldReturn401WhenNotAuthenticated() throws Exception {
        mockMvc.perform(get("/api/v1/orders/1"))
            .andExpect(status().isUnauthorized());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void shouldReturn403WhenAccessingAdminEndpoint() throws Exception {
        mockMvc.perform(delete("/api/v1/admin/orders/1"))
            .andExpect(status().isForbidden());
    }
}
```

---

### 4. TestContainers for Real Dependencies

```java
@SpringBootTest
@Testcontainers
@ActiveProfiles("integration-test")
class OrderServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.4.0")
    );
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
        
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Test
    void shouldCreateAndRetrieveOrder() {
        // Given
        CreateOrderRequest request = CreateOrderRequest.builder()
            .customerId(1L)
            .items(List.of(new OrderItemRequest(1L, 2)))
            .build();
        
        // When
        OrderDTO created = orderService.createOrder(request);
        Optional<OrderDTO> retrieved = orderService.findById(created.getId());
        
        // Then
        assertThat(retrieved).isPresent();
        assertThat(retrieved.get().getId()).isEqualTo(created.getId());
    }
}
```

---

## Production & Deployment

### 1. Spring Boot Actuator

**Actuator Configuration:**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,env
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true
  health:
    redis:
      enabled: true
    db:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
```

**Custom Health Indicator:**

```java
@Component
public class PaymentServiceHealthIndicator implements HealthIndicator {
    
    private final PaymentServiceClient paymentClient;
    
    @Override
    public Health health() {
        try {
            // Check if payment service is accessible
            paymentClient.healthCheck();
            
            return Health.up()
                .withDetail("payment-service", "Available")
                .withDetail("last-check", LocalDateTime.now())
                .build();
                
        } catch (Exception e) {
            return Health.down()
                .withDetail("payment-service", "Unavailable")
                .withDetail("error", e.getMessage())
                .withDetail("last-check", LocalDateTime.now())
                .build();
        }
    }
}
```

**Custom Metrics:**

```java
@Service
public class OrderMetricsService {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCreatedCounter;
    private final Counter orderFailedCounter;
    private final Timer orderProcessingTimer;
    
    public OrderMetricsService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        this.orderCreatedCounter = Counter.builder("orders.created")
            .description("Total number of orders created")
            .tag("status", "success")
            .register(meterRegistry);
        
        this.orderFailedCounter = Counter.builder("orders.failed")
            .description("Total number of failed orders")
            .tag("status", "failed")
            .register(meterRegistry);
        
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
    }
    
    public void recordOrderCreated() {
        orderCreatedCounter.increment();
    }
    
    public void recordOrderFailed(String reason) {
        orderFailedCounter.increment();
        meterRegistry.counter("orders.failed.reason", "reason", reason).increment();
    }
    
    public void recordProcessingTime(Runnable task) {
        orderProcessingTimer.record(task);
    }
    
    // Gauge for active orders
    @PostConstruct
    public void registerGauges() {
        Gauge.builder("orders.active", orderRepository, OrderRepository::countActiveOrders)
            .description("Number of active orders")
            .register(meterRegistry);
    }
}
```

**Monitoring Endpoints:**

| Endpoint | Purpose | Example |
|----------|---------|---------|
| `/actuator/health` | Application health | Liveness/readiness probes |
| `/actuator/metrics` | Available metrics | List all metrics |
| `/actuator/metrics/{metric}` | Specific metric | JVM memory, HTTP requests |
| `/actuator/prometheus` | Prometheus format | Scraping endpoint |
| `/actuator/info` | Application info | Version, build info |
| `/actuator/env` | Environment properties | Configuration values |
| `/actuator/loggers` | Logger configuration | Change log levels |

---

### 2. Logging Best Practices

**Logging Configuration:**

```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
    max-size: 10MB
    max-history: 30
```

**Structured Logging with Logback:**

```xml
<!-- logback-spring.xml -->
<configuration>
    <springProperty scope="context" name="appName" source="spring.application.name"/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app":"${appName}"}</customFields>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy 
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

**Logging in Code:**

```java
@Service
@Slf4j
public class OrderService {
    
    public Order createOrder(OrderRequest request) {
        // Use MDC for correlation
        MDC.put("orderId", request.getOrderId().toString());
        MDC.put("customerId", request.getCustomerId().toString());
        
        try {
            log.info("Creating order for customer: {}", request.getCustomerId());
            log.debug("Order details: {}", request);
            
            Order order = processOrder(request);
            
            log.info("Order created successfully with ID: {}", order.getId());
            return order;
            
        } catch (InsufficientInventoryException e) {
            log.warn("Insufficient inventory for order: {}", request.getOrderId(), e);
            throw e;
        } catch (Exception e) {
            log.error("Failed to create order: {}", request.getOrderId(), e);
            throw new OrderCreationException("Order creation failed", e);
        } finally {
            MDC.clear();
        }
    }
}
```

---

### 3. Docker Configuration

**Multi-stage Dockerfile:**

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Expose port
EXPOSE 8080

# JVM options
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Docker Compose for Local Development:**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orderdb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper

  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    ports:
      - "2181:2181"

  order-service:
    build: .
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/orderdb
      SPRING_DATASOURCE_USERNAME: admin
      SPRING_DATASOURCE_PASSWORD: secret
      SPRING_REDIS_HOST: redis
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      kafka:
        condition: service_started
    healthcheck:
      test: ["CMD", "wget", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

volumes:
  postgres_data:
```

---

### 4. Kubernetes Deployment

**Deployment YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: order-service
        version: v1
    spec:
      containers:
      - name: order-service
        image: order-service:1.0.0
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: SPRING_DATASOURCE_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: config
          mountPath: /config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: order-service-config

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: order-service

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**ConfigMap:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  application.yml: |
    spring:
      application:
        name: order-service
      jpa:
        hibernate:
          ddl-auto: validate
        show-sql: false
    management:
      endpoints:
        web:
          exposure:
            include: health,info,metrics,prometheus
```

---

## Performance Optimization

### 1. Database Query Optimization

**N+1 Query Problem:**

```java
// BAD: N+1 queries
@GetMapping("/orders")
public List<OrderDTO> getOrders() {
    List<Order> orders = orderRepository.findAll();
    // This triggers N queries for order items
    return orders.stream()
        .map(order -> new OrderDTO(order, order.getItems()))
        .collect(Collectors.toList());
}

// GOOD: Use fetch join
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items")
    List<Order> findAllWithItems();
    
    // Or with EntityGraph
    @EntityGraph(attributePaths = {"items", "customer"})
    List<Order> findAll();
}
```

**Pagination for Large Result Sets:**

```java
@Service
public class OrderService {
    
    public Page<OrderDTO> getOrders(int page, int size) {
        Pageable pageable = PageRequest.of(
            page, 
            size, 
            Sort.by("orderDate").descending()
        );
        
        return orderRepository.findAll(pageable)
            .map(this::toDTO);
    }
    
    // Slice for infinite scroll (better performance)
    public Slice<OrderDTO> getOrdersSlice(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return orderRepository.findAllBy(pageable)
            .map(this::toDTO);
    }
}
```

**Query Projection for Performance:**

```java
// Projection interface
public interface OrderSummary {
    Long getId();
    String getCustomerName();
    BigDecimal getTotalAmount();
    String getStatus();
}

@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    // Returns only needed fields
    List<OrderSummary> findAllProjectedBy();
    
    // DTO projection
    @Query("SELECT new com.example.dto.OrderDTO(o.id, o.totalAmount, o.status) " +
           "FROM Order o")
    List<OrderDTO> findAllDTOs();
}
```

---

### 2. Caching Strategies

**Cache Configuration:**

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
        
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        
        // Different TTL for different caches
        cacheConfigurations.put("users", config.entryTtl(Duration.ofHours(1)));
        cacheConfigurations.put("products", config.entryTtl(Duration.ofMinutes(30)));
        cacheConfigurations.put("orders", config.entryTtl(Duration.ofMinutes(5)));
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```

**Cache Annotations:**

```java
@Service
public class ProductService {
    
    // Cache result
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }
    
    // Cache with condition
    @Cacheable(value = "products", key = "#id", condition = "#id > 0")
    public Product getProductConditional(Long id) {
        return productRepository.findById(id).orElse(null);
    }
    
    // Update cache
    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }
    
    // Evict cache
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
    
    // Evict all entries
    @CacheEvict(value = "products", allEntries = true)
    public void clearProductCache() {
        // Method body
    }
    
    // Multiple cache operations
    @Caching(
        evict = {
            @CacheEvict(value = "products", key = "#product.id"),
            @CacheEvict(value = "productList", allEntries = true)
        },
        put = {
            @CachePut(value = "products", key = "#product.id")
        }
    )
    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }
}
```

**Cache Comparison:**

| Cache Type | Use Case | TTL | Distribution | Performance |
|-----------|----------|-----|--------------|-------------|
| In-Memory (Caffeine) | Local cache | Short | Single instance | Fastest |
| Redis | Distributed cache | Configurable | Multi-instance | Fast |
| Hazelcast | Distributed cache | Configurable | Multi-instance | Fast |
| Database | Persistent cache | Long/Forever | Multi-instance | Slower |

---

### 3. Async Processing

**Async Configuration:**

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
    
    @Bean(name = "emailExecutor")
    public Executor emailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("email-");
        executor.initialize();
        return executor;
    }
}
```

**Async Methods:**

```java
@Service
public class NotificationService {
    
    @Async("emailExecutor")
    public CompletableFuture<String> sendEmail(String to, String subject, String body) {
        try {
            // Simulate email sending
            Thread.sleep(2000);
            log.info("Email sent to: {}", to);
            return CompletableFuture.completedFuture("Email sent successfully");
        } catch (Exception e) {
            log.error("Failed to send email", e);
            return CompletableFuture.failedFuture(e);
        }
    }
    
    @Async("taskExecutor")
    public void processOrderAsync(Order order) {
        log.info("Processing order asynchronously: {}", order.getId());
        // Long-running process
        generateInvoice(order);
        updateInventory(order);
        notifyWarehouse(order);
    }
    
    // Return CompletableFuture for composition
    @Async
    public CompletableFuture<OrderReport> generateReport(Long orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();
        OrderReport report = createReport(order);
        return CompletableFuture.completedFuture(report);
    }
}

// Using async methods
@Service
public class OrderService {
    
    private final NotificationService notificationService;
    
    public void completeOrder(Long orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();
        order.setStatus(OrderStatus.COMPLETED);
        orderRepository.save(order);
        
        // Fire and forget
        notificationService.processOrderAsync(order);
        
        // With callback
        notificationService.sendEmail(
            order.getCustomerEmail(),
            "Order Completed",
            "Your order has been completed"
        ).thenAccept(result -> {
            log.info("Email notification result: {}", result);
        }).exceptionally(ex -> {
            log.error("Failed to send notification", ex);
            return null;
        });
    }
    
    // Combine multiple async operations
    public CompletableFuture<OrderSummary> getOrderSummary(Long orderId) {
        CompletableFuture<Order> orderFuture = 
            CompletableFuture.supplyAsync(() -> getOrder(orderId));
        
        CompletableFuture<List<Payment>> paymentsFuture = 
            CompletableFuture.supplyAsync(() -> getPayments(orderId));
        
        CompletableFuture<List<Shipment>> shipmentsFuture = 
            CompletableFuture.supplyAsync(() -> getShipments(orderId));
        
        return CompletableFuture.allOf(orderFuture, paymentsFuture, shipmentsFuture)
            .thenApply(v -> new OrderSummary(
                orderFuture.join(),
                paymentsFuture.join(),
                shipmentsFuture.join()
            ));
    }
}
```

---

### 4. Connection Pool Tuning

**HikariCP Best Practices:**

```yaml
spring:
  datasource:
    hikari:
      # Pool sizing formula: connections = ((core_count * 2) + effective_spindle_count)
      maximum-pool-size: 20
      minimum-idle: 10
      
      # Connection management
      connection-timeout: 20000      # 20 seconds
      idle-timeout: 300000           # 5 minutes
      max-lifetime: 1200000          # 20 minutes
      
      # Performance
      auto-commit: false
      
      # Leak detection (development only)
      leak-detection-threshold: 60000  # 60 seconds
      
      # Connection testing
      connection-test-query: SELECT 1
      validation-timeout: 3000
      
      # Monitoring
      register-mbeans: true
      
      # Pool name for monitoring
      pool-name: OrderServicePool
```

**Connection Pool Monitoring:**

```java
@Component
public class HikariMetrics {
    
    @Autowired
    private DataSource dataSource;
    
    @Autowired
    private MeterRegistry meterRegistry;
    
    @PostConstruct
    public void registerMetrics() {
        if (dataSource instanceof HikariDataSource) {
            HikariDataSource hikariDS = (HikariDataSource) dataSource;
            HikariPoolMXBean poolProxy = hikariDS.getHikariPoolMXBean();
            
            Gauge.builder("hikari.connections.active", poolProxy, 
                HikariPoolMXBean::getActiveConnections)
                .register(meterRegistry);
            
            Gauge.builder("hikari.connections.idle", poolProxy,
                HikariPoolMXBean::getIdleConnections)
                .register(meterRegistry);
            
            Gauge.builder("hikari.connections.total", poolProxy,
                HikariPoolMXBean::getTotalConnections)
                .register(meterRegistry);
            
            Gauge.builder("hikari.connections.pending", poolProxy,
                HikariPoolMXBean::getThreadsAwaitingConnection)
                .register(meterRegistry);
        }
    }
}
```

---

## Real-World Use Cases

### 1. E-Commerce Platform

**Architecture Overview:**

```
┌─────────────────┐
│   API Gateway   │
└────────┬────────┘
         │
    ┌────┴────┬──────────┬──────────┐
    │         │          │          │
┌───▼───┐ ┌──▼──┐  ┌────▼────┐ ┌──▼──────┐
│Product│ │Order│  │Payment  │ │Inventory│
│Service│ │Svc  │  │Service  │ │Service  │
└───┬───┘ └──┬──┘  └────┬────┘ └──┬──────┘
    │        │           │          │
    └────────┴───────────┴──────────┘
                    │
            ┌───────▼────────┐
            │  Event Bus     │
            │  (Kafka)       │
            └───────┬────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
    ┌───▼────┐  ┌──▼────┐  ┌──▼──────┐
    │Notif   │  │Analytics│ │Warehouse│
    │Service │  │Service  │ │Service  │
    └────────┘  └─────────┘ └─────────┘
```

**Key Patterns Used:**

| Component | Patterns | Reason |
|-----------|----------|--------|
| Product Catalog | CQRS, Cache-Aside | Read-heavy workload |
| Shopping Cart | Repository, Memento | State persistence, undo |
| Order Processing | Saga, Event Sourcing | Distributed transactions, audit |
| Payment | Strategy, Adapter | Multiple payment gateways |
| Inventory | Observer, Command | Stock updates, operations |
| Recommendations | Strategy, Decorator | Multiple algorithms, enhancements |

**Order Service Implementation:**

```java
@Service
@Transactional
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final ApplicationEventPublisher eventPublisher;
    
    // Saga pattern for distributed transaction
    public Order createOrder(CreateOrderRequest request) {
        // 1. Create order in PENDING state
        Order order = Order.builder()
            .customerId(request.getCustomerId())
            .items(request.getItems())
            .status(OrderStatus.PENDING)
            .build();
        
        order = orderRepository.save(order);
        
        try {
            // 2. Reserve inventory
            inventoryService.reserveItems(order.getId(), order.getItems());
            
            // 3. Process payment
            PaymentResult payment = paymentService.processPayment(
                order.getId(),
                order.getTotalAmount()
            );
            
            if (!payment.isSuccessful()) {
                // Compensating transaction
                inventoryService.releaseItems(order.getId());
                order.setStatus(OrderStatus.PAYMENT_FAILED);
                orderRepository.save(order);
                throw new PaymentFailedException("Payment failed");
            }
            
            // 4. Confirm order
            order.setStatus(OrderStatus.CONFIRMED);
            order.setPaymentId(payment.getPaymentId());
            order = orderRepository.save(order);
            
            // 5. Publish event
            eventPublisher.publishEvent(new OrderConfirmedEvent(order));
            
            return order;
            
        } catch (Exception e) {
            // Rollback: Release inventory
            try {
                inventoryService.releaseItems(order.getId());
            } catch (Exception rollbackEx) {
                log.error("Failed to rollback inventory", rollbackEx);
            }
            
            order.setStatus(OrderStatus.FAILED);
            orderRepository.save(order);
            throw e;
        }
    }
}
```

---

### 2. Banking/FinTech System

**Key Requirements:**
- ACID transactions
- Audit trail (compliance)
- High security
- Real-time processing
- 99.99% uptime

**Patterns & Implementation:**

```java
@Service
public class TransferService {
    
    @Transactional(
        isolation = Isolation.SERIALIZABLE,
        timeout = 30,
        rollbackFor = Exception.class
    )
    @AuditLog(action = "MONEY_TRANSFER")
    public TransferResult transferMoney(TransferRequest request) {
        // 1. Validate accounts
        Account fromAccount = accountRepository.findByIdForUpdate(
            request.getFromAccountId()
        );
        Account toAccount = accountRepository.findByIdForUpdate(
            request.getToAccountId()
        );
        
        validateTransfer(fromAccount, toAccount, request.getAmount());
        
        // 2. Check balance
        if (fromAccount.getBalance().compareTo(request.getAmount()) < 0) {
            throw new InsufficientFundsException("Insufficient balance");
        }
        
        // 3. Perform transfer
        fromAccount.debit(request.getAmount());
        toAccount.credit(request.getAmount());
        
        accountRepository.save(fromAccount);
        accountRepository.save(toAccount);
        
        // 4. Create transaction records
        Transaction debitTx = Transaction.builder()
            .accountId(fromAccount.getId())
            .type(TransactionType.DEBIT)
            .amount(request.getAmount())
            .balance(fromAccount.getBalance())
            .build();
        
        Transaction creditTx = Transaction.builder()
            .accountId(toAccount.getId())
            .type(TransactionType.CREDIT)
            .amount(request.getAmount())
            .balance(toAccount.getBalance())
            .build();
        
        transactionRepository.saveAll(List.of(debitTx, creditTx));
        
        // 5. Publish event for audit
        eventPublisher.publishEvent(new TransferCompletedEvent(
            fromAccount.getId(),
            toAccount.getId(),
            request.getAmount()
        ));
        
        return TransferResult.success(debitTx.getId());
    }
    
    private void validateTransfer(Account from, Account to, BigDecimal amount) {
        if (from.getStatus() != AccountStatus.ACTIVE) {
            throw new AccountInactiveException("Source account is not active");
        }
        if (to.getStatus() != AccountStatus.ACTIVE) {
            throw new AccountInactiveException("Destination account is not active");
        }
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException("Amount must be positive");
        }
        if (amount.compareTo(from.getDailyLimit()) > 0) {
            throw new LimitExceededException("Daily limit exceeded");
        }
    }
}

// Audit aspect
@Aspect
@Component
public class AuditAspect {
    
    @Around("@annotation(auditLog)")
    public Object audit(ProceedingJoinPoint joinPoint, AuditLog auditLog) throws Throwable {
        String username = SecurityContextHolder.getContext()
            .getAuthentication().getName();
        
        AuditEntry entry = AuditEntry.builder()
            .action(auditLog.action())
            .username(username)
            .timestamp(LocalDateTime.now())
            .ipAddress(getClientIp())
            .build();
        
        try {
            Object result = joinPoint.proceed();
            entry.setStatus("SUCCESS");
            return result;
        } catch (Exception e) {
            entry.setStatus("FAILED");
            entry.setErrorMessage(e.getMessage());
            throw e;
        } finally {
            auditRepository.save(entry);
        }
    }
}
```

---

### 3. Social Media Platform

**Key Requirements:**
- High read throughput
- Real-time feeds
- Scalable notifications
- Media storage
- Social graph

**Feed Service with Cache:**

```java
@Service
public class FeedService {
    
    private final PostRepository postRepository;
    private final FollowRepository followRepository;
    private final CacheManager cacheManager;
    
    // Fan-out on write pattern
    @Async
    public void publishPost(Post post) {
        // 1. Save post
        postRepository.save(post);
        
        // 2. Get followers
        List<Long> followers = followRepository.findFollowerIds(post.getAuthorId());
        
        // 3. Fan-out: Add to each follower's feed cache
        followers.forEach(followerId -> {
            String feedKey = "feed:" + followerId;
            Cache cache = cacheManager.getCache("userFeeds");
            
            List<Post> feed = cache.get(feedKey, List.class);
            if (feed != null) {
                feed.add(0, post); // Add to beginning
                if (feed.size() > 100) {
                    feed = feed.subList(0, 100); // Keep latest 100
                }
                cache.put(feedKey, feed);
            }
        });
        
        // 4. Publish event for notifications
        eventPublisher.publishEvent(new PostPublishedEvent(post));
    }
    
    // Fan-out on read pattern (for users with many followers)
    @Cacheable(value = "userFeeds", key = "#userId")
    public List<Post> getUserFeed(Long userId, Pageable pageable) {
        // Get list of people user follows
        List<Long> followingIds = followRepository.findFollowingIds(userId);
        
        // Fetch recent posts from followed users
        return postRepository.findByAuthorIdInOrderByCreatedAtDesc(
            followingIds,
            pageable
        );
    }
    
    // Hybrid approach for celebrities
    public List<Post> getHybridFeed(Long userId, Pageable pageable) {
        List<Long> followingIds = followRepository.findFollowingIds(userId);
        
        // Separate celebrities from regular users
        List<Long> celebrities = followingIds.stream()
            .filter(id -> followRepository.getFollowerCount(id) > 100000)
            .collect(Collectors.toList());
        
        List<Long> regularUsers = new ArrayList<>(followingIds);
        regularUsers.removeAll(celebrities);
        
        // Pre-computed feed for regular users
        List<Post> cachedPosts = getCachedFeedPosts(regularUsers);
        
        // Fetch celebrity posts on demand
        List<Post> celebrityPosts = postRepository
            .findRecentByAuthorIds(celebrities, pageable);
        
        // Merge and sort
        return mergePosts(cachedPosts, celebrityPosts, pageable);
    }
}
```

---

### 4. IoT Platform

**Key Requirements:**
- High-volume data ingestion
- Real-time processing
- Time-series data
- Device management
- Rule engine

**Device Data Ingestion:**

```java
@RestController
@RequestMapping("/api/devices")
public class DeviceDataController {
    
    private final KafkaTemplate<String, DeviceData> kafkaTemplate;
    private final DeviceMetricsService metricsService;
    
    @PostMapping("/{deviceId}/data")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void ingestData(
        @PathVariable String deviceId,
        @RequestBody DeviceDataRequest request
    ) {
        // Validate device
        if (!deviceService.isDeviceRegistered(deviceId)) {
            throw new DeviceNotFoundException("Device not registered");
        }
        
        // Create data point
        DeviceData data = DeviceData.builder()
            .deviceId(deviceId)
            .timestamp(Instant.now())
            .metrics(request.getMetrics())
            .build();
        
        // Send to Kafka for processing
        kafkaTemplate.send("device-data", deviceId, data);
        
        // Update metrics
        metricsService.recordDataPoint(deviceId);
    }
    
    @PostMapping("/batch")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void ingestBatch(@RequestBody List<DeviceDataRequest> batch) {
        // Batch ingestion for efficiency
        List<ProducerRecord<String, DeviceData>> records = batch.stream()
            .map(this::toProducerRecord)
            .collect(Collectors.toList());
        
        records.forEach(record -> 
            kafkaTemplate.send(record)
        );
    }
}

// Kafka Consumer for processing
@Service
public class DeviceDataProcessor {
    
    @KafkaListener(
        topics = "device-data",
        groupId = "device-processor",
        concurrency = "3"
    )
    public void processDeviceData(
        @Payload DeviceData data,
        @Header(KafkaHeaders.RECEIVED_KEY) String deviceId
    ) {
        try {
            // 1. Store in time-series database
            influxDbService.writeDataPoint(data);
            
            // 2. Apply rules
            List<Alert> alerts = ruleEngine.evaluate(data);
            if (!alerts.isEmpty()) {
                alertService.sendAlerts(alerts);
            }
            
            // 3. Update device state
            deviceStateService.updateState(deviceId, data);
            
            // 4. Check for anomalies
            if (anomalyDetector.isAnomalous(data)) {
                eventPublisher.publishEvent(
                    new AnomalyDetectedEvent(deviceId, data)
                );
            }
            
        } catch (Exception e) {
            log.error("Failed to process device data: {}", deviceId, e);
            // Send to DLQ
            kafkaTemplate.send("device-data-dlq", deviceId, data);
        }
    }
}

// Rule Engine
@Service
public class IoTRuleEngine {
    
    private final RuleRepository ruleRepository;
    
    public List<Alert> evaluate(DeviceData data) {
        List<Alert> alerts = new ArrayList<>();
        
        // Get rules for device type
        List<Rule> rules = ruleRepository.findByDeviceType(data.getDeviceType());
        
        for (Rule rule : rules) {
            if (evaluateCondition(rule.getCondition(), data)) {
                Alert alert = Alert.builder()
                    .deviceId(data.getDeviceId())
                    .ruleId(rule.getId())
                    .severity(rule.getSeverity())
                    .message(rule.getMessage())
                    .timestamp(Instant.now())
                    .build();
                
                alerts.add(alert);
            }
        }
        
        return alerts;
    }
    
    private boolean evaluateCondition(String condition, DeviceData data) {
        // Simple expression evaluation
        // In production, use Spring Expression Language (SpEL)
        // or a rules engine like Drools
        
        StandardEvaluationContext context = new StandardEvaluationContext(data);
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression(condition);
        
        return Boolean.TRUE.equals(exp.getValue(context, Boolean.class));
    }
}
```

---

### 5. Microservices Communication Patterns

**Synchronous Communication (REST):**

```java
@Service
public class OrderOrchestrationService {
    
    private final PaymentServiceClient paymentClient;
    private final InventoryServiceClient inventoryClient;
    private final ShippingServiceClient shippingClient;
    
    // Orchestration pattern
    public OrderResult processOrder(OrderRequest request) {
        try {
            // 1. Validate inventory
            InventoryResponse inventory = inventoryClient.checkAvailability(
                request.getItems()
            );
            
            if (!inventory.isAvailable()) {
                return OrderResult.failed("Items not available");
            }
            
            // 2. Process payment
            PaymentResponse payment = paymentClient.processPayment(
                PaymentRequest.builder()
                    .amount(request.getTotalAmount())
                    .customerId(request.getCustomerId())
                    .build()
            );
            
            if (!payment.isSuccessful()) {
                return OrderResult.failed("Payment failed");
            }
            
            // 3. Create shipment
            ShipmentResponse shipment = shippingClient.createShipment(
                ShipmentRequest.builder()
                    .orderId(request.getOrderId())
                    .address(request.getShippingAddress())
                    .build()
            );
            
            return OrderResult.success(shipment.getTrackingNumber());
            
        } catch (Exception e) {
            log.error("Order processing failed", e);
            // Compensate
            compensateOrder(request);
            return OrderResult.failed("Processing error");
        }
    }
}
```

**Asynchronous Communication (Event-Driven):**

```java
@Service
public class OrderEventHandler {
    
    @KafkaListener(topics = "order-created")
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Processing order created event: {}", event.getOrderId());
        
        // Each service processes independently
        inventoryService.reserveItems(event.getOrderId(), event.getItems());
    }
}

@Service
public class InventoryEventPublisher {
    
    private final KafkaTemplate<String, Object> kafkaTemplate;
    
    public void publishInventoryReserved(Long orderId, List<OrderItem> items) {
        InventoryReservedEvent event = InventoryReservedEvent.builder()
            .orderId(orderId)
            .items(items)
            .timestamp(Instant.now())
            .build();
        
        kafkaTemplate.send("inventory-reserved", orderId.toString(), event);
    }
}

// Saga Orchestrator
@Service
public class OrderSagaOrchestrator {
    
    @KafkaListener(topics = "order-created")
    public void startOrderSaga(OrderCreatedEvent event) {
        SagaInstance saga = sagaRepository.create(event.getOrderId());
        
        try {
            // Step 1: Reserve inventory
            saga.addStep("INVENTORY_RESERVE");
            kafkaTemplate.send("inventory-reserve-command", event);
            
        } catch (Exception e) {
            saga.fail();
            compensateSaga(saga);
        }
    }
    
    @KafkaListener(topics = "inventory-reserved")
    public void continueOrderSaga(InventoryReservedEvent event) {
        SagaInstance saga = sagaRepository.findByOrderId(event.getOrderId());
        
        try {
            // Step 2: Process payment
            saga.addStep("PAYMENT_PROCESS");
            kafkaTemplate.send("payment-process-command", event);
            
        } catch (Exception e) {
            saga.fail();
            compensateSaga(saga);
        }
    }
    
    private void compensateSaga(SagaInstance saga) {
        // Execute compensation in reverse order
        for (String step : saga.getCompletedSteps().reversed()) {
            switch (step) {
                case "INVENTORY_RESERVE":
                    kafkaTemplate.send("inventory-release-command", saga.getOrderId());
                    break;
                case "PAYMENT_PROCESS":
                    kafkaTemplate.send("payment-refund-command", saga.getOrderId());
                    break;
            }
        }
    }
}
```

---

## Spring Boot Best Practices Summary

### 1. Project Structure

```
src/main/java/com/example/
├── config/              # Configuration classes
├── controller/          # REST controllers
├── service/            # Business logic
├── repository/         # Data access
├── entity/             # JPA entities
├── dto/                # Data transfer objects
├── exception/          # Custom exceptions
├── security/           # Security configuration
├── util/               # Utility classes
└── Application.java    # Main class

src/main/resources/
├── application.yml
├── application-dev.yml
├── application-prod.yml
└── db/migration/       # Flyway migrations
```

### 2. Configuration Best Practices

| Practice | Example | Benefit |
|----------|---------|---------|
| Externalize config | Use environment variables | 12-factor app |
| Profile-specific | application-{profile}.yml | Environment separation |
| Type-safe config | @ConfigurationProperties | Validation, IDE support |
| Secrets management | Vault, AWS Secrets Manager | Security |
| Default values | @Value("${key:default}") | Fail-safe |

### 3. API Design Best Practices

| Practice | Implementation | Benefit |
|----------|---------------|---------|
| Versioning | /api/v1/users | Backward compatibility |
| Pagination | Page/Slice return types | Performance |
| Filtering | @RequestParam for filters | Flexibility |
| Sorting | Pageable with Sort | User control |
| HATEOAS | Links in responses | Discoverability |
| Error handling | @RestControllerAdvice | Consistency |
| Documentation | Swagger/OpenAPI | API documentation |

### 4. Security Best Practices

| Practice | Implementation | Protection Against |
|----------|---------------|-------------------|
| Authentication | JWT, OAuth2 | Unauthorized access |
| Authorization | Method security | Privilege escalation |
| Input validation | @Valid, @Validated | Injection attacks |
| CSRF protection | CsrfTokenRepository | Cross-site attacks |
| CORS configuration | CorsConfigurationSource | Unauthorized origins |
| SQL injection | Parameterized queries | SQL injection |
| Password encoding | BCryptPasswordEncoder | Weak passwords |
| Rate limiting | @RateLimiter | DoS attacks |

### 5. Performance Best Practices

| Practice | Implementation | Impact |
|----------|---------------|--------|
| Query optimization | @EntityGraph, fetch joins | Reduce N+1 queries |
| Pagination | Pageable, Slice | Memory efficiency |
| Caching | @Cacheable, Redis | Reduce DB load |
| Async processing | @Async, CompletableFuture | Improve throughput |
| Connection pooling | HikariCP tuning | Database performance |
| Lazy loading | FetchType.LAZY | Reduce data transfer |
| Projection | DTO projections | Reduce payload size |
| Batch operations | saveAll(), batch size | Reduce round trips |

### 6. Testing Best Practices

| Test Type | Tools | Coverage |
|-----------|-------|----------|
| Unit tests | JUnit, Mockito | 80%+ |
| Integration tests | @SpringBootTest, TestContainers | Critical paths |
| API tests | MockMvc, RestAssured | All endpoints |
| Security tests | @WithMockUser | Auth/authz |
| Performance tests | JMeter, Gatling | Load scenarios |
| Contract tests | Pact | API contracts |

### 7. Monitoring & Observability

| Aspect | Tool/Method | Purpose |
|--------|------------|---------|
| Health checks | Actuator /health | Liveness/readiness |
| Metrics | Micrometer, Prometheus | Performance monitoring |
| Logging | Logback, ELK Stack | Debugging |
| Tracing | Sleuth, Zipkin | Request flow |
| APM | New Relic, Datadog | Application performance |
| Alerts | PagerDuty, OpsGenie | Incident management |

---

## Common Pitfalls & Solutions

### 1. N+1 Query Problem

**Problem:**
```java
// Fetches 1 query for orders + N queries for items
List<Order> orders = orderRepository.findAll();
orders.forEach(order -> 
    order.getItems().size() // Triggers query for each order
);
```

**Solution:**
```java
// Single query with JOIN FETCH
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items")
List<Order> findAllWithItems();
```

### 2. Transaction Boundaries

**Problem:**
```java
@Transactional
public void processOrder(Order order) {
    // Long-running external API call inside transaction
    paymentGateway.processPayment(order); // Holds DB connection
}
```

**Solution:**
```java
public void processOrder(Order order) {
    // Process payment outside transaction
    PaymentResult result = paymentGateway.processPayment(order);
    
    // Short transaction for DB update
    updateOrderStatus(order, result);
}

@Transactional
private void updateOrderStatus(Order order, PaymentResult result) {
    order.setStatus(result.getStatus());
    orderRepository.save(order);
}
```

### 3. Exception Handling in Transactions

**Problem:**
```java
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);
    // Checked exception doesn't rollback by default
    throw new IOException("Error");
}
```

**Solution:**
```java
@Transactional(rollbackFor = Exception.class)
public void createOrder(Order order) throws Exception {
    orderRepository.save(order);
    throw new IOException("Error"); // Will rollback
}
```

### 4. Lazy Loading Outside Transaction

**Problem:**
```java
public Order getOrder(Long id) {
    return orderRepository.findById(id).get();
    // Items accessed outside transaction
}

// In controller
order.getItems().size(); // LazyInitializationException
```

**Solution:**
```java
@Transactional(readOnly = true)
public Order getOrderWithItems(Long id) {
    Order order = orderRepository.findById(id).get();
    // Force initialization within transaction
    order.getItems().size();
    return order;
}

// Or use fetch join
@Query("SELECT o FROM Order o LEFT JOIN FETCH o.items WHERE o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);
```

### 5. Improper Use of @Autowired

**Problem:**
```java
@Service
public class OrderService {
    @Autowired
    private OrderRepository repository; // Field injection
}
```

**Solution:**
```java
@Service
public class OrderService {
    private final OrderRepository repository;
    
    // Constructor injection (recommended)
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

---

## Quick Reference Tables

### HTTP Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 422 | Unprocessable | Business logic error |
| 500 | Internal Error | Server error |
| 503 | Service Unavailable | Service down |

### JPA Fetch Types

| Type | When Loaded | Use Case | N+1 Risk |
|------|------------|----------|----------|
| EAGER | Immediately | Small, always needed | High |
| LAZY | On access | Large, rarely needed | Low |

### Transaction Isolation Levels

| Level | Dirty Read | Non-Repeatable | Phantom | Performance |
|-------|-----------|---------------|---------|-------------|
| READ_UNCOMMITTED | Yes | Yes | Yes | Highest |
| READ_COMMITTED | No | Yes | Yes | High |
| REPEATABLE_READ | No | No | Yes | Medium |
| SERIALIZABLE | No | No | No | Lowest |

### Cache Strategies

| Strategy | Pattern | Use Case |
|----------|---------|----------|
| Cache-Aside | Load on miss | Read-heavy |
| Write-Through | Write to cache & DB | Write-heavy |
| Write-Behind | Async DB write | High write volume |
| Refresh-Ahead | Proactive refresh | Predictable access |