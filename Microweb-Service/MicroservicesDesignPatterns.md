# Microservices Design Patterns & Principles

**Comprehensive guide to microservices architecture patterns, principles, and implementation strategies**

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [Decomposition Patterns](#decomposition-patterns)
3. [Integration Patterns](#integration-patterns)
4. [Data Management Patterns](#data-management-patterns)
5. [Communication Patterns](#communication-patterns)
6. [Resilience Patterns](#resilience-patterns)
7. [Observability Patterns](#observability-patterns)
8. [Deployment Patterns](#deployment-patterns)
9. [Security Patterns](#security-patterns)
10. [Best Practices & Anti-Patterns](#best-practices--anti-patterns)

---

## Core Principles

### 1. Single Responsibility Principle (SRP)

**Definition:** Each microservice should have one and only one reason to change, focusing on a single business capability.

**Why It Matters:**
- Easier to understand and maintain
- Reduces coupling between services
- Enables independent scaling
- Simplifies testing and deployment

**Example:**
```
❌ Bad: OrderService handles orders, payments, inventory, and shipping
✅ Good: 
   - OrderService: Order management only
   - PaymentService: Payment processing only
   - InventoryService: Stock management only
   - ShippingService: Delivery coordination only
```

**Implementation Guidelines:**
- Define clear bounded contexts
- One database per service
- Service should be replaceable
- Team ownership aligned with service

---

### 2. Autonomy

**Definition:** Services should be autonomous and independently deployable without requiring coordination with other services.

**Key Aspects:**
- **Independent Deployment** - Deploy without affecting other services
- **Independent Scaling** - Scale based on own requirements
- **Independent Technology Stack** - Choose best tools for the job
- **Independent Data Store** - Own database schema

**Example Architecture:**
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  User Service   │     │  Order Service  │     │ Payment Service │
│  Node.js        │     │  Java/Spring    │     │  Python/Flask   │
│  MongoDB        │     │  PostgreSQL     │     │  MySQL          │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

### 3. Decentralized Data Management

**Definition:** Each service manages its own database. No shared databases between services.

**Benefits:**
- Service independence
- Technology diversity (polyglot persistence)
- Easier scaling
- Reduced coupling

**Challenges:**
- Data consistency (eventual consistency)
- Distributed transactions
- Data duplication

**Pattern: Database per Service**
```
Service A → Database A
Service B → Database B
Service C → Database C

Communication via APIs/Events, NOT direct database access
```

---

### 4. Design for Failure

**Definition:** Assume that services and infrastructure will fail, and design systems to handle failures gracefully.

**Key Patterns:**
- Circuit Breaker
- Retry with exponential backoff
- Timeout
- Bulkhead
- Fallback

**Example Failure Scenarios:**
- Service unavailable (503)
- Network timeout
- Database connection failure
- Downstream service slow response
- Partial system failure

---

### 5. API First Design

**Definition:** Design APIs before implementation, treating them as contracts between services.

**Best Practices:**
- Use OpenAPI/Swagger specifications
- Version APIs from the start
- Design for backward compatibility
- Document thoroughly
- Use standard HTTP methods and status codes

**API Versioning Strategies:**
```
1. URI Versioning: /api/v1/users, /api/v2/users
2. Header Versioning: Accept: application/vnd.company.v1+json
3. Query Parameter: /api/users?version=1
```

---

### 6. Domain-Driven Design (DDD)

**Definition:** Align service boundaries with business domains and bounded contexts.

**Key Concepts:**

**Bounded Context:**
- Clear boundary within which a domain model applies
- Each microservice represents one bounded context

**Ubiquitous Language:**
- Common language between developers and domain experts
- Used in code, documentation, and communication

**Aggregates:**
- Cluster of domain objects treated as a single unit
- Root entity controls access to aggregate

**Example - E-commerce System:**
```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Order Context   │  │ Catalog Context  │  │ Customer Context │
│  - Order         │  │  - Product       │  │  - Customer      │
│  - OrderItem     │  │  - Category      │  │  - Address       │
│  - OrderStatus   │  │  - Inventory     │  │  - Preferences   │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

### 7. Observability

**Definition:** System's ability to provide insights into its internal state through logs, metrics, and traces.

**Three Pillars:**

**1. Logging**
- Structured logging (JSON format)
- Correlation IDs across services
- Centralized log aggregation

**2. Metrics**
- RED metrics (Rate, Errors, Duration)
- Business metrics
- Infrastructure metrics

**3. Distributed Tracing**
- Track requests across services
- Identify bottlenecks
- Understand dependencies

**Tools:**
- Logging: ELK Stack, Splunk, Fluentd
- Metrics: Prometheus, Grafana
- Tracing: Jaeger, Zipkin, OpenTelemetry

---

## Decomposition Patterns

### 1. Decompose by Business Capability

**Pattern:** Identify business capabilities and create services around them.

**Steps:**
1. Identify business capabilities (what the business does)
2. Define services for each capability
3. Ensure loose coupling

**Example - E-commerce:**
```
Business Capabilities → Services

Product Catalog Management → Catalog Service
Order Management → Order Service
Customer Management → Customer Service
Payment Processing → Payment Service
Inventory Management → Inventory Service
Shipping & Delivery → Shipping Service
```

**Benefits:**
- Stable architecture (business capabilities change slowly)
- Clear service boundaries
- Team alignment with business

---

### 2. Decompose by Subdomain (DDD)

**Pattern:** Use Domain-Driven Design to identify subdomains and create services for each.

**Types of Subdomains:**

**Core Domain:**
- Key differentiator for business
- Most valuable
- Example: Recommendation engine for Netflix

**Supporting Domain:**
- Supports core domain
- Example: User management, notifications

**Generic Domain:**
- Common functionality
- Can use off-the-shelf solutions
- Example: Payment gateway, email service

**Example - Banking:**
```
Core Subdomains:
- Loan Processing Service
- Risk Assessment Service

Supporting Subdomains:
- Account Management Service
- Transaction Service

Generic Subdomains:
- Authentication Service
- Notification Service
```

---

### 3. Strangler Fig Pattern

**Pattern:** Gradually migrate from monolith to microservices by incrementally replacing functionality.

**Steps:**
1. Identify functionality to extract
2. Create new microservice
3. Route traffic to new service
4. Deprecate old functionality
5. Repeat

**Implementation:**
```
┌─────────────────────────────────────┐
│         API Gateway/Proxy           │
│  (Routes traffic based on rules)    │
└──────────┬──────────────────┬───────┘
           │                  │
           ▼                  ▼
    ┌─────────────┐    ┌─────────────┐
    │  Monolith   │    │ New Service │
    │  - Feature A│    │  - Feature B│
    │  - Feature C│    │             │
    └─────────────┘    └─────────────┘

Gradually migrate features from Monolith → New Services
```

**Benefits:**
- Low risk incremental migration
- No big-bang rewrite
- Continuous delivery during migration

---

## Integration Patterns

### 1. API Gateway Pattern

**Definition:** Single entry point for all client requests, routing to appropriate microservices.

**Responsibilities:**
- Request routing
- Authentication & authorization
- Rate limiting
- Request/response transformation
- Protocol translation (REST to gRPC)
- API composition

**Architecture:**
```
┌──────────────┐
│   Clients    │
│ (Web, Mobile)│
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────┐
│        API Gateway               │
│  - Authentication                │
│  - Rate Limiting                 │
│  - Request Routing               │
│  - Response Aggregation          │
└──────┬───────────────────────────┘
       │
   ┌───┴────┬─────────┬─────────┐
   ▼        ▼         ▼         ▼
┌────┐  ┌────┐   ┌────┐   ┌────┐
│Svc1│  │Svc2│   │Svc3│   │Svc4│
└────┘  └────┘   └────┘   └────┘
```

**Implementation (Spring Cloud Gateway):**
```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("order_service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .stripPrefix(2)
                    .addRequestHeader("X-Gateway", "API-Gateway")
                    .circuitBreaker(config -> config
                        .setName("orderServiceCB")
                        .setFallbackUri("forward:/fallback/orders")))
                .uri("lb://ORDER-SERVICE"))
            
            .route("user_service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .stripPrefix(2)
                    .rateLimit(config -> config
                        .setRateLimiter(redisRateLimiter())))
                .uri("lb://USER-SERVICE"))
            
            .build();
    }
}
```

**Popular API Gateways:**
- Kong
- Netflix Zuul
- Spring Cloud Gateway
- AWS API Gateway
- Azure API Management

---

### 2. Backend for Frontend (BFF)

**Definition:** Create separate backend services for each client type (web, mobile, IoT).

**Why BFF?**
- Different clients have different needs
- Avoid bloated generic APIs
- Optimize for specific client requirements
- Independent evolution of client experiences

**Architecture:**
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│   Web    │  │  Mobile  │  │   IoT    │
│  Client  │  │  Client  │  │  Client  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     ▼             ▼             ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Web BFF │  │Mobile BFF│ │ IoT BFF │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────┬───┴────────────┘
              ▼
    ┌──────────────────────┐
    │  Microservices Layer │
    └──────────────────────┘
```

**Example:**
```java
// Mobile BFF - Optimized for mobile bandwidth
@RestController
@RequestMapping("/mobile/api")
public class MobileBFFController {
    
    @GetMapping("/dashboard")
    public MobileDashboard getDashboard(@AuthenticationPrincipal User user) {
        return MobileDashboard.builder()
            .userSummary(getUserSummary(user))
            .recentOrders(getRecentOrders(user, 5)) // Only 5 orders
            .recommendations(getRecommendations(user, 3))
            .build();
    }
}

// Web BFF - Rich data for web interface
@RestController
@RequestMapping("/web/api")
public class WebBFFController {
    
    @GetMapping("/dashboard")
    public WebDashboard getDashboard(@AuthenticationPrincipal User user) {
        return WebDashboard.builder()
            .userProfile(getFullUserProfile(user))
            .recentOrders(getRecentOrders(user, 20)) // More orders
            .orderHistory(getOrderHistory(user))
            .recommendations(getRecommendations(user, 10))
            .analytics(getUserAnalytics(user))
            .build();
    }
}
```

---

### 3. Service Registry & Discovery

**Definition:** Mechanism for services to register themselves and discover other services dynamically.

**Why Needed?**
- Dynamic service instances (scaling up/down)
- IP addresses change in cloud environments
- Load balancing across instances
- Health checking

**Patterns:**

**Client-Side Discovery:**
```
1. Service registers with registry
2. Client queries registry
3. Client selects instance
4. Client makes direct request
```

**Server-Side Discovery:**
```
1. Client makes request to load balancer
2. Load balancer queries registry
3. Load balancer routes to instance
```

**Implementation (Eureka):**
```java
// Service Registration
@SpringBootApplication
@EnableEurekaClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

// application.yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    preferIpAddress: true
    leaseRenewalIntervalInSeconds: 10

// Service Discovery with Load Balanced RestTemplate
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public User getUserDetails(String userId) {
        // Automatically discovers and load balances
        return restTemplate.getForObject(
            "http://USER-SERVICE/users/" + userId, 
            User.class
        );
    }
}
```

**Popular Service Registries:**
- Netflix Eureka
- Consul
- Zookeeper
- etcd
- Kubernetes Service Discovery

---

## Data Management Patterns

### 1. Database per Service

**Definition:** Each microservice has its own private database that cannot be accessed directly by other services.

**Benefits:**
- Service independence
- Technology diversity (polyglot persistence)
- Easier scaling
- Loose coupling

**Challenges:**
- Data consistency
- Distributed transactions
- Data duplication
- Querying across services

**Architecture:**
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Order Service  │     │  User Service   │     │Payment Service  │
│  ┌───────────┐  │     │  ┌───────────┐  │     │  ┌───────────┐  │
│  │PostgreSQL │  │     │  │  MongoDB  │  │     │  │   MySQL   │  │
│  └───────────┘  │     │  └───────────┘  │     │  └───────────┘  │
└─────────────────┘     └─────────────────┘     └─────────────────┘

❌ No direct database access between services
✅ Communication only via APIs or events
```

**Implementation:**
```java
// Order Service - Owns order data
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private String orderId;
    private String userId;  // Reference, not foreign key
    private BigDecimal totalAmount;
    private OrderStatus status;
}

// Order Service needs user data - Call User Service API
@Service
public class OrderService {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    public OrderDetails getOrderDetails(String orderId) {
        Order order = orderRepository.findById(orderId);
        User user = userServiceClient.getUser(order.getUserId());
        
        return OrderDetails.builder()
            .order(order)
            .user(user)
            .build();
    }
}
```

---

### 2. Saga Pattern

**Definition:** Manage distributed transactions across multiple services using a sequence of local transactions.

**Problem:**
- Traditional ACID transactions don't work across microservices
- Need to maintain data consistency across services

**Solution:**
- Break transaction into multiple local transactions
- Each service commits its own transaction
- If one fails, execute compensating transactions to rollback

**Types:**

**Choreography-Based Saga:**
Services communicate via events, no central coordinator.

```
Order Created Event
    ↓
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│Order Service│────→│Payment Service│────→│Inventory Svc│
└─────────────┘     └──────────────┘     └─────────────┘
                           │
                    Payment Failed Event
                           ↓
                    ┌─────────────┐
                    │Order Service│ (Compensate: Cancel Order)
                    └─────────────┘
```

**Implementation:**
```java
// Order Service - Initiates saga
@Service
public class OrderService {
    
    @Autowired
    private EventPublisher eventPublisher;
    
    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = new Order(request);
        order.setStatus(OrderStatus.PENDING);
        orderRepository.save(order);
        
        eventPublisher.publish(new OrderCreatedEvent(order));
        return order;
    }
    
    @EventListener
    public void handlePaymentFailed(PaymentFailedEvent event) {
        Order order = orderRepository.findById(event.getOrderId());
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
    }
}

// Payment Service - Continues saga
@Service
public class PaymentService {
    
    @Autowired
    private EventPublisher eventPublisher;
    
    @EventListener
    @Transactional
    public void handleOrderCreated(OrderCreatedEvent event) {
        try {
            Payment payment = processPayment(event.getOrder());
            paymentRepository.save(payment);
            eventPublisher.publish(new PaymentCompletedEvent(payment));
        } catch (PaymentException e) {
            eventPublisher.publish(new PaymentFailedEvent(event.getOrderId()));
        }
    }
}
```

**Orchestration-Based Saga:**
Central orchestrator coordinates the saga.

```
                    ┌──────────────────┐
                    │ Saga Orchestrator│
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│Order Service│     │Payment Service│     │Inventory Svc│
└─────────────┘     └──────────────┘     └─────────────┘
```

**Implementation:**
```java
@Service
public class OrderSagaOrchestrator {
    
    @Autowired
    private OrderService orderService;
    @Autowired
    private PaymentService paymentService;
    @Autowired
    private InventoryService inventoryService;
    
    public void executeOrderSaga(OrderRequest request) {
        try {
            Order order = orderService.createOrder(request);
            Payment payment = paymentService.processPayment(order);
            inventoryService.reserveInventory(order);
            orderService.completeOrder(order.getId());
            
        } catch (PaymentException e) {
            orderService.cancelOrder(order.getId());
            throw new SagaFailedException("Payment failed", e);
            
        } catch (InsufficientStockException e) {
            paymentService.refundPayment(payment.getId());
            orderService.cancelOrder(order.getId());
            throw new SagaFailedException("Inventory reservation failed", e);
        }
    }
}
```

---

### 3. CQRS (Command Query Responsibility Segregation)

**Definition:** Separate read and write operations into different models.

**Why CQRS?**
- Different read and write requirements
- Optimize reads and writes independently
- Scale reads and writes separately
- Complex domain logic

**Architecture:**
```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ├─────────────────┬─────────────────┐
     │                 │                 │
     ▼                 ▼                 ▼
┌─────────┐    ┌──────────┐    ┌──────────┐
│ Command │    │  Query   │    │  Query   │
│  API    │    │  API 1   │    │  API 2   │
└────┬────┘    └────┬─────┘    └────┬─────┘
     │              │                │
     ▼              ▼                ▼
┌─────────┐    ┌──────────┐    ┌──────────┐
│ Write   │    │  Read    │    │  Read    │
│  DB     │───→│  DB 1    │    │  DB 2    │
└─────────┘    └──────────┘    └──────────┘
        Events/Sync
```

**Implementation:**
```java
// Command Side - Handles writes
@RestController
@RequestMapping("/api/commands")
public class OrderCommandController {
    
    @Autowired
    private OrderCommandService commandService;
    
    @PostMapping("/orders")
    public ResponseEntity<OrderCreatedResponse> createOrder(
            @RequestBody CreateOrderCommand command) {
        String orderId = commandService.createOrder(command);
        return ResponseEntity.ok(new OrderCreatedResponse(orderId));
    }
}

@Service
public class OrderCommandService {
    
    @Autowired
    private OrderWriteRepository writeRepository;
    @Autowired
    private EventPublisher eventPublisher;
    
    @Transactional
    public String createOrder(CreateOrderCommand command) {
        Order order = new Order(command);
        writeRepository.save(order);
        eventPublisher.publish(new OrderCreatedEvent(order));
        return order.getId();
    }
}

// Query Side - Handles reads
@RestController
@RequestMapping("/api/queries")
public class OrderQueryController {
    
    @Autowired
    private OrderQueryService queryService;
    
    @GetMapping("/orders/{orderId}")
    public ResponseEntity<OrderDetailsView> getOrderDetails(
            @PathVariable String orderId) {
        OrderDetailsView view = queryService.getOrderDetails(orderId);
        return ResponseEntity.ok(view);
    }
}

@Service
public class OrderQueryService {
    
    @Autowired
    private OrderReadRepository readRepository;
    
    public OrderDetailsView getOrderDetails(String orderId) {
        return readRepository.findOrderDetailsById(orderId);
    }
}

// Event Handler - Synchronizes read side
@Service
public class OrderReadModelUpdater {
    
    @Autowired
    private OrderReadRepository readRepository;
    
    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        OrderDetailsView view = new OrderDetailsView(event.getOrder());
        readRepository.save(view);
    }
}
```

---

### 4. Event Sourcing

**Definition:** Store all changes to application state as a sequence of events instead of storing current state.

**Key Concepts:**
- Events are immutable
- Current state is derived by replaying events
- Complete audit trail
- Time travel (reconstruct state at any point)

**Architecture:**
```
Commands → Aggregate → Events → Event Store
                                     ↓
                              Event Handlers
                                     ↓
                              Read Models
```

**Implementation:**
```java
// Event
public abstract class OrderEvent {
    private String orderId;
    private LocalDateTime occurredAt;
    private long version;
}

public class OrderCreatedEvent extends OrderEvent {
    private String userId;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
}

// Aggregate
public class OrderAggregate {
    private String orderId;
    private OrderStatus status;
    private long version;
    private List<OrderEvent> uncommittedEvents = new ArrayList<>();
    
    public void createOrder(String userId, List<OrderItem> items) {
        OrderCreatedEvent event = new OrderCreatedEvent();
        event.setOrderId(UUID.randomUUID().toString());
        event.setUserId(userId);
        event.setItems(items);
        event.setVersion(version + 1);
        apply(event);
    }
    
    private void apply(OrderEvent event) {
        if (event instanceof OrderCreatedEvent) {
            OrderCreatedEvent e = (OrderCreatedEvent) event;
            this.orderId = e.getOrderId();
            this.status = OrderStatus.PENDING;
        }
        this.version = event.getVersion();
        this.uncommittedEvents.add(event);
    }
    
    public static OrderAggregate fromEvents(List<OrderEvent> events) {
        OrderAggregate aggregate = new OrderAggregate();
        events.forEach(aggregate::apply);
        aggregate.uncommittedEvents.clear();
        return aggregate;
    }
}

// Event Store
@Service
public class EventStore {
    
    @Autowired
    private EventRepository eventRepository;
    
    public void saveEvents(String aggregateId, List<OrderEvent> events, 
                          long expectedVersion) {
        long currentVersion = eventRepository.getLatestVersion(aggregateId);
        if (currentVersion != expectedVersion) {
            throw new ConcurrencyException("Aggregate has been modified");
        }
        events.forEach(event -> eventRepository.save(aggregateId, event));
    }
    
    public List<OrderEvent> getEvents(String aggregateId) {
        return eventRepository.findByAggregateId(aggregateId);
    }
}
```

---

## Communication Patterns

### 1. Synchronous Communication (REST/gRPC)

**REST (Representational State Transfer):**

**When to Use:**
- Simple CRUD operations
- Request-response pattern
- Client needs immediate response
- Public APIs

**Best Practices:**
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .header("Location", "/api/orders/" + order.getId())
            .body(new OrderResponse(order));
    }
    
    @GetMapping("/{orderId}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable String orderId) {
        return orderService.findById(orderId)
            .map(order -> ResponseEntity.ok(new OrderResponse(order)))
            .orElse(ResponseEntity.notFound().build());
    }
    
    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(OrderNotFoundException ex) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }
}

// Feign Client for inter-service communication
@FeignClient(name = "user-service", fallback = UserServiceFallback.class)
public interface UserServiceClient {
    
    @GetMapping("/api/users/{userId}")
    UserResponse getUser(@PathVariable("userId") String userId);
}

@Component
public class UserServiceFallback implements UserServiceClient {
    
    @Override
    public UserResponse getUser(String userId) {
        return UserResponse.builder()
            .userId(userId)
            .name("Unknown User")
            .build();
    }
}
```

**gRPC (Google Remote Procedure Call):**

**When to Use:**
- High-performance requirements
- Internal service communication
- Streaming data
- Polyglot environments

**Protocol Buffer Definition:**
```protobuf
syntax = "proto3";

service OrderService {
  rpc CreateOrder (CreateOrderRequest) returns (OrderResponse);
  rpc GetOrder (GetOrderRequest) returns (OrderResponse);
  rpc ListOrders (ListOrdersRequest) returns (stream OrderResponse);
}

message CreateOrderRequest {
  string user_id = 1;
  repeated OrderItem items = 2;
}

message OrderResponse {
  string order_id = 1;
  string user_id = 2;
  double total_amount = 3;
  string status = 4;
}
```

---

### 2. Asynchronous Communication (Messaging)

**When to Use:**
- Decoupled services
- Event-driven architecture
- Fire-and-forget operations
- High throughput requirements

**Message Broker Patterns:**

**Point-to-Point (Queue):**
```
Producer → Queue → Consumer
(One message consumed by one consumer)
```

**Publish-Subscribe (Topic):**
```
Producer → Topic → Consumer 1
                 → Consumer 2
                 → Consumer 3
(One message consumed by all subscribers)
```

**Implementation (Kafka):**
```java
// Producer
@Service
public class OrderEventProducer {
    
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        kafkaTemplate.send("order-events", order.getId(), event);
    }
}

// Consumer
@Service
public class OrderEventConsumer {
    
    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void handleOrderCreated(OrderCreatedEvent event) {
        notificationService.sendOrderConfirmation(
            event.getUserId(),
            event.getOrderId()
        );
    }
}

// Configuration
@Configuration
public class KafkaConfig {
    
    @Bean
    public ProducerFactory<String, OrderEvent> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        config.put(ProducerConfig.ACKS_CONFIG, "all");
        return new DefaultKafkaProducerFactory<>(config);
    }
}
```

---

## Resilience Patterns

### 1. Circuit Breaker Pattern

**Definition:** Prevent cascading failures by stopping calls to a failing service and providing fallback behavior.

**States:**
- **Closed:** Normal operation, requests pass through
- **Open:** Service is failing, requests fail immediately
- **Half-Open:** Test if service has recovered

**State Transitions:**
```
Closed ──(failures > threshold)──→ Open
  ↑                                  │
  │                                  │
  └──(success)──← Half-Open ←──(timeout)
```

**Implementation (Resilience4j):**
```java
@Service
public class OrderService {
    
    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    public User getUser(String userId) {
        return userServiceClient.getUser(userId);
    }
    
    public User getUserFallback(String userId, Exception ex) {
        log.warn("User service call failed, using fallback", ex);
        return User.builder()
            .userId(userId)
            .name("Unknown User")
            .build();
    }
}

// Configuration
@Configuration
public class CircuitBreakerConfig {
    
    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofSeconds(30))
            .slidingWindowSize(10)
            .minimumNumberOfCalls(5)
            .build();
        return CircuitBreakerRegistry.of(config);
    }
}
```

---

### 2. Retry Pattern

**Definition:** Automatically retry failed operations with configurable backoff strategy.

**Strategies:**
- **Fixed Delay:** Wait same time between retries
- **Exponential Backoff:** Increase wait time exponentially
- **Random Jitter:** Add randomness to avoid thundering herd

**Implementation:**
```java
@Service
public class PaymentService {
    
    @Retry(name = "paymentGateway", fallbackMethod = "processPaymentFallback")
    public Payment processPayment(Order order) {
        return paymentGateway.charge(order.getUserId(), order.getTotalAmount());
    }
    
    public Payment processPaymentFallback(Order order, Exception ex) {
        log.error("Payment failed after retries", ex);
        throw new PaymentFailedException("Unable to process payment", ex);
    }
}

// Configuration
@Configuration
public class RetryConfig {
    
    @Bean
    public RetryRegistry retryRegistry() {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofSeconds(1))
            .retryExceptions(IOException.class, TimeoutException.class)
            .build();
        return RetryRegistry.of(config);
    }
    
    // Exponential backoff with jitter
    @Bean
    public RetryRegistry exponentialRetryRegistry() {
        IntervalFunction intervalFn = IntervalFunction
            .ofExponentialRandomBackoff(1000, 2.0, 0.5);
        
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(5)
            .intervalFunction(intervalFn)
            .build();
        return RetryRegistry.of(config);
    }
}
```

---

### 3. Bulkhead Pattern

**Definition:** Isolate resources to prevent one failing component from taking down the entire system.

**Types:**
- **Thread Pool Bulkhead:** Separate thread pools for different operations
- **Semaphore Bulkhead:** Limit concurrent calls

**Implementation:**
```java
@Service
public class OrderService {
    
    @Bulkhead(name = "userService", type = Bulkhead.Type.THREADPOOL)
    public User getUser(String userId) {
        return userServiceClient.getUser(userId);
    }
    
    @Bulkhead(name = "inventoryService", type = Bulkhead.Type.SEMAPHORE)
    public Inventory checkInventory(String productId) {
        return inventoryServiceClient.getInventory(productId);
    }
}

// Configuration
@Configuration
public class BulkheadConfig {
    
    @Bean
    public BulkheadRegistry bulkheadRegistry() {
        ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
            .maxThreadPoolSize(10)
            .coreThreadPoolSize(5)
            .queueCapacity(20)
            .build();
        return BulkheadRegistry.of(config);
    }
}
```

---

### 4. Timeout Pattern

**Definition:** Set maximum time to wait for a response before failing.

**Implementation:**
```java
@Service
public class OrderService {
    
    @TimeLimiter(name = "default")
    public CompletableFuture<User> getUserAsync(String userId) {
        return CompletableFuture.supplyAsync(() -> 
            userServiceClient.getUser(userId)
        );
    }
}

// Configuration
@Configuration
public class TimeLimiterConfig {
    
    @Bean
    public TimeLimiterRegistry timeLimiterRegistry() {
        TimeLimiterConfig config = TimeLimiterConfig.custom()
            .timeoutDuration(Duration.ofSeconds(3))
            .cancelRunningFuture(true)
            .build();
        return TimeLimiterRegistry.of(config);
    }
}
```

---

### 5. Rate Limiting

**Definition:** Limit the number of requests to prevent overload.

**Implementation:**
```java
@Service
public class OrderService {
    
    @RateLimiter(name = "orderCreation")
    public Order createOrder(CreateOrderRequest request) {
        return orderRepository.save(new Order(request));
    }
}

// Configuration
@Configuration
public class RateLimiterConfig {
    
    @Bean
    public RateLimiterRegistry rateLimiterRegistry() {
        RateLimiterConfig config = RateLimiterConfig.custom()
            .limitForPeriod(100)
            .limitRefreshPeriod(Duration.ofSeconds(1))
            .timeoutDuration(Duration.ofMillis(500))
            .build();
        return RateLimiterRegistry.of(config);
    }
}
```

---

## Observability Patterns

### 1. Distributed Tracing

**Definition:** Track requests as they flow through multiple services.

**Implementation (OpenTelemetry):**
```java
@Configuration
public class TracingConfig {
    
    @Bean
    public OpenTelemetry openTelemetry() {
        JaegerGrpcSpanExporter jaegerExporter = JaegerGrpcSpanExporter.builder()
            .setEndpoint("http://localhost:14250")
            .build();
        
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
            .addSpanProcessor(BatchSpanProcessor.builder(jaegerExporter).build())
            .setResource(Resource.create(Attributes.of(
                ResourceAttributes.SERVICE_NAME, "order-service"
            )))
            .build();
        
        return OpenTelemetrySdk.builder()
            .setTracerProvider(tracerProvider)
            .buildAndRegisterGlobal();
    }
}

@Service
public class OrderService {
    
    private final Tracer tracer;
    
    public Order createOrder(CreateOrderRequest request) {
        Span span = tracer.spanBuilder("createOrder")
            .setSpanKind(SpanKind.SERVER)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            span.setAttribute("user.id", request.getUserId());
            Order order = processOrder(request);
            span.setStatus(StatusCode.OK);
            return order;
        } catch (Exception e) {
            span.recordException(e);
            span.setStatus(StatusCode.ERROR, e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
}
```

---

### 2. Centralized Logging

**Implementation (Structured Logging):**
```java
@Service
public class OrderService {
    
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
    
    public Order createOrder(CreateOrderRequest request) {
        String correlationId = MDC.get("correlationId");
        
        log.info("Creating order",
            kv("correlationId", correlationId),
            kv("userId", request.getUserId()),
            kv("itemCount", request.getItems().size())
        );
        
        try {
            Order order = orderRepository.save(new Order(request));
            log.info("Order created successfully",
                kv("correlationId", correlationId),
                kv("orderId", order.getId())
            );
            return order;
        } catch (Exception e) {
            log.error("Failed to create order",
                kv("correlationId", correlationId),
                kv("error", e.getMessage()),
                e
            );
            throw e;
        }
    }
}

// Correlation ID Filter
@Component
public class CorrelationIdFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) {
        String correlationId = request.getHeader("X-Correlation-ID");
        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }
        MDC.put("correlationId", correlationId);
        response.setHeader("X-Correlation-ID", correlationId);
        
        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

---

### 3. Metrics & Monitoring

**Implementation (Micrometer + Prometheus):**
```java
@Service
public class OrderService {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCreatedCounter;
    private final Timer orderProcessingTimer;
    
    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.orderCreatedCounter = Counter.builder("orders.created")
            .description("Total orders created")
            .tag("service", "order-service")
            .register(meterRegistry);
        
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
    }
    
    public Order createOrder(CreateOrderRequest request) {
        return orderProcessingTimer.record(() -> {
            Order order = orderRepository.save(new Order(request));
            orderCreatedCounter.increment();
            
            meterRegistry.gauge("orders.total.amount", order.getTotalAmount());
            
            return order;
        });
    }
}
```

---

## Deployment Patterns

### 1. Blue-Green Deployment

**Definition:** Two identical environments for zero-downtime deployments.

**Process:**
1. Blue (current) serves production traffic
2. Deploy new version to Green
3. Test Green environment
4. Switch traffic from Blue to Green
5. Keep Blue as rollback option

---

### 2. Canary Deployment

**Definition:** Gradual rollout to subset of users.

**Process:**
1. Deploy new version to small percentage (5%)
2. Monitor metrics
3. Gradually increase traffic (10%, 25%, 50%, 100%)
4. Rollback if issues detected

---

### 3. Rolling Deployment

**Definition:** Update instances incrementally.

**Process:**
1. Update one instance at a time
2. Health check before moving to next
3. Continue until all updated

---

## Security Patterns

### 1. API Gateway Security

**Implementation:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .antMatchers("/api/**").authenticated()
            .and()
            .oauth2ResourceServer()
                .jwt();
    }
}
```

---

### 2. Service-to-Service Authentication

**JWT Token Propagation:**
```java
@Component
public class JwtTokenRelayInterceptor implements ClientHttpRequestInterceptor {
    
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                                       ClientHttpRequestExecution execution) {
        String token = SecurityContextHolder.getContext()
            .getAuthentication()
            .getCredentials()
            .toString();
        
        request.getHeaders().setBearerAuth(token);
        return execution.execute(request, body);
    }
}
```

---

### 3. Secret Management

**Using Spring Cloud Config with Vault:**
```yaml
spring:
  cloud:
    config:
      server:
        vault:
          host: localhost
          port: 8200
          scheme: https
          authentication: TOKEN
          token: ${VAULT_TOKEN}
```

---

## Best Practices & Anti-Patterns

### Best Practices

1. **Start with a Monolith** - Don't over-engineer early
2. **Define Clear Boundaries** - Use DDD
3. **Automate Everything** - CI/CD, testing, deployment
4. **Monitor Proactively** - Logs, metrics, traces
5. **Design for Failure** - Resilience patterns
6. **Version APIs** - Backward compatibility
7. **Document Thoroughly** - APIs, architecture, decisions
8. **Test at All Levels** - Unit, integration, contract, E2E
9. **Secure by Default** - Authentication, encryption
10. **Keep Services Small** - Focused, manageable

---

### Anti-Patterns to Avoid

**1. Distributed Monolith**
- Services tightly coupled
- Must deploy together
- Shared database

**2. Chatty Services**
- Too many inter-service calls
- Network overhead
- Performance issues

**3. Shared Database**
- Violates autonomy
- Tight coupling
- Difficult to scale

**4. No API Versioning**
- Breaking changes
- Client compatibility issues

**5. Insufficient Monitoring**
- Can't diagnose issues
- No visibility into system

**6. Ignoring Failures**
- No circuit breakers
- Cascading failures
- Poor user experience

**7. Premature Optimization**
- Over-engineering
- Unnecessary complexity

**8. Lack of Automation**
- Manual deployments
- Inconsistent environments
- Slow delivery

---

## Summary

Microservices architecture provides significant benefits but requires careful design and implementation. Key takeaways:

- **Apply core principles** - SRP, autonomy, DDD
- **Choose right patterns** - Based on requirements
- **Design for failure** - Resilience is critical
- **Ensure observability** - Logs, metrics, traces
- **Automate deployment** - CI/CD pipelines
- **Secure properly** - Authentication, authorization
- **Start simple** - Evolve architecture over time

---

**Last Updated:** March 2026  
**Version:** 1.0
