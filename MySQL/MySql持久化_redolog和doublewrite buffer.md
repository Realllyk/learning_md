# MySQL 持久化机制学习笔记

本文总结了 MySQL（InnoDB 引擎）中关于 **redo log** 与 **doublewrite buffer** 的机制，梳理其在事务持久化与宕机恢复中的作用。

---

## 1. 数据页与写入粒度

- **InnoDB 数据页大小**：16KB（默认）。
- **操作系统写磁盘的最小单位**：通常是 4KB block。
- **问题**：写一个 16KB 页到磁盘时，操作系统可能拆成 4KB × 4 次写。如果宕机，可能出现只写了一半的情况，称为 **torn page（撕裂页）**。

---

## 2. Redo Log

### 2.1 定义
- Redo log（重做日志）是 **物理日志**，记录对某个数据页的修改操作，而不是 SQL 语句。
- 一条 redo log 记录包括：
  - Page ID（页号）
  - Offset（偏移量）
  - Data（修改的字节内容）
  - Length（长度）
  - LSN（日志序列号，递增）
  - Checksum（校验值）

### 2.2 特点
- Redo log 按顺序写入文件，写入效率高。
- 应用 redo log 的前提：需要有一份 **完整且一致的数据页**。
- 事务提交时会打上 **commit 标记**，保证原子性。

### 2.3 作用
- 保证事务提交后的修改 **不会丢失**。  
- 宕机恢复时，可以根据 redo log 把缺失的修改重放到数据页上。

---

## 3. Doublewrite Buffer

### 3.1 定义
- InnoDB 在共享表空间（ibdata1）中预留了 **2MB 的磁盘区域**作为 doublewrite buffer。
- 内存中也有对应的 2MB buffer，用来缓存要写的页。

### 3.2 工作过程
1. 脏页刷盘时，先把完整的 **16KB 页**写入 doublewrite buffer（顺序写）。
2. 确认成功后，再把页写入目标表空间文件（.ibd 文件，随机写）。
3. 如果宕机导致 torn page，可以用 doublewrite buffer 中的副本覆盖。

### 3.3 作用
- 保证数据页写盘时不会出现 **半新半旧的 torn page**。
- 提供 redo log 重放所需的完整基准页。

---

## 4. Redo log 与 Doublewrite buffer 的互补关系

- **Redo log**：解决“事务持久性”问题，保证已提交的修改不会丢失。
- **Doublewrite buffer**：解决“页完整性”问题，防止 torn page。
- **配合机制**：
  1. doublewrite buffer 先修复坏页 → 保证页完整。
  2. redo log 再重放事务修改 → 保证数据正确。

---

## 5. 不同阶段的动作对比

| 阶段                  | Buffer Pool（内存页） | Redo Log                        | Doublewrite Buffer                  |
|-----------------------|-----------------------|---------------------------------|-------------------------------------|
| **事务修改时**        | 页被修改为脏页        | 生成 redo log 写入 log buffer   | 不参与                              |
| **事务提交时**        | 脏页可能还在内存      | 刷盘 redo log（保证持久性）      | 不参与                              |
| **脏页刷盘时**        | 将脏页写回磁盘        | 已经有 redo log 可用于恢复       | 先写 doublewrite，再写表空间        |
| **宕机恢复时**        | Buffer Pool 丢失      | 重放 redo log（恢复事务修改）    | 修复 torn page（提供完整页基准）     |

---

## 6. 宕机恢复流程

1. **检查并修复 torn page**
   - 利用 doublewrite buffer 中的完整页覆盖损坏页。

2. **分析 redo log**
   - 从 checkpoint LSN 之后开始扫描。
   - 校验 checksum 确保日志完整。
   - 过滤掉没有 commit 标记的事务。

3. **重放 redo log**
   - 对比 Page LSN 与 Redo LSN：
     - Redo LSN > Page LSN → 应用日志。
     - Redo LSN ≤ Page LSN → 已落盘，跳过。

---

## 7. 总结

- **redo log**：保证事务不丢失，但需要完整页。  
- **doublewrite buffer**：保证页不坏，为 redo log 提供基准。  
- **两者配合**：先保证页完整，再保证数据正确。  

这样 MySQL InnoDB 就能同时保证 **持久性 (Durability)** 和 **一致性 (Consistency)**。
