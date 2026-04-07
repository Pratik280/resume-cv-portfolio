# 📘 Consistent API Response Design (Spring Boot)

## ✅ Why do we need a Base Response?

Without a standard format:

* Every API returns different JSON
* Hard for frontend to handle
* Debugging becomes messy

👉 Solution: **Wrap every response in a common structure**

---

## 📦 BaseResponse (Generic Wrapper)

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class BaseResponse<T> {

    private int status;              // HTTP status code
    private String message;          // Success/Error message

    // For success responses
    private T data;

    // For error responses
    private List<Map<String, Object>> errors;
}
```

### ✔ Key Idea

| Scenario | Field Used |
| -------- | ---------- |
| Success  | `data`     |
| Error    | `errors`   |

---

## 🛠 ResponseBuilder (Centralized Builder)

👉 Avoid repeating response creation logic everywhere

```java
public class ResponseBuilder {

    public static <T> ResponseEntity<BaseResponse<T>> success(
            HttpStatus status, String message, T data) {

        BaseResponse<T> response = BaseResponse.<T>builder()
                .status(status.value())
                .message(message)
                .data(data)
                .errors(null)
                .build();

        return new ResponseEntity<>(response, status);
    }

    public static <T> ResponseEntity<BaseResponse<T>> error(
            HttpStatus status, String message, List<Map<String, Object>> errors) {

        BaseResponse<T> response = BaseResponse.<T>builder()
                .status(status.value())
                .message(message)
                .data(null)
                .errors(errors)
                .build();

        return new ResponseEntity<>(response, status);
    }
}
```

---

# 🌍 Global Exception Handling

## 🔥 Why?

Instead of writing try-catch in every controller:
👉 Handle all exceptions in one place

---

## 🧠 @RestControllerAdvice

* Global exception handler
* Applies to all controllers
* Returns JSON directly

---

## ⚙️ GlobalExceptionHandler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 404 - Resource Not Found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<BaseResponse<Object>> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {

        return ResponseBuilder.error(
                HttpStatus.NOT_FOUND,
                ex.getMessage(),
                List.of(errorItem("RESOURCE_NOT_FOUND", ex.getMessage(), request))
        );
    }

    // 400 - Invalid Request Body
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<BaseResponse<Object>> handleNotReadable(
            HttpMessageNotReadableException ex, HttpServletRequest request) {

        return ResponseBuilder.error(
                HttpStatus.BAD_REQUEST,
                "Request body is missing or malformed",
                List.of(errorItem("INVALID_REQUEST_BODY", ex.getMessage(), request))
        );
    }

    // 400 - Validation Errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<BaseResponse<Object>> handleValidation(
            MethodArgumentNotValidException ex, HttpServletRequest request) {

        String errorMsg = ex.getBindingResult()
                            .getFieldError()
                            .getDefaultMessage();

        return ResponseBuilder.error(
                HttpStatus.BAD_REQUEST,
                "Validation Failed",
                List.of(errorItem("VALIDATION_FAILED", errorMsg, request))
        );
    }

    // Business Exception
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<BaseResponse<Object>> handleBusiness(
            BusinessException ex, HttpServletRequest request) {

        return ResponseBuilder.error(
                HttpStatus.INTERNAL_SERVER_ERROR,
                "Business Error",
                List.of(errorItem(ex.getErrorCode(), ex.getMessage(), request))
        );
    }

    // Generic Exception
    @ExceptionHandler(Exception.class)
    public ResponseEntity<BaseResponse<Object>> handleAll(
            Exception ex, HttpServletRequest request) {

        return ResponseBuilder.error(
                HttpStatus.INTERNAL_SERVER_ERROR,
                "Unexpected Error",
                List.of(errorItem("INTERNAL_ERROR", ex.getMessage(), request))
        );
    }

    // Helper method
    private Map<String, Object> errorItem(
            String code, String message, HttpServletRequest request) {

        return Map.of(
                "code", code,
                "message", message,
                "path", request.getRequestURI()
        );
    }
}
```

---

# 🧩 Layered Architecture (Clean Structure)

```
Controller → Service → Repository → Database
           ↓
          DTO
```

---

## 🎮 Controller Layer

```java
@RestController
@RequestMapping("/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @GetMapping
    public ResponseEntity<BaseResponse<List<OrderResponse>>> getAllOrders() {

        return ResponseBuilder.success(
                HttpStatus.OK,
                "All orders fetched successfully",
                orderService.getAllOrders()
        );
    }
}
```

---

## ⚙️ Service Layer

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;
    private final ModelMapper modelMapper;

    @Override
    public List<OrderResponse> getAllOrders() {

        List<OrderResponse> orders = orderRepository.findAll()
                .stream()
                .map(order -> modelMapper.map(order, OrderResponse.class))
                .toList();

        if (orders.isEmpty()) {
            throw new ResourceNotFoundException("No orders found");
        }

        return orders;
    }
}
```

---

## 🗄 Repository Layer

```java
@Repository
public interface OrderRepository extends JpaRepository<OrderEntity, Long> {
}
```

---

## 🧾 Entity

```java
@Entity
@Getter
@Setter
public class OrderEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;
}
```

---

## 📤 DTO (Response)

```java
@Getter
@Setter
@NoArgsConstructor
public class OrderResponse {

    private Long id;
    private String productName;
}
```

---

# 🔄 Flow Summary

```
Client Request
      ↓
Controller
      ↓
Service
      ↓
Repository (DB)
      ↓
Service (map Entity → DTO)
      ↓
Controller (wrap in BaseResponse)
      ↓
Client Response
```

---

# ❗ Error Flow

```
Service throws Exception
      ↓
GlobalExceptionHandler catches it
      ↓
ResponseBuilder.error()
      ↓
BaseResponse(errors filled)
      ↓
Client gets structured error JSON
```

---

# 🎯 Final Response Examples

## ✅ Success Response

```json
{
  "status": 200,
  "message": "All orders fetched successfully",
  "data": [
    {
      "id": 1,
      "productName": "Laptop"
    }
  ],
  "errors": null
}
```

---

## ❌ Error Response

```json
{
  "status": 404,
  "message": "No orders found",
  "data": null,
  "errors": [
    {
      "code": "RESOURCE_NOT_FOUND",
      "message": "No orders found",
      "path": "/orders"
    }
  ]
}
```

---

# 🚀 Interview Points (VERY IMPORTANT)

* `@RestControllerAdvice` = global exception handling
* `@ExceptionHandler` = map exception → response
* Use **generic BaseResponse<T>**
* Separate:

  * Entity (DB)
  * DTO (API)
* Never return Entity directly
* Use **ResponseBuilder** to avoid duplication
* Consistent API contract = production-ready design

---