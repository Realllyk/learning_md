# Kafka 与 RocketMQ 消息持久化机制总结

## 1. Kafka 的持久化机制

### 存储结构
- **Topic → Partition → Segment**
- 每个 Partition 是一个独立的有序日志（Log）。
- Segment 文件包括：
  - `.log`：消息内容
  - `.index`：offset → 物理地址映射（稀疏索引）
  - `.timeindex`：时间戳 → 物理地址映射

### 写入流程
1. Producer 将消息发送到 Broker。
2. Broker 将消息写入 **Page Cache**（内存缓冲）。
3. 消息顺序追加到日志文件（.log）。
4. 操作系统定期或按配置刷盘（flush）到磁盘。

### 副本与可靠性
- Partition 可配置多个副本（Replica）。
- Leader 负责写入，Follower 异步拉取同步。
- `acks` 参数控制确认策略：
  - `acks=0`：只要写入 Page Cache 就算成功。
  - `acks=1`：Leader 写入成功即返回。
  - `acks=all`：所有 ISR 副本写入成功才返回。

### 消费流程
- Consumer 按照 offset 拉取数据。
- Broker 通过 `.index` 文件快速定位 offset → 文件位置。
- 实际读取时：
  - 命中 Page Cache → 内存读取。
  - 未命中 → 触发磁盘读取。
- Kafka 使用 **零拷贝（sendfile）** 提升读取效率。


## 2. RocketMQ 的持久化机制

### 存储结构
- **CommitLog + ConsumeQueue**
- 所有消息统一顺序写入 **CommitLog**。
- **ConsumeQueue** 是逻辑队列的稠密索引文件，记录：
  - 消息在 CommitLog 的物理偏移量。
  - 消息大小。
  - 消息 Tag 的 hashCode。

### 写入流程
1. Producer 将消息写入 Broker。
2. Broker 将消息写入 Page Cache。
3. 消息顺序写入 CommitLog 文件。
4. 异步或同步刷盘策略：
   - **异步刷盘**：写入 Page Cache 即返回，由后台线程定期刷盘。
   - **同步刷盘**：必须 fsync 落盘才返回。

### 副本与可靠性
- 早期主要依赖刷盘策略保障可靠性。
- 现在支持 **Dledger 多副本模式**，副本同步写入后才返回 ACK。

### 消费流程
- Consumer 从 **ConsumeQueue** 找到消息在 CommitLog 的物理位置。
- 根据物理位置直接读取 CommitLog。
- 和 Kafka 一样，优先从 Page Cache 读取，未命中才触发磁盘 I/O。
- RocketMQ 也使用 mmap + 零拷贝 提升读性能。


## 3. Kafka vs RocketMQ 对比

| 特性 | Kafka | RocketMQ |
|------|-------|----------|
| **日志结构** | Partition 自己就是日志 | CommitLog（全局日志）+ ConsumeQueue（索引） |
| **索引机制** | 稀疏索引（.index 文件） | 稠密索引（ConsumeQueue） |
| **写入方式** | 顺序写 Partition Log | 顺序写 CommitLog |
| **刷盘策略** | 依赖 OS Page Cache + flush.ms/messages | 异步刷盘 / 同步刷盘 |
| **副本机制** | ISR 副本同步（acks 参数控制） | Dledger 多副本同步 |
| **消费读取** | offset + index → Log | ConsumeQueue → CommitLog |

## 4. 总结
- Kafka 偏重 **高吞吐**，通过 Partition + Page Cache + 副本保证性能与可靠性。
- RocketMQ 偏重 **可控可靠性**，通过 CommitLog + ConsumeQueue + 刷盘策略保证高可靠。

**一句话**：  
Kafka 更像是“分布式日志系统”，而 RocketMQ 更像是“金融级可靠消息系统”。
