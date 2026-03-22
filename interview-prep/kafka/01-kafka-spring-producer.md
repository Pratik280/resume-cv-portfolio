# 🚀 Kafka in Spring Boot — End-to-End Deep Dive (Order System)

---

# 🧠 1. Problem Statement (Real-World Context)

We have an **Order Service**:

```text
POST /orders → create order
```

Initially:

```text
Client → Controller → Service → DB Save
```

Now we want:

```text
Client → Order Service → DB Save → Kafka Event → Other Services
```

---

# ⚙️ 2. Kafka Setup (KRaft)

* Run Kafka locally (no Zookeeper)
* Create topic: `order-topic`
* Start console consumer to verify messages

---

# 🔥 3. Spring Boot Kafka Setup

## Dependency

```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
</dependency>
```

---

## Configuration

```properties
spring.kafka.bootstrap-servers=localhost:9092

spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

---

# 🔹 4. Serializer Explanation (VERY IMPORTANT)

---

## 🔑 Key Serializer

```properties
key-serializer=StringSerializer
```

👉 Converts:

```text
Java String → byte[]
```

Example:

```java
"123" → [49, 50, 51]
```

✔ Used for:

* Partitioning
* Ordering

---

## 📦 Value Serializer

```properties
value-serializer=JsonSerializer
```

👉 Converts:

```text
OrderResponse (Java Object)
→ JSON
→ byte[]
```

Example:

```json
{
  "id": 1,
  "productName": "Mobile"
}
```

---

## 🔥 Why Serialization is Needed

Kafka only understands:

```text
byte[]
```

So:

```text
Java Object → Serializer → byte[] → Kafka
```

---

# 🧩 5. Kafka Producer (KafkaTemplate)

```java
@Service
public class OrderProducer {

    private final KafkaTemplate<String, OrderResponse> kafkaTemplate;

    public OrderProducer(KafkaTemplate<String, OrderResponse> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendOrder(OrderResponse orderResponse) {

        CompletableFuture<SendResult<String, OrderResponse>> future =
                kafkaTemplate.send(
                        "order-topic",
                        String.valueOf(orderResponse.getId()),
                        orderResponse
                );

        // 🔥 Attach callback
        future.whenComplete((result, ex) -> {
            if (ex != null) {
                log.error("Failed to send order {}", orderResponse.getId(), ex);
            } else {
                log.info("Order {} sent to partition {}, offset {}",
                        orderResponse.getId(),
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
            }
        });
    }
}
```

---

# 🔥 6. What happens inside `send()`

```text
Controller → KafkaTemplate
        ↓
ProducerRecord created
        ↓
Serialization (Object → JSON → bytes)
        ↓
Partition selection (using key)
        ↓
Batching (RecordAccumulator)
        ↓
Sender Thread
        ↓
Kafka Broker
        ↓
Partition Log (Disk)
```

---

# ⚡ 7. Async Behavior

```java
kafkaTemplate.send(...)
```

✔ Non-blocking
✔ Returns immediately

👉 Actual send happens in background

---

# 🔄 8. CompletableFuture (Callback Handling)

---

## Why it exists?

Kafka is async → result comes later

---

## How we use it:

```java
future.whenComplete((result, ex) -> { ... });
```

---

## Two Scenarios

### ✅ Success

```java
result.getRecordMetadata()
```

Gives:

* partition
* offset

---

### ❌ Failure

```java
ex != null
```

Examples:

* broker down
* timeout
* serialization error

---

# 💣 9. Dual Write Problem

```text
DB Save → SUCCESS
Kafka Send → FAIL
```

---

## Result:

```text
Data inconsistency
```

---

# 🔥 10. Outbox Pattern Solution

---

## Instead of:

```text
Save Order → Send Kafka ❌
```

## Do:

```text
Save Order + Save Event (same DB transaction) ✅
→ Later publish to Kafka
```

---

# 🗄️ 11. Outbox Table

```sql
CREATE TABLE outbox (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    payload TEXT,
    status VARCHAR(20)
);
```

---

# 🧩 12. Save Order + Event

```java
@Transactional
public OrderResponse createOrder(OrderRequest request) {

    Order order = orderRepository.save(mapToEntity(request));

    OutboxEvent event = new OutboxEvent();
    event.setPayload(convertToJson(order));
    event.setStatus("NEW");

    outboxRepository.save(event);

    return mapToResponse(order);
}
```

---

# 🔁 13. Scheduler + Kafka Publish

```java
@Scheduled(fixedDelay = 5000)
@Transactional
public void publish() {

    List<OutboxEvent> events = outboxRepository.fetchBatchForUpdate(10);

    for (OutboxEvent event : events) {
        try {
            kafkaTemplate.send("order-topic", event.getPayload());
            event.setStatus("SENT");
        } catch (Exception e) {
            log.error("Kafka failed", e);
        }
    }
}
```

---

# 🔒 14. Multi-Instance Handling (DB Lock)

```sql
SELECT * FROM outbox
WHERE status = 'NEW'
FOR UPDATE SKIP LOCKED
LIMIT 10;
```

---

## What happens?

```text
Instance A → locks row
Instance B → skips locked row
```

✔ No duplicate processing

---

# 🔁 15. Status Flow

```text
NEW → PROCESSING → SENT
```

---

# 🔥 16. Final Architecture

```text
Client
  ↓
Order Service
  ↓
DB (Order + Outbox) ✅
  ↓
Scheduler
  ↓
Kafka Producer
  ↓
Kafka Topic
  ↓
Consumers
```

---

# 🎯 17. Key Takeaways

* Kafka is async (CompletableFuture used)
* Serialization converts objects → bytes
* Keys ensure ordering
* Dual write problem is critical
* Outbox pattern ensures reliability
* DB locking enables scaling
* Status prevents infinite retries

---

# 🧠 18. Interview Answer (Refined)

> In our order system, we use KafkaTemplate to publish events asynchronously, which returns a CompletableFuture allowing us to attach callbacks for success and failure handling. We configure serializers to convert keys and payloads into byte arrays, as Kafka only works with bytes. To handle the dual write problem, we implemented the outbox pattern, storing events in the database within the same transaction as the order. A scheduled job reads and publishes these events using database locking to ensure only one instance processes each event, achieving reliability and scalability.

---
