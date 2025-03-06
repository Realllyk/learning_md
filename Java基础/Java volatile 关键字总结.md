## 1. `volatile` 的作用

在 Java 并发编程中，`volatile` 关键字主要用于保证**可见性**和**防止指令重排序**，但**不保证原子性**。

### `volatile` **提供的特性**

1. **保证变量的可见性（Visibility）**
    - `volatile` 变量的修改会立刻刷新到主存，并通知其他线程，使其读取最新值。
    - 适用于**多个线程共享变量**的场景。
2. **禁止指令重排序（Ordering）**
    - `volatile` 变量的读写操作不会被编译器或 CPU 乱序优化。
    - 适用于 **双重检查锁（DCL, Double-Checked Locking）** 等场景。

## 2. `volatile` 的工作原理

`volatile` 变量的修改会触发 **内存屏障（Memory Barrier）** 机制，保证：
1. **写** `volatile` **变量时**，会将值立刻写入主存，并让其他 CPU 核心的缓存失效。
2. **读** `volatile` **变量时**，会直接从主存读取，而不是从 CPU 缓存读取。
    

## 3. `volatile` 示例

### **示例 1：保证可见性**

```java
class VolatileExample {
    private static volatile boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (running) {
                // 没有 `volatile`，可能会一直循环
            }
            System.out.println("Thread stopped");
        }).start();

        Thread.sleep(2000);
        running = false; // 让线程停止
    }
}
```

- 如果 `running` 不是 `volatile`，子线程可能无法感知 `false`，导致无限循环。
- **加上** `volatile`，**子线程可见主线程的修改**。

### **示例 2：**`volatile` **不能保证原子性**

```java
class NonAtomicVolatile {
    private static volatile int count = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                count++;  // 不是原子操作
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                count++;
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final count: " + count);  // 可能小于 2000
    }
}
```

**为什么** `count++` **不是原子操作？**

- `count++` 其实是三步操作：
    1. **读取** `count`
    2. **增加 1**
    3. **写回** `count`
- **多线程同时执行时，可能会丢失更新**。

### **解决方案**

✅ **使用** `**AtomicInteger**`

```java
import java.util.concurrent.atomic.AtomicInteger;

class AtomicIntegerExample {
    private static AtomicInteger count = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                count.incrementAndGet();  // 原子操作
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                count.incrementAndGet();
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final count: " + count.get());  // 结果保证是 2000
    }
}
```

## 4. `volatile` vs `synchronized`

|特性|`volatile`|`synchronized`|
|---|---|---|
|**保证可见性**|✅|✅|
|**保证原子性**|❌|✅|
|**防止指令重排序**|✅|✅|
|**适用场景**|变量可见性控制，适用于**读多写少**|临界区保护，适用于**读写都频繁的情况**|
|**性能**|高（不会阻塞）|可能低（涉及上下文切换）|

## 5. 经典应用：双重检查锁（DCL）

```java
class Singleton {
    private static volatile Singleton instance;  // 禁止指令重排序

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

## 6. `volatile` 适用场景

✅ **适用**

1. **多线程共享变量，且只需要保证可见性，不需要保证原子性**
2. **布尔标志变量（如** `running**`）
3. **双重检查锁（DCL）**
4. **轻量级的状态标志**
    

❌ **不适用**

1. **需要保证原子性的场景（如** `count++`）
2. **涉及多个变量的同步更新**

## 7. 总结
- `volatile` **保证可见性**，但**不保证原子性**。
- `volatile` **禁止指令重排序**，适用于 **DCL 单例模式**。
- **适用于** `读多写少` **的共享变量，避免使用在复杂并发场景**。
- 如果涉及**原子性**，应使用 `synchronized` 或 `Atomic` 变量。