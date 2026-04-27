---

# 🧠 Spring Boot + Redis + Dev Setup (Complete Interview Notes)

---

# 🔹 Overview

* Spring caching abstraction: **Spring Cache**
* Redis integration via: **Spring Data Redis**
* Used for:

  * reducing DB calls
  * improving latency
  * distributed system use cases

---

# 🔹 Enable Caching

```java id="9a1m6y"
@EnableCaching
@SpringBootApplication
public class Application {}
```

---

# 🔹 Cache Annotations

## 🔸 @Cacheable (Read → Cache if missing)

```java id="3jexnf"
@Cacheable(value = "users", key = "#id")
public User getUserById(String id) {
    return userRepository.findById(id).orElse(null);
}
```

### ✅ Behavior

* First call → DB hit → cache result
* Next calls → return from cache
* Method NOT executed on cache hit

---

## 🔸 @CacheEvict (Remove Cache)

```java id="2z2y83"
@CacheEvict(value = "users", key = "#id")
public void deleteUser(String id) {
    userRepository.deleteById(id);
}
```

---

## 🔸 @CachePut (Update Cache)

```java id="6h9s6s"
@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}
```

---

## 🔸 Differences

| Annotation  | Behavior                    |
| ----------- | --------------------------- |
| @Cacheable  | Skip method if cache exists |
| @CacheEvict | Remove cache                |
| @CachePut   | Update cache                |

---

## 🔸 Evict All

```java id="0h7v8c"
@CacheEvict(value = "users", allEntries = true)
public void clearCache() {}
```

---

## 🔸 Conditional Caching

```java id="3xk0qv"
@Cacheable(value = "users", key = "#id", condition = "#id.length() > 3")
```

```java id="b7paz7"
@Cacheable(value = "users", key = "#id", unless = "#result == null")
```

---

# 🔹 Redis Behind the Scenes

```
cacheName::key
```

Example:

```
users::101 → {"id":101,"name":"Pratik"}
```

---

# 🔹 TTL (Time To Live)

```yaml id="g3g6e1"
spring:
  cache:
    redis:
      time-to-live: 600000
```

---

# 🔹 RedisTemplate (Manual Control)

Spring provides **RedisTemplate**

---

# 🔹 opsForValue() (String Operations)

## 🔸 Write

```java id="2q5r8x"
redisTemplate.opsForValue().set("user:1", "Pratik");
```

---

## 🔸 Read

```java id="98y0xg"
redisTemplate.opsForValue().get("user:1");
```

---

## 🔸 TTL

```java id="7k4yr5"
redisTemplate.opsForValue()
             .set("otp:1234", "5678", 5, TimeUnit.MINUTES);
```

---

## 🔸 Exists

```java id="1l9n93"
redisTemplate.hasKey("user:1");
```

---

## 🔸 Delete

```java id="8h2n7c"
redisTemplate.delete("user:1");
```

---

## 🔸 Increment / Decrement

```java id="1j9nfu"
redisTemplate.opsForValue().increment("counter");
redisTemplate.opsForValue().increment("counter", 5);
redisTemplate.opsForValue().decrement("counter");
```

---

## 🔸 setIfAbsent (Lock)

```java id="8dj5pt"
redisTemplate.opsForValue()
    .setIfAbsent("lock:order:1", "locked", 10, TimeUnit.SECONDS);
```

---

## 🔸 Append

```java id="3b7z6n"
redisTemplate.opsForValue().append("key", "data");
```

---

# 🔹 Data Structures Mapping

| Redis Type | Spring        |
| ---------- | ------------- |
| String     | opsForValue() |
| Hash       | opsForHash()  |
| List       | opsForList()  |
| Set        | opsForSet()   |
| ZSet       | opsForZSet()  |

---

# 🔹 Cache vs RedisTemplate

| Feature     | @Cacheable     | RedisTemplate  |
| ----------- | -------------- | -------------- |
| Abstraction | High           | Low            |
| Control     | Limited        | Full           |
| Use case    | Simple caching | Advanced logic |

---

# 🔹 Advanced Concepts

## 🔸 Cache Stampede

* Many requests hit DB after expiry

## 🔸 Cache Penetration

* Requests for invalid keys

## 🔸 Cache Aside Pattern

```
App → Cache → DB → Cache
```

---

# 🔹 Common Mistakes

* Missing `@EnableCaching`
* Wrong keys
* No eviction
* Self-invocation issue
* No TTL
* Large objects

---

# 🔹 Real Use Cases

* DB caching
* sessions
* rate limiting
* distributed locking

---

# 🔹 Dev Setup: VS Code Dotfiles Symlink

Editor: **Visual Studio Code**

---

## 🔸 Source (Dotfiles Repo)

```
/var/home/pratikc/repos/dotfiles/config/vscode/
├── settings.json
└── keybindings.json
```

---

## 🔸 Target (VS Code Config)

```
~/.config/Code/User/
```

---

## 🔸 Create Directory (if needed)

```bash id="9c3b6d"
mkdir -p ~/.config/Code/User
```

---

## 🔸 Create Symlinks

```bash id="4d7k2m"
ln -sf /var/home/pratikc/repos/dotfiles/config/vscode/settings.json ~/.config/Code/User/settings.json

ln -sf /var/home/pratikc/repos/dotfiles/config/vscode/keybindings.json ~/.config/Code/User/keybindings.json
```

---

## 🔸 Verify

```bash id="5f8m1z"
ls -l ~/.config/Code/User/
```

Expected:

```
settings.json -> /var/home/pratikc/repos/dotfiles/config/vscode/settings.json
keybindings.json -> /var/home/pratikc/repos/dotfiles/config/vscode/keybindings.json
```

---

## 🔸 Backup Existing Config (Optional)

```bash id="2p7s9c"
mv ~/.config/Code/User/settings.json ~/.config/Code/User/settings.json.bak
mv ~/.config/Code/User/keybindings.json ~/.config/Code/User/keybindings.json.bak
```

---

## 🔸 Notes

* Close VS Code before linking
* Use `-sf` to overwrite existing files
* OSS version path:

  ```
  ~/.config/Code - OSS/User/
  ```

---

# 🔹 Final Interview Summary

> `@Cacheable` caches method results and skips execution on cache hit.
> `@CacheEvict` removes stale cache.
> `@CachePut` updates cache after execution.
> `RedisTemplate` with `opsForValue()` gives fine-grained control for key-value operations like TTL, atomic increment, and locking.
> Redis is widely used for caching, sessions, rate limiting, and distributed systems.

---
