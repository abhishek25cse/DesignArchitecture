# Rate Limiter - System Design

## 1. Problem Definition

Design a distributed rate limiting system that can:
- Throttle requests based on user identity, IP address, or API key
- Support multiple rate limiting rules (per-user, per-endpoint, tiered limits)
- Scale horizontally to handle millions of requests per second
- Maintain accuracy across distributed servers
- Provide graceful degradation on component failures

**Constraints:**
- Latency overhead: < 1ms per request
- Availability: 99.99%
- Accuracy: < 1% false positive rate
- Scale: 1M+ requests/second at peak

---

## 2. Capacity Estimation

### Traffic Analysis
```
Assumptions:
- 1 billion users globally
- Average: 10 requests/user/day
- Peak traffic: 10x average (during events, launches)

Calculations:
Daily requests:     1B × 10 = 10B requests/day
Average RPS:        10B / 86,400 ≈ 115,740 RPS
Peak RPS:           115,740 × 10 ≈ 1,157,400 RPS
```

### Storage Requirements
```
Per-user metadata:
- User ID: 16 bytes (UUID)
- Timestamp entries: 8 bytes × 100 (sliding window) = 800 bytes
- Metadata: ~200 bytes
Total per active user: ~1 KB

Active users (1-hour window):
- 1B / 24 ≈ 42M active users/hour

Memory required:
- 42M × 1 KB ≈ 42 GB
- With replication (3x): ~126 GB
- With overhead (2x): ~250 GB total
```

### Network Bandwidth
```
Per request:
- Rate limit check: ~200 bytes (key + metadata)
- Response: ~100 bytes

Bandwidth at peak:
- 1.16M RPS × 300 bytes ≈ 348 MB/s
- ~2.8 Gbps
```

---

## 3. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer                             │
│  (Web, Mobile, IoT devices)                                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  CDN / Edge Layer                            │
│  (CloudFlare, Akamai - Basic DDoS protection)              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Load Balancer (Layer 7)                         │
│  (AWS ALB, nginx) - Distributes across API gateways         │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│   API    │  │   API    │  │   API    │
│ Gateway  │  │ Gateway  │  │ Gateway  │
│   Node   │  │   Node   │  │   Node   │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     │   ┌─────────┴─────────┐   │
     │   │                   │   │
     ▼   ▼                   ▼   ▼
┌─────────────────────────────────────┐
│     Rate Limiter Service Layer      │
│  ┌─────────────────────────────┐   │
│  │  Rate Limit Decision Engine │   │
│  │  - Algorithm execution      │   │
│  │  - Rule evaluation          │   │
│  │  - Cache management         │   │
│  └─────────────────────────────┘   │
└────────────┬────────────────────────┘
             │
    ┌────────┼────────┐
    ▼        ▼        ▼
┌────────────────────────────────────────────┐
│        Distributed Cache Layer             │
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐│
│  │  Redis   │  │  Redis   │  │  Redis   ││
│  │ Master 1 │  │ Master 2 │  │ Master 3 ││
│  │ (Shard)  │  │ (Shard)  │  │ (Shard)  ││
│  └────┬─────┘  └────┬─────┘  └────┬─────┘│
│       │             │             │       │
│  ┌────▼─────┐  ┌────▼─────┐  ┌────▼─────┐│
│  │ Replica  │  │ Replica  │  │ Replica  ││
│  └──────────┘  └──────────┘  └──────────┘│
└────────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│      Configuration & Rules Store           │
│  (PostgreSQL/DynamoDB)                     │
│  - Rate limit rules                        │
│  - User tier mappings                      │
│  - Endpoint configurations                 │
└────────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│      Monitoring & Analytics Layer          │
│  - Metrics aggregation                     │
│  - Alerting                                │
│  - Audit logs                              │
└────────────────────────────────────────────┘
```

---

## 4. Component Design

### 4.1 Rate Limiting Algorithms

#### Algorithm Selection Matrix

| Algorithm | Time Complexity | Space Complexity | Accuracy | Burst Handling | Distributed Friendly |
|-----------|----------------|------------------|----------|----------------|---------------------|
| **Token Bucket** | O(1) | O(1) | High | Excellent | Yes |
| **Leaky Bucket** | O(1) | O(n) | High | Poor | Moderate |
| **Fixed Window** | O(1) | O(1) | Low | Poor | Yes |
| **Sliding Window Log** | O(n) | O(n) | Very High | Good | Moderate |
| **Sliding Window Counter** | O(1) | O(1) | High | Good | **Yes** ✓ |

**Recommended: Sliding Window Counter** - Optimal balance for distributed systems.

---

### 4.2 Algorithm Implementations (Java)

#### Token Bucket Algorithm

```java
import java.util.concurrent.locks.ReentrantLock;

public class TokenBucket {
    private final long capacity;
    private final double refillRate;
    private double tokens;
    private long lastRefillTimestamp;
    private final ReentrantLock lock;

    public TokenBucket(long capacity, double refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.tokens = capacity;
        this.lastRefillTimestamp = System.currentTimeMillis();
        this.lock = new ReentrantLock();
    }

    public boolean allowRequest() {
        lock.lock();
        try {
            refill();
            if (tokens >= 1) {
                tokens -= 1;
                return true;
            }
            return false;
        } finally {
            lock.unlock();
        }
    }

    private void refill() {
        long now = System.currentTimeMillis();
        double elapsedSeconds = (now - lastRefillTimestamp) / 1000.0;
        double tokensToAdd = elapsedSeconds * refillRate;
        
        tokens = Math.min(capacity, tokens + tokensToAdd);
        lastRefillTimestamp = now;
    }

    public long getAvailableTokens() {
        lock.lock();
        try {
            refill();
            return (long) tokens;
        } finally {
            lock.unlock();
        }
    }
}

// Usage
TokenBucket limiter = new TokenBucket(10, 1.0); // 10 tokens, 1 per second
if (limiter.allowRequest()) {
    System.out.println("Request allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

**Characteristics:**
- **Pros**: Handles bursts, memory efficient, smooth traffic flow
- **Cons**: Complex parameter tuning, can allow short bursts
- **Use Case**: API gateways, when controlled bursts are acceptable

---

#### Leaky Bucket Algorithm

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.ReentrantLock;

public class LeakyBucket {
    private final int capacity;
    private final double leakRate;
    private final Queue<Long> queue;
    private long lastLeakTimestamp;
    private final ReentrantLock lock;

    public LeakyBucket(int capacity, double leakRate) {
        this.capacity = capacity;
        this.leakRate = leakRate;
        this.queue = new LinkedList<>();
        this.lastLeakTimestamp = System.currentTimeMillis();
        this.lock = new ReentrantLock();
    }

    public boolean allowRequest() {
        lock.lock();
        try {
            leak();
            if (queue.size() < capacity) {
                queue.offer(System.currentTimeMillis());
                return true;
            }
            return false;
        } finally {
            lock.unlock();
        }
    }

    private void leak() {
        long now = System.currentTimeMillis();
        double elapsedSeconds = (now - lastLeakTimestamp) / 1000.0;
        int requestsToLeak = (int) (elapsedSeconds * leakRate);
        
        for (int i = 0; i < requestsToLeak && !queue.isEmpty(); i++) {
            queue.poll();
        }
        
        lastLeakTimestamp = now;
    }

    public int getQueueSize() {
        lock.lock();
        try {
            leak();
            return queue.size();
        } finally {
            lock.unlock();
        }
    }
}

// Usage
LeakyBucket limiter = new LeakyBucket(10, 1.0); // 10 capacity, 1 req/sec
if (limiter.allowRequest()) {
    System.out.println("Request allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

**Characteristics:**
- **Pros**: Smooth constant output, prevents bursts
- **Cons**: Rejects traffic during bursts, head-of-line blocking
- **Use Case**: Traffic shaping, QoS requirements

---

#### Fixed Window Counter Algorithm

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

public class FixedWindowCounter {
    private final int limit;
    private final long windowSizeMs;
    private final ConcurrentHashMap<String, WindowData> counters;

    private static class WindowData {
        final AtomicInteger count;
        final long windowStart;

        WindowData(long windowStart) {
            this.count = new AtomicInteger(0);
            this.windowStart = windowStart;
        }
    }

    public FixedWindowCounter(int limit, long windowSizeMs) {
        this.limit = limit;
        this.windowSizeMs = windowSizeMs;
        this.counters = new ConcurrentHashMap<>();
    }

    public boolean allowRequest(String userId) {
        long now = System.currentTimeMillis();
        long currentWindow = now / windowSizeMs;
        String key = userId + ":" + currentWindow;

        WindowData data = counters.computeIfAbsent(key, k -> new WindowData(currentWindow));
        
        // Clean old windows
        cleanupOldWindows(currentWindow);
        
        return data.count.incrementAndGet() <= limit;
    }

    private void cleanupOldWindows(long currentWindow) {
        counters.entrySet().removeIf(entry -> {
            long windowId = Long.parseLong(entry.getKey().split(":")[1]);
            return windowId < currentWindow - 1;
        });
    }

    public int getCurrentCount(String userId) {
        long currentWindow = System.currentTimeMillis() / windowSizeMs;
        String key = userId + ":" + currentWindow;
        WindowData data = counters.get(key);
        return data != null ? data.count.get() : 0;
    }
}

// Usage
FixedWindowCounter limiter = new FixedWindowCounter(100, 60000); // 100 req/min
if (limiter.allowRequest("user123")) {
    System.out.println("Request allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

**Characteristics:**
- **Pros**: Simple, minimal memory, easy to understand
- **Cons**: Boundary burst problem (can 2x limit), inaccurate
- **Use Case**: Internal services, approximate limiting

---

#### Sliding Window Log Algorithm

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.List;

public class SlidingWindowLog {
    private final int limit;
    private final long windowSizeMs;
    private final ConcurrentHashMap<String, CopyOnWriteArrayList<Long>> logs;

    public SlidingWindowLog(int limit, long windowSizeMs) {
        this.limit = limit;
        this.windowSizeMs = windowSizeMs;
        this.logs = new ConcurrentHashMap<>();
    }

    public boolean allowRequest(String userId) {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;

        CopyOnWriteArrayList<Long> userLogs = logs.computeIfAbsent(
            userId, 
            k -> new CopyOnWriteArrayList<>()
        );

        // Remove expired timestamps
        userLogs.removeIf(timestamp -> timestamp <= windowStart);

        if (userLogs.size() < limit) {
            userLogs.add(now);
            return true;
        }
        return false;
    }

    public int getCurrentCount(String userId) {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;
        
        List<Long> userLogs = logs.get(userId);
        if (userLogs == null) return 0;
        
        return (int) userLogs.stream()
            .filter(timestamp -> timestamp > windowStart)
            .count();
    }
}

// Usage
SlidingWindowLog limiter = new SlidingWindowLog(100, 60000); // 100 req/min
if (limiter.allowRequest("user123")) {
    System.out.println("Request allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

**Characteristics:**
- **Pros**: Very accurate, no boundary issues
- **Cons**: Memory intensive, O(n) cleanup
- **Use Case**: Low-traffic endpoints, strict accuracy requirements

---

#### Sliding Window Counter Algorithm (RECOMMENDED)

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.locks.ReentrantLock;

public class SlidingWindowCounter {
    private final int limit;
    private final long windowSizeMs;
    private final ConcurrentHashMap<String, WindowCounters> counters;

    private static class WindowCounters {
        int prevCount;
        int currCount;
        long windowStart;
        final ReentrantLock lock;

        WindowCounters() {
            this.prevCount = 0;
            this.currCount = 0;
            this.windowStart = 0;
            this.lock = new ReentrantLock();
        }
    }

    public SlidingWindowCounter(int limit, long windowSizeMs) {
        this.limit = limit;
        this.windowSizeMs = windowSizeMs;
        this.counters = new ConcurrentHashMap<>();
    }

    public boolean allowRequest(String userId) {
        long now = System.currentTimeMillis();
        long currentWindow = now / windowSizeMs;

        WindowCounters data = counters.computeIfAbsent(userId, k -> new WindowCounters());

        data.lock.lock();
        try {
            // Slide window if needed
            if (currentWindow > data.windowStart) {
                data.prevCount = data.currCount;
                data.currCount = 0;
                data.windowStart = currentWindow;
            }

            // Calculate weighted count
            long elapsedInWindow = now - (currentWindow * windowSizeMs);
            double weight = 1.0 - ((double) elapsedInWindow / windowSizeMs);
            double estimatedCount = data.prevCount * weight + data.currCount;

            if (estimatedCount < limit) {
                data.currCount++;
                return true;
            }
            return false;
        } finally {
            data.lock.unlock();
        }
    }

    public int getEstimatedCount(String userId) {
        long now = System.currentTimeMillis();
        long currentWindow = now / windowSizeMs;
        
        WindowCounters data = counters.get(userId);
        if (data == null) return 0;

        data.lock.lock();
        try {
            long elapsedInWindow = now - (currentWindow * windowSizeMs);
            double weight = 1.0 - ((double) elapsedInWindow / windowSizeMs);
            return (int) (data.prevCount * weight + data.currCount);
        } finally {
            data.lock.unlock();
        }
    }
}

// Usage
SlidingWindowCounter limiter = new SlidingWindowCounter(100, 60000); // 100 req/min
if (limiter.allowRequest("user123")) {
    System.out.println("Request allowed");
} else {
    System.out.println("Rate limit exceeded");
}
```

**Mathematical Model:**
```
Estimated count = Cp × (1 - (Tc - Tw) / W) + Cc

Where:
- Cp = count in previous window
- Cc = count in current window
- Tc = current timestamp
- Tw = current window start
- W = window size
```

**Characteristics:**
- **Pros**: O(1) time/space, 97-99% accurate, no boundary burst
- **Cons**: Slight approximation
- **Use Case**: High-traffic distributed systems (RECOMMENDED)

---

### 4.3 Spring Boot Integration

#### Rate Limiter Filter

```java
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class RateLimitFilter extends OncePerRequestFilter {
    
    private final RateLimiterService rateLimiterService;
    
    public RateLimitFilter(RateLimiterService rateLimiterService) {
        this.rateLimiterService = rateLimiterService;
    }

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {
        
        String userId = extractUserId(request);
        String endpoint = request.getRequestURI();
        
        RateLimitResult result = rateLimiterService.checkRateLimit(userId, endpoint);
        
        // Add rate limit headers
        response.setHeader("X-RateLimit-Limit", String.valueOf(result.getLimit()));
        response.setHeader("X-RateLimit-Remaining", String.valueOf(result.getRemaining()));
        response.setHeader("X-RateLimit-Reset", String.valueOf(result.getResetTime()));
        
        if (!result.isAllowed()) {
            response.setStatus(429);
            response.setHeader("Retry-After", String.valueOf(result.getRetryAfter()));
            response.getWriter().write("{\"error\": \"Rate limit exceeded\"}");
            return;
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String extractUserId(HttpServletRequest request) {
        String apiKey = request.getHeader("X-API-Key");
        if (apiKey != null) {
            return apiKey;
        }
        return request.getRemoteAddr();
    }
}
```

#### Rate Limiter Service

```java
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.RedisScript;
import org.springframework.stereotype.Service;
import java.util.Collections;
import java.util.List;

@Service
public class RateLimiterService {
    
    private final RedisTemplate<String, String> redisTemplate;
    private final RateLimitConfigService configService;
    private final RedisScript<List> rateLimitScript;
    
    public RateLimiterService(
            RedisTemplate<String, String> redisTemplate,
            RateLimitConfigService configService,
            RedisScript<List> rateLimitScript) {
        this.redisTemplate = redisTemplate;
        this.configService = configService;
        this.rateLimitScript = rateLimitScript;
    }
    
    public RateLimitResult checkRateLimit(String userId, String endpoint) {
        RateLimitRule rule = configService.getRule(userId, endpoint);
        
        String key = "rate_limit:" + userId + ":" + endpoint;
        long now = System.currentTimeMillis();
        
        List<Long> result = redisTemplate.execute(
            rateLimitScript,
            Collections.singletonList(key),
            String.valueOf(now),
            String.valueOf(rule.getWindowMs()),
            String.valueOf(rule.getLimit())
        );
        
        boolean allowed = result.get(0) == 1;
        long remaining = result.get(1);
        
        return RateLimitResult.builder()
            .allowed(allowed)
            .limit(rule.getLimit())
            .remaining(remaining)
            .resetTime(now + rule.getWindowMs())
            .retryAfter(rule.getWindowMs() / 1000)
            .build();
    }
}
```

#### Redis Lua Script Configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.data.redis.core.script.RedisScript;
import java.util.List;

@Configuration
public class RedisScriptConfig {
    
    @Bean
    public RedisScript<List> rateLimitScript() {
        DefaultRedisScript<List> script = new DefaultRedisScript<>();
        script.setLocation(new ClassPathResource("scripts/rate_limit.lua"));
        script.setResultType(List.class);
        return script;
    }
}
```

**Lua Script** (`resources/scripts/rate_limit.lua`):
```lua
local key = KEYS[1]
local now = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])

local prev_key = key .. ":prev"
local curr_key = key .. ":curr"
local window_key = key .. ":window"

local curr_window = math.floor(now / window)
local stored_window = tonumber(redis.call('GET', window_key) or 0)

if curr_window > stored_window then
    redis.call('SET', prev_key, redis.call('GET', curr_key) or 0)
    redis.call('SET', curr_key, 0)
    redis.call('SET', window_key, curr_window)
end

local prev_count = tonumber(redis.call('GET', prev_key) or 0)
local curr_count = tonumber(redis.call('GET', curr_key) or 0)

local elapsed = now - (curr_window * window)
local weight = 1 - (elapsed / window)
local estimated = prev_count * weight + curr_count

if estimated < limit then
    redis.call('INCR', curr_key)
    redis.call('EXPIRE', curr_key, window * 2 / 1000)
    return {1, math.floor(limit - estimated - 1)}
end

return {0, 0}
```

---

### 4.4 Data Models

#### Rate Limit Rule Entity

```java
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "rate_limit_rules")
public class RateLimitRule {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String ruleName;
    
    @Column(nullable = false)
    private String userTier;
    
    private String endpointPattern;
    
    private String httpMethod;
    
    @Column(nullable = false)
    private Integer requestsPerWindow;
    
    @Column(nullable = false)
    private Integer windowSeconds;
    
    private Integer burstAllowance;
    
    private Integer priority;
    
    @Column(nullable = false)
    private Boolean active = true;
    
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getRuleName() { return ruleName; }
    public void setRuleName(String ruleName) { this.ruleName = ruleName; }
    
    public String getUserTier() { return userTier; }
    public void setUserTier(String userTier) { this.userTier = userTier; }
    
    public String getEndpointPattern() { return endpointPattern; }
    public void setEndpointPattern(String endpointPattern) { 
        this.endpointPattern = endpointPattern; 
    }
    
    public Integer getRequestsPerWindow() { return requestsPerWindow; }
    public void setRequestsPerWindow(Integer requestsPerWindow) { 
        this.requestsPerWindow = requestsPerWindow; 
    }
    
    public Integer getWindowSeconds() { return windowSeconds; }
    public void setWindowSeconds(Integer windowSeconds) { 
        this.windowSeconds = windowSeconds; 
    }
    
    public long getWindowMs() {
        return windowSeconds * 1000L;
    }
    
    public Integer getLimit() {
        return requestsPerWindow;
    }
}
```

#### Rate Limit Result DTO

```java
import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class RateLimitResult {
    private boolean allowed;
    private int limit;
    private long remaining;
    private long resetTime;
    private long retryAfter;
}
```

#### Rate Limit Override Entity

```java
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "rate_limit_overrides")
public class RateLimitOverride {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String userId;
    
    @ManyToOne
    @JoinColumn(name = "rule_id")
    private RateLimitRule rule;
    
    private Integer customLimit;
    
    private LocalDateTime expiresAt;
    
    private String reason;
    
    private String createdBy;
    
    private LocalDateTime createdAt;
    
    // Getters and setters
}
```

---

### 4.5 Configuration Service

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
import java.util.Optional;

@Service
public class RateLimitConfigService {
    
    private final RateLimitRuleRepository ruleRepository;
    private final RateLimitOverrideRepository overrideRepository;
    
    public RateLimitConfigService(
            RateLimitRuleRepository ruleRepository,
            RateLimitOverrideRepository overrideRepository) {
        this.ruleRepository = ruleRepository;
        this.overrideRepository = overrideRepository;
    }
    
    @Cacheable(value = "rateLimitRules", key = "#userId + ':' + #endpoint")
    public RateLimitRule getRule(String userId, String endpoint) {
        // 1. Check user-specific override
        Optional<RateLimitOverride> override = overrideRepository
            .findActiveOverrideByUserId(userId);
        if (override.isPresent()) {
            return override.get().getRule();
        }
        
        // 2. Check endpoint-specific rule
        Optional<RateLimitRule> endpointRule = ruleRepository
            .findByEndpointPattern(endpoint);
        if (endpointRule.isPresent()) {
            return endpointRule.get();
        }
        
        // 3. Check user tier default
        String userTier = getUserTier(userId);
        Optional<RateLimitRule> tierRule = ruleRepository
            .findByUserTier(userTier);
        if (tierRule.isPresent()) {
            return tierRule.get();
        }
        
        // 4. Return global default
        return getDefaultRule();
    }
    
    private String getUserTier(String userId) {
        // Fetch from user service or cache
        return "free"; // Default
    }
    
    private RateLimitRule getDefaultRule() {
        RateLimitRule rule = new RateLimitRule();
        rule.setRequestsPerWindow(100);
        rule.setWindowSeconds(3600);
        return rule;
    }
}
```

---

## 5. Database Schema

```sql
-- Rate limit rules
CREATE TABLE rate_limit_rules (
    id BIGSERIAL PRIMARY KEY,
    rule_name VARCHAR(100) NOT NULL,
    user_tier VARCHAR(50),
    endpoint_pattern VARCHAR(255),
    http_method VARCHAR(10),
    requests_per_window INT NOT NULL,
    window_seconds INT NOT NULL,
    burst_allowance INT DEFAULT 0,
    priority INT DEFAULT 0,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tier_endpoint ON rate_limit_rules(user_tier, endpoint_pattern);
CREATE INDEX idx_priority ON rate_limit_rules(priority DESC);

-- User overrides
CREATE TABLE rate_limit_overrides (
    id BIGSERIAL PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    rule_id BIGINT REFERENCES rate_limit_rules(id),
    custom_limit INT,
    expires_at TIMESTAMP,
    reason TEXT,
    created_by VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_override ON rate_limit_overrides(user_id, expires_at);

-- Audit log (partitioned by month)
CREATE TABLE rate_limit_events (
    id BIGSERIAL,
    user_id VARCHAR(255) NOT NULL,
    endpoint VARCHAR(255),
    action VARCHAR(20),
    rule_applied VARCHAR(100),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id, timestamp)
) PARTITION BY RANGE (timestamp);

CREATE INDEX idx_events_user_time ON rate_limit_events(user_id, timestamp);

-- Sample data
INSERT INTO rate_limit_rules (rule_name, user_tier, endpoint_pattern, requests_per_window, window_seconds)
VALUES 
    ('free_tier_default', 'free', '/api/*', 100, 3600),
    ('pro_tier_default', 'pro', '/api/*', 1000, 3600),
    ('enterprise_tier_default', 'enterprise', '/api/*', 10000, 3600),
    ('search_endpoint_limit', 'free', '/api/search', 10, 60),
    ('upload_endpoint_limit', 'free', '/api/upload', 5, 3600);
```

---

## 6. Scalability Considerations

### 6.1 Redis Cluster Configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import java.util.Arrays;

@Configuration
public class RedisClusterConfig {
    
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisClusterConfiguration clusterConfig = new RedisClusterConfiguration(
            Arrays.asList(
                "redis-node-1:6379",
                "redis-node-2:6379",
                "redis-node-3:6379"
            )
        );
        
        return new LettuceConnectionFactory(clusterConfig);
    }
    
    @Bean
    public RedisTemplate<String, String> redisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, String> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new StringRedisSerializer());
        return template;
    }
}
```

### 6.2 Circuit Breaker Pattern

```java
import io.github.resilience4j.circuitbreaker.CircuitBreaker;
import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.circuitbreaker.CircuitBreakerRegistry;
import org.springframework.stereotype.Component;
import java.time.Duration;

@Component
public class RateLimiterWithCircuitBreaker {
    
    private final RateLimiterService rateLimiterService;
    private final CircuitBreaker circuitBreaker;
    
    public RateLimiterWithCircuitBreaker(RateLimiterService rateLimiterService) {
        this.rateLimiterService = rateLimiterService;
        
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofSeconds(30))
            .slidingWindowSize(10)
            .build();
        
        CircuitBreakerRegistry registry = CircuitBreakerRegistry.of(config);
        this.circuitBreaker = registry.circuitBreaker("rateLimiter");
    }
    
    public RateLimitResult checkRateLimit(String userId, String endpoint) {
        return circuitBreaker.executeSupplier(() -> {
            try {
                return rateLimiterService.checkRateLimit(userId, endpoint);
            } catch (Exception e) {
                // Fail-open: Allow request if Redis is down
                return RateLimitResult.builder()
                    .allowed(true)
                    .limit(Integer.MAX_VALUE)
                    .remaining(Integer.MAX_VALUE)
                    .resetTime(System.currentTimeMillis() + 60000)
                    .retryAfter(0)
                    .build();
            }
        });
    }
}
```

### 6.3 Local Cache Layer

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.stereotype.Component;
import java.time.Duration;

@Component
public class LocalCacheRateLimiter {
    
    private final Cache<String, CachedRateLimitData> localCache;
    private final RateLimiterService redisRateLimiter;
    
    private static class CachedRateLimitData {
        int count;
        long timestamp;
        int limit;
        
        CachedRateLimitData(int count, long timestamp, int limit) {
            this.count = count;
            this.timestamp = timestamp;
            this.limit = limit;
        }
    }
    
    public LocalCacheRateLimiter(RateLimiterService redisRateLimiter) {
        this.redisRateLimiter = redisRateLimiter;
        this.localCache = Caffeine.newBuilder()
            .maximumSize(10_000)
            .expireAfterWrite(Duration.ofSeconds(1))
            .build();
    }
    
    public RateLimitResult checkRateLimit(String userId, String endpoint) {
        String key = userId + ":" + endpoint;
        long now = System.currentTimeMillis();
        
        CachedRateLimitData cached = localCache.getIfPresent(key);
        
        if (cached != null && now - cached.timestamp < 1000) {
            // Use local cache (1 second TTL)
            if (cached.count < cached.limit) {
                cached.count++;
                return RateLimitResult.builder()
                    .allowed(true)
                    .limit(cached.limit)
                    .remaining(cached.limit - cached.count)
                    .resetTime(now + 60000)
                    .retryAfter(0)
                    .build();
            }
            return RateLimitResult.builder()
                .allowed(false)
                .limit(cached.limit)
                .remaining(0)
                .resetTime(now + 60000)
                .retryAfter(60)
                .build();
        }
        
        // Cache miss - check Redis
        RateLimitResult result = redisRateLimiter.checkRateLimit(userId, endpoint);
        
        localCache.put(key, new CachedRateLimitData(
            result.isAllowed() ? 1 : result.getLimit(),
            now,
            result.getLimit()
        ));
        
        return result;
    }
}
```

---

## 7. Monitoring & Observability

### 7.1 Metrics Collection

```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Component;

@Component
public class RateLimiterMetrics {
    
    private final MeterRegistry meterRegistry;
    
    public RateLimiterMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    public void recordRateLimitCheck(
            String userId, 
            String endpoint, 
            boolean allowed, 
            long latencyMs) {
        
        meterRegistry.counter("rate_limit.checks",
            "user_tier", getUserTier(userId),
            "endpoint", endpoint,
            "result", allowed ? "allowed" : "blocked"
        ).increment();
        
        meterRegistry.timer("rate_limit.latency",
            "endpoint", endpoint
        ).record(Duration.ofMillis(latencyMs));
        
        if (!allowed) {
            meterRegistry.counter("rate_limit.exceeded",
                "user_id", userId,
                "endpoint", endpoint
            ).increment();
        }
    }
    
    public void recordRedisError(String operation) {
        meterRegistry.counter("rate_limit.redis.errors",
            "operation", operation
        ).increment();
    }
    
    private String getUserTier(String userId) {
        // Fetch user tier
        return "free";
    }
}
```

### 7.2 Health Check

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

@Component
public class RateLimiterHealthIndicator implements HealthIndicator {
    
    private final RedisTemplate<String, String> redisTemplate;
    
    public RateLimiterHealthIndicator(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
    @Override
    public Health health() {
        try {
            String pong = redisTemplate.getConnectionFactory()
                .getConnection()
                .ping();
            
            if ("PONG".equals(pong)) {
                return Health.up()
                    .withDetail("redis", "connected")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("redis", "disconnected")
                .withException(e)
                .build();
        }
        
        return Health.unknown().build();
    }
}
```

---

## 8. Trade-offs & Design Decisions

| Decision | Option A | Option B | Choice | Rationale |
|----------|----------|----------|--------|-----------|
| **Algorithm** | Token Bucket | Sliding Window Counter | **B** | Better accuracy, simpler distributed impl |
| **Storage** | Redis | Cassandra | **A** | Lower latency, simpler ops |
| **Consistency** | Strong | Eventual | **B** | Acceptable for rate limiting, better performance |
| **Failure Mode** | Fail-open | Fail-closed | **A** | Availability > strict limits (except critical endpoints) |
| **Cache** | L1 + L2 | L2 only | **A** | 10x performance gain, acceptable accuracy loss |

---

## 9. Testing Strategy

### 9.1 Unit Tests

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class SlidingWindowCounterTest {
    
    @Test
    void testAllowRequestWithinLimit() {
        SlidingWindowCounter limiter = new SlidingWindowCounter(10, 60000);
        
        for (int i = 0; i < 10; i++) {
            assertTrue(limiter.allowRequest("user1"));
        }
        
        assertFalse(limiter.allowRequest("user1"));
    }
    
    @Test
    void testWindowSliding() throws InterruptedException {
        SlidingWindowCounter limiter = new SlidingWindowCounter(5, 1000);
        
        // Fill the limit
        for (int i = 0; i < 5; i++) {
            assertTrue(limiter.allowRequest("user1"));
        }
        assertFalse(limiter.allowRequest("user1"));
        
        // Wait for window to slide
        Thread.sleep(1100);
        
        // Should allow again
        assertTrue(limiter.allowRequest("user1"));
    }
    
    @Test
    void testMultipleUsers() {
        SlidingWindowCounter limiter = new SlidingWindowCounter(5, 60000);
        
        for (int i = 0; i < 5; i++) {
            assertTrue(limiter.allowRequest("user1"));
            assertTrue(limiter.allowRequest("user2"));
        }
        
        assertFalse(limiter.allowRequest("user1"));
        assertFalse(limiter.allowRequest("user2"));
    }
}
```

### 9.2 Integration Tests

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RateLimiterIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void testRateLimitHeaders() {
        ResponseEntity<String> response = restTemplate.getForEntity("/api/test", String.class);
        
        assertNotNull(response.getHeaders().get("X-RateLimit-Limit"));
        assertNotNull(response.getHeaders().get("X-RateLimit-Remaining"));
        assertNotNull(response.getHeaders().get("X-RateLimit-Reset"));
    }
    
    @Test
    void testRateLimitExceeded() {
        // Make requests until limit exceeded
        for (int i = 0; i < 100; i++) {
            restTemplate.getForEntity("/api/test", String.class);
        }
        
        ResponseEntity<String> response = restTemplate.getForEntity("/api/test", String.class);
        assertEquals(HttpStatus.TOO_MANY_REQUESTS, response.getStatusCode());
        assertNotNull(response.getHeaders().get("Retry-After"));
    }
}
```

---

## 10. Summary

### Best Practices

1. **Use Sliding Window Counter** for most use cases
2. **Implement in middleware** for centralized control
3. **Use Redis Cluster** for distributed state
4. **Add proper headers** for API transparency
5. **Monitor and alert** on key metrics
6. **Fail-open** on Redis failures (except critical endpoints)
7. **Use Lua scripts** for atomic operations
8. **Implement circuit breaker** for resilience

### Key Takeaways

- Rate limiting is essential for API stability and security
- Choose algorithm based on accuracy vs performance needs
- Redis is the standard for distributed rate limiting
- Always include rate limit information in response headers
- Monitor and tune based on actual traffic patterns
- Use local cache for performance (with acceptable accuracy trade-off)
- Test thoroughly under load conditions

---

**Recommended Stack:**
- **Algorithm**: Sliding Window Counter
- **Storage**: Redis Cluster (3+ nodes)
- **Framework**: Spring Boot with Resilience4j
- **Monitoring**: Micrometer + Prometheus + Grafana
- **Caching**: Caffeine (L1) + Redis (L2)

---

**Last Updated**: February 2026  
**Version**: 1.0
