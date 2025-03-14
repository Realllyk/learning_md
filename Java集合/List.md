## 为什么ArrayList不是线程安全的
在并发环境下，ArrayList 会暴露三个问题： 1. 部分值为 `null`（我们并没有 `add null` 进去）。 2. 索引越界异常。 3. `size` 与 `add` 的数量不符。 ## 代码分析 为了知道这三种情况是怎么发生的，`ArrayList` 的 `add` 增加元素的代码如下：
```java 
public boolean add(E e) { 
	ensureCapacityInternal(size + 1); // Increments modCount!! 
	elementData[size++] = e; 
	return true; 
}

```

`ensureCapacityInternal()` 这个方法的详细代码我们可以暂时不看，它的作用就是判断如果当前的新元素加到列表尾后，列表 `elementData` 额外的大小是否满足，如果 `size + 1` 这个需求长度大于了 `elementData` 这个数组的长度，那么就要对这个数组进行扩容。

大体可以分为三步：
- 判断是否需要扩容，需要扩容时，调用 `grow` 方法进行扩容。
- 将数据放在 `size` 位置处（因为数组的下标是从 0 开始的）。
- 将当前集合的大小 `size` 加 1。

### 线程安全问题分析
下面我们分析在多线程情况下，这三种情况是如何产生的：
- **部分值为 `null`**：
    - 线程 A 进入 `add()` 扩容逻辑，假设 `size=9`，数组容量为 `10`，但还未扩容。
    - 此时 CPU 调度线程 B 执行 `add()`，B 也进入扩容逻辑，发现 `size=9`，仍未扩容，于是开始扩容。
    - 线程 B 扩容后，`size` 仍然是 `9`，然后将元素写入 `elementData[9]`，并递增 `size`。
    - 线程 A 继续执行 `add()`，由于已经扩容完成，它也向 `elementData[9]` 写入元素，导致覆盖，可能出现 `null`。
- **索引越界异常**：
    - 线程 A 和线程 B 几乎同时执行 `add()`，但 `size` 还未增长，同时访问 `elementData[size]`。
    - 由于 `size` 没有及时更新，可能导致访问超过数组最大长度，引发 `IndexOutOfBoundsException`。
- **`size` 和 `add` 的数量不符**：
    - 假设 `size=10`，但 `elementData` 只有 `10` 的空间，线程 A 进入扩容逻辑但未完成。
    - 线程 B 也进入 `add()` 并执行 `size++`，但由于扩容未完成，可能导致 `size` 递增两次或丢失更新。


--- 

## ArrayList的扩容机制

`ArrayList` 在添加元素时，如果当前元素个数已经达到了内部数组的容量上限，就会触发扩容操作。`ArrayList` 的扩容操作主要包括以下几个步骤：

1. **计算新的容量**：
   - 一般情况下，新的容量会扩大会原容量的 `1.5` 倍（在 JDK 10 之后，扩容策略做了调整）。
   - 然后检查是否超过了最大容量限制。

2. **创建新的数组**：
   - 根据计算得到的新容量，创建一个新的、更大的数组。

3. **元素复制**：
   - 将原数组中的元素逐个复制到新数组中。

4. **更新引用**：
   - `ArrayList` 内部的引用指向新的数组。

5. **完成扩容**：
   - 容量完成扩大，可以继续添加新元素。

### 扩容机制的影响

`ArrayList` 的扩容操作涉及到数组的复制和内存的重新分配，所以在频繁添加大量元素时，扩容操作可能会影响性能。为了减少扩容带来的性能损耗，可以在初始化 `ArrayList` 时预分配足够大的容量，避免频繁触发扩容操作。

### 为什么扩容倍数是 1.5？

扩容倍数选择 `1.5`，是因为 `1.5` 可以充分利用移位操作，减少浮点数计算带来的运算时间和算术开销。

```java
// 新容量计算
int newCapacity = oldCapacity + (oldCapacity >> 1);
```


### 扩容后的地址变化

`ArrayList` 类型的引用变量 `list` 在 **栈** 上，它指向 **堆** 上的 `ArrayList` 对象，而 `ArrayList` 对象内部维护了 `elementData` 这个 **数组引用**，这个引用最终指向真正存储数据的 **堆内存数组**。

当 `ArrayList` 触发扩容机制后，它的底层数组会重新分配内存，并且引用变量会指向新的数组地址。

```java
import java.lang.reflect.Field;
import java.util.ArrayList;

public class ArrayListTest {
    public static void main(String[] args) throws Exception {
        ArrayList<Integer> list = new ArrayList<>(2); // 初始化容量2
        list.add(1);
        list.add(2);

        // 获取原始 elementData 地址
        Field elementDataField = ArrayList.class.getDeclaredField("elementData");
        elementDataField.setAccessible(true);
        Object[] oldArray = (Object[]) elementDataField.get(list);
        System.out.println("扩容前 elementData 地址: " + oldArray);

        list.add(3); // 触发扩容

        Object[] newArray = (Object[]) elementDataField.get(list);
        System.out.println("扩容后 elementData 地址: " + newArray);

        System.out.println("是否是同一块内存？" + (oldArray == newArray));
    }
}

```

```css
扩容前 elementData 地址: [Ljava.lang.Object;@1b6d3586
扩容后 elementData 地址: [Ljava.lang.Object;@4554617c
是否是同一块内存？false
```

#### 总结
- **`ArrayList` 触发扩容后，底层 `elementData` 数组会指向新的地址**，旧数组会被垃圾回收。
- **`ArrayList` 变量本身的引用不会变**，它仍然是同一个对象，只是内部数组的引用更新了。
- **如果有其他地方持有旧 `elementData` 数组的引用，它仍然可以访问旧数据，直到 GC 回收它**。


---


## CopyOnWriteArrayList是如何实现的
### 1. CopyOnWriteArrayList 的线程安全机制

`CopyOnWriteArrayList` 底层使用的是**数组**来存储数据，并通过 `volatile` 关键字保证对该数组对象的可见性，确保线程可以及时获取最新数据。

```java
private transient volatile Object[] array;
```

### 2. 插入操作的线程安全

在写入操作时，`CopyOnWriteArrayList` 使用 **ReentrantLock**（可重入锁）保证线程安全。

```java
public boolean add(E e) {
    // 获取锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 获取当前 List 备份数组的快照
        Object[] elements = getArray();
        int len = elements.length;
        
        // 创建一个新数组，比原数组大 1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        
        // 在新数组的最后一个位置添加新元素
        newElements[len] = e;
        
        // 用新数组替换旧数组
        setArray(newElements);
        
        return true;
    } finally {
        lock.unlock();
    }
}
```

#### 解析：

- **读取旧数组**：在添加元素前，先获取当前 `array` 数组的快照。
- **复制数组**：创建一个新数组，长度比原数组大 1，并复制旧数组的所有元素到新数组。
- **添加元素**：将新元素插入到新数组的最后一个位置。
- **替换数组**：用 `setArray(newElements)` 替换旧数组。
- **加锁保证线程安全**：在操作期间使用 `ReentrantLock` 进行加锁，防止多个线程同时修改。

这种写时复制（Copy-On-Write）策略确保了**写操作的线程安全性**，但每次写入都需要复制整个数组，性能较低，因此适用于 **读多写少** 的场景。


### 3. 修改元素的线程安全

当 `CopyOnWriteArrayList` 修改数组中的某个元素时（`set(int index, E element)` 方法），同样会创建一个新数组，并在新数组上进行修改，然后替换旧数组。

```java
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        Object oldValue = elements[index];
        
        // 复制数组并修改指定索引的值
        Object[] newElements = Arrays.copyOf(elements, elements.length);
        newElements[index] = element;
        
        // 替换数组
        setArray(newElements);
        
        return (E) oldValue;
    } finally {
        lock.unlock();
    }
}
```

### 解析：

- `set(int index, E element)` 方法不会直接修改原数组，而是复制一个新数组，在新数组上修改后再替换。
- 由于每次修改都创建新数组，**写入成本较高**。
- 适用于**读多写少**的场景。


### 4. 读取操作的线程安全

由于 `CopyOnWriteArrayList` 在写入时复制新数组并替换旧数组，而读取操作不会修改数据，因此 `get()` 方法无需加锁，可以直接读取 `array` 快照，提高读取性能。

```java
public E get(int index) {
    return get(getArray(), index);
}
```

#### 解析：

- 读取时，直接获取当前数组 `array`，保证读取操作不会受到并发写入的影响。
- 由于 `array` 是 `volatile` 变量，保证了可见性，读操作不会获取旧数据。

### 5. CopyOnWriteArrayList 适用场景

|适用场景|说明|
|---|---|
|**读多写少**|由于写操作会复制整个数组，因此适用于高并发读、低并发写的场景，如缓存、黑名单等。|
|**遍历不受干扰**|由于 `CopyOnWriteArrayList` 在写入时创建新数组，因此遍历时不会抛 `ConcurrentModificationException`，适合并发环境中的遍历操作。|
|**适用于不可变数据**|适合存储相对固定的数据，减少频繁的数组复制。|

### 6. CopyOnWriteArrayList 与 ArrayList 的对比

|   |   |   |
|---|---|---|
|特性|CopyOnWriteArrayList|ArrayList|
|线程安全|✅ 是|❌ 否（需手动同步）|
|读取性能|✅ 高|✅ 高|
|写入性能|❌ 低（需要复制数组）|✅ 高（直接修改原数组）|
|适合场景|读多写少|写多读少|

### 7. 总结

- `CopyOnWriteArrayList` 通过 **volatile** 关键字保证可见性，并在写入时采用 **写时复制** 方式，确保线程安全。
- 适用于**读多写少**的场景，例如**缓存、黑名单、系统配置数据等**。
- 由于写操作涉及数组复制，**写入性能较低**，不适用于频繁写入的场景。
- 迭代时不会抛 `ConcurrentModificationException`，适用于并发环境下的遍历操作。

这种 `Copy-On-Write` 机制虽然牺牲了写入性能，但带来了**线程安全的无锁读**，在高并发读的场景中表现优越。


--- 
