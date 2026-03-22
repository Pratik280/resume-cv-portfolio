# 🚀 Kafka Consumer in Spring Boot — Complete Notes (with Order Example)

---

# 🧠 1. Quick Producer Context (Why Consumer Matters)

In our Order system, we produce messages like:

```java
kafkaTemplate.send(
    "order-topic",
    event.getId().toString(),   // key
    orderResponse               // value (Java object)
);
```

---

## 🔹 What happens internally?

```text
OrderResponse (Java Object)
→ JsonSerializer
→ JSON
→ byte[]
→ Kafka
```

👉 Kafka **always stores byte[]**, not objects

---

# 🔴 2. Consumer Responsibility

Consumer does the reverse:

```text
byte[]
→ Deserializer
→ JSON
→ Java Object
```

👉 This is why **deserializer configuration is critical**

---

# 🔥 3. Consumer Configuration (application.yml)

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092

    consumer:
      group-id: order-group
      auto-offset-reset: earliest

      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer

      properties:
        spring.json.trusted.packages: "*"
```

---

# 🔹 4. Explanation of Each Property

---

## ✅ 1. group-id

```yaml
group-id: order-group
```

👉 Identifies consumer group

* Same group → load balancing
* Different group → each gets all messages

---

## ✅ 2. auto-offset-reset

```yaml
auto-offset-reset: earliest
```

👉 If no offset found:

* `earliest` → read from beginning
* `latest` → read only new messages

---

## ✅ 3. key-deserializer

```yaml
StringDeserializer
```

👉 Converts:

```text
byte[] → String (key)
```

---

## ✅ 4. value-deserializer

```yaml
JsonDeserializer
```

👉 Converts:

```text
byte[] → JSON → Java Object
```

---

## ✅ 5. trusted.packages

```yaml
spring.json.trusted.packages: "*"
```

👉 Required for security
Allows deserialization into your classes

---

# 🔥 5. How Deserialization Works

---

## Flow:

```text
Kafka (byte[])
        ↓
JsonDeserializer
        ↓
Uses target class
        ↓
Creates Java Object
```

---

## 🔴 Important

JSON does NOT contain class info

So Spring needs to know:

```text
Which class should I convert into?
```

---

# 🔹 6. Providing Target Type

---

## ✅ Option 1 (Best — inferred from method)

```java
@KafkaListener(topics = "order-topic")
public void consume(OrderResponse order) {
    log.info("Received: {}", order);
}
```

👉 Spring uses method parameter type

---

## ✅ Option 2 (Explicit config)

```yaml
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.OrderResponse
```

---

# 🔥 7. @KafkaListener Explained

---

## Basic Usage

```java
@KafkaListener(
    topics = "order-topic",
    groupId = "order-group"
)
public void consume(OrderResponse order) {
    log.info("Order received: {}", order);
}
```

---

## 🔹 What it does

* Subscribes to topic
* Polls Kafka automatically
* Deserializes message
* Calls your method

---

## 🔹 Behind the scenes

```text
Kafka Consumer Thread
        ↓
Poll messages
        ↓
Deserialize
        ↓
Invoke method
```

---

# 🔥 8. Topic Handling

---

## Producer

```java
kafkaTemplate.send("order-topic", key, value);
```

---

## Consumer

```java
@KafkaListener(topics = "order-topic")
```

👉 Topic name must match

---

# 🔴 9. Serializer vs Deserializer (Core Concept)

---

## Producer

```text
Java Object → Serializer → byte[]
```

---

## Consumer

```text
byte[] → Deserializer → Java Object
```

---

## 🔥 Rule

> Serializer and Deserializer must match

---

### Example

| Producer         | Consumer           |
| ---------------- | ------------------ |
| JsonSerializer   | JsonDeserializer   |
| StringSerializer | StringDeserializer |

---

# ⚠️ 10. Common Mistakes

---

## ❌ 1. Missing group-id

```text
Error: No group.id found
```

---

## ❌ 2. Mismatched serializer

```text
Producer → JSON
Consumer → StringDeserializer ❌
```

---

## ❌ 3. Missing trusted packages

```text
Class not trusted error
```

---

## ❌ 4. Sending String but expecting Object

```java
@KafkaListener
public void consume(OrderResponse order) ❌
```

---

# 🔥 11. End-to-End Flow

```text
Producer:
OrderResponse → JSON → Kafka

Consumer:
Kafka → JSON → OrderResponse
```

---

# 🎯 12. Interview Summary

> In Spring Kafka, the consumer uses deserializers to convert byte array messages into Java objects. JsonDeserializer is used to map JSON messages to POJOs, either by inferring the type from the method parameter or using configuration. The @KafkaListener annotation simplifies message consumption by handling polling, deserialization, and method invocation automatically. Proper configuration of group-id, topic, and serializers ensures reliable and scalable message processing.

---

# 🚀 13. Key Takeaways

* Kafka stores only byte[]
* Serializer → producer side
* Deserializer → consumer side
* @KafkaListener handles polling automatically
* group-id is mandatory
* Type mapping is critical for JSON

---

# 🔁 Retry & DLQ (Dead Letter Queue)

---

## 🔹 Why needed?

When a Kafka consumer processes a message:

```text
Message → Consumer → Processing
```

👉 It can fail due to:

* DB issues
* temporary network issues
* invalid data

---

## 🔁 Retry

👉 Retry means reprocessing the same message multiple times

```text
Fail → Retry 1 → Retry 2 → Retry 3
```

✔ Handles temporary failures
✔ Improves reliability

---

## 💀 DLQ (Dead Letter Queue)

👉 If message still fails after retries:

```text
Send to another topic (DLQ)
```

Example:

```text
order-topic       → normal messages  
order-topic-dlt   → failed messages
```

---

## 🔥 Flow

```text
Kafka → Consumer
        ↓
     Failure
        ↓
     Retry (n times)
        ↓
   Still fails ❌
        ↓
   Send to DLQ
```

---

## 🎯 Key Points

* Retry → for temporary issues
* DLQ → for permanent failures
* Prevents infinite retry loops
* Ensures no data loss

---

## 🧠 Interview Line

> Retry is used to handle transient failures by reprocessing messages, and DLQ is used to store messages that fail permanently after retries, ensuring system reliability and debuggability.

---

# 📦 DTO / Event Design in Kafka (Producer vs Consumer)

---

# 🔴 1. Problem Statement

In a microservices system:

```text
Order Service (Producer) → Kafka → Inventory Service (Consumer)
```

👉 Question:

Do we use the same `OrderResponse` class in both services?

---

# ❌ 2. Wrong Approach (Tight Coupling)

```text
Shared library (common JAR)
→ Both services use same OrderResponse class
```

### Problems:

* Tight coupling between services
* Version conflicts during deployment
* Any change breaks multiple services
* Not true microservice architecture

---

# ✅ 3. Correct Approach (Loose Coupling)

👉 Each service defines its **own DTO / Event class**

---

## 🔹 Producer Side (Order Service)

```java
public class OrderCreatedEvent {

    private Long id;
    private String productName;

}
```

---

## 🔹 Consumer Side (Inventory Service)

```java
public class OrderCreatedEventDTO {

    private Long id;
    private String productName;

}
```

---

# 🔥 4. Why This Works

Kafka sends:

```text
JSON (not Java objects)
```

Example message:

```json
{
  "id": 1,
  "productName": "Mobile"
}
```

👉 Consumer maps JSON → its own class

---

# 🔑 5. Key Principle

> Kafka communication is based on **schema (JSON)**, not Java classes

---

# 🔹 6. Naming Best Practices

---

## ❌ Avoid

```text
OrderResponse (API DTO)
```

👉 Why?

* API DTO ≠ Event DTO
* Mixing responsibilities

---

## ✅ Use Event-Based Names

```text
OrderCreatedEvent         (Producer)
OrderCreatedEventDTO      (Consumer)
```

---

## 💡 Even better (clean naming)

```text
Producer → OrderCreatedEvent
Consumer → OrderCreatedEvent (same name, different package)
```

✔ Same meaning
✔ Still independent

---

# 🔹 7. What Must Match?

👉 Only this:

```text
JSON structure (fields + types)
```

---

## Example

Producer sends:

```json
{
  "id": 1,
  "productName": "Mobile"
}
```

Consumer class:

```java
class OrderCreatedEvent {
    Long id;
    String productName;
}
```

✔ Works perfectly

---

# 🔹 8. Schema Evolution (Important)

---

## Adding new field

Producer:

```json
{
  "id": 1,
  "productName": "Mobile",
  "price": 1000
}
```

👉 Old consumer:

✔ Still works (ignores extra fields)

---

## Removing field

👉 Dangerous ❗
→ Can break consumer

---

# 🔥 9. Best Practice Summary

* Do NOT share DTO classes across services
* Use **event-specific classes**
* Keep services independent
* Ensure JSON structure compatibility
* Prefer backward-compatible changes

---

# 🎯 10. Interview Answer

> In Kafka-based microservices, we should not share Java DTO classes between producer and consumer services, as it creates tight coupling. Instead, each service defines its own event class with the same JSON structure. Kafka communicates using serialized data like JSON, so as long as the schema matches, services remain loosely coupled and independently deployable.

---

# 🚀 11. One-Line Memory Trick

```text
Same JSON schema → different classes → loosely coupled microservices
```

---
