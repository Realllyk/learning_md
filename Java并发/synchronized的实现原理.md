# Java synchronized 底层实现原理学习笔记

## 1. synchronized 的字节码实现
- 同步代码块：编译后生成 `monitorenter` 和 `monitorexit` 指令。
- 同步方法：方法表上加 `ACC_SYNCHRONIZED` 标志。
- 这两个字节码都依赖 JVM 的 **Monitor** 来实现互斥和可见性。

---

## 2. 对象头（Object Header）与 Mark Word
- HotSpot 每个对象都有对象头，其中 **Mark Word** 存储锁信息：
  - **无锁状态**：保存哈希、GC 年龄等。
  - **轻量级锁**：Mark Word 指向线程栈中的 **Lock Record**。
  - **重量级锁**：Mark Word 指向堆中的 **ObjectMonitor**。

---

## 3. Monitor 的结构
Java 层的 Monitor 是一种管程结构，主要包括：
- **Owner**：当前持有锁的线程。
- **EntryList**：正在等待进入临界区的线程队列。
- **WaitSet**：调用 `wait()` 进入的等待队列。
- **Recursions**：可重入计数。
- **Succ (Successor hint)**：候选下一个唤醒的线程。

Monitor 的作用：
- **互斥**：同一时刻只允许一个线程持有锁。
- **条件变量**：依靠 `WaitSet` 支持 `wait/notify`。

---

## 4. HotSpot C++ 实现：ObjectMonitor
在 HotSpot 的 `objectMonitor.hpp` 中，Monitor 的实现是 **`ObjectMonitor`**，主要字段：

- `_owner`：持有锁的线程 (JavaThread*)。
- `_recursions`：重入计数。
- `_EntryList`：等待获取锁的线程队列。
- `_WaitSet`：调用 `wait()` 后挂起的线程队列。
- `_cxq`：新竞争线程的单向链表（CAS 入队，LIFO）。
- `_succ`：继任者提示，减少无效唤醒。

---

## 5. 加锁与解锁流程

### monitorenter 流程
1. 尝试轻量级锁：CAS 将 Mark Word 指向 Lock Record。
2. 成功 → 直接获得锁。
3. 失败 → 膨胀为 `ObjectMonitor`。
4. 尝试设置 `_owner`，若失败 → 入 `_cxq`，自旋 / 挂起。

### monitorexit 流程
1. 如果是轻量级锁 → 恢复 Mark Word。
2. 如果是重量级锁 (ObjectMonitor)：
   - `_recursions--`，为 0 时释放 `_owner`。
   - 唤醒 `EntryList` / `_cxq` 的线程。
   - 设置 `_succ`，避免惊群。

---

## 6. 内存语义
- `monitorenter` = 获取锁 → **acquire 屏障**。
- `monitorexit` = 释放锁 → **release 屏障**。
- 保证临界区写入对后续线程可见，即 `happens-before` 语义。

---

## 7. 锁状态与对象头的关系
- **无锁**：Mark Word 保存哈希等信息。
- **轻量级锁**：Mark Word 指向线程栈中的 Lock Record。
- **重量级锁**：Mark Word 指向 `ObjectMonitor`。

👉 只有在 **重量级锁** 状态下，才会真正使用 C++ 的 `ObjectMonitor`。

---

## 8. 总结
- Java `synchronized` 通过字节码的 `monitorenter/monitorexit` 实现。
- HotSpot 内部使用 **对象头 + Monitor** 管理锁。
- Monitor 的具体实现是 **C++ 的 `ObjectMonitor`**，包括 `_owner`、`_recursions`、`_EntryList`、`_WaitSet`、`_cxq` 等。
- 轻量级锁时不使用 `ObjectMonitor`，只有在锁膨胀时才会指向 `ObjectMonitor`。
