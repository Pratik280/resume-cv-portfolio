
* Actuator (A/B/C)
* DB health (Postgres)
* Kafka metrics (lag)
* Transaction stats (Kafka → Redis)

---

# 🏦 Monitoring System (Service D) — Complete Notes

---

# 🧩 Objective

Build a monitoring system that:

* Tracks **service health (A, B, C)**
* Tracks **DB (Postgres) health**
* Tracks **Kafka metrics (lag)**
* Tracks **transaction metrics (success/failure)**
* Exposes a **single API for dashboard**

---

# 🏗️ Architecture

```text
A/B/C Services (multiple instances)
    → /actuator/health
    → Kafka (transaction + monitoring)

                ↓
        Monitoring Service (D)
                ↓
             Redis (cache)
                ↓
         Aggregated API (/dashboard)
                ↓
             Dashboard UI
```

---

# 🧠 Key Design Principles

* ❌ Do NOT call services on every API request
* ✅ Periodically collect data (scheduler)
* ✅ Store in Redis
* ✅ Serve fast via API

---

# 🧩 Data Sources

---

## 1. Service Health (Actuator)

```text
GET /actuator/health
```

Provides:

* Service status
* DB status
* Kafka status
* Redis status

---

## 2. Kafka Metrics

* Consumer lag
* Topic partitions
* Offsets

---

## 3. Transaction Data

* Comes via Kafka (`monitoring-topic`)
* Aggregated in Redis

---

## 4. DB Health

* Extracted from actuator response (`components.db.status`)

---

# 🧩 Redis Data Model

---

## 🔹 Service Health

```text
health:service-a → {instance1: UP, instance2: DOWN}
health:service-b → {instance1: UP}
```

---

## 🔹 DB Health

```text
db:service-a → UP
db:service-b → DOWN
```

---

## 🔹 Kafka Lag

```text
kafka:lag → {
  transaction-topic:0 → 10,
  transaction-topic:1 → 5
}
```

---

## 🔹 Transaction Metrics

```text
txn:total → 1000
txn:success → 900
txn:failed → 100
```

---

## 🔹 Time-Based Metrics

```text
txn:minute:YYYYMMDDHHmm → count
txn:hour:YYYYMMDDHH → count
txn:hour:failed:YYYYMMDDHH → count
```

---

# 🧩 Scheduler (Central Collector)

---

## Runs periodically

```java
@Scheduled(fixedDelay = 10000) // every 10 seconds
public void collectAllMetrics() {

    fetchServiceHealth();
    fetchKafkaLag();
    fetchDbHealth();
}
```

---

# 🧩 Fetch Service Health (Actuator Polling)

---

```java
public void fetchServiceHealth() {

    List<String> instances = serviceDiscovery.getInstances("service-b");

    for (String instance : instances) {

        Map<String, Object> health =
            restTemplate.getForObject(instance + "/actuator/health", Map.class);

        String status = (String) health.get("status");

        redisTemplate.opsForHash()
            .put("health:service-b", instance, status);
    }
}
```

---

# 🧩 Extract DB Health

---

```java
Map<String, Object> components =
    (Map<String, Object>) health.get("components");

Map<String, Object> db =
    (Map<String, Object>) components.get("db");

String dbStatus = (String) db.get("status");

redisTemplate.opsForValue().set("db:service-b", dbStatus);
```

---

# 🧩 Kafka Lag Collection

---

```java
public void fetchKafkaLag() {

    Map<String, Long> lagMap = kafkaMonitoringService.getLag();

    for (Map.Entry<String, Long> entry : lagMap.entrySet()) {

        redisTemplate.opsForHash().put(
            "kafka:lag",
            entry.getKey(),
            entry.getValue()
        );
    }
}
```

---

# 🧩 Transaction Data (Kafka Consumer)

---

```java
@KafkaListener(topics = "monitoring-topic")
public void consume(TransactionEvent event) {

    // total
    redisTemplate.opsForValue().increment("txn:total");

    // success/failure
    if ("SUCCESS".equals(event.getStatus())) {
        redisTemplate.opsForValue().increment("txn:success");
    } else {
        redisTemplate.opsForValue().increment("txn:failed");
    }
}
```

---

# 🧩 Aggregated Dashboard API

---

```java
@GetMapping("/dashboard")
public Map<String, Object> getDashboard() {

    Map<String, Object> response = new HashMap<>();

    // Service health
    response.put("services",
        redisTemplate.opsForHash().entries("health:service-b"));

    // DB health
    response.put("db", Map.of(
        "A", redisTemplate.opsForValue().get("db:service-a"),
        "B", redisTemplate.opsForValue().get("db:service-b"),
        "C", redisTemplate.opsForValue().get("db:service-c")
    ));

    // Kafka lag
    response.put("kafkaLag",
        redisTemplate.opsForHash().entries("kafka:lag"));

    // Transaction stats
    response.put("transactions", Map.of(
        "total", redisTemplate.opsForValue().get("txn:total"),
        "success", redisTemplate.opsForValue().get("txn:success"),
        "failed", redisTemplate.opsForValue().get("txn:failed")
    ));

    return response;
}
```

---

# 🔁 End-to-End Flow

---

```text
1. Scheduler runs every 10 sec
   → calls actuator of A/B/C
   → fetches Kafka lag
   → stores in Redis

2. Kafka events arrive
   → update transaction metrics in Redis

3. Dashboard calls API
   → reads Redis
   → returns aggregated result
```

---

# 🧠 Why This Design Works

| Concern            | Solution                |
| ------------------ | ----------------------- |
| High load          | scheduler-based polling |
| Slow APIs          | Redis caching           |
| Real-time updates  | Kafka                   |
| Scalability        | decoupled services      |
| Multiple instances | service discovery       |

---

# ⚠️ Production Considerations

---

## ✅ Polling Frequency

```text
Health → 10–30 sec
Kafka lag → 5–15 sec
```

---

## ✅ Error Handling

```java
try {
   call actuator
} catch (Exception e) {
   mark service DOWN
}
```

---

## ✅ TTL (Auto Cleanup)

```java
redisTemplate.expire("health:service-b", Duration.ofSeconds(30));
```

---

## ✅ Security

* Do not expose `/actuator/*` publicly
* Use API gateway / auth

---

# 💥 Interview-Ready Summary

> “We built a monitoring service that periodically polls actuator endpoints of all microservices to collect health and database status, fetches Kafka lag metrics, and consumes transaction events in real time. All data is stored in Redis, and a single aggregated API provides fast and scalable dashboard responses.”

---

# 🔥 Killer Line

> **“We separate data collection from data serving using scheduled aggregation and Redis caching.”**

---

# 🔚 Final Mental Model

```text
Collectors (Scheduler + Kafka)
        ↓
     Redis Cache
        ↓
   Aggregated API
        ↓
     Dashboard
```

---