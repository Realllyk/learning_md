## 简介
`Stream API` 是 Java 8 引入的一种新特性，主要用于处理**集合（Collection**的数据。它提供了一种声明式（Declarative）的方式来操作数据，就像 SQL 操作数据库一样，而不用写传统的 `for` 循环来遍历和处理集合。


## Stream 的基本操作流程

Stream 主要有三大操作：

1. **创建流**（来源）：从集合、数组、文件、甚至 `Stream.generate()` 等创建流。
    
2. **中间操作**（转换）：对数据进行处理（如 `map`、`filter`、`sorted`），但不会立刻执行。
    
3. **终结操作**（收集）：如 `collect()`、`forEach()`、`count()`，真正执行流操作。

---

## 1. Stream 的创建

Stream 可以从**集合**、**数组**、**指定范围的数据**创建：

```java
import java.util.*;
import java.util.stream.*;

public class StreamExample {
    public static void main(String[] args) {
        // 1. 从 List 创建 Stream
        List<String> list = Arrays.asList("apple", "banana", "cherry");
        Stream<String> stream1 = list.stream();

        // 2. 从数组创建 Stream
        String[] array = {"Java", "Python", "C++"};
        Stream<String> stream2 = Arrays.stream(array);

        // 3. 使用 Stream.of() 直接创建
        Stream<Integer> stream3 = Stream.of(1, 2, 3, 4, 5);

        // 4. 创建范围流
        IntStream intStream = IntStream.range(1, 10);  // 1~9

        // 5. 无限流 (生成 5 个随机数)
        Stream<Double> randomStream = Stream.generate(Math::random).limit(5);
    }
}
```

---

## 2. Stream 的中间操作
**中间操作**用于转换数据，不会立刻执行，只有在终结操作时才真正处理。

### 1. `filter()`：过滤数据

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date");
List<String> filtered = words.stream()
    .filter(s -> s.startsWith("b")) // 只保留以 'b' 开头的元素
    .collect(Collectors.toList());  // 终结操作，收集结果

System.out.println(filtered); // 输出: [banana]
```

### 2. `map()`：映射（转换）数据

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
List<Integer> nameLengths = names.stream()
    .map(String::length) // 把名字转换为长度
    .collect(Collectors.toList());

System.out.println(nameLengths); // 输出: [5, 3, 7]
```

### 3. `sorted()`：排序

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1);
List<Integer> sortedNumbers = numbers.stream()
    .sorted()
    .collect(Collectors.toList());

System.out.println(sortedNumbers); // 输出: [1, 2, 5, 8]
```

### 4. `distinct()`：去重

```java
List<Integer> nums = Arrays.asList(1, 2, 2, 3, 4, 4, 5);
List<Integer> uniqueNums = nums.stream()
    .distinct()
    .collect(Collectors.toList());

System.out.println(uniqueNums); // 输出: [1, 2, 3, 4, 5]
```

### 5. `limit()` & `skip()`：限制/跳过元素
```java
List<String> items = Arrays.asList("A", "B", "C", "D", "E");

List<String> limited = items.stream().limit(3).collect(Collectors.toList());
System.out.println(limited); // 输出: [A, B, C]

List<String> skipped = items.stream().skip(2).collect(Collectors.toList());
System.out.println(skipped); // 输出: [C, D, E]
```

## 3. Stream 的终结操作

终结操作会真正执行流的计算，如**遍历、计数、聚合、收集**等。

### 1. `forEach()`：遍历输出

```java
List<String> items = Arrays.asList("A", "B", "C");
items.stream().forEach(System.out::println);
```

### 2. `count()`：统计元素数量

```java
long count = items.stream().count();
System.out.println(count); // 输出: 3
```

### 3. `collect()`：收集结果（如转换回 `List`）

```java
List<String> upperCaseList = items.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

System.out.println(upperCaseList); // 输出: [A, B, C]
```


### 4. `toArray()`：转换为数组
1. 转换为基本类型数组： 先转换为基本类型对应的流，再调用toArray
   ```java
int[] intArray = Arrays.asList(1, 2, 3, 4, 5)
    .stream()
    .mapToInt(Integer::intValue)
    .toArray();

System.out.println(Arrays.toString(intArray)); // 输出: [1, 2, 3, 4, 5]
```

2. 引用类型数组：需要传入对应对象数组的::new函数引用，来指明创建的是哪种类型的数组
```java
class A {
    String name;
    A(String name) { this.name = name; }
    @Override
    public String toString() { return name; }
}

List<A> list = Arrays.asList(new A("Apple"), new A("Banana"), new A("Cherry"));
A[] array = list.stream().toArray(A[]::new);
System.out.println(Arrays.toString(array)); // 输出: [Apple, Banana, Cherry]
```

### 5. `reduce()`：聚合计算

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .reduce(0, Integer::sum);  // 0 是初始值

System.out.println(sum); // 输出: 15
```

---

## 4. 并行流（Parallel Stream）

如果数据量大，可以使用**并行流**来提升处理速度：

```java
List<Integer> bigList = IntStream.range(1, 1000000).boxed().collect(Collectors.toList());

long startTime = System.currentTimeMillis();
long sum1 = bigList.stream().mapToLong(i -> i).sum();
long endTime = System.currentTimeMillis();
System.out.println("普通流耗时：" + (endTime - startTime) + " ms");

startTime = System.currentTimeMillis();
long sum2 = bigList.parallelStream().mapToLong(i -> i).sum();
endTime = System.currentTimeMillis();
System.out.println("并行流耗时：" + (endTime - startTime) + " ms");
```

**并行流（`parallelStream()`）**会自动利用多核 CPU，提高数据处理效率。

---

## 总结

- **Stream 让集合操作更简单、声明式，避免** ****``**** **循环。**
- **主要有三类操作：**
    1. **创建流**：`stream()`、`Stream.of()`、`Arrays.stream()`
    2. **中间操作（转换）**：`filter()`、`map()`、`sorted()`、`distinct()`
    3. **终结操作（收集）**：`collect()`、`forEach()`、`reduce()`
