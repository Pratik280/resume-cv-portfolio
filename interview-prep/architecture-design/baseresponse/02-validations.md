# 📘 Validation in Spring Boot (Jakarta Validation / Bean Validation)

## ✅ Why validation?

* Prevent bad data from entering system
* Fail fast at API layer
* Standardized error responses

---

# 🧩 1. Add Validation Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

# 🧾 2. Request DTO with Validation Annotations

👉 Validation is applied on **DTO (NOT Entity)**

```java
@Getter
@Setter
@NoArgsConstructor
public class CreateOrderRequest {

    @NotBlank(message = "Product name is required")
    private String productName;

    @NotNull(message = "Quantity is required")
    @Min(value = 1, message = "Quantity must be greater than 0")
    private Integer quantity;

    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.1", message = "Price must be greater than 0")
    private Double price;
}
```

---

## 📌 Common Validation Annotations

| Annotation    | Use                              |
| ------------- | -------------------------------- |
| `@NotNull`    | Cannot be null                   |
| `@NotBlank`   | Not null + not empty + no spaces |
| `@NotEmpty`   | Not null + not empty             |
| `@Min / @Max` | Number range                     |
| `@Size`       | String length                    |
| `@Email`      | Valid email                      |
| `@Pattern`    | Regex validation                 |

---

# 🎮 3. Controller Layer (Trigger Validation)

👉 You MUST use `@Valid`

```java
@PostMapping
public ResponseEntity<BaseResponse<OrderResponse>> createOrder(
        @Valid @RequestBody CreateOrderRequest request) {

    OrderResponse response = orderService.createOrder(request);

    return ResponseBuilder.success(
            HttpStatus.CREATED,
            "Order created successfully",
            response
    );
}
```

---

# ⚠️ What happens internally?

```
Request → @Valid triggers validation
        ↓
If invalid → MethodArgumentNotValidException thrown
        ↓
Handled by GlobalExceptionHandler
```

---

# 🌍 4. Global Validation Exception Handling (IMPORTANT)

Your current version handles only **one error** 👇

```java
exception.getBindingResult().getFieldError().getDefaultMessage()
```

❌ Problem: Only first error returned
✅ Fix: Return **all field errors**

---

## ✅ Improved Validation Handler

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<BaseResponse<Object>> handleValidation(
        MethodArgumentNotValidException ex,
        HttpServletRequest request) {

    List<Map<String, Object>> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> Map.of(
                    "code", "VALIDATION_FAILED",
                    "field", error.getField(),
                    "message", error.getDefaultMessage(),
                    "path", request.getRequestURI()
            ))
            .toList();

    return ResponseBuilder.error(
            HttpStatus.BAD_REQUEST,
            "Validation Failed",
            errors
    );
}
```

---

# 📤 Example Error Response (Multiple Errors)

```json
{
  "status": 400,
  "message": "Validation Failed",
  "data": null,
  "errors": [
    {
      "code": "VALIDATION_FAILED",
      "field": "productName",
      "message": "Product name is required",
      "path": "/orders"
    },
    {
      "code": "VALIDATION_FAILED",
      "field": "quantity",
      "message": "Quantity must be greater than 0",
      "path": "/orders"
    }
  ]
}
```

---

# 🧠 Advanced (Interview + Real World)

## ✅ 1. Nested Validation

```java
public class OrderRequest {

    @Valid
    @NotNull
    private CustomerRequest customer;
}
```

👉 Without `@Valid` → nested object won't be validated

---

## ✅ 2. Custom Validation Annotation

Example: Only allow specific product types

```java
@Constraint(validatedBy = ProductTypeValidator.class)
@Target({ FIELD })
@Retention(RUNTIME)
public @interface ValidProductType {
    String message() default "Invalid product type";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

---

## ✅ 3. Service Layer Validation (Business Validation)

👉 Not everything is annotation-based

```java
if (price > 100000) {
    throw new BusinessException("PRICE_TOO_HIGH", "Price exceeds limit");
}
```

---

# ⚠️ Common Mistakes (VERY IMPORTANT)

❌ Putting validation on Entity
✔ Always use DTO

❌ Forgetting `@Valid`
✔ Validation won’t trigger

❌ Returning raw error messages
✔ Always wrap in BaseResponse

❌ Handling only first error
✔ Always return list

---

# 🚀 Final Flow

```
Client Request
      ↓
Controller (@Valid)
      ↓
Validation fails → Exception
      ↓
GlobalExceptionHandler
      ↓
ResponseBuilder.error()
      ↓
BaseResponse (errors[])
```

---

# 🎯 Interview Summary

* Validation is handled by **Jakarta Bean Validation**
* Triggered using `@Valid`
* Exception: `MethodArgumentNotValidException`
* Best practice:

  * DTO + annotations
  * Global exception handling
  * Return structured error list

---