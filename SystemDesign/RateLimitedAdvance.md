# Rate Limiting: A Comprehensive Architectural Deep Dive

## For Senior & Principal Architects | Distributed Systems Edition

---

## 1. Foundational Premise: Why Rate Limiting is a Non-Negotiable Cross-Cutting Concern

Rate limiting is not merely a defensive mechanism — it is a **Quality of Service (QoS) enforcement primitive** that sits at the intersection of **system reliability engineering**, **capacity planning**, and **API monetization governance**.

In a microservices topology, the absence of rate limiting creates **cascading failure amplification** — a single tenant's unbounded traffic can saturate connection pools, exhaust thread pools, trigger GC pressure, and ultimately cause **systemic brownout** across your entire service mesh.

### Architectural Motivations (Beyond the Basics):

| Concern | Architectural Implication |
|---|---|
| **Back-pressure Propagation** | Rate limiting acts as a **back-pressure valve** at the edge, preventing downstream saturation in asynchronous/reactive pipelines. Without it, unbounded producers overwhelm bounded consumers (violating the Reactive Manifesto's back-pressure principle). |
| **Fair-Use Policy Enforcement** | Enables **multi-tenant resource isolation** — critical in SaaS platforms where noisy-neighbor syndrome can violate SLA guarantees for other tenants. |
| **Cost Attribution & Chargeback** | In FinOps-driven organizations, rate limiting feeds into **metering and billing pipelines**, enabling usage-based pricing models with granular per-endpoint, per-tier attribution. |
| **Regulatory & Compliance Guardrails** | Certain financial (PCI-DSS) and healthcare (HIPAA) systems require demonstrable controls against abuse vectors. Rate limiting serves as an auditable control plane artifact. |
| **Graceful Degradation under Load** | Enables **load shedding** strategies — intentionally rejecting excess traffic to preserve system stability for priority traffic classes (weighted fair queuing). |
| **Service Mesh Observability** | Rate limiter rejection metrics (`429` response codes) provide leading indicators for capacity planning — if your `429` rate exceeds a threshold, it signals organic demand growth, not abuse. |

---

## 2. Taxonomy of Rate Limiting Algorithms — Comparative Architectural Analysis

Each algorithm embodies different **trade-offs across accuracy, memory footprint, burst tolerance, and implementation complexity**. Understanding these trade-offs is what separates a senior architect from a junior developer pasting code from Stack Overflow.

### 2.1 Fixed Window Counter

```
Window: [0s ────────── 60s] [60s ────────── 120s]
         ████████░░░░░░░░    ░░░░░░░░░░░░░░░░░░
         (8 requests)         (reset to 0)
```

**Mechanism:** Divides time into fixed-duration windows. Maintains an atomic counter per window. Resets at each boundary.

**Critical Flaw — Boundary Burst Vulnerability:**
If you allow 100 requests per minute:
- A client sends 100 requests at `t=59s` (end of Window 1)
- Then sends 100 requests at `t=60s` (beginning of Window 2)
- Result: **200 requests in 2 seconds** — double the intended rate

This **temporal aliasing** at window boundaries makes Fixed Window unsuitable for systems with strict throughput invariants.

**Space Complexity:** `O(1)` — just a counter and a timestamp.

---

### 2.2 Sliding Window Log

**Mechanism:** Maintains a **sorted set** (e.g., `TreeSet` or Redis `ZSET`) of timestamps for every request. On each incoming request, it evicts expired entries, then checks the cardinality.

**Accuracy:** Near-perfect — no boundary burst issue.

**Critical Flaw — Unbounded Memory Growth:**
For a high-throughput API (e.g., 50,000 req/sec with a 60-second window), you're storing **3 million timestamps per client**. In a multi-tenant system with 10,000 active clients, that's **30 billion entries**. This is architecturally untenable for memory-constrained systems.

**Space Complexity:** `O(N)` where N = requests in the window. **Not production-viable at scale without significant memory budgeting.**

---

### 2.3 Sliding Window Counter (Hybrid)

**Mechanism:** Combines the efficiency of Fixed Window with the accuracy of Sliding Window Log. It uses **weighted interpolation** between the current and previous window counters:

```
Effective Count = (Previous Window Count × Overlap %) + (Current Window Count)
```

**Example:** Window = 60s, current time = `t=75s` (25% into current window)
```
Effective = (PrevCount × 0.75) + (CurrCount × 1.0)
```

**Space Complexity:** `O(1)` — only two counters and two timestamps.
**Accuracy:** Approximately 99.97% accurate (CloudFlare's published benchmarks).

This is the algorithm used by **CloudFlare** at their edge network — processing **millions of requests per second globally**.

---

### 2.4 Leaky Bucket (FIFO Queue Model)

```
        ┌─── Incoming Requests (variable rate)
        ▼
   ┌──────────┐
   │ ░░░░░░░░ │ ← Bounded queue (bucket)
   │ ░░░░░░░░ │
   └────┬─────┘
        │
        ▼ Outflow at FIXED rate (constant drip)
   [Processed]
```

**Mechanism:** Essentially a **FIFO queue with a bounded capacity** and a constant-rate drain. Requests that arrive when the queue is full are immediately dropped (or a `429` is returned).

**Architectural Use Case:** Perfect for **traffic shaping** where you need a **constant, predictable outflow rate** — e.g., writing to a legacy mainframe system that cannot tolerate any burst whatsoever.

**Critical Limitation:** Zero burst tolerance. In real-world systems where traffic is inherently **bursty** (Poisson-distributed arrival patterns), this causes excessive rejection even when aggregate throughput is well within capacity. This makes it suboptimal for user-facing APIs.

**Relationship to Token Bucket:** The Leaky Bucket is the **mathematical dual** of the Token Bucket — they are equivalent in steady-state throughput but differ fundamentally in burst behavior.

---

### 2.5 Token Bucket ⭐ (The Production Standard)

```
   Tokens refill at rate 'r'
        │
        ▼
   ┌──────────┐  capacity = B (max tokens)
   │ ●●●●●●●● │  
   │ ●●●●○○○○ │  ← availableTokens at any given instant
   └────┬─────┘
        │
        ▼ Each request consumes 1 token
   [ALLOW] if tokens > 0
   [DENY]  if tokens == 0
```

**Why the Industry Converged on Token Bucket:**

1. **Burst Tolerance with Rate Enforcement:** It allows accumulated burst capacity (up to `B`) while enforcing an average rate of `r` tokens/second over time. This models real-world API usage patterns far better than Leaky Bucket.

2. **Constant Space:** `O(1)` — only two variables: `availableTokens` and `lastRefillTimestamp`.

3. **Lazy Evaluation Compatible:** No background thread/timer required. The refill is calculated mathematically on-demand — critical for systems managing millions of per-client buckets.

4. **Composable:** Multiple Token Buckets can be layered for **hierarchical rate limiting** (e.g., "10 req/sec AND 1,000 req/hour AND 50,000 req/day").

5. **Adopted By:** AWS API Gateway, Stripe, Shopify, Envoy Proxy, NGINX, Linux kernel (`tc` traffic control), Cisco IOS QoS.

---

### Algorithm Comparison Matrix

| Dimension | Fixed Window | Sliding Log | Sliding Counter | Leaky Bucket | Token Bucket |
|---|---|---|---|---|---|
| **Space Complexity** | O(1) | O(N) ⚠️ | O(1) | O(N) queue | O(1) |
| **Time Complexity** | O(1) | O(log N) | O(1) | O(1) | O(1) |
| **Boundary Burst** | ⚠️ 2× burst | ✅ None | ✅ ~None | ✅ None | ✅ None |
| **Burst Tolerance** | No | No | No | ❌ None | ✅ Yes (configurable) |
| **Memory at Scale** | Excellent | ⚠️ Terrible | Excellent | Moderate | Excellent |
| **Accuracy** | ~Low | Perfect | ~99.97% | Perfect | Excellent |
| **Use Case** | Simple analytics | Audit trails | Edge/CDN | Traffic shaping | ⭐ General purpose |

---

## 3. Production-Grade Java Implementation: Token Bucket with CAS Semantics

The naive `synchronized` approach introduces **mutex contention** under high concurrency — a throughput bottleneck in systems handling thousands of concurrent requests. Below is a **lock-free, CAS-based implementation** using `AtomicLong` for non-blocking thread safety.

### 3.1 Lock-Free Token Bucket (CAS Spin Loop)

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;

/**
 * A high-throughput, lock-free Token Bucket rate limiter using CAS
 * (Compare-And-Swap) semantics for non-blocking thread safety.
 *
 * <p>This implementation packs both the token count and the last-refill
 * timestamp into a single {@code AtomicLong} using bit manipulation,
 * enabling atomic updates to both fields in a single CAS operation
 * without requiring explicit locks or synchronized blocks.</p>
 *
 * <h3>Design Rationale — Why CAS over synchronized?</h3>
 * <ul>
 *   <li><b>No kernel-level context switching:</b> CAS operates entirely
 *       in user-space via CPU instructions (e.g., x86 CMPXCHG), avoiding
 *       the expensive thread parking/unparking that synchronized entails.</li>
 *   <li><b>No priority inversion:</b> Unlike mutex-based synchronization,
 *       CAS loops don't suffer from priority inversion scenarios where a
 *       low-priority thread holds a lock needed by a high-priority thread.</li>
 *   <li><b>Throughput under contention:</b> Under moderate contention (typical
 *       for rate limiters), CAS spin loops outperform locks. Under extreme
 *       contention, consider striped/partitioned buckets.</li>
 * </ul>
 *
 * <h3>Bit Packing Layout (64-bit long):</h3>
 * <pre>
 * [   32 bits: token count   |   32 bits: timestamp (seconds)   ]
 *  MSB ◄──────────────────────────────────────────────────────► LSB
 * </pre>
 *
 * @author Senior Architecture Team
 * @since 2024
 */
public class LockFreeTokenBucketRateLimiter {

    private static final int TIMESTAMP_BITS = 32;
    private static final long TIMESTAMP_MASK = 0xFFFFFFFFL;

    private final long maxTokens;
    private final long refillTokensPerPeriod;
    private final long refillPeriodNanos;

    /**
     * Packed state: upper 32 bits = token count, lower 32 bits = 
     * timestamp offset (seconds since limiter creation).
     *
     * Using a single AtomicLong ensures that token count and timestamp
     * are ALWAYS updated atomically — no torn reads/writes possible.
     */
    private final AtomicLong packedState;
    private final long creationNanoTime;

    public LockFreeTokenBucketRateLimiter(long maxTokens,
                                           long refillTokensPerPeriod,
                                           long period,
                                           TimeUnit unit) {
        this.maxTokens = maxTokens;
        this.refillTokensPerPeriod = refillTokensPerPeriod;
        this.refillPeriodNanos = unit.toNanos(period);
        this.creationNanoTime = System.nanoTime();

        long initialTimestamp = 0L; // offset from creation
        this.packedState = new AtomicLong(pack(maxTokens, initialTimestamp));
    }

    /**
     * Attempts to acquire the specified number of tokens using an
     * optimistic CAS spin loop.
     *
     * <p><b>Algorithmic Flow:</b></p>
     * <ol>
     *   <li>Read current packed state (snapshot).</li>
     *   <li>Unpack tokens and timestamp from snapshot.</li>
     *   <li>Calculate tokens to refill based on elapsed time.</li>
     *   <li>Compute new state (tokens after refill and consumption).</li>
     *   <li>CAS: attempt to swap old state with new state.</li>
     *   <li>If CAS fails (another thread modified state), RETRY from step 1.</li>
     * </ol>
     *
     * <p>This loop is guaranteed to terminate because at least one thread
     * always makes forward progress on each CAS attempt (lock-freedom
     * guarantee).</p>
     *
     * @param tokensRequested number of tokens to consume
     * @return true if tokens were acquired; false if rate limit exceeded
     */
    public boolean tryAcquire(int tokensRequested) {
        while (true) { // CAS spin loop — will retry on contention
            long currentState = packedState.get();
            long currentTokens = unpackTokens(currentState);
            long currentTimestamp = unpackTimestamp(currentState);

            // Calculate elapsed time since last refill
            long nowOffset = elapsedSecondsSinceCreation();
            long elapsedSeconds = nowOffset - currentTimestamp;

            // Refill tokens based on elapsed time
            long tokensToAdd = (elapsedSeconds * refillTokensPerPeriod * 
                                1_000_000_000L) / refillPeriodNanos;
            long newTokens = Math.min(maxTokens, currentTokens + tokensToAdd);

            // Advance timestamp only by consumed duration (preserve fractional time)
            long consumedSeconds = (tokensToAdd * refillPeriodNanos) / 
                                   (refillTokensPerPeriod * 1_000_000_000L);
            long newTimestamp = currentTimestamp + consumedSeconds;

            if (newTokens < tokensRequested) {
                return false; // ❌ Insufficient tokens — THROTTLE
            }

            // Consume tokens
            newTokens -= tokensRequested;
            long newState = pack(newTokens, newTimestamp);

            // Atomic CAS: only succeeds if no other thread modified
            // the state between our read and this write attempt
            if (packedState.compareAndSet(currentState, newState)) {
                return true; // ✅ Successfully acquired — ALLOW
            }

            // CAS failed — another thread got there first.
            // Loop back and retry with fresh state.
            // Thread.onSpinWait() is a JVM intrinsic (JEP 285) that
            // emits a PAUSE instruction on x86, reducing pipeline
            // speculation waste and power consumption during spin waits.
            Thread.onSpinWait();
        }
    }

    // ─── Bit Packing Utilities ───────────────────────────────────

    private static long pack(long tokens, long timestamp) {
        return (tokens << TIMESTAMP_BITS) | (timestamp & TIMESTAMP_MASK);
    }

    private static long unpackTokens(long packed) {
        return packed >>> TIMESTAMP_BITS;
    }

    private static long unpackTimestamp(long packed) {
        return packed & TIMESTAMP_MASK;
    }

    private long elapsedSecondsSinceCreation() {
        return (System.nanoTime() - creationNanoTime) / 1_000_000_000L;
    }
}
```

### 3.2 Architectural Notes on the CAS Implementation

| Design Element | Rationale |
|---|---|
| **Bit Packing into AtomicLong** | Eliminates the need for a separate lock to atomically update two correlated fields. Without this, you'd need either `synchronized` or a `ReentrantLock` to prevent **state tearing** (reading a new timestamp with an old token count). |
| **`Thread.onSpinWait()` (JEP 285)** | Emits the x86 `PAUSE` instruction, signaling to the CPU that this is a spin loop. This reduces **speculative execution waste**, lowers power consumption, and improves SMT (Symmetric Multi-Threading) performance on hyperthreaded cores. |
| **Monotonic Clock (`System.nanoTime()`)** | Immune to NTP clock adjustments, leap second insertions, and wall-clock drift. Critical for correctness in duration-based calculations. |
| **Lock-Freedom Guarantee** | The CAS loop provides **lock-freedom** (not wait-freedom): at least one thread is guaranteed to make progress in any concurrent execution. This is stronger than mutex-based approaches where thread starvation is theoretically possible under extreme contention. |

### 3.3 When CAS Becomes a Bottleneck — Striped Bucketing

Under **extreme contention** (e.g., 100,000+ concurrent threads competing for a single `AtomicLong`), CAS retries increase exponentially, causing **livelock-like behavior**. The architectural solution is **bucket striping**:

```java
/**
 * Striped rate limiter that distributes contention across N independent
 * buckets, analogous to ConcurrentHashMap's segment-based locking or
 * LongAdder's cell-based striping.
 *
 * Each thread is deterministically mapped to a stripe via thread ID hashing,
 * reducing CAS contention by a factor of N.
 *
 * TRADE-OFF: Accuracy decreases slightly because each stripe independently
 * tracks its own tokens. Effective global rate = N × per-stripe rate.
 * You must divide your target rate by N when configuring each stripe.
 */
public class StripedRateLimiter {

    private final LockFreeTokenBucketRateLimiter[] stripes;
    private final int stripeMask;

    public StripedRateLimiter(int stripeCount, long totalCapacity,
                               long totalRefillRate, long period, TimeUnit unit) {
        // Ensure stripe count is a power of 2 for efficient bitwise modulo
        stripeCount = Integer.highestOneBit(stripeCount - 1) << 1;
        this.stripeMask = stripeCount - 1;
        this.stripes = new LockFreeTokenBucketRateLimiter[stripeCount];

        long perStripeCapacity = totalCapacity / stripeCount;
        long perStripeRate = totalRefillRate / stripeCount;

        for (int i = 0; i < stripeCount; i++) {
            stripes[i] = new LockFreeTokenBucketRateLimiter(
                perStripeCapacity, perStripeRate, period, unit);
        }
    }

    public boolean tryAcquire(int tokens) {
        // Map current thread to a stripe using thread ID hashing
        // Fibonacci hashing provides better distribution than simple modulo
        int stripe = (int) (Thread.currentThread().getId() * 0x9E3779B97F4A7C15L 
                            >>> 48) & stripeMask;
        return stripes[stripe].tryAcquire(tokens);
    }
}
```

> **Architectural Analogy:** This is the same principle behind `java.util.concurrent.atomic.LongAdder` — which stripes additions across cells to reduce contention, at the cost of slightly delayed global reads.

---

## 4. Distributed Rate Limiting: Redis + Lua Script Atomicity

In a horizontally-scaled microservices architecture with N replicas behind a load balancer, local in-memory rate limiters allow **N × intended rate** — completely defeating the purpose. You must externalize state to a **shared, consistent data store**.

### 4.1 Why Redis?

| Property | Relevance |
|---|---|
| **Sub-millisecond latency** | Rate limiting is on the **hot path** of every request. Any solution adding >1ms latency per request is architecturally unacceptable. |
| **Lua script atomicity** | Redis executes Lua scripts atomically (single-threaded event loop). No race conditions between `GET → compute → SET` — equivalent to a distributed `synchronized` block. |
| **Native TTL** | Keys auto-expire, preventing memory leaks from inactive client buckets. |
| **Cluster-mode partitioning** | Redis Cluster hashes keys across slots, enabling horizontal scaling of the rate limiter state itself. |

### 4.2 Atomic Token Bucket via Redis Lua Script

```lua
-- rate_limiter.lua
-- Atomic Token Bucket implementation executed as a single Redis
-- transaction (EVAL). No race conditions possible because Redis
-- Lua scripts are executed serially on Redis's single-threaded
-- event loop.
--
-- KEYS[1] = rate limiter key (e.g., "rl:{client_id}:{endpoint}")
-- ARGV[1] = maxTokens (bucket capacity)
-- ARGV[2] = refillRate (tokens per second)
-- ARGV[3] = nowMicroseconds (server timestamp in microseconds)
-- ARGV[4] = requestedTokens (usually 1)
--
-- Returns: {allowed (0/1), remainingTokens, retryAfterMs}

local key = KEYS[1]
local maxTokens = tonumber(ARGV[1])
local refillRate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])
local requested = tonumber(ARGV[4])

-- Retrieve current bucket state (or initialize)
local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
local tokens = tonumber(bucket[1])
local lastRefill = tonumber(bucket[2])

-- Initialize bucket on first access
if tokens == nil then
    tokens = maxTokens
    lastRefill = now
end

-- Calculate token refill based on elapsed time
local elapsedMicros = math.max(0, now - lastRefill)
local tokensToAdd = (elapsedMicros * refillRate) / 1000000
tokens = math.min(maxTokens, tokens + tokensToAdd)

-- Advance timestamp (preserve fractional time for accuracy)
local consumedMicros = (tokensToAdd / refillRate) * 1000000
lastRefill = lastRefill + consumedMicros

-- Attempt to consume tokens
local allowed = 0
local retryAfterMs = 0

if tokens >= requested then
    tokens = tokens - requested
    allowed = 1
else
    -- Calculate how long the client must wait for sufficient tokens
    local deficit = requested - tokens
    retryAfterMs = math.ceil((deficit / refillRate) * 1000)
end

-- Persist updated state with TTL for automatic cleanup
-- TTL = 2 × window duration — ensures inactive buckets don't leak memory
redis.call('HMSET', key, 'tokens', tokens, 'last_refill', lastRefill)
redis.call('PEXPIRE', key, math.ceil((maxTokens / refillRate) * 2000))

return {allowed, math.floor(tokens), retryAfterMs}
```

### 4.3 Java Client Integration (Jedis + Lua)

```java
import redis.clients.jedis.JedisPooled;
import java.nio.file.Files;
import java.nio.file.Path;
import java.time.Instant;
import java.util.List;

/**
 * Distributed rate limiter backed by Redis.
 *
 * <p>Architecture: Each microservice instance calls this class, which
 * delegates to a centralized Redis cluster. The Lua script executes
 * atomically on Redis, providing linearizable consistency for the
 * token bucket state across all service instances.</p>
 *
 * <h3>Consistency Model:</h3>
 * <p>This implementation provides <b>linearizable</b> rate limiting
 * when using a single Redis master. When using Redis Cluster with
 * replicas, during a failover event there is a brief window where
 * the rate limit state may be lost (async replication lag). For
 * strict consistency, consider RedLock or Redis with WAIT command.</p>
 */
public class DistributedRateLimiter {

    private final JedisPooled jedis;
    private final String luaScriptSha;
    private final long maxTokens;
    private final long refillRate;

    public DistributedRateLimiter(JedisPooled jedis, long maxTokens,
                                   long refillRate) throws Exception {
        this.jedis = jedis;
        this.maxTokens = maxTokens;
        this.refillRate = refillRate;

        // Pre-load Lua script into Redis (EVALSHA is faster than EVAL)
        // EVALSHA uses the SHA1 hash to reference cached scripts,
        // avoiding retransmitting the script body on every call
        String script = Files.readString(Path.of("rate_limiter.lua"));
        this.luaScriptSha = jedis.scriptLoad(script);
    }

    /**
     * Attempts to acquire a permit for the given client + endpoint.
     *
     * @param clientId  Unique tenant/client identifier
     * @param endpoint  API endpoint (enables per-endpoint rate policies)
     * @return RateLimitResult containing the decision and metadata
     */
    public RateLimitResult tryAcquire(String clientId, String endpoint) {
        String key = String.format("rl:{%s}:%s", clientId, endpoint);

        // Use microsecond precision for accurate token calculations
        long nowMicros = Instant.now().toEpochMilli() * 1000;

        @SuppressWarnings("unchecked")
        List<Long> result = (List<Long>) jedis.evalsha(
            luaScriptSha,
            1,                              // number of KEYS
            key,                            // KEYS[1]
            String.valueOf(maxTokens),      // ARGV[1]
            String.valueOf(refillRate),     // ARGV[2]
            String.valueOf(nowMicros),      // ARGV[3]
            "1"                             // ARGV[4] = tokens requested
        );

        return new RateLimitResult(
            result.get(0) == 1,      // allowed
            result.get(1),            // remaining tokens
            result.get(2)             // retry-after (ms)
        );
    }

    public record RateLimitResult(
        boolean allowed,
        long remainingTokens,
        long retryAfterMs
    ) {
        /**
         * Populates standard HTTP rate limit response headers
         * as per IETF RFC 6585 and draft-ietf-httpapi-ratelimit-headers.
         */
        public void applyHeaders(jakarta.servlet.http.HttpServletResponse resp) {
            resp.setHeader("X-RateLimit-Remaining", 
                           String.valueOf(remainingTokens));
            if (!allowed) {
                resp.setStatus(429);
                resp.setHeader("Retry-After", 
                               String.valueOf((retryAfterMs + 999) / 1000));
            }
        }
    }
}
```

---

## 5. Hierarchical / Multi-Dimensional Rate Limiting

Production APIs rarely have a single rate limit. Real-world policies are **multi-dimensional**:

```
┌─────────────────────────────────────────────────────────────┐
│                    Rate Limit Policy Stack                   │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Global System Protection                          │
│           → 1,000,000 req/min across ALL tenants            │
│                                                             │
│  Layer 2: Per-Tenant Quota (SLA-driven)                     │
│           → Free tier:    100 req/min                        │
│           → Premium tier: 10,000 req/min                    │
│           → Enterprise:   100,000 req/min                   │
│                                                             │
│  Layer 3: Per-Endpoint Throttle (Resource-weighted)         │
│           → GET  /api/users:     1,000 req/min              │
│           → POST /api/reports:   50 req/min (expensive)     │
│           → POST /api/export:    5 req/min  (very expensive)│
│                                                             │
│  Layer 4: Per-User Rate Limit (Abuse Prevention)            │
│           → 20 req/sec per individual user session          │
│                                                             │
│  Layer 5: Concurrent Request Limiter (Semaphore)            │
│           → Max 5 in-flight requests per user               │
└─────────────────────────────────────────────────────────────┘

Request must pass ALL layers to be admitted.
```

### Java Implementation — Composite Rate Limiter (Chain of Responsibility)

```java
import java.util.List;

/**
 * Composite rate limiter implementing the Chain of Responsibility pattern.
 * A request must pass ALL constituent limiters to be admitted.
 *
 * <p>This models the hierarchical policy stack where each layer
 * represents a different dimension of rate limiting (global, tenant,
 * endpoint, user, concurrency).</p>
 *
 * <p><b>Ordering Semantics:</b> Limiters are evaluated in insertion order.
 * Place the cheapest/most-likely-to-reject limiter first (fail-fast)
 * to minimize unnecessary downstream evaluations.</p>
 */
public class CompositeRateLimiter {

    @FunctionalInterface
    public interface RateLimiter {
        RateLimitDecision tryAcquire(RequestContext ctx);
    }

    public record RateLimitDecision(
        boolean allowed,
        String rejectedByLayer,
        long retryAfterMs
    ) {
        public static RateLimitDecision allow() {
            return new RateLimitDecision(true, null, 0);
        }

        public static RateLimitDecision deny(String layer, long retryMs) {
            return new RateLimitDecision(false, layer, retryMs);
        }
    }

    public record RequestContext(
        String tenantId,
        String userId,
        String endpoint,
        String tier        // "free", "premium", "enterprise"
    ) {}

    private final List<RateLimiter> limiterChain;

    public CompositeRateLimiter(List<RateLimiter> limiterChain) {
        this.limiterChain = List.copyOf(limiterChain); // Immutable defensive copy
    }

    /**
     * Evaluates the request against every limiter in the chain.
     * Short-circuits on the first rejection (fail-fast semantics).
     *
     * IMPORTANT: On rejection, we must NOT roll back tokens consumed
     * by earlier (passed) limiters. Rolling back would create a subtle
     * vulnerability where an attacker could probe rate limits without
     * cost by crafting requests that always fail at a later layer.
     */
    public RateLimitDecision evaluate(RequestContext ctx) {
        for (RateLimiter limiter : limiterChain) {
            RateLimitDecision decision = limiter.tryAcquire(ctx);
            if (!decision.allowed()) {
                return decision; // Short-circuit — request denied
            }
        }
        return RateLimitDecision.allow();
    }
}
```

---

## 6. Integration Patterns in Enterprise Architecture

### 6.1 Optimal Placement in the Request Pipeline

```
                        ┌──────────────────────────────────────┐
                        │        REQUEST FLOW TOPOLOGY          │
                        └──────────────────────────────────────┘

 Client                                                          Backend
   │                                                               │
   │  ① CDN/WAF (Cloudflare, AWS Shield)                          │
   │     └─ L3/L4 DDoS mitigation (volumetric attacks)            │
   │     └─ IP reputation scoring                                  │
   │     └─ Coarse rate limiting (IP-level)                        │
   │                                                               │
   │  ② API Gateway (Kong, Envoy, AWS API Gateway)                │
   │     └─ ⭐ PRIMARY RATE LIMITING ENFORCEMENT POINT             │
   │     └─ Authentication / JWT validation                        │
   │     └─ Per-tenant, per-endpoint, per-tier rate limits         │
   │     └─ Request transformation & routing                       │
   │                                                               │
   │  ③ Service Mesh Sidecar (Istio/Envoy)                        │
   │     └─ Inter-service (east-west) rate limiting                │
   │     └─ Circuit breaking (complementary to rate limiting)      │
   │     └─ mTLS termination                                       │
   │                                                               │
   │  ④ Application Layer (Spring Boot / Quarkus)                  │
   │     └─ Business-rule rate limiting                             │
   │     └─ e.g., "Max 3 password reset attempts per hour"         │
   │     └─ Fine-grained, context-aware throttling                 │
   │                                                               │
   ▼                                                               ▼
```

### 6.2 Spring Boot Integration (Servlet Filter)

```java
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import org.springframework.stereotype.Component;
import org.springframework.core.annotation.Order;
import java.io.IOException;

/**
 * Rate limiting servlet filter.
 * 
 * <p><b>Filter ordering:</b> This filter MUST execute AFTER authentication
 * (to have access to tenant/user identity) but BEFORE any business logic
 * (to prevent resource waste on rejected requests).</p>
 *
 * <p><b>@Order(2):</b> Assumes authentication filter is @Order(1).</p>
 */
@Component
@Order(2)
public class RateLimitFilter implements Filter {

    private final DistributedRateLimiter rateLimiter;

    public RateLimitFilter(DistributedRateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpReq = (HttpServletRequest) request;
        HttpServletResponse httpResp = (HttpServletResponse) response;

        // Extract identity from authenticated security context
        String clientId = extractClientId(httpReq);
        String endpoint = httpReq.getMethod() + ":" + httpReq.getRequestURI();

        var result = rateLimiter.tryAcquire(clientId, endpoint);

        // Always set informational headers (even on success)
        // Per IETF draft-ietf-httpapi-ratelimit-headers
        httpResp.setHeader("RateLimit-Remaining",
                           String.valueOf(result.remainingTokens()));
        httpResp.setHeader("RateLimit-Policy",
                           "100;w=60");  // 100 requests per 60-second window

        if (!result.allowed()) {
            httpResp.setStatus(429);
            httpResp.setHeader("Retry-After",
                               String.valueOf((result.retryAfterMs() + 999) / 1000));
            httpResp.setContentType("application/problem+json");
            httpResp.getWriter().write("""
                {
                    "type": "https://api.example.com/errors/rate-limit-exceeded",
                    "title": "Rate Limit Exceeded",
                    "status": 429,
                    "detail": "You have exceeded the request rate limit. Please retry after %d seconds.",
                    "retryAfterMs": %d
                }
                """.formatted(
                    (result.retryAfterMs() + 999) / 1000,
                    result.retryAfterMs()
                ));
            return; // DO NOT continue the filter chain
        }

        chain.doFilter(request, response); // ✅ Proceed to next filter/controller
    }

    private String extractClientId(HttpServletRequest req) {
        // In production: extract from JWT claims, API key header, or 
        // Spring Security's SecurityContextHolder
        String apiKey = req.getHeader("X-API-Key");
        return apiKey != null ? apiKey : req.getRemoteAddr(); // Fallback to IP
    }
}
```

---

## 7. Observability & Operational Excellence

A rate limiter you can't observe is a rate limiter you can't trust. Instrument everything.

### 7.1 Essential Metrics (Micrometer/Prometheus)

```java
import io.micrometer.core.instrument.*;

/**
 * Observable rate limiter wrapper that emits Prometheus-compatible
 * metrics for every rate limiting decision.
 */
public class ObservableRateLimiter {

    private final DistributedRateLimiter delegate;
    private final Counter allowedCounter;
    private final Counter throttledCounter;
    private final Timer latencyTimer;
    private final DistributionSummary remainingTokensSummary;

    public ObservableRateLimiter(DistributedRateLimiter delegate,
                                  MeterRegistry registry) {
        this.delegate = delegate;

        this.allowedCounter = Counter.builder("rate_limiter.decisions")
            .tag("result", "allowed")
            .description("Number of requests allowed through the rate limiter")
            .register(registry);

        this.throttledCounter = Counter.builder("rate_limiter.decisions")
            .tag("result", "throttled")
            .description("Number of requests rejected by the rate limiter")
            .register(registry);

        this.latencyTimer = Timer.builder("rate_limiter.evaluation.duration")
            .description("Time taken to evaluate rate limit decision")
            .publishPercentiles(0.5, 0.95, 0.99, 0.999)
            .register(registry);

        this.remainingTokensSummary = DistributionSummary
            .builder("rate_limiter.remaining_tokens")
            .description("Distribution of remaining tokens after each decision")
            .register(registry);
    }

    public DistributedRateLimiter.RateLimitResult tryAcquire(
            String clientId, String endpoint) {
        return latencyTimer.record(() -> {
            var result = delegate.tryAcquire(clientId, endpoint);

            if (result.allowed()) {
                allowedCounter.increment();
            } else {
                throttledCounter.increment();
            }

            remainingTokensSummary.record(result.remainingTokens());
            return result;
        });
    }
}
```

### 7.2 Critical Alerting Rules (Prometheus/Grafana)

```yaml
# prometheus-alerts.yml
groups:
  - name: rate_limiter_alerts
    rules:
      # ALERT: High throttle rate indicates either an attack or
      # organic growth that requires capacity expansion
      - alert: HighThrottleRate
        expr: |
          rate(rate_limiter_decisions_total{result="throttled"}[5m]) /
          rate(rate_limiter_decisions_total[5m]) > 0.15
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Rate limiter throttling >15% of requests"
          runbook: "https://wiki.internal/runbooks/rate-limiter-throttle"

      # ALERT: Rate limiter latency spike could indicate Redis
      # connectivity issues or Lua script performance degradation
      - alert: RateLimiterLatencySpike
        expr: |
          histogram_quantile(0.99, 
            rate(rate_limiter_evaluation_duration_seconds_bucket[5m])
          ) > 0.005
        for: 3m
        labels:
          severity: critical
        annotations:
          summary: "Rate limiter p99 latency exceeds 5ms"
```

---

## 8. Failure Mode Analysis & Resilience Patterns

### What happens when Redis is down?

This is the most critical architectural decision. You have two options:

| Strategy | Behavior When Redis Fails | Trade-off |
|---|---|---|
| **Fail-Open** | Allow all requests through (disable rate limiting) | Prioritizes **availability** over protection. Risk: system overload during outage. |
| **Fail-Closed** | Reject all requests (return `503`) | Prioritizes **protection** over availability. Risk: total service outage. |
| **Fail-Open with Local Fallback** ⭐ | Fall back to in-memory rate limiter per instance | Best compromise — provides degraded-but-present protection. Rate limits become N× less accurate (N = instance count). |

### Recommended: Circuit-Breaker-Wrapped Fallback

```java
/**
 * Resilient rate limiter with automatic fallback from distributed
 * (Redis) to local (in-memory) mode when Redis becomes unavailable.
 *
 * Uses Resilience4j's CircuitBreaker to detect Redis failures and
 * switch to the local fallback without manual intervention.
 */
public class ResilientRateLimiter {

    private final DistributedRateLimiter primary;      // Redis-backed
    private final LockFreeTokenBucketRateLimiter fallback; // Local JVM
    private final CircuitBreaker circuitBreaker;

    public boolean tryAcquire(String clientId, String endpoint) {
        try {
            return circuitBreaker.executeSupplier(() ->
                primary.tryAcquire(clientId, endpoint).allowed()
            );
        } catch (Exception e) {
            // Circuit is OPEN — Redis is unavailable
            // Fall back to local limiter (degraded accuracy, but still protected)
            log.warn("Rate limiter falling back to local mode: {}", 
                     e.getMessage());
            return fallback.tryAcquire(1);
        }
    }
}
```

---

## 9. Architectural Decision Record (ADR) Summary

| Decision | Choice | Rationale |
|---|---|---|
| **Algorithm** | Token Bucket | O(1) space, burst tolerance, lazy refill, industry standard |
| **Concurrency Model** | CAS (lock-free) for local; Lua atomicity for distributed | Eliminates mutex contention; provides linearizable distributed consistency |
| **State Store** | Redis with Lua scripts | Sub-ms latency, atomic operations, native TTL, cluster-mode scalable |
| **Clock Source** | `System.nanoTime()` (local), server-provided microsecond timestamp (distributed) | Monotonic, immune to NTP drift |
| **Failure Mode** | Fail-open with local fallback via circuit breaker | Balances availability with degraded protection |
| **Placement** | API Gateway (primary) + Application layer (business rules) | Reject traffic as early as possible in the pipeline |
| **Observability** | Prometheus counters, timers, distribution summaries | Enables capacity planning, anomaly detection, SLA reporting |
| **Response Format** | RFC 6585 (429) + IETF draft ratelimit headers + RFC 7807 problem JSON | Standards-compliant, client-friendly, machine-parseable |

---

## 10. Final Thought: Rate Limiting as an Architectural Primitive

Rate limiting is not a "nice-to-have" bolt-on — it is a **foundational architectural primitive** on par with authentication, encryption, and circuit breaking. It sits at the intersection of **reliability engineering**, **security architecture**, **capacity management**, and **business policy enforcement**.

The best rate limiter is one that:
- You never have to think about when it's working correctly
- Screams loudly (via observability) when it starts rejecting more than expected
- Degrades gracefully when its own dependencies fail
- Is configured declaratively, not buried in application code

> *"The goal of rate limiting is not to reject requests — it's to protect the requests you do accept."*
