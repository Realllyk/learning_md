# Java 性能优化与缓存机制总结

## 1. Java 程序加速执行的常见手段

### 1.1 JVM 层面优化
- **JIT 编译**：将热点代码编译为机器码，加快执行速度。
- **GC 调优**：选择合适的垃圾回收器（如 G1、ZGC、Shenandoah），减少停顿。
- **内存参数优化**：合理配置堆大小、线程栈大小，减少内存压力。

### 1.2 I/O 与数据库优化
- **连接池**：使用 HikariCP / Druid 等，避免频繁创建销毁数据库连接。
- **批量操作**：减少数据库 round-trip，例如批量插入、更新。
- **缓存**：本地缓存（Guava、Caffeine）或分布式缓存（Redis），减少数据库访问。
- **异步 I/O**：使用 NIO/Netty 提高并发处理能力。

### 1.3 并发与多线程
- **线程池**：复用线程，避免频繁创建销毁。
- **并发集合**：使用 `ConcurrentHashMap`、`CopyOnWriteArrayList` 等。
- **锁优化**：使用读写锁、StampedLock，减少竞争。
- **无锁操作**：使用 CAS 和 `Atomic` 类。
- **异步编程**：`CompletableFuture`、Reactor 等。

### 1.4 代码层面优化
- **减少对象创建**：避免重复创建大对象。
- **String 优化**：使用 `StringBuilder` 或 `StringBuffer` 替代 `+` 拼接。
- **懒加载**：按需加载资源。
- **合适的数据结构与算法**：选择复杂度更优的实现。

### 1.5 缓存与分布式
- **多级缓存**：JVM 本地缓存 + 分布式缓存。
- **结果缓存**：接口调用结果缓存，减少重复计算。

### 1.6 部署与监控
- **类加载优化**：减少反射、动态代理带来的开销。
- **AOT 编译**：使用 GraalVM Native Image 提前编译。
- **性能监控**：使用 JFR、Arthas、JVisualVM 分析性能热点。

---

## 2. Guava Cache

### 2.1 与普通 Map 的区别
- **ConcurrentHashMap**：只是存储容器，无过期、淘汰机制。
- **Guava Cache**：在 `ConcurrentHashMap` 基础上增加：
  - 过期策略（写入后过期 / 访问后过期）。
  - 容量限制 + LRU 淘汰。
  - 自动加载（CacheLoader）。
  - 统计功能（命中率、加载次数等）。

### 2.2 常见配置
- `.maximumSize(n)`：设置最大容量，LRU 淘汰。
- `.expireAfterWrite(n, TimeUnit)`：写入后多久过期。
- `.expireAfterAccess(n, TimeUnit)`：最后访问后多久过期。
- `.recordStats()`：开启统计功能。

### 2.3 使用示例
```java
import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;
import java.util.concurrent.TimeUnit;

public class GuavaCacheDemo {
    public static void main(String[] args) throws Exception {
        LoadingCache<String, String> cache = CacheBuilder.newBuilder()
                .maximumSize(100)
                .expireAfterWrite(10, TimeUnit.SECONDS)
                .build(new CacheLoader<String, String>() {
                    @Override
                    public String load(String key) {
                        return "Value_for_" + key;
                    }
                });

        System.out.println(cache.get("A")); // 第一次，触发 load
        System.out.println(cache.get("A")); // 第二次，命中缓存
        Thread.sleep(11000);
        System.out.println(cache.get("A")); // 第三次，过期后再次 load
    }
}
```

---

## 3. StringBuilder vs StringBuffer

| 特性 | StringBuilder | StringBuffer |
|------|---------------|--------------|
| 线程安全 | ❌ 不保证 | ✅ synchronized 方法保证 |
| 性能 | 🚀 更快 | 🐢 稍慢 |
| 使用场景 | 单线程环境 | 多线程环境 |
| 底层实现 | 基于 AbstractStringBuilder，无锁 | 基于 AbstractStringBuilder，方法加锁 |

**总结**：  
- `StringBuilder`：推荐在单线程场景下使用，性能更好。  
- `StringBuffer`：在需要线程安全的多线程场景使用。  

---

## 4. 总结
Java 提供了多层次的性能优化方法：  
- JVM 参数调优  
- 数据库连接池与批量操作  
- 使用缓存（本地 Guava/分布式 Redis）  
- 并发工具类（线程池、并发集合）  
- 合理选择 `StringBuilder` / `StringBuffer`  

其中 **Guava Cache** 是一种轻量级的本地缓存方案，比普通 Map 更智能，适合单机快速缓存场景。
