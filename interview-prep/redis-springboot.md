---

# 🧠 Spring Boot + Redis Caching (Final Interview Notes)

---

# 🔹 Overview

* Spring caching abstraction: **Spring Cache**
* Redis integration: **Spring Data Redis**

👉 Why we use Redis:

* Reduce DB calls
* Cache external API / microservice responses
* Improve latency
* Handle distributed use cases (locks, counters, sessions)

---

# 🔹 1. Add Redis Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

👉 Uses **Lettuce** by default (non-blocking, production-friendly)

---

# 🔹 2. Configure Redis Connection

### ✅ application.yml (most common)

```yaml
spring:
  redis:
    host: localhost
    port: 6379
```

---

### ✅ OR Java Config

```java
@Bean
public RedisConnectionFactory redisConnectionFactory() {
    return new LettuceConnectionFactory("localhost", 6379);
}
```

---

# 🔹 3. Do We Need RedisTemplate Bean?

## ❗ Key Interview Concept

👉 **No, not mandatory**

Spring Boot auto-configures:

* `RedisConnectionFactory`
* `RedisTemplate<Object, Object>`
* `StringRedisTemplate`

So this works without any custom bean.

---

## ⚠️ But in Real Systems → You SHOULD create one

### Problem with default:

* Uses **Java serialization (binary)**
* Not readable
* Can break across versions
* Hard to debug

---

## ✅ Recommended Production Config

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {

    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(factory);

    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

    return template;
}
```

👉 Now data in Redis is JSON → readable + portable

---

## 🔹 When is custom RedisTemplate needed?

| Scenario              | Needed? |
| --------------------- | ------- |
| Only @Cacheable       | ❌ No    |
| Simple apps           | ❌ No    |
| Production            | ✅ Yes   |
| Debugging Redis       | ✅ Yes   |
| Cross-service sharing | ✅ Yes   |

---

# 🔹 4. Enable Caching

```java
@EnableCaching
@SpringBootApplication
public class Application {}
```

---

# 🔹 5. Cache Annotations (Core)

---

## 🔸 @Cacheable (Read → Cache if missing)

### Concept

* First call → actual execution (DB/API)
* Next calls → Redis
* Method is **skipped on cache hit**

---

## Example 1: DB Call

```java
@Cacheable(value = "users", key = "#id")
public User getUserById(String id) {
    return userRepository.findById(id).orElse(null);
}
```

---

## Example 2: REST Call (Microservice via RestTemplate)

```java
@Cacheable(value = "userProfile", key = "#userId")
public UserProfile getUserProfile(String userId) {
    return restTemplate.getForObject(
        "http://user-service/api/users/" + userId,
        UserProfile.class
    );
}
```

👉 Avoids repeated network calls

---

## Example 3: Feign Client (REAL-WORLD MICROservices)

Feign: **OpenFeign**

---

### Feign Client

```java
@FeignClient(name = "config-service")
public interface ConfigClient {

    @GetMapping("/config/merchant/{id}")
    MerchantConfig getMerchantConfig(@PathVariable("id") String id);
}
```

---

### Service Layer (Caching applied)

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

## 🔍 What’s happening internally?

1. First request:

   * Calls `config-service`
   * Stores result in Redis

   ```
   merchantConfig::123 → JSON(MerchantConfig)
   ```

2. Next request:

   * Directly served from Redis
   * No network call

---

## 🎯 Why this is important

* Reduces inter-service latency
* Prevents overload on downstream services
* Improves resilience

---

# 🔹 6. @CacheEvict (Remove stale cache)

### When data changes → cache must be cleared

---

## Example (Feign update)

```java
@CacheEvict(value = "merchantConfig", key = "#merchantId")
public void updateMerchantConfig(String merchantId, MerchantConfig config) {
    configClient.updateConfig(merchantId, config);
}
```

---

👉 Without eviction:

* Redis returns **old data ❌**

👉 With eviction:

* Next call fetches fresh data ✅

---

# 🔹 7. @CachePut (Update cache directly)

```java
@CachePut(value = "merchantConfig", key = "#merchantId")
public MerchantConfig updateAndReturn(String merchantId, MerchantConfig config) {
    return configClient.updateConfig(merchantId, config);
}
```

---

## 🔹 Difference (very important)

| Annotation  | Behavior              |
| ----------- | --------------------- |
| @Cacheable  | Skip method if cached |
| @CacheEvict | Remove cache          |
| @CachePut   | Update cache          |

---

# 🔹 8. TTL (Time To Live)

```yaml
spring:
  cache:
    redis:
      time-to-live: 600000
```

👉 Prevents stale data + memory issues

---

# 🔹 9. RedisTemplate (Manual Control)

Spring provides **RedisTemplate**

---

## 🔸 opsForValue() (String operations)

Used for:

* caching
* counters
* locks

---

Here are the same operations rewritten with **generic placeholders** instead of hardcoded values:

```java
// Write value
redisTemplate.opsForValue().set(key, value);

// Read value
Object result = redisTemplate.opsForValue().get(key);

// Write with TTL
redisTemplate.opsForValue()
    .set(key, value, ttl, timeUnit);

// Increment (by 1 or custom step)
redisTemplate.opsForValue().increment(key);
redisTemplate.opsForValue().increment(key, delta);

// Set if absent (distributed lock pattern with expiry)
Boolean success = redisTemplate.opsForValue()
    .setIfAbsent(key, value, ttl, timeUnit);
```

---

## 🔍 Meaning of placeholders

* `key` → unique identifier (e.g., `"user:123"`, `"otp:phone:9999"`)
* `value` → data stored (String / Object depending on serializer)
* `ttl` → duration (e.g., `5`)
* `timeUnit` → unit (`TimeUnit.SECONDS`, `MINUTES`, etc.)
* `delta` → increment step (e.g., `1`, `5`)

---

## 🧠 Key Understanding (important)

* `set(key, value)` → basic cache write
* `get(key)` → read from Redis
* `set(key, value, ttl, timeUnit)` → auto-expiring cache
* `increment(key)` → atomic counter
* `setIfAbsent(...)` → used for locking (only sets if key doesn’t exist)

---

## 🎯 Interview Tip

Always talk in terms of:

* **key design** (`user:123`, `order:456`)
* **TTL usage** (avoid stale data)
* **atomic operations** (increment, locking)


---

## 🔍 Key Insight

* `opsForValue()` → Redis **String type**
* All operations are **atomic**
* Used for:

  * rate limiting
  * distributed locking
  * counters

---

# 🔹 10. Cache vs RedisTemplate

| Feature     | @Cacheable     | RedisTemplate  |
| ----------- | -------------- | -------------- |
| Abstraction | High           | Low            |
| Control     | Limited        | Full           |
| Use case    | Simple caching | Advanced logic |

---

# 🔹 11. Advanced Concepts

## 🧠 1. Cache Aside (Cache-Aside Pattern)

### 🔹 Explanation (from scratch)

Cache Aside means:

* Application is responsible for cache
* It **checks cache first**
* If not found → fetch from DB → store in cache → return

👉 Cache is **lazy-loaded** (only filled when needed)

---

### 🔹 Flow

```text id="l7jyku"
Request → Cache
        → HIT → return
        → MISS → DB → Cache → return
```

---

### 🔹 Example (simple)

```java id="ehar2y"
public User getUser(String id) {

    String key = "user:" + id;

    // 1. Check cache
    User cached = (User) redisTemplate.opsForValue().get(key);
    if (cached != null) return cached;

    // 2. Fetch from DB
    User user = userRepository.findById(id).orElse(null);

    // 3. Store in cache
    if (user != null) {
        redisTemplate.opsForValue().set(key, user, 10, TimeUnit.MINUTES);
    }

    return user;
}
```

---

### 🎯 Interview Answer

> Cache Aside is a caching pattern where the application first checks the cache, and on a miss, fetches data from the database and updates the cache. It is simple and widely used for read-heavy systems.

---

## 🔥 2. Cache Stampede

### 🔹 Explanation

Cache stampede happens when:

👉 Many requests hit DB at the same time
👉 Because cache expired for a popular key

---

### 🔹 Example

```text id="u0jhzh"
user:123 expires at 10:00

At 10:00:
1000 requests → cache MISS → DB hit 💥
```

👉 DB gets overloaded

---

### 🔹 Why it happens

* Same TTL for all keys
* No protection on cache miss

---

### 🔹 Solutions

* Random TTL (spread expiry)
* Locking (only one request fetches DB)
* Cache pre-warming

---

### 🎯 Interview Answer

> Cache stampede occurs when many requests simultaneously hit the database after a cache entry expires. It can be mitigated using techniques like randomizing TTL, using distributed locks, or pre-warming the cache.

---

## 🚫 3. Cache Penetration

### 🔹 Explanation

Cache penetration happens when:

👉 Requests are for **data that does not exist**

---

### 🔹 Example

```text id="pdt8ac"
user:999 (does not exist)

Request → Cache MISS → DB → not found
Repeat → DB hit again and again ❌
```

---

### 🔹 Why it’s dangerous

* DB gets unnecessary load
* Can be used in attacks

---

### 🔹 Solutions

* Cache null values (short TTL)
* Bloom filter (advanced)
* Input validation

---

### 🎯 Interview Answer

> Cache penetration occurs when repeated requests for non-existent data bypass the cache and hit the database every time. It can be prevented by caching null responses, using bloom filters, or validating inputs.

---

## ⚖️ Quick Summary

| Concept     | Meaning                                          |
| ----------- | ------------------------------------------------ |
| Cache Aside | App controls cache (read → DB → cache)           |
| Stampede    | Many requests hit DB after expiry                |
| Penetration | Requests for non-existent data hit DB repeatedly |


---

# 🔹 12. Common Mistakes

* Missing `@EnableCaching`
* Wrong keys → collision
* No eviction → stale data
* Self-invocation (Spring proxy issue)
* No TTL
* Large objects in Redis

---

# 🔹 Final Interview Answer (Polished)

> We integrate Redis using spring-boot-starter-data-redis, configure connection via properties or a RedisConnectionFactory, and optionally define a custom RedisTemplate with JSON serialization for production use.
>
> We use @Cacheable to cache DB or microservice (Feign/REST) responses, @CacheEvict to remove stale data after updates, and @CachePut to update cache.
>
> For advanced scenarios like distributed locking, counters, or rate limiting, we use RedisTemplate with opsForValue().

---