# Java Interview Questions

---

## 1. Java Basics & Core Concepts

- What is Java?
- What are the features of Java?
- Is Java 100% object-oriented?
- What are the advantages of Java being partially object-oriented?
- What is the use of public static void main(String[] args)?
- What happens if we don’t declare main as static?
- Can we override the main method?
- Can we overload the main method?
- Can we print something on console without main method?
- What is method hiding?
- Difference between == and .equals()
- What is Enum in Java?
- What is import and static import?
- What is wildcard parameter in Java?

---

## 2. JVM, JDK, JRE & Memory

- What is JVM and why is it important?
- Difference between JDK, JRE, and JVM
- Can a Java application run without JRE?
- Is it possible to have JDK without JRE?
- What are the key components of JVM architecture?
- What memory storages are available in JVM?
- Which is faster: heap or stack, and why?
- What is Java Memory Model?

---

## 3. Garbage Collection & Memory Management

- How does garbage collection work in Java?
- What algorithm does JVM use for garbage collection?
- Role of finalize() method
- How can memory leaks occur in Java?

---

## 4. Data Types & Wrapper Classes

- Difference between primitive and non-primitive data types
- Can we declare pointers in Java?
- What are wrapper classes?
- What is autoboxing and unboxing?
- Why use wrapper classes in collections?
- Scenarios where autoboxing causes unexpected behavior
- Can autoboxing/unboxing cause NullPointerException?
- What are wrapper class pools?

---

## 5. Exception Handling

- What is Exception?
- What is Exception Handling?
- Role of try, catch, and finally
- Does finally execute if return is used?
- When will the finally block not execute?
- Can we execute a program without catch block?
- Can we write multiple finally blocks?
- What is stack trace?
- Handling multiple exceptions in a single catch
- Difference between checked and unchecked exceptions
- What is throws keyword?
- try with resources
- Custom Exception
- Performance impact of exception handling
- Difference between Throwable and Exception

---

## 6. Strings & Immutability

- What is String pool?
- Difference between String, StringBuilder, StringBuffer
- Why is String immutable?
- String literal vs String object
- When is StringBuffer better than String?
- What does immutability mean?
- How to create an immutable class?
- String.join() use

---

## 7. OOP Concepts (Deep Dive)

- What is inheritance in Java?
- Why multiple inheritance is not allowed?
- Composition vs inheritance
- Why composition is preferred over inheritance?
- Association, aggregation, and composition differences
- What is polymorphism in Java?
- What is encapsulation and its benefits?
- What is abstraction in Java?
- Difference between interface and abstract class
- When to use interface vs inheritance
- Can constructors be polymorphic?
- What is dynamic method dispatch?

---

## 8. Constructors & Object Creation

- What is a constructor?
- Can constructors be overloaded?
- Ways to create an object in Java
- Can a class extend itself?

---

## 9. Access Modifiers & Keywords

- What are access modifiers?
- Can a top-level class be private or protected?
- Why use getters/setters instead of public fields?
- What is static keyword used for?
- Can static block throw exception?
- Can static methods access non-static members?
- Can this and super be used in static context?
- What is final keyword and its impact?
- What is volatile keyword?

---

## 10. Singleton & Design Patterns

- What is Singleton class?
- How to create Singleton class?
- Are Singleton implementations thread-safe?
- What is fail-fast and fail-safe iterator?
- What is idempotency?
- What is Circuit Breaker Design Pattern?
- How have you implemented Circuit Breaker using Resilience4j?

---

## 11. Collections Framework

- What is Java Collections Framework?
- Enhancements in Collections in Java 8
- Difference between Iterator and ListIterator
- Common methods in Collection interface
- Algorithms used in Arrays.sort and Collections.sort
- ArrayList vs LinkedList internal implementation
- How HashSet handles duplicates
- Why override equals and hashCode together?
- TreeSet vs HashSet vs LinkedHashSet use cases
- Internal working of HashMap
- HashMap changes in Java 8
- HashMap vs TreeMap
- Using objects/classes as keys in maps
- How to choose the right collection?
- How ConcurrentHashMap works internally?
- Difference between ConcurrentHashMap before and after Java 8

---

## 12. Concurrency & Multithreading

- What is a thread?
- What is multithreading?
- Difference between Runnable and Thread
- Thread safety techniques in Java
- What is synchronized keyword?
- wait() and notify() use cases
- Challenges in multithreading
- How to handle concurrent updates
- What is Exchanger class?
- What is volatile keyword in multithreading?
- What is Java Memory Model in multithreading?

---

## 13. Java 8+ Features

- What are new features in Java 8?
- What is functional interface?
- Can functional interface extend another interface?
- Advantages of @FunctionalInterface
- Why Optional, Lambda, and Streams were introduced?
- Difference between filter and map
- Difference between intermediate and terminal operations
- Performance difference between for-loop and streams
- When to prefer loops vs streams
- Interface changes from Java 7 to Java 8
- What are default methods and diamond problem?

---

## 14. Modern Java Versions

- What are new features in Java 11?
- What are new features in Java 17?
- What are new features in Java 21?
- What is Date and Time API in Java?

---

## Generics / Wildcard / Diamond brackets

- What is generics and why generics were introduced?
- Generic class and generic method.
- What is wildcard?
- What is Type Erasure?
- Difference between List<Object> and List<?>
- What is upper/lower bounded exdends/super wildcard?

---

## 18. Experience & Scenario-Based Questions

- Explain your project architecture
- Explain how your microservices communicate
- Explain how Kafka is used in your project
- Explain how Redis is used in your project
- Explain how you handle failures in production
- Explain how you ensure idempotency in APIs
- Explain performance issues you faced and how you solved them
- Explain concurrency issues you handled
- Explain your CI/CD pipeline
- Explain your deployment architecture

---