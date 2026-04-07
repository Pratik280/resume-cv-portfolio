Below are **complete, interview-ready technical notes** in a **story + implementation format**. You can revise this directly before interviews.

---

# 🏦 BFSI Microservices Design: Config Fetch + Cache + Kafka Eviction + Outbox

---

# 🧩 Problem Statement

We have two microservices:

### 🔹 Service A — Config Management

* Stores:

  * Merchant config
  * BIN data
  * Feature flags
* Source of truth
* Exposes REST APIs
* Publishes config change events

---

### 🔹 Service B — Middleware

* Receives request from frontend
* Validates:

  * Merchant
  * BIN
* Calls external processors (Lyra, TPSL)
* Needs config from A

---

# 🎯 Goal

Design a system where:

* B fetches config from A efficiently
* Avoids latency (using cache)
* Avoids stale data (using Kafka events)
* Handles failures (using TTL + Outbox)

---

# 🧠 High-Level Design

```text
Frontend → B (Middleware)

B:
  → Redis Cache (@Cacheable)
  → Feign Client → A (Config Service)
  → Kafka Consumer → Cache Eviction

A:
  → DB (source of truth)
  → Outbox Table
  → Kafka Producer
```

---

# 🔁 Flow Overview

## ✅ Normal Flow

```text
Request → B
→ Check cache
   → HIT → return
   → MISS → call A → cache → return
```

---

## 🔄 Config Update Flow

```text
A updates DB
→ Writes to Outbox
→ Kafka publishes event
→ B consumes event
→ Cache evicted
→ Next request gets fresh data
```

---

# 🏗️ Implementation in Service B (Middleware)

---

# ✅ 1. Feign Client (Call Service A)

```java
@FeignClient(name = "config-service")
public interface ConfigClient {

    @GetMapping("/config/merchant/{id}")
    MerchantConfig getMerchantConfig(@PathVariable("id") String id);
}
```

---

# ✅ 2. Config Service Layer (Caching Logic)

```java
@Service
public class ConfigService {

    @Autowired
    private ConfigClient configClient;

    @Cacheable(value = "merchantConfig", key = "#merchantId")
    public MerchantConfig getMerchantConfig(String merchantId) {
        return configClient.getMerchantConfig(merchantId);
    }
}
```

---

# ✅ 3. Transaction Flow in B

```java
@Service
public class TransactionService {

    @Autowired
    private ConfigService configService;

    public void processTransaction(String merchantId) {

        MerchantConfig config =
                configService.getMerchantConfig(merchantId);

        // Validate merchant + bin
        // Process transaction
    }
}
```

---

# 🔁 Cache Behavior

```text
Case 1: Cache HIT
→ Fast response (no call to A)

Case 2: Cache MISS
→ Feign call to A
→ Store in Redis
→ Return response
```

---

# ⚙️ Enable Caching

```java
@EnableCaching
@SpringBootApplication
public class Application {
}
```

---

# ⚙️ Redis Configuration

```yaml
spring:
  cache:
    type: redis
```

---

# 🔥 Kafka-Based Cache Eviction (Core Concept)

---

# ✅ 4. Event Model

```java
public class MerchantEvent {
    private String eventType;
    private String merchantId;

    // getters/setters
}
```

---

# 🏗️ Implementation in Service A (Producer)

---

# ✅ 5. Kafka Producer Config

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

---

# ✅ 6. Publish Event on Update

```java
@Service
public class MerchantService {

    @Autowired
    private KafkaTemplate<String, MerchantEvent> kafkaTemplate;

    public void disableMerchant(String merchantId) {

        // 1. Update DB
        merchantRepository.disable(merchantId);

        // 2. Publish event
        MerchantEvent event = new MerchantEvent();
        event.setEventType("MERCHANT_DISABLED");
        event.setMerchantId(merchantId);

        kafkaTemplate.send("merchant-config-events", merchantId, event);
    }
}
```

---

# ⚠️ Problem Here

```text
DB updated ✅
Kafka failed ❌
→ Inconsistency
```

---

# 🚀 Solution: Transactional Outbox Pattern

---

# ✅ 7. Outbox Table

```sql
CREATE TABLE outbox_events (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    event_type VARCHAR(50),
    payload JSON/TEXT,
    status VARCHAR(20),
    created_at TIMESTAMP
);
```

---

# ✅ 8. Save Event in Same Transaction

```java
@Transactional
public void disableMerchant(String merchantId) {

    // Update DB
    merchantRepository.disable(merchantId);

    // Insert into outbox
    OutboxEvent event = new OutboxEvent();
    event.setEventType("MERCHANT_DISABLED");
    event.setPayload("{\"merchantId\":\"" + merchantId + "\"}");
    event.setStatus("NEW");

    outboxRepository.save(event);
}
```

---

# ✅ 9. Background Publisher

```java
@Scheduled(fixedDelay = 5000)
public void publishEvents() {

    // List<OutboxEvent> events = outboxRepository.findByStatus("NEW");
    List<OutboxEvent> events = outboxRepository.fetchBatchForUpdate(10);

    for (OutboxEvent event : events) {

        kafkaTemplate.send("merchant-config-events", event.getPayload())
            .whenComplete((result, ex) -> {

                OutboxEvent updatedEvent = outboxRepository
                        .findById(event.getId())
                        .orElseThrow();

                if (ex == null) {
                    updatedEvent.setStatus("SENT");
                } else {
                    updatedEvent.setStatus("FAILED");
                }

                outboxRepository.save(updatedEvent);
            });
    }
}
```

```java
@Query(value = """
    SELECT * FROM outbox_event
    WHERE status = 'NEW'
    ORDER BY id
    LIMIT :limit
    FOR UPDATE SKIP LOCKED
""", nativeQuery = true)
List<OutboxEvent> fetchBatchForUpdate(@Param("limit") int limit);
```

---

# 🏗️ Implementation in Service B (Consumer)

---

# ✅ 10. Kafka Consumer Config

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: middleware-group
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
```

---

# ✅ 11. Kafka Listener (Evict Cache)

```java
@Service
public class MerchantEventListener {

    @Autowired
    private CacheManager cacheManager;

    @KafkaListener(topics = "merchant-config-events")
    public void handleMerchantUpdate(MerchantEvent event) {

        Cache cache = cacheManager.getCache("merchantConfig");

        if (cache != null) {
            cache.evict(event.getMerchantId());
        }
    }
}
```

---

# ❗ Why Manual Eviction?

* `@CacheEvict` may not work reliably inside Kafka listener
* Direct control using `CacheManager` is safer

---

# 🔁 Final End-to-End Flow

---

## 🟢 Scenario 1: First Request

```text
→ Cache MISS
→ Call A via Feign
→ Store in Redis
→ Process txn
```

---

## 🟢 Scenario 2: Cached Request

```text
→ Cache HIT
→ Fast response
```

---

## 🔴 Scenario 3: Config Update

```text
A updates DB
→ Outbox insert
→ Kafka publish
→ B consumes event
→ Cache evicted
```

---

## 🟢 Scenario 4: Next Request After Update

```text
→ Cache MISS
→ Fetch fresh config
→ No stale data
```

---

# ⚠️ Important Edge Cases

---

## ❓ New Merchant Added

```text
Cache MISS → Call A → Works fine
```

👉 TTL does NOT affect new entries

---

## ❓ Stale Cache Problem

```text
Merchant disabled in A ❌
Cache still says enabled in B ❌
```

👉 Solved by Kafka eviction

---

## ❓ Kafka Failure

Solution:

* Retry
* DLQ (Dead Letter Queue)
* TTL fallback

---

## ❓ Cache Stampede

Solution:

* Locking
* Staggered TTL
* Refresh ahead

---

# ⏱️ TTL Strategy

| Data Type       | TTL                     |
| --------------- | ----------------------- |
| Merchant config | 5–15 min                |
| BIN data        | 10–30 min               |
| Critical flags  | Very low / event-driven |

---

# 🧠 Key Design Patterns Used

---

## ✅ Cache-Aside Pattern

* Application manages cache
* On miss → fetch + store

---

## ✅ Event-Driven Architecture

* Kafka used for async updates

---

## ✅ Transactional Outbox

* Ensures DB + event consistency

---

## ✅ Idempotent Consumer

* Safe for duplicate events

---

# 💥 Interview-Ready Final Answer

> “Middleware service fetches configuration from config service using Feign client and caches it in Redis using @Cacheable. To maintain consistency, config service publishes Kafka events using the transactional outbox pattern whenever data changes. Middleware consumes these events and evicts the relevant cache entries using CacheManager. TTL acts as a fallback mechanism.”

---

# 🚀 Bonus (Senior-Level Add-on)

> “For critical payment flows, we may also perform final synchronous validation with config service to avoid stale decisions.”

---

# 🔚 Final Summary

| Component      | Role                               |
| -------------- | ---------------------------------- |
| A (Config)     | Source of truth + publishes events |
| Outbox         | Ensures reliability                |
| Kafka          | Event transport                    |
| B (Middleware) | Fetch + cache + evict              |
| Redis          | Distributed cache                  |

---

If you want next step:
👉 I can turn this into a **perfect diagram + whiteboard explanation script (exactly how to speak in interview)**
