# Java Most Asked Stream API Coding Questions

---

## 1. Filter Even Numbers

**Problem:** Given a list of integers, return only even numbers.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

**Explanation:**

* `filter` keeps only even numbers
* `collect` gathers results into a list

---

## 2. Find Maximum

**Problem:** Find the maximum value.

```java
Optional<Integer> max = numbers.stream()
    .max(Integer::compare);
```

**Explanation:**
Returns the maximum wrapped in `Optional`.

---

## 3. Sum of Elements

**Problem:** Calculate sum.

```java
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
```

**Explanation:**
Converts to `IntStream` and uses `sum()`.

---

## 4. List of Names to Uppercase

**Problem:** Convert strings to uppercase.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 5. Sort List

**Problem:** Sort integers.

```java
List<Integer> sortedNumbers = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
```

---

## 6. Count Elements

**Problem:** Count numbers > 5.

```java
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();
```

---

## 7. Get Distinct Elements

**Problem:** Remove duplicates.

```java
List<Integer> distinctNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
```

---

## 8. Reduce to Sum

**Problem:** Reduce to total.

```java
int total = numbers.stream()
    .reduce(0, Integer::sum);
```

---

## 9. Find Any

**Problem:** Return any element.

```java
Optional<Integer> anyElement = numbers.stream()
    .findAny();
```

---

## 10. List First Names

**Problem:** Extract first names.

```java
List<String> fullNames = Arrays.asList(
    "Alice Johnson", "Bob Harris", "Charlie Lou"
);

List<String> firstNames = fullNames.stream()
    .map(name -> name.split(" ")[0])
    .collect(Collectors.toList());
```

---

## 11. All Match

**Problem:** Check if all numbers are positive.

```java
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);
```

---

## 12. None Match

**Problem:** Check if no numbers are negative.

```java
boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);
```

---

## 13. Find First

**Problem:** Get first element.

```java
Optional<Integer> first = numbers.stream()
    .findFirst();
```

---

## 14. FlatMap for Nested Lists

**Problem:** Flatten nested lists.

```java
List<List<Integer>> nestedNumbers = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4, 5)
);

List<Integer> flatList = nestedNumbers.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
```

---

## 15. Grouping Elements

**Problem:** Group users by age.

```java
Map<Integer, List<User>> usersByAge = users.stream()
    .collect(Collectors.groupingBy(User::getAge));
```

---

## 16. Peek Elements

**Problem:** Print elements during processing.

```java
List<Integer> peekedAtNumbers = numbers.stream()
    .peek(System.out::println)
    .collect(Collectors.toList());
```

---

## 17. Limit Stream

**Problem:** Limit to first 3 elements.

```java
List<Integer> limited = numbers.stream()
    .limit(3)
    .collect(Collectors.toList());
```

---

## 18. Skip Elements

**Problem:** Skip first 2 elements.

```java
List<Integer> skipped = numbers.stream()
    .skip(2)
    .collect(Collectors.toList());
```

---

## 19. Convert to Set

**Problem:** Remove duplicates using Set.

```java
Set<Integer> uniqueNumbers = numbers.stream()
    .collect(Collectors.toSet());
```

---

## 20. Summarizing Statistics

**Problem:** Get stats (min, max, avg, sum, count).

```java
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();
```

---

If you want, I can:

* Turn this into **interview notes (short + easy to revise)**
* Add **time complexity + when to use each**
* Or convert into **flashcards / cheat sheet**
