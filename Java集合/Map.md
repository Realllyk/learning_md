## HashMap实现原理
![[hash_implements.png]]

### 1. Java 7 及以前的 HashMap 结构

在 JDK 1.7 版本之前，`HashMap` 采用**数组 + 链表**的数据结构。

- `HashMap` 通过哈希算法计算键（`Key`）的哈希值，并将其映射到数组中的某个槽位（`bucket`）。
- 如果多个键映射到同一个槽位，则采用**链地址法**，即这些键值对将以**链表**的形式存储在同一槽位上。
- 由于链表的查找时间复杂度为 **O(n)**，当哈希冲突严重时，查询性能会大幅下降。
### 2. Java 8 之后的优化

在 JDK 1.8 版本，`HashMap` 对哈希冲突的处理方式进行了优化，引入了**红黑树（Red-Black Tree）**：
- **当链表长度超过 8** 时，自动将链表转换为**红黑树**，查询时间复杂度由 **O(n)** 降为 **O(log n)**，提高查询性能。
- **当红黑树节点数小于 6** 时，重新转换回链表，以降低存储开销。
- 红黑树的引入减少了大规模数据存储时的查询开销，使 `HashMap` 适用于更复杂的应用场景。

## 哈希冲突解决方法
在哈希表（Hash Table）中，**哈希冲突**（Hash Collision）是指多个键（Key）映射到相同的哈希桶（Bucket）。常见的哈希冲突解决方法如下：

### 1. 链接法（Chaining）
- 采用**链表**或其他数据结构（如平衡树）存储冲突的键值对。
- 多个哈希冲突的键值对共享同一个哈希桶，并通过**链表**连接。
- JDK 1.8 以后，当链表长度超过 **8**，会转换为**红黑树**以提高查找效率。

### 2. 开放寻址法（Open Addressing）

- 当发生哈希冲突时，寻找哈希表中的**另一个可用位置**存储键值对。
- 主要策略：
    - **线性探测（Linear Probing）**：每次冲突后查找下一个相邻的空闲槽。
    - **二次探测（Quadratic Probing）**：查找空闲槽时，步长呈二次方递增（1, 4, 9…）。
    - **双重散列（Double Hashing）**：使用**另一个哈希函数**计算偏移量，减少聚集效应。

### 3. 再哈希法（Rehashing）

- 发生哈希冲突时，使用**另一个哈希函数**重新计算哈希值，直至找到空槽。
    
- 适用于**特定应用场景**，但增加了计算开销。
    

### 4. 哈希桶扩容（Resizing）
- 当哈希冲突过多时，可以**动态扩展**哈希桶的数量。
- **重新计算**所有键值对的哈希值，并分配到新的哈希桶中，以降低冲突率。
- `HashMap` 默认负载因子为 **0.75**，超过阈值后自动扩容 **2 倍**。


## hashMap的put流程
![[hash_process.png]]


---


## HashMap 调用get方法安全吗
在使用 `HashMap` 的 `get` 方法时，需要注意以下几点：
### 1. 空指针异常（NullPointerException）
- 如果尝试使用 `null` 作为键调用 `get` 方法，而 `HashMap` **尚未初始化**（即 `null`），则会抛出 **空指针异常**。
- 但如果 `HashMap` **已初始化**，则 `null` 作为键是**允许的**，因为 `HashMap` 支持 `null` 作为键。

### 2.线程安全问题
- `HashMap` **不是线程安全**的，在**多线程环境**下，如果没有适当的同步措施，可能会导致**不可预测的行为**。
- 例如，在一个线程中调用 `get` 方法读取数据，而另一个线程正在修改 `HashMap`（如**插入或删除元素**），可能会导致读取到错误的结果，甚至抛出 `ConcurrentModificationException`。
- **解决方案**：
    - 如果需要**多线程环境下的** `HashMap` **替代方案**，可以使用 `ConcurrentHashMap`，它是**线程安全**的哈希表实现。

在多线程场景下，尽量避免直接使用 `HashMap`，而是选择 `ConcurrentHashMap` 以确保数据一致性和线程安全。


---


## 为什么String适合做Key
### 1. 为什么 `String` 适合作为 `HashMap` 的 Key？
`String` 是 `HashMap` 中最常用的 Key，其原因如下：

#### **1.1** `String` **是不可变对象（Immutable）**
- `String` 在 Java 中是 **不可变对象**，一旦创建就不会修改。
- `HashMap` 依赖 `hashCode()` 进行存储和查找，而 `String` 的 `hashCode()` **始终不变**，避免了哈希值变化导致的 `get()` 失败。
- 其他可变对象（如 `ArrayList`、`HashSet`）如果作为 `Key`，在 `put()` 之后修改内容，`hashCode()` 可能会变化，导致 `HashMap` 失效。
    
#### **1.2** `String` 的 hashCode()**` 计算结果稳定

- `String` 的 `hashCode()` 方法经过优化，计算速度快，且哈希分布均匀，降低了哈希冲突的概率。
- `String` 的 `equals()` 方法实现高效，适合作为 `HashMap` 的 Key。
    

#### **1.3** `String` **是** `final` **类，安全性高**

- `String` 作为 `final` 类，无法被继承，避免了子类重写 `hashCode()` 或 `equals()` 方法导致的问题。
    

### 2. 使用可变对象作为 Key 可能导致的问题

如果使用**可变对象**（如 `ArrayList`、`Person` 类）作为 `HashMap` 的 Key，可能会导致以下问题：

#### **2.1** `hashCode()` **变化导致** `HashMap` **查找失败**

- `HashMap` 依赖 `hashCode()` 计算存储位置。
- 如果 `Key` 在 `put()` 之后被修改，使得 `hashCode()` 发生变化，`get()` 方法将无法找到正确的 `Bucket`，导致查找失败。
    

#### **示例代码**

```java
import java.util.HashMap;

class Person {
    String name;
    
    Person(String name) {
        this.name = name;
    }
    
    @Override
    public int hashCode() {
        return name.hashCode();
    }
    
    @Override
    public boolean equals(Object obj) {
        return obj instanceof Person && ((Person) obj).name.equals(this.name);
    }
}

public class HashMapTest {
    public static void main(String[] args) {
        HashMap<Person, String> map = new HashMap<>();
        Person key = new Person("Alice");
        map.put(key, "Data");

        // 修改 key 的 name
        key.name = "Bob";
        
        // 尝试获取原始 key
        System.out.println(map.get(new Person("Alice"))); // 可能返回 null
        System.out.println(map.get(key)); // 可能返回 null
    }
}
```

#### **问题分析**

1. `map.put(key, "Data")` 时，`key` 的 `hashCode()` 计算基于 `name="Alice"`。
2. 之后 `key.name = "Bob"`，导致 `hashCode()` 发生变化。
3. `get(new Person("Alice"))` 计算出的 `hashCode()` 仍然是 `Alice` 的，但 `HashMap` 仍然存储在 `Bob` 的 `Bucket` 中，导致 `get()` 失败。
    

#### **2.2** `equals()` **与** `hashCode()` **不一致，导致** `HashMap` **结构混乱**

- `HashMap` 依赖 `hashCode()` 定位 `Bucket`，再用 `equals()` 查找具体 `Key`。
- 如果 `hashCode()` 变了，但 `equals()` 仍然认为是相同对象，可能导致 `HashMap` 结构混乱，甚至出现重复 `Key`。
    

#### **2.3** `HashMap.keySet()` **仍然持有修改后的对象**
- `HashMap` 存储的是 `Key` **的引用**，而不是拷贝。
- 修改 `Key` 之后，`keySet()` 仍然引用修改后的对象，但 `get()` 可能查找失败。

#### **示例代码**
```java
System.out.println(map.keySet());
```

**输出**：
```css
[Person{name='Bob'}]
```
但 `get(new Person("Alice"))` 仍然找不到值。

### 3. 解决方案

#### ✅ **方法 1：使用不可变对象作为 Key**
- `String`、`Integer`、`UUID` 等不可变对象适合作为 `Key`。

```
HashMap<String, String> map = new HashMap<>();
map.put("Alice", "Data"); // 永远不会因 Key 变更导致 get 失败
```

#### ✅ **方法 2：确保 Key 在存入** `HashMap` **后不可变**
- **禁止修改** `**Key**` **的字段**，如：

```java
class ImmutableKey {
    private final String name; // 使用 final 防止修改

    ImmutableKey(String name) {
        this.name = name;
    }

    @Override
    public int hashCode() {
        return name.hashCode();
    }
    
    @Override
    public boolean equals(Object obj) {
        return obj instanceof ImmutableKey && ((ImmutableKey) obj).name.equals(this.name);
    }
}
```

#### ✅ **方法 3：如果** `Key` **必须修改，先** `remove()` **再** `put()`
- **先删除旧 Key，再插入新 Key**，确保 `Key` 存入正确的 `Bucket`。
```
map.remove(key);
key.name = "Bob";
map.put(key, "Data");
```

### 4. 总结

| 问题                                             | 解决方案                               |
| ---------------------------------------------- | ---------------------------------- |
| `hashCode()` 变更导致 `get()` 失败                   | 使用不可变对象（如 `String`）作为 `Key`        |
| `equals()` 与 `hashCode()` 不一致导致 `HashMap` 结构混乱 | 确保 `Key` 在 `put()` 后不可变            |
| `keySet()` 持有修改后的对象，但 `get()` 失败               | 先 `remove()` 旧 Key，再 `put()` 新 Key |
|                                                |                                    |

使用 `String` 作为 `HashMap` 的 `Key`，可以**确保 Key 的稳定性**，避免 `hashCode()` 变化导致的查找失败，提高 `HashMap` 的正确性和效率。


---


## 重写HashMap的key类型equals方法不当会出现什么问题

默认情况下，如果一个类**没有重写 `equals()` 和 `hashCode()` 方法**，那么 `equals()` 返回 `true` 时，`hashCode()` **一定相等**。
因为默认情况下，`euqals()`方法是判断两个对象的内存地址是否相同；而`hashcode()`则默认依赖内存地址。

### 问题
如果**重写了** `**equals()**` **方法但没有重写** `**hashCode()**` **方法**，可能会导致：
1. `**equals()**` **返回** `**true**`**，但** `**hashCode()**` **不相等**。
2. `HashMap` 可能会在不同的 `Bucket` 中存储多个“相等”的键，导致无法正确覆盖已有数据。
3. 违反 `HashMap` 只能存储唯一 `Key` 的规范，可能引发数据丢失或重复存储问题。


---


## HashMap的扩容机制概述

首先来介绍 HashMap 是如何定位buckect的。HashMap的capacity总是 **2^n** ，那么就可以键值key计算得到的HashCode的 **n-1** 位来定位bucket。**HashMap (capacity 为 2^n) 寻找 bucket 的机制是：**
- 计算 `hashCode`，通常还会经过 `hash()` 方法进行扰动，减少 hash 冲突。
- 使用 **位运算** 来定位 bucket：`index = hash & (capacity - 1)`。

这一机制是基于 **位运算** 的 **高效替代取模运算**，因为 `& (capacity - 1)` 比 `% capacity` 要快得多。


### 1. HashMap的扩容规则

HashMap 在允许最大载负固定为 **0.75**，即当元素数量超过总容量的 **75%** 时，将会触发扩容操作。

### 2. 扩容过程
HashMap 扩容主要分为 **两步**：
1. **增加哈希表的容量**：容量变为原来的 **2 倍**。
2. **重新分配元素位置**：将旧表中的元素放入新的哈希表。

### 3. HashMap 扩容时的算法

由于我们使用的是 **2倍扩容**（表的容量被伸长为原来的 2 倍），因此元素的新位置仅有 **两种可能**：
- 保持元素在原位置不变
- 移动到 `原位置 + oldCap`

### 4. 具体例子
假设 HashMap 从 **16 扩容到 32**，那么 `n-1` 的乘码 (`mask`)也会增加 **一个位元为1**（红色部分）。
实际计算系数时， `index = hash & (n - 1)`，由于 `n` 变为 **2 倍**，那么 `n-1` 乘码将会在二进制中增加 **一个 1**。
这个额外的 `1`影响系数计算，致使部分元素的新位置变为 `原位置 + oldCap`。

**示例：**
- `hash1`计算后 `index = 5`，保持在原位置。
- `hash2`计算后 `index = 21 = 5 + 16`，被移动到 `oldCap` 之后的新位置。

### 5. HashMap 扩容不需要重新计算 hash 值的原因

扩容时，只需要判断 **hash 值新增的那一位是 0 还是 1**：
- **如果是 0**，元素的索引 **不变**。
- **如果是 1**，新索引 = `原索引 + oldCap`。

这样就避免了 **重新计算 hashCode**，提高了扩容效率。

![[hash_capacity.png]]


---


## HashMap和HashTable的区别

### **(1) 线程安全性**
- **HashMap 线程不安全**，效率较高。适用于单线程环境或需要手动同步的场景。
- **Hashtable 线程安全**，但效率较低。其内部方法使用 `synchronized` 关键字修饰，适用于多线程环境。


### **(2)** `**null**` **处理**
- **HashMap**：允许 `null` 作为 **key**（只能有一个），允许多个 `null` 作为 **value**。
- **Hashtable**：**不允许** `null` 作为 **key** 或 **value**。

### **(3) 初始容量与扩容机制**
- **HashMap**：
    - 默认 **初始容量** 为 `16`，扩容为 **原来的 2 倍**。
    - 创建时如果提供了初始容量，则最终容量是 **大于等于该值的 2 的幂**。
- **Hashtable**：
    - 默认 **初始容量** 为 `11`，扩容为 **原容量的 2n + 1**。
    - 创建时如果提供了初始容量，会**直接使用**指定的值，而不会调整为 2 的幂次。
        

### **(4) 底层数据结构**
- **HashMap**：数组 + **链表/红黑树**（JDK 1.8+）。
    - **链表转红黑树**：当链表长度超过 **8**，且桶大小 ≥ 64 时，转换为红黑树，否则保持链表。
- **Hashtable**：数组 + **链表**，无红黑树优化。

### **(5) 适用场景**
- **单线程环境**：优先使用 `HashMap`，性能更好。
- **多线程环境**：
    - 需要线程安全时可以使用 `Hashtable`，但一般推荐 `ConcurrentHashMap` 代替。


---

## ConcurrentHashMap 实现原理
### JDK 1.7 ConcurrentHashMap 实现原理

#### 1. 数据结构
JDK 1.7 的 `ConcurrentHashMap` 采用 **数组 + 链表** 结构，并使用 **分段锁（Segment 锁）** 来实现并发控制。

##### **(1) 组成部分**
- **Segment[]**（大数组）：每个 `Segment` 负责一部分数据，相当于一个小的 `HashMap`，每个 `Segment` 拥有独立的锁。
- **HashEntry[]**（小数组）：每个 `Segment` 内部包含多个 `HashEntry`，其中每个 `HashEntry` 代表链表中的一个元素。

##### **(2) 细节解析**
- **Segment 作用**：
    - `Segment` 继承自 `ReentrantLock`，用于保证并发安全。
    - 允许多个线程同时访问 `ConcurrentHashMap`，只要它们访问的 `Segment` 不同。
- **HashEntry 作用**：
    - 存储键值对数据。
    - 采用链表解决哈希冲突。

#### 2. 并发控制
JDK 1.7 `ConcurrentHashMap` 采用 **分段锁** 实现并发访问：
- `ConcurrentHashMap` 被划分为 **多个 Segment**，每个 Segment 维护自己的 `HashEntry` 数组。
- 每个 `Segment` 有一把 `ReentrantLock`，当线程访问某个 `Segment` 时，只会锁住该 `Segment`，不会影响其他 `Segment`。
- 这样可以 **大幅提高并发能力**，支持多个线程同时访问不同 `Segment` 内的数据。

#### 3. 主要优势
- **支持高并发**：多个线程可以并发访问不同 `Segment`，避免整个 `Map` 加锁导致的性能下降。
- **降低锁粒度**：相比 `Hashtable` 的全局同步锁，`ConcurrentHashMap` 采用 **局部锁**（分段锁），提升性能。
- **线程安全**：使用 `ReentrantLock` 确保 `Segment` 内部操作的安全性。

![[1_7_concurrent.png]]


### JDK 1.8 ConcurrentHashMap 实现原理
在 JDK 1.7 中，ConcurrentHashMap 虽然是线程安全的，但由于其底层实现是数组 + 链表的形式，在数据量较大时访问速度较慢，因为需要遍历整个链表。而 JDK 1.8 对 ConcurrentHashMap 进行了优化，采用了 **数组 + 链表/红黑树** 结构。

#### 实现结构
JDK 1.8 的 ConcurrentHashMap 主要通过 **volatile + CAS（Compare-And-Swap）+ synchronized** 来实现线程安全。

#### 添加元素的步骤
1. 先判断容器是否为空：
    - 为空时，使用 **volatile + CAS** 进行初始化。
2. 如果容器不为空，则根据存储的元素计算存放位置：
    - 计算位置为空时，使用 **CAS** 设置该节点。
    - 计算位置不为空，判断该节点是否为链表：
        - 是链表时，使用 **synchronized** 进行加锁，然后遍历链表插入数据，并检查是否需要转换为红黑树。
        - 不是链表（即已是红黑树），直接插入。


#### 主要优化点
- **减少锁的粒度**：在 JDK 1.7 中，ConcurrentHashMap 采用 **Segment** 作为分段锁，而 JDK 1.8 取消了分段锁，改用 **节点级别的锁**，减少锁的竞争，提高并发性能。
- **优化查询效率**：
    - JDK 1.7 采用数组 + 链表结构，查询时间复杂度为 **O(n)**。
    - JDK 1.8 在链表节点超过 **8** 个时，将链表转换为 **红黑树**，查询时间复杂度降至 **O(log n)**，提升了查询效率。


#### 已经用了 synchronized，为什么还要用 CAS 呢？

**ConcurrentHashMap** 使用 **synchronized** 和 **CAS（Compare-And-Swap）** 这两种手段来保证线程安全，主要是基于**权衡性能和锁竞争的考虑**。在某些操作中使用 **synchronized**，而在某些情况下使用 **CAS**，主要取决于锁的竞争程度。

##### CAS 与 synchronized 的使用场景
- 在 `putVal` 方法中：
  - 如果计算出的 **hash 位置为空**，可以直接使用 **CAS** 进行设置。  
    - 因为**空位置插入**时，不涉及数据迁移，冲突概率低，使用 CAS 可以减少锁竞争，提高性能。
  - 如果 **hash 位置不为空**，则可能发生 **哈希碰撞**，这时：
    - 需要遍历链表或红黑树，插入数据；
    - 这种情况下**锁竞争激烈**，因此采用 **synchronized** 进行加锁。

##### 选择 synchronized 还是 CAS？
- **当发生哈希碰撞较少**时，使用 CAS 操作，**减少锁竞争**，提升并发性能；
- **当哈希碰撞较多**（如容器接近饱和或有大量线程访问）时，使用 **synchronized** 进行加锁，保证操作的正确性。

##### 结论
ConcurrentHashMap 结合 **CAS** 和 **synchronized** 来平衡**性能与安全性**：
- **CAS 适用于低冲突情况**，如插入新元素到空位置，避免不必要的锁竞争；
- **synchronized 适用于高冲突情况**，如哈希碰撞严重时，提高操作的稳定性。

这种机制使得 ConcurrentHashMap 既能保持**高并发性能**，又能保证**线程安全**。

