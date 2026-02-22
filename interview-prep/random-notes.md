## coding quesions


``` java
Collectors.groupingBy() – Short Notes

1. groupingBy(classifier)
   Groups elements based on key → value becomes List<T>

Example:
Map<Integer, List<Integer>> map =
list.stream()
.collect(Collectors.groupingBy(x -> x));

2. groupingBy(classifier, downstream)
   Groups elements and applies another collector (counting, mapping, toSet, etc.)

Example:
Map<Integer, Long> freq =
list.stream()
.collect(Collectors.groupingBy(x -> x, Collectors.counting()));

3. groupingBy(classifier, mapSupplier, downstream)
   Allows choosing Map implementation (LinkedHashMap, TreeMap, etc.) + downstream collector

Example:
Map<Integer, Long> freq =
list.stream()
.collect(Collectors.groupingBy(
x -> x,
LinkedHashMap::new,
Collectors.counting()
));
``` 

```java
Write a function:
 
class Solution {
    public int solution(int[] A);
}
 
 
that, given an array A of N integers, returns the smallest positive integer (greater than 0) that does not occur in A.
 
Examples
 
Given A = [1, 3, 6, 4, 1, 2], the function should return 5.
Given A = [1, 2, 3], the function should return 4.
Given A = [-1, -3], the function should return 1.

Assumptions
 
N is an integer within the range [1..100,000]
 
Each element of array A is an integer within the range [-1,000,000..1,000,000]
 
Solve this problem using Array.Streams of A

    public static int solution(int[] a){
        
        if(a.length < 1 ){
            throw new IllegalArgumentException("wrong array");
        }
        
        
        Set<Integer> positives = new TreeSet<>();
        
        for(int i : a){
            if(i > 0){
                positives.add(i);
            }
        }
        
        if(positives.size() < 1 ){
            return 1;
        }

        int counter = 1;
        for(int i : positives){
            if(i != counter){
                return counter;
            }
            counter++;
        }
        
        return counter;
    }
```

## interview data

- interview data:
- failfast failsafe (in exp maybe)
- idempotency
- What is the Circuit Breaker Design Pattern, and how have you implemented it using Resilience4 j?
- wildcard parameter in java
- odd rows in sql
- copy rows
- springboot question
- resttemplate vs webclient====
- lazy loading in hibernate
- Date and Time API
- springboot 
- coding logic
- cv