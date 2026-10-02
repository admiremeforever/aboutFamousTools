# Spring Boot SDE2 Interview Notes

## 1. SpringApplication.run() — Application Startup

### Interview Definition

`SpringApplication.run()` Spring Boot application ka startup process initiate karta hai.

### Startup Flow

```text
main()
  ↓
SpringApplication.run()
  ↓
Prepare Environment
  ↓
Create ApplicationContext
  ↓
Load Bean Definitions
  ↓
Refresh ApplicationContext
  ↓
Create & Initialize Beans
  ↓
Start Embedded Server
  ↓
ApplicationReadyEvent
```

### Important Points

* `Environment` properties load karta hai:

  * application.properties
  * application.yml
  * environment variables
  * system properties
  * command-line arguments
* Web application ke liye ApplicationContext create hota hai.
* Component scanning se bean definitions register hoti hain.
* `context.refresh()` bean creation/lifecycle ka important phase hai.
* Embedded Tomcat/Jetty/Undertow start hota hai.
* Startup complete hone par `ApplicationReadyEvent` publish hota hai.

### Interview Follow-up

**Q: Application startup complete hone ka signal?**

`ApplicationReadyEvent`

---

# 2. @SpringBootApplication

`@SpringBootApplication` mainly 3 annotations ka combination hai:

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

### @SpringBootConfiguration

Application ki main configuration class identify karta hai.

### @ComponentScan

Components scan karta hai:

```text
@Component
@Service
@Repository
@Controller
@Configuration
```

### @EnableAutoConfiguration

Classpath aur configuration ke basis par required beans automatically configure karta hai.

### Interview Line

> "`@SpringBootApplication` is a convenience annotation combining configuration, component scanning and auto-configuration."

---

# 3. Spring Boot Auto-Configuration

### Definition

Auto-configuration classpath aur application configuration ke basis par Spring beans automatically configure karti hai.

Example:

```text
spring-boot-starter-data-jpa
          ↓
JPA/Hibernate detected
          ↓
Relevant auto-configuration
          ↓
Required beans configured
```

### Conditional Configuration

Common annotations:

```java
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnProperty
@ConditionalOnBean
```

Example:

```java
@Bean
@ConditionalOnMissingBean
DataSource dataSource() {
    ...
}
```

Meaning:

> Agar user ne already DataSource define nahi kiya hai, tab default DataSource create karo.

### Important

Auto-configuration blindly beans create nahi karti.

It uses conditions.

Modern Spring Boot auto-configuration discovery:

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

---

# 4. Spring Bean Lifecycle

### Complete Flow

```text
Bean Definition
      ↓
Bean Instantiation
      ↓
Dependency Injection
      ↓
Aware Callbacks
      ↓
BeanPostProcessor - Before
      ↓
@PostConstruct
      ↓
InitializingBean
      ↓
Custom init-method
      ↓
BeanPostProcessor - After
      ↓
Bean Ready
      ↓
Application Uses Bean
      ↓
@PreDestroy
      ↓
DisposableBean
      ↓
Custom destroy-method
```

### Important Annotations

```java
@PostConstruct
```

Initialization ke baad execute hota hai.

```java
@PreDestroy
```

Bean destroy hone se pehle execute hota hai.

### SDE2 Point

`BeanPostProcessor` Spring ke internal mechanisms mein heavily used hota hai.

Examples:

```text
@Autowired
AOP
@Transactional
@Configuration
```

---

# 5. Constructor Injection vs Field Injection

## Field Injection

```java
@Autowired
private PaymentService paymentService;
```

## Constructor Injection

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### Constructor Injection Preferred

Reasons:

1. Dependencies explicit hoti hain.
2. Required dependency missing ho to object creation fail hota hai.
3. Object immutable banaya ja sakta hai.
4. Unit testing easy hoti hai.
5. Dependencies constructor mein clearly visible hoti hain.
6. Circular dependencies identify karna easier hota hai.

### Interview Line

> "I prefer constructor injection because dependencies are explicit, required dependencies are enforced at construction time, and the class becomes easier to test."

---

# 6. BeanPostProcessor

`BeanPostProcessor` bean lifecycle mein hooks provide karta hai.

Main methods:

```java
postProcessBeforeInitialization()
postProcessAfterInitialization()
```

### Flow

```text
Create Bean
   ↓
Dependency Injection
   ↓
Before Initialization
   ↓
@PostConstruct
   ↓
InitializingBean
   ↓
After Initialization
   ↓
Bean Ready
```

### Why Important?

Spring infrastructure internally post-processors ka use karta hai for things like:

```text
Dependency Injection
AOP
@Transactional
@Configuration
```

### Interview Point

Spring bean ko directly return karne ke bajay post-processing ke through modify/wrap/proxy kar sakta hai.

---

# 7. Spring MVC Request Flow

Suppose:

```http
GET /users/10
```

### Flow

```text
Client
  ↓
Tomcat
  ↓
Servlet Filters
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
HandlerAdapter
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
  ↓
Response
```

### DispatcherServlet

Spring MVC ka Front Controller hai.

### HandlerMapping

Determine karta hai ki request kis controller method ko jayegi.

Example:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id)
```

### HandlerAdapter

Resolved controller method ko invoke karne mein help karta hai.

### HttpMessageConverter

Java object ko JSON mein convert kar sakta hai.

Example:

```java
User
```

becomes:

```json
{
  "id": 10,
  "name": "Manish"
}
```

Jackson commonly JSON serialization/deserialization handle karta hai.

---

# 8. @Transactional Internals

Example:

```java
@Transactional
public void transferMoney() {
    debit();
    credit();
}
```

### Simplified Flow

```text
Client
  ↓
Spring Proxy
  ↓
Start Transaction
  ↓
Actual Method
  ↓
DB Operations
  ↓
Commit
```

Exception:

```text
Exception
   ↓
Rollback
```

### Important Concept

Spring commonly proxy-based transaction management use karta hai.

Therefore:

```java
this.transferMoney();
```

jaisi self-invocation mein proxy bypass ho sakta hai.

Result:

```text
@Transactional
     ↓
Advice may not execute
```

### Default Propagation

```text
REQUIRED
```

Meaning:

```text
Existing transaction?
    YES → Join it
    NO  → Create new transaction
```

### Interview Follow-ups

Know these:

```text
REQUIRED
REQUIRES_NEW
NESTED
MANDATORY
SUPPORTS
NOT_SUPPORTED
NEVER
```

---

# 9. Persistence Context + Dirty Checking

## Persistence Context

JPA EntityManager managed entities ka context maintain karta hai.

Example:

```java
@Transactional
public void updateUser(Long id) {

    User user = entityManager.find(User.class, id);

    user.setName("Manish");
}
```

Explicit UPDATE query nahi likhi.

But transaction flush/commit ke time Hibernate change detect karke SQL generate kar sakta hai.

### Flow

```text
find()
  ↓
Entity Loaded
  ↓
Persistence Context
  ↓
Entity Modified
  ↓
Dirty Checking
  ↓
UPDATE SQL
  ↓
Commit
```

### Dirty Checking

> Hibernate managed entity ke changes detect karke required SQL generate karta hai.

### Important

Dirty checking primarily **managed entities** par apply hoti hai.

---

# 10. N+1 Query Problem

Suppose:

```java
List<Order> orders = orderRepository.findAll();

for (Order order : orders) {
    order.getCustomer().getName();
}
```

Suppose 10 orders hain.

Queries:

```text
1 query  → Orders

10 queries → Customers

Total = 11 queries
```

This is:

```text
N + 1 Query Problem
```

### Solutions

## 1. JOIN FETCH

```jpql
SELECT o
FROM Order o
JOIN FETCH o.customer
```

## 2. EntityGraph

```java
@EntityGraph(attributePaths = "customer")
List<Order> findAll();
```

## 3. Batch Fetching

Hibernate batch fetching use kar sakta hai.

### Interview Answer

> "I would first identify the N+1 through SQL logs or APM, then choose JOIN FETCH, EntityGraph, or batching depending on the access pattern."

---

# 11. Spring Security Filter Chain

### Basic Flow

```text
Client
  ↓
Security Filter Chain
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
```

### JWT Flow

```text
Request
  ↓
JWT Filter
  ↓
Extract Authorization Header
  ↓
Validate JWT
  ↓
Create Authentication
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
```

Example:

```http
Authorization: Bearer <token>
```

### Authentication vs Authorization

Authentication:

> Tum kaun ho?

Authorization:

> Tum kya access kar sakte ho?

Example:

```text
Authentication → Manish
Authorization  → ADMIN APIs allowed
```

### SecurityContext

Authenticated user's security information ko current execution context mein hold karta hai.

---

# 12. HikariCP / Database Connection Pool

Without connection pooling:

```text
Request
  ↓
Create DB Connection
  ↓
Execute Query
  ↓
Close Connection
```

Every request ke liye connection create karna expensive hai.

With HikariCP:

```text
Application
     ↓
HikariCP
     ↓
Connection Pool
     ↓
Database
```

Request:

```text
Get Connection
     ↓
Execute Query
     ↓
Return Connection to Pool
```

### Example

```text
maximumPoolSize = 20
```

100 concurrent requests DB connection maangti hain.

```text
20 → Connections
80 → Wait
```

Agar queries slow hain:

```text
DB Slow
   ↓
Connections occupied
   ↓
Pool Exhaustion
   ↓
Requests wait
   ↓
Timeout
```

### Production Debugging

Check:

```text
Active connections
Idle connections
Pending threads
Maximum pool size
Connection timeout
Query latency
Database health
```

---

# 13. Thread Pool + Request Handling

Simplified flow:

```text
Incoming Requests
       ↓
Tomcat Thread Pool
       ↓
Controller
       ↓
Service
       ↓
DB / External Service
```

Suppose:

```text
max threads = 200
```

Aur 1000 concurrent requests aa gayi.

Approximately 200 worker threads simultaneously work kar sakti hain; remaining requests queue/wait kar sakti hain depending on server configuration.

### Problem

Suppose each request DB call ke liye 10 seconds wait kar rahi hai.

```text
Slow DB
   ↓
Threads occupied/waiting
   ↓
Thread pool exhaustion
   ↓
Latency increases
   ↓
Timeouts
```

### Don't blindly increase threads

Investigate:

```text
DB latency
External API latency
Connection pool
CPU
GC
Thread dumps
Lock contention
```

Possible solutions:

```text
Optimize DB
Add proper timeout
Circuit breaker
Bulkhead
Caching
Async/non-blocking I/O where appropriate
Connection-pool tuning
```

---

# 14. Resilience4j / Circuit Breaker

Suppose:

```text
Service A
    ↓
Service B
```

Service B down/slow hai.

Without protection:

```text
A → B
A → B
A → B
A → B
...
```

Eventually Service A ke resources bhi exhaust ho sakte hain.

### Circuit Breaker States

```text
CLOSED
   ↓
Failure Threshold
   ↓
OPEN
   ↓
Wait Duration
   ↓
HALF_OPEN
   ↓
Test Requests
   ↓
CLOSED / OPEN
```

## CLOSED

Normal requests pass.

## OPEN

Requests downstream service ko call nahi karti.

Fast failure/fallback possible.

## HALF_OPEN

Limited requests test karti hain whether dependency recover hui hai.

### Common Resilience Mechanisms

```text
Timeout
Retry
Circuit Breaker
Bulkhead
Rate Limiting
```

### Important

Retry blindly nahi lagana.

Example:

```text
POST /payment
```

ko blindly retry karna duplicate payment create kar sakta hai.

Retry + idempotency ko together consider karna chahiye.

---

# 15. Production API Suddenly Slow — Debugging Approach

Question:

> API latency suddenly 200 ms se 5 sec ho gayi. What will you do?

## Step 1 — Establish Scope

Check:

```text
Which API?
All APIs or one?
One instance or all?
When did issue start?
```

## Step 2 — Application Metrics

Check:

```text
CPU
Memory
GC
Request latency
Throughput
Error rate
Thread count
```

## Step 3 — Database

Check:

```text
DB CPU
Slow queries
Query execution plan
Locks
Deadlocks
Connection pool
Connection wait time
```

## Step 4 — External Dependencies

Check:

```text
Other microservices
Redis
Kafka
External APIs
Network
```

## Step 5 — Thread Dump

If threads are blocked/waiting:

```text
jstack
```

Look for:

```text
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
```

Also investigate lock contention.

## Step 6 — GC

Check:

```text
GC frequency
GC pause time
Heap usage
Allocation rate
```

High GC pressure can increase latency.

## Step 7 — Distributed Tracing

Example:

```text
API
 ↓ 50ms
Service A
 ↓ 100ms
Service B
 ↓ 4.5 sec
Database
```

Now bottleneck clear hai.

### Strong SDE2 Answer

> "I would first determine whether the latency increase is application-wide or isolated to a particular endpoint or dependency. Then I would correlate latency with CPU, GC, thread-pool, database connection-pool and downstream-service metrics. I would use logs and distributed tracing to identify the slow segment, and thread dumps if threads appear blocked. I would avoid blindly increasing thread pools before identifying the bottleneck."

---

# 🔥 MASTER REVISION FLOW

Spring Boot ko is complete flow se yaad rakho:

```text
                    CLIENT
                       │
                       ▼
                    TOMCAT
                       │
                       ▼
                    FILTERS
                       │
                       ▼
               DISPATCHERSERVLET
                       │
                       ▼
                  CONTROLLER
                       │
                       ▼
                   SERVICE
                       │
              ┌────────┴────────┐
              ▼                 ▼
            REDIS              KAFKA
              │
              ▼
          REPOSITORY
              │
              ▼
        HIKARICP / JPA
              │
              ▼
           DATABASE
```

Cross-cutting concerns:

```text
Security
Transactions
AOP
Logging
Metrics
Tracing
Circuit Breaker
```

## SDE2 Interview Priority

```text
1. SpringApplication.run()
2. @SpringBootApplication
3. Auto-Configuration
4. Bean Lifecycle
5. Dependency Injection
6. BeanPostProcessor
7. DispatcherServlet Request Flow
8. @Transactional
9. Persistence Context + Dirty Checking
10. N+1 Query
11. Security Filter Chain
12. HikariCP
13. Thread Pool
14. Circuit Breaker / Resilience4j
15. Production Debugging
```

## Golden Rule

SDE2 interview mein har topic par ye 4 questions mentally prepare rakho:

```text
WHAT?
  ↓
HOW?
  ↓
WHY?
  ↓
WHAT IF IT FAILS IN PRODUCTION?
```

Example:

```text
@Transactional
   ↓
What? → Transaction management
How?  → Proxy/AOP
Why?  → Atomicity/consistency
What if? → Rollback, propagation, timeout, deadlock
```
