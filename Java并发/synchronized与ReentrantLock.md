/mnt/data/synchronized_vs_reentrantlock_fairness.md

# synchronized vs ReentrantLock：公平锁 / 非公平锁学习笔记

> 目标：一眼看懂二者在**公平性（fairness）**与**非公平性（non-fairness）**上的实现机制、差异、常见陷阱与选型建议。

---

## 0. TL;DR（要点速览）
- **synchronized**：天生**非公平**，无配置项可改。唤醒后“大家同时起跑”，老线程与新线程在释放时刻**并发抢占**所有权。
- **ReentrantLock**：基于 **AQS**；默认**非公平**（`new ReentrantLock()`），可选**公平**（`new ReentrantLock(true)`）。
  - **非公平**：新线程**入队前**先“直冲 CAS 抢锁”一次；失败才入队等待（队列近似 FIFO 唤醒）。
  - **公平**：新线程在抢占前检查 `hasQueuedPredecessors()`，前有等待者则**必须入队**，严格 FIFO 尝试。

---

## 1. 基础概念与名词
- **公平锁（Fair Lock）**：尽量按请求顺序（FIFO）获取锁，强调先来先得。
- **非公平锁（Non-Fair Lock）**：允许“插队”，以减少上下文切换与提升吞吐，但可能牺牲个体等待时间的可预测性。
- **AQS（AbstractQueuedSynchronizer）**：JUC 同步器的核心抽象，内部维护一个 **CLH 双向队列**，配合 `park/unpark` 管理阻塞与唤醒。

---

## 2. `synchronized` 的公平性行为
- **实现基底**：HotSpot 下由对象头 +（膨胀后）**ObjectMonitor** 实现；进入/退出对应 `monitorenter/monitorexit` 字节码。
- **为什么是非公平**：
  1. `notify/notifyAll` 唤醒线程**不会直接持有锁**，仅是“具备竞争资格”。
  2. 在锁**释放瞬间**，被唤醒的老线程与**新到的线程**一起参与竞争，**没有显式 FIFO 队列**保证顺序。
  3. 为性能采用**自旋→失败→挂起**等策略，进一步弱化了先来先得。

**一句话**：`synchronized` 的非公平主要体现在**“释放/唤醒之后的并发争抢”**。

> **内存语义**：`monitorexit` 对应的释放 **happens-before** 随后的 `monitorenter` 成功获取；保证临界区内写对后继获取者可见。

---

## 3. `ReentrantLock` 的公平/非公平（AQS 视角）
### 3.1 非公平 ReentrantLock（默认）
- **策略**：当线程请求锁时，**先不顾队列**直接 `CAS(state, 0→1)`：
  ```java
  // 化简示意
  boolean tryAcquire(int acquires) {
    if (compareAndSetState(0, acquires)) { setExclusiveOwnerThread(current); return true; }
    if (current == owner) { state += acquires; return true; } // 重入
    return false; // 失败 → 入队
  }
  ```
- **失败后入队**：调用 `acquireQueued()` 进入 **CLH 队列**，队列采用**近似 FIFO 唤醒**（见 3.3）。

**非公平发生在“入队之前”**：新线程拥有一次“直冲”机会，可能**插队**成功。

### 3.2 公平 ReentrantLock
- **策略**：在 CAS 之前检查 `hasQueuedPredecessors()`：
  ```java
  boolean tryAcquire(int acquires) {
    if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) { ... }
    // 前面有人则不直冲，必须入队
  }
  ```
- **效果**：基本遵守 FIFO 次序尝试获取，减少“插队”，但**吞吐量通常低于非公平**。

### 3.3 AQS 队列与唤醒
- **结构**：CLH 双向链表，节点含等待状态与前驱/后继。
- **释放**：`unlock()` → `AQS.release()`，当 `state` 归零时调用 `unparkSuccessor(head)`，**唤醒队首的后继节点**。
- **唤醒后仍需 CAS**：被唤醒线程也要与竞争者在释放瞬间 **CAS**；因此称**“近似 FIFO”**（wake-up FIFO，acquire 仍看 CAS 竞态）。

---

## 4. 二者“非公平”的机制差异
| 对比点 | synchronized | ReentrantLock（非公平） |
|---|---|---|
| 非公平发生位置 | **释放/唤醒之后**：老线程与新线程**同时起跑**争抢所有权 | **入队之前**：新线程先**直冲 CAS**，失败再入队 |
| 队列抽象 | 无显式 FIFO 队列 | 有 AQS CLH 队列（近似 FIFO 唤醒） |
| 可配置公平性 | 不可配置 | 构造可选 `true/false` |
| 可中断/超时 | 不支持 | 支持 `lockInterruptibly()`/`tryLock(timeout)` |
| 条件队列 | `wait/notify`（单队列） | `Condition`（多条件队列） |
| 可重入计数 | 有（隐式） | 有（`state` 计数） |

**类比**：  
- `synchronized`：**“大家同时起跑”**的非公平。  
- `ReentrantLock` 非公平：**“新人先摸门把”**，不行才排队。

---

## 5. 常见问题（FAQ）
**Q1：非公平 ReentrantLock 新线程直冲失败后会怎样？**  
A：会**入队**，随后按队列**FIFO 唤醒**进行再次尝试。公平性主要体现在“**已入队**的线程”上。

**Q2：非公平是否必然导致饥饿？**  
A：理论上在高竞争下**存在饥饿风险**；但 AQS 的唤醒策略倾向队首，实践中较少出现长期饥饿。需要严格顺序请用公平锁，但注意吞吐下降。

**Q3：`Condition.await()` 会不会“丢唤醒”？**  
A：`Condition` 与 `signal` 的语义由 AQS 保证配合使用；典型写法需搭配**显式条件判断**（`while (!condition) await()`）以抵御**伪唤醒**。

**Q4：公平锁一定更快吗？**  
A：通常**更慢**（上下文切换更多、直冲优化被抑制）。公平锁换**可预测等待**与**较少抖动**。

---

## 6. 时序/流程（ASCII）

### 6.1 非公平 ReentrantLock（新线程入队前直冲）
```
Thread Tn: tryLock()
   ├─ CAS(state:0→1) 成功 → 获取锁（可能插队）
   └─ 失败 → 入队（CLH）→ park
Unlock()
   └─ release → 唤醒 head 的后继 → 其被唤醒后 CAS 再试
```

### 6.2 synchronized（释放/唤醒后并发争抢）
```
notify/notifyAll → 唤醒等待者（仅获得竞争资格）
Monitorexit（释放） → 新老线程一起并发抢占 owner
→ 成功者进入；失败者可能继续等待/挂起
```

---

## 7. 选型建议
- **吞吐优先 / 低延迟**：首选**非公平**（`synchronized` 或 `new ReentrantLock(false)`）。
- **强顺序 / 公平性要求**：使用 **`new ReentrantLock(true)`**，注意整体吞吐与尾延迟可能提升。
- **读多写少**：考虑 `ReentrantReadWriteLock`（可选公平/非公平）或 `StampedLock`（乐观读，注意不可重入与中断特性）。
- **需要超时/可中断/多条件队列**：优先 `ReentrantLock` + `Condition`。

---

## 8. 代码片段（公平 vs 非公平）

```java
// 非公平（默认）
Lock lock = new ReentrantLock(); // equals new ReentrantLock(false)

// 公平
Lock fairLock = new ReentrantLock(true);

// 典型使用
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();
}

// 可中断与超时
lock.lockInterruptibly();
if (lock.tryLock(200, TimeUnit.MILLISECONDS)) { ... }
```

---

## 9. 一页对照表

| 维度 | synchronized | ReentrantLock(非公平) | ReentrantLock(公平) |
|---|---|---|---|
| 公平性 | 固定非公平 | 默认非公平 | 可选公平 |
| 非公平点位 | 释放/唤醒后并发争抢 | 入队前直冲 CAS | 无直冲，遵循排队 |
| 队列 | 无显式 FIFO | AQS CLH 队列 | AQS CLH 队列 |
| 可中断/超时 | × | ✓ | ✓ |
| 条件变量 | `wait/notify`（单） | `Condition`（多） | `Condition`（多） |
| 重入 | ✓ | ✓ | ✓ |
| 可观测/控制度 | 低（语法级） | 高（API 丰富） | 高（API 丰富） |

---

### 参考实践建议
- 服务端高并发热点锁：优先 **非公平**，减少上下文切换抖动。
- 任务队列/网关等强调先来先服务场景：考虑 **公平锁**，保障等待时间可预测。
- 条件较多的协作（生产/消费、多阶段流程）：`ReentrantLock` + 多 `Condition` 更清晰。
