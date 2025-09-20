# Java vs C++ 迭代器在遍历期的“修改影响”对照表

> 速查目标：当**正在用迭代器遍历**时，不同“修改”会导致什么结果？（是否报错 / 是否失效 / 是否安全）

---

## 一、Java 迭代器（Collection/Map 家族，重点是 fail‑fast 语义）

- **核心机制**：大多数非并发集合（`ArrayList`, `HashMap`, `HashSet` 等）的迭代器是 *fail‑fast*。集合维护 `modCount`，迭代器保存期望值 `expectedModCount`；遍历过程中如检测到**非自身**的结构性修改（结构变化），会抛出 `ConcurrentModificationException (CME)`。该检查是 **best‑effort**，不是绝对保证。
- **结构性修改**：改变集合结构的操作，如 `add`, `remove`, `clear`, 自动扩容/再散列等。
- **非结构性修改**：不改变结构的操作，如替换某个下标/键对应的 **值**（`set(index, v)`, `map.put(key, newVal)` 对已存在键），或直接修改元素对象内部字段。
- **并发容器**（如 `ConcurrentHashMap`, `CopyOnWriteArrayList`）的迭代器通常是 **弱一致 (weakly consistent)** 或 **快照式**，不抛 CME，但看到的视图可能是遍历开始时或动态变化中的一个一致片段。

### Java 遍历 × 修改行为表

| 场景（遍历中）                                 | 修改来源              | 修改类型                             | 典型容器  | 结果/行为                                                |
| --------------------------------------- | ----------------- | -------------------------------- | ----- | ---------------------------------------------------- |
| 使用 `Iterator` 遍历 `ArrayList`            | **迭代器自身**         | `iterator.remove()`              | 非并发集合 | **允许**，不抛异常（视为合法的结构性修改）                              |
| 使用 `ListIterator` 遍历 `ArrayList`        | **迭代器自身**         | `listIterator.add(e)` / `set(e)` | 非并发集合 | **允许**（`add`/`set` 由该迭代器掌控）                          |
| 使用 `Iterator` 遍历 `ArrayList`            | **其他代码**（同线程或他线程） | `list.add/remove/clear` 导致结构变化   | 非并发集合 | **通常抛 `ConcurrentModificationException`**（fail‑fast） |
| 使用 `Iterator` 遍历 `HashMap.entrySet()`   | **其他代码**          | 新增/删除键（再散列/结构变化）                 | 非并发集合 | **通常抛 `ConcurrentModificationException`**            |
| 使用 `Iterator` 遍历 `HashMap.entrySet()`   | **其他代码**          | 仅 `put` 到**已存在键**（覆盖旧值，不新增键）     | 非并发集合 | 一般 **不抛异常**（视实现，通常不改变结构）                             |
| 使用 `Iterator` 遍历任意集合                    | **其他代码**          | **修改元素对象内部字段**（不触及集合结构）          | 非并发集合 | **不抛异常**（非结构性修改；但可能破坏基于 equals/hash 的不变量，风险自担）       |
| 使用 `Iterator` 遍历 `ConcurrentHashMap`    | **其他代码**          | 插入/删除                            | 并发集合  | **不抛 CME**；**弱一致**视图（遍历可能看到部分新/旧数据）                  |
| 使用 `Iterator` 遍历 `CopyOnWriteArrayList` | **其他代码**          | 插入/删除                            | 并发集合  | **不抛 CME**；迭代的是**快照**，遍历中对原容器的修改不会反映到当前迭代            |

> 备注：fail‑fast 是“尽早发现错误”的策略，并非并发安全的保证。即便没抛异常，也不能据此推断你的代码是线程安全的。

---

## 二、C++ 迭代器（STL 容器，迭代器失效规则为主）

- **核心哲学**：不做运行时检查，**不抛异常**；一旦违反规则，通常是 **Undefined Behavior（未定义行为，UB）**。是否“失效”取决于**容器类型**与**具体操作**。
- **结构性修改**：插入/删除导致存储重排、rehash、节点拆装等。
- **非结构性修改**：通过引用/迭代器修改元素值（若容器允许）。注意：**有序关联容器**（`std::set`, `std::map`）元素的 **键 key 为 const**，不可修改；`std::map` 的 `mapped_type`（value 的第二部分）可改。

### C++ 不同容器的“遍历期 × 修改影响”速查表

> 结论优先：遇到 **reallocation/rehash/中间插入删除** 往往会使**一批迭代器失效**；`list`/`forward_list` 最稳；有序容器插入基本不失效，删除仅失效被删处。

| 容器                                          | 操作（遍历期发生）                       | 迭代器有效性                               | 引用/指针有效性           | 说明                 |
| ------------------------------------------- | ------------------------------- | ------------------------------------ | ------------------ | ------------------ |
| `std::vector`                               | `push_back`/`insert` **触发重新分配** | **全部失效**                             | **全部失效**           | 容量不足时 reallocation |
| `std::vector`                               | `push_back`（**无**重新分配）          | 仍有效                                  | 仍有效                | 小心后续操作触发扩容         |
| `std::vector`                               | `erase(pos)`                    | **从 `pos` 起到 end 的迭代器失效**，`end()` 失效 | 对这些元素的引用/指针也失效     | 元素左移               |
| `std::deque`                                | 在 **任意位置** `insert/erase`       | **通常全部迭代器失效**                        | 引用/指针大多仍有效（被删除的除外） | 分段存储结构导致迭代器不稳定     |
| `std::list` / `std::forward_list`           | `insert/erase/splice`           | **仅指向被删元素的迭代器失效**；其他**保持有效**         | 其他引用/指针也**保持有效**   | 链表节点稳定，不移动         |
| `std::set` / `std::map`（有序）                 | `insert`                        | **不使现有迭代器失效**                        | 引用/指针仍有效           | 基于平衡树              |
| `std::set` / `std::map`                     | `erase(it)`                     | **仅使该元素处迭代器失效**                      | 指向被删元素的引用/指针失效     | 其他不受影响             |
| `std::unordered_set` / `std::unordered_map` | `rehash` / 负载因子触发扩容             | **全部迭代器失效**                          | 引用/指针也可能失效         | 桶重排                |
| `std::unordered_*`                          | `insert`（未触发 rehash）            | 仍有效                                  | 仍有效                | 注意负载因子阈值           |
| `std::unordered_*`                          | `erase(it)`                     | **仅使指向被删元素的迭代器失效**                   | 指向被删元素的引用/指针失效     | 其他不受影响             |
| `std::string`                               | 与 `vector` 类似（小字符串优化细节依实现）      | 参照 `vector`                          | 参照 `vector`        | 当内存重分配时失效          |

### C++ 非结构性修改（值修改）
- `vector[i] = v`、通过迭代器解引用写入：**安全**，不致使迭代器失效（前提：**本次不触发结构性操作**）。  
- `std::map<Key, T>` 可修改 `it->second`；`std::set`/`std::map` 的 **键 `Key` 为 const**，**不可**通过迭代器修改。  
- 多线程下，标准容器**不提供线程安全保证**：并发读写通常为 UB。

---

## 三、面试/实战要点备忘

- **Java**：
  - 记住：遍历中 **非自身**结构性修改 ⇒ 大概率 `ConcurrentModificationException`（best‑effort）。
  - 自身修改：`Iterator.remove()`、`ListIterator.add()/set()` 合法。  
  - 需要并发遍历：使用并发容器（弱一致/快照式）。
- **C++**：
  - 背下“**迭代器失效表**”即可应对 90% 场景：`vector` 扩容全失效、`list` 稳定、`map/set` 插入不失效删除只失一个、`unordered_*` rehash 全失效、`deque` 迭代器很脆。
  - 违反规则不报错但会 UB，**绝不要赌**。

---

## 四、简版口播（30 秒）
- Java 的非并发集合是 **fail‑fast**：遍历中如果不是“我这个迭代器自己”做的结构改动，就可能抛 `ConcurrentModificationException`；改值一般不算结构改动。并发容器的迭代器弱一致或快照，不抛 CME。  
- C++ 标准容器**不做检查**：是否失效取决于容器类型与操作；`vector` 扩容全失效、`list` 基本不失效、`map/set` 插入不失效删除只失该元素、`unordered` rehash 全失效；违规则 UB。

