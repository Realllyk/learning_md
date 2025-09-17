# Redis Listpack 结构与原理整理

## 1. Listpack 的整体结构
一个 listpack 由以下部分组成：
- **总字节数**：整个 listpack 占用的字节长度；
- **元素数量**：entry 的数量；
- **多个 listpack entry**：每个 entry 存储一个数据元素；
- **结尾标识**：标记 listpack 的结束。

整体布局：
```
[listpack 总字节数][元素数量][entry1][entry2]...[entryN][结尾标识]
```

---

## 2. Listpack entry 的结构
每个 entry 由三个部分组成：
```
[encoding][data][len]
```

- **encoding**：定义数据的类型和长度信息。
  - 用于区分整数 / 字符串；
  - 对不同长度的字符串采用不同的编码方式（1 字节 / 2 字节 / 5 字节）；
  - 对整数采用不同位宽（int16/int32/int64）。
- **data**：实际存储的数据内容。
- **len**：当前 entry 的总长度，即 encoding + data 的字节数。
  - 存储在 entry 的末尾，采用变长编码。

---

## 3. encoding 的设计
encoding 不定长，取决于数据类型：
- **短字符串（≤63 字节）**：1 字节 encoding（低 6 位存储长度）。
- **中等字符串（≤4095 字节）**：2 字节 encoding（12 位存储长度）。
- **长字符串（≤ 2^32-1 字节）**：5 字节 encoding。
- **整数**：1 字节标识不同整数类型（如 int16/int32/int64）。

作用：
- 告诉解析器 **data 的字节数**；
- 告诉解析器 **如何解释 data**。

---

## 4. len 的设计与作用
- **位置**：位于 entry 的末尾。
- **内容**：存储当前 entry 的总长度（encoding + data）。
- **作用**：
  1. 支持 O(1) 的反向遍历：已知 entry 末尾位置，可以通过 len 快速跳到 entry 开头；
  2. 顺序解析时可以直接跳过 entry，提高效率。

---

## 5. encoding 与 len 的关系
- **encoding**：描述数据的类型与长度（仅保证能读懂 data）。
- **len**：描述整个 entry 的总大小（保证能快速跳过/反向遍历）。
- **互补关系**：encoding 解决“内容是什么”，len 解决“整体占多少”。

---

## 6. 与 ziplist 的对比
- **ziplist**：每个 entry 带有 prevlen（前一个 entry 长度），方便双向遍历，但会导致“连锁更新”。
- **listpack**：去掉 prevlen，改为在末尾记录 len，避免连锁更新，同时支持双向遍历。

---

## 7. 小结
- listpack entry = **encoding + data + len**。
- encoding：定义类型和数据长度（变长编码）。
- data：实际内容。
- len：entry 总长度，主要用于高效反向遍历。
- 优点：结构紧凑、避免连锁更新、支持双向遍历。

