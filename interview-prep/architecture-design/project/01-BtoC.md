---

# 🏦 Step 2 (Updated): Transaction → Outbox → Kafka → Service C

---

# 🧩 System Overview

```text
Client → B (Middleware)
        → External Processor
        → Outbox (DB in B)
        → Kafka (transaction-topic)
        → C (Transaction Service)
        → DB (source of truth)
```

---

# 🎯 Goal

* Process transaction safely
* Ensure **no data loss**
* Ensure **scalable event publishing**
* Ensure **idempotent persistence**

---

# 🧠 Core Design

* B is **stateless (no txn DB)**
* Outbox ensures **reliable delivery**
* `@Async + ThreadPoolTaskExecutor` ensures **concurrency**
* C ensures **idempotency via UPSERT**

---

# 🧩 Step 2.1 — Transaction Processing in B

---

## ✅ Flow

```text
1. Receive request
2. Generate txnId
3. Call processor (WebClient)
4. Get response
5. Create transaction event
6. Save to OUTBOX
```

---

## 💻 Code

```java
@Service
public class TransactionService {

    @Autowired
    private WebClient webClient;

    @Autowired
    private OutboxRepository outboxRepository;

    public void processTransaction(TransactionRequest request) {

        ProcessorResponse response = webClient.post()
                .uri("/process")
                .bodyValue(request)
                .retrieve()
                .bodyToMono(ProcessorResponse.class)
                .timeout(Duration.ofSeconds(5))
                .onErrorReturn(buildFailureResponse(request))
                .block();

        TransactionEvent event = new TransactionEvent();
        event.setTxnId(request.getTxnId());
        event.setMerchantId(request.getMerchantId());
        event.setAmount(request.getAmount());
        event.setStatus(response.getStatus());

        OutboxEvent outbox = new OutboxEvent();
        outbox.setEventType("TRANSACTION_CREATED");
        outbox.setPayload(convertToJson(event));
        outbox.setStatus("NEW");

        outboxRepository.save(outbox);
    }
}
```

---

# 🧩 Step 2.2 — Async Outbox Publisher (UPDATED)

---

# ✅ Enable Async

```java
@EnableAsync
@Configuration
public class AsyncConfig {
}
```

---

# ✅ ThreadPoolTaskExecutor (Custom Bean)

```java
@Configuration
public class ExecutorConfig {

    @Bean(name = "outboxExecutor")
    public ThreadPoolTaskExecutor outboxExecutor() {

        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("outbox-");

        executor.initialize();
        return executor;
    }
}
```

---

# ✅ Async Processor

```java
@Service
public class OutboxAsyncProcessor {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    private OutboxRepository outboxRepo;

    @Async("outboxExecutor")
    public void processEvent(OutboxEvent event) {

        try {
            kafkaTemplate.send(
                "transaction-topic",
                event.getTxnId(),
                event.getPayload()
            ).get(); // wait for Kafka ack

            event.setStatus("SENT");

        } catch (Exception e) {
            event.setStatus("FAILED");
        }

        outboxRepo.save(event);
    }
}
```

---

# ✅ Scheduler (Triggers Async Execution)

```java
@Service
public class OutboxPublisher {

    @Autowired
    private OutboxRepository outboxRepo;

    @Autowired
    private OutboxAsyncProcessor asyncProcessor;

    @Scheduled(fixedDelay = 2000)
    public void publishEvents() {

        List<OutboxEvent> events =
            outboxRepo.fetchBatchForUpdate(); // uses SKIP LOCKED

        for (OutboxEvent event : events) {
            asyncProcessor.processEvent(event); // async call
        }
    }
}
```

---

# ⚠️ DB Locking (CRITICAL)

```sql
SELECT * FROM outbox_events
WHERE status = 'NEW'
FOR UPDATE SKIP LOCKED
LIMIT 100;
```

---

# 🔁 Flow with Async

```text
Scheduler thread
→ fetch batch

For each event:
→ call @Async method

Spring:
→ submits to outboxExecutor
→ runs in parallel threads
```

---

# 🧠 Why This is Correct

| Problem              | Solution            |
| -------------------- | ------------------- |
| Slow publishing      | Async execution     |
| Thread explosion     | Thread pool         |
| Duplicate processing | DB locking          |
| Blocking Kafka call  | controlled `.get()` |

---

# 🧩 Step 2.3 — Kafka → Service C

---

## ✅ Consumer

```java
@Service
public class TransactionConsumer {

    @Autowired
    private TransactionService transactionService;

    @KafkaListener(topics = "transaction-topic", groupId = "txn-group")
    public void consume(TransactionEvent event) {

        transactionService.saveOrUpdate(event);
    }
}
```

---

# 🧩 Step 2.4 — Idempotent Persistence (UPSERT)

---

## 💻 Query

```sql
INSERT INTO transactions (txn_id, status, amount, status_priority)
VALUES (:txnId, :status, :amount, :priority)

ON CONFLICT (txn_id)
DO UPDATE SET
    status = EXCLUDED.status,
    status_priority = EXCLUDED.status_priority
WHERE transactions.status_priority < EXCLUDED.status_priority;
```

---

## 💻 Service

```java
@Transactional
public void saveOrUpdate(TransactionEvent event) {

    int priority = getPriority(event.getStatus());

    repository.upsertTransaction(
        event.getTxnId(),
        event.getStatus(),
        event.getAmount(),
        priority
    );
}
```

---

# 🧠 Status Priority

```text
INITIATED = 1
PENDING   = 2
SUCCESS   = 3
FAILED    = 3
```

---

# 🔁 End-to-End Flow

```text
1. Client → B

2. B:
   → Call processor
   → Save event in outbox

3. Scheduler:
   → Fetch batch (locked)
   → Call async processor

4. Async Threads:
   → Send to Kafka
   → Update status

5. Kafka:
   → transaction-topic

6. C:
   → Consume event
   → UPSERT into DB
```

---

# 🧠 Key Guarantees

| Concern              | Solution             |
| -------------------- | -------------------- |
| Data loss            | Outbox               |
| Slow publishing      | @Async + thread pool |
| Duplicate processing | SKIP LOCKED          |
| Kafka retry          | Outbox retry         |
| Duplicate events     | UPSERT               |
| Out-of-order         | priority             |

---

# 🎯 Interview Summary

> “We use the outbox pattern for reliable event publishing and leverage Spring’s @Async with a dedicated ThreadPoolTaskExecutor to process events concurrently. Database-level locking ensures safe parallel processing, and the consumer uses an upsert strategy to maintain idempotency.”

---

# 💥 Killer Line

> “We combine async processing for scalability and database guarantees for correctness.”

---
