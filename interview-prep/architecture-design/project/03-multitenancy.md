---

# 🧠 Hibernate Multi-Tenancy (Database per Tenant) — Complete Notes

---

# 🧾 1. Problem Statement

In a multi-tenant system:

* Each **tenant (bank)** has its **own database**
* Application is shared
* Requests contain a **tenant identifier (bankId)**

👉 Goal:
Route each request to the **correct database dynamically**

---

# 🏗️ 2. Multi-Tenancy Strategies (Know This)

| Strategy             | Description                               |
| -------------------- | ----------------------------------------- |
| Database per tenant  | Each tenant has separate DB ✅ (your case) |
| Schema per tenant    | Same DB, different schemas                |
| Column discriminator | Shared DB, tenantId column                |

---

### 🎯 Interview Line

“I implemented database-per-tenant multi-tenancy, where each tenant has an isolated database, ensuring strong data separation.”

---

# 🔑 3. Core Components Overview

Your implementation depends on:

1. **TenantContext (ThreadLocal)**
2. **Filter (sets tenant per request)**
3. **AbstractRoutingDataSource (routes DB)**
4. **Multiple DataSource beans**
5. **Hibernate integration (resolver)**

---

# 🧵 4. ThreadLocal & TenantContext

## 👉 What is ThreadLocal?

* Provides **thread-scoped storage**
* Each thread has its own copy
* Prevents concurrency issues

---

### 🎯 Interview Line

“ThreadLocal allows us to store tenant information per request thread, avoiding the need to pass tenantId across multiple layers.”

---

## 👉 TenantContext Implementation

```java
public class TenantContext {

    private static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();

    public static void setTenant(String tenant) {
        CURRENT_TENANT.set(tenant);
    }

    public static String getTenant() {
        return CURRENT_TENANT.get();
    }

    public static void clear() {
        CURRENT_TENANT.remove();
    }
}
```

---

## ⚠️ Important

* Must **clear after request**
* Otherwise → **thread reuse issues**

---

### 🎯 Interview Line

“We use ThreadLocal to maintain tenant isolation per request, and we clear it after execution to prevent data leakage due to thread pooling.”

---

# 🌐 5. Filter (Request Entry Point)

## 👉 What is a Filter?

* Runs **before and after every HTTP request**
* Ideal place to set tenant

---

## 👉 Implementation

```java
public class TenantFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        try {
            String bankId = request.getHeader("bankId");
            TenantContext.setTenant(bankId);

            filterChain.doFilter(request, response);

        } finally {
            TenantContext.clear();
        }
    }
}
```

---

## ⚠️ Key Points

Short answer: **your `OncePerRequestFilter` runs *before* the request reaches Spring’s `DispatcherServlet` and your controllers.**

Let’s place it precisely so you can explain it confidently.

---

# 🔄 Full Request Lifecycle (Spring Boot + Tomcat)

```text
Client
  ↓
Tomcat (Servlet Container)
  ↓
Filter Chain  ← 🔥 YOUR TenantFilter runs HERE
  ↓
DispatcherServlet (Spring MVC entry point)
  ↓
HandlerMapping → Controller → Service → Repository
  ↓
Response goes back (same chain in reverse)
```

---

# 🧠 What actually happens step-by-step

### 1. Request hits Tomcat

* Tomcat receives HTTP request
* It looks up registered **Filters**

---

### 2. Filter chain executes

* All filters run in order
* Your `TenantFilter` runs here

👉 At this point:

* No controller yet
* No Spring MVC logic yet

---

### 3. Your filter logic executes

```java
TenantContext.setTenant(bankId);
```

✔ Tenant is now available **before Spring starts processing**

---

### 4. You call:

```java
filterChain.doFilter(request, response);
```

👉 This hands over control to:

```text
→ Next filter (if any)
→ Eventually DispatcherServlet
```

---

### 5. DispatcherServlet takes over

From here Spring starts:

* Finds controller
* Executes service
* Calls repository
* Hibernate uses tenant → routing happens


### 6. Response comes back

Then:

```java
finally {
    TenantContext.clear();
}
```

👉 Filter runs at **Servlet container level (Tomcat)**
👉 DispatcherServlet is just **one step after filters**

> “Does filter run before DispatcherServlet or controller?”

✅ Answer:

* Before DispatcherServlet ✅
* Before Controller ✅
* Before Service/Repo ✅

---



👉 “When does your filter execute?”

“My TenantFilter runs in the servlet filter chain before the request reaches Spring’s DispatcherServlet. Since filters are part of the servlet container (Tomcat), they execute at the very beginning of the request lifecycle. This ensures the tenant is set in ThreadLocal before any business logic, including controllers and database access, is executed.”

“Because it runs before DispatcherServlet, it guarantees that even Hibernate and transaction management have access to the tenant context.”


* `filterChain.doFilter()` → continues request
* `finally` → ensures cleanup

---

### 🎯 Interview Line

“I used a OncePerRequestFilter to extract the tenant from the request header and store it in ThreadLocal before processing. The context is cleared in a finally block.”

---

# 🗄️ 6. Multiple DataSource Configuration

## 👉 Application Properties Example

```properties
tenant1.datasource.url=jdbc:mysql://localhost:3306/db1
tenant1.datasource.username=root
tenant1.datasource.password=xxx

tenant2.datasource.url=jdbc:mysql://localhost:3306/db2
tenant2.datasource.username=root
tenant2.datasource.password=yyy
```

---

## 👉 Define DataSources

```java
@Bean
public DataSource tenant1DataSource() {
    return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/db1")
            .username("root")
            .password("xxx")
            .build();
}

@Bean
public DataSource tenant2DataSource() {
    return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/db2")
            .username("root")
            .password("yyy")
            .build();
}
```

---

### 🎯 Interview Line

“I configured separate DataSource beans for each tenant using application properties.”

---

# 🔀 7. AbstractRoutingDataSource (Core Logic)

## 👉 Purpose

* Dynamically chooses correct DataSource at runtime

---

## 👉 Implementation

```java
public class MultiTenantRoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return TenantContext.getTenant();
    }
}
```

---

## 👉 Configuration

```java
@Bean
public DataSource dataSource() {

    Map<Object, Object> targetDataSources = new HashMap<>();

    targetDataSources.put("1", tenant1DataSource());
    targetDataSources.put("2", tenant2DataSource());

    MultiTenantRoutingDataSource routingDataSource = new MultiTenantRoutingDataSource();
    routingDataSource.setTargetDataSources(targetDataSources);
    routingDataSource.setDefaultTargetDataSource(tenant1DataSource());

    return routingDataSource;
}
```

---

## 🔥 How it Works

* Hibernate asks for connection
* Routing DS calls:

```java
determineCurrentLookupKey()
```

* Returns tenant → picks correct DB

---

### 🎯 Interview Line

“I used AbstractRoutingDataSource, which determines the current tenant dynamically using ThreadLocal and routes to the corresponding DataSource.”

---

# 🔄 8. End-to-End Flow

### Step-by-step:

1. Request comes with `bankId`
2. Filter sets tenant in ThreadLocal
3. Controller → Service → Repository
4. Hibernate requests connection
5. Routing DataSource selects DB
6. Query executes
7. Finally → TenantContext cleared

---

### 🎯 Interview Line

“The tenant is extracted at the request boundary, stored in ThreadLocal, and used by the routing DataSource during DB access to dynamically select the correct database.”

---

# ⚠️ 9. Critical Edge Cases

---

## ❗ Thread Reuse Problem

* Threads are reused from pool
* If not cleared → wrong tenant used

---

## ❗ Missing Tenant

Options:

* Throw exception
* Use default (risky)

---

## ❗ Key Mismatch

```java
targetDataSources.put("2", ds)
TenantContext.getTenant() = "2" // must match
```

---

## ❗ Too Many Tenants

* Many DataSources = memory overhead
* Use:

  * Lazy loading
  * Dynamic creation

---

### 🎯 Interview Line

“ThreadLocal must be cleared after each request to prevent tenant leakage due to thread reuse in server thread pools.”

---

# ⚙️ 10. Hibernate Integration (Important)

## 👉 Key Interfaces

* `CurrentTenantIdentifierResolver`
* `MultiTenantConnectionProvider`

---

## 👉 Resolver Example

```java
public class TenantIdentifierResolver 
        implements CurrentTenantIdentifierResolver {

    @Override
    public String resolveCurrentTenantIdentifier() {
        return TenantContext.getTenant();
    }
}
```

---

### 🎯 Interview Line

“Hibernate uses CurrentTenantIdentifierResolver to fetch the tenant dynamically during session creation.”

---

# ❓ 11. Common Interview Questions

---

### Q: Why ThreadLocal?

👉 Avoid passing tenantId across layers

---

### Q: When is routing decided?

👉 At DB connection time

---

### Q: Why Filter?

👉 Runs at request boundary, before business logic

---

### Q: What if async call?

👉 ThreadLocal won’t propagate ❌
→ Must handle manually

---

### Q: Is this scalable?

👉 Yes, but optimize for large tenant counts

---

# 🚀 12. Strong Final Answer (Use This)

---

“I implemented database-per-tenant multi-tenancy using Hibernate and Spring Boot. Each tenant has a separate DataSource configured via application properties.

I used a ThreadLocal-based TenantContext to store the tenant identifier per request. A OncePerRequestFilter extracts the tenant from the request header and sets it before processing.

I extended AbstractRoutingDataSource to dynamically select the correct DataSource at runtime using the tenant from ThreadLocal.

Finally, the TenantContext is cleared after each request to prevent data leakage due to thread reuse.”

---

# 🧠 13. Bonus (To Impress)

Mention this if possible:

* ThreadLocal breaks in `@Async`
* Can integrate tenant from JWT
* Can implement dynamic tenant onboarding

---
