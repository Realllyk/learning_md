## SQL vs NoSQL
### **SQL 数据库**

SQL 数据库指**关系型数据库**，主要代表有：
- **SQL Server**
- **Oracle**
- **MySQL（开源）**
- **PostgreSQL（开源）**

关系型数据库用于存储**结构化数据**，其数据逻辑上**以行列二维表的形式存在**。
- **每一列代表数据的一种属性**
- **每一行代表一个数据实体**

### **NoSQL 数据库**
NoSQL 指**非关系型数据库**，主要代表有：
- **MongoDB**
- **Redis**

NoSQL 数据库在逻辑结构上提供了**不同于二维表的存储方式**，例如：
- **JSON 文档存储**
- **哈希表存储**
- **键值对存储**

### **选择 SQL vs NoSQL 的考量因素**
### **ACID vs BASE**
- **关系型数据库（SQL）支持 ACID**，即 **原子性、一致性、隔离性、持久性**。
- **NoSQL 采用 BASE 模型**（基本可用、软状态、最终一致性）。

#### **ACID 适用场景**
- 适用于**银行、支付系统**，保证数据一致性，避免事务丢失。
- 确保数据更新**对所有用户立即可见**。

#### **BASE 适用场景**
- 适用于**社交网络、日志分析**等应用。
- 允许数据**短暂不一致，但最终一致**，提升性能。

➡ **如果应用需要严格保证 ACID，选择 SQL。** ➡ **如果应用可以容忍一定时间的不一致性，可选择 NoSQL。**

### **扩展性对比**

- **NoSQL 数据库之间无强关联，非常易扩展，适用于分布式架构。**
- 例如：
    - **Redis** 提供 **主从复制模式、哨兵模式、分片集群模式**，方便横向扩展。
- **SQL 数据库扩展较难，水平扩展受限**，需要解决：
    - **数据库分片*
    - **分布式事务**
    - **多表关联（JOIN）**

➡ **如果需要大规模扩展，选择 NoSQL。** ➡ **如果数据模型复杂，SQL 仍然是更好的选择。**



---


## **数据库的三大范式**

### **1. 第一范式（1NF）：确保每列的原子性**
**要求：** 数据表的**每一列**都是**不可再分的原子值**，即所有字段都应存储**最小的、不可拆分的数据**。

### **2. 第二范式（2NF）：消除部分依赖**
**要求：** **在满足 1NF 的前提下，消除非主键列对主键的** **部分依赖**。

### **3. 第三范式（3NF）：消除传递依赖**
**要求：** **在满足 2NF 的前提下，消除非主键列对主键的传递依赖。**


---


## **MySQL 如何避免重复插入数据？**

### **方式一：使用 UNIQUE 约束**
在表的相关列上添加 UNIQUE 约束，确保每个值在列中唯一。

```mysql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE,
    name VARCHAR(255)
);
```

如果尝试插入重复的 email，MySQL 会返回错误。

### **方式二：使用 INSERT ... ON DUPLICATE KEY UPDATE**
如果插入的记录与现有的记录冲突，可以选择更新现有记录：
```mysql
INSERT INTO users (email, name)
VALUES ('example@example.com', 'John Doe')
ON DUPLICATE KEY UPDATE name = VALUES(name);
```

### **方式三：使用 INSERT IGNORE**
如果 email 已存在，该条插入语句将被忽略而不会返回错
```mysql
INSERT IGNORE INTO users (email, name)
VALUES ('example@example.com', 'John Doe');
```


### **选择方法依据**
- **需要全局唯一性**：使用 `UNIQUE` 约束。
- **需要插入和更新结合**：使用 `ON DUPLICATE KEY UPDATE`。
- **快速忽略重复插入**：使用 `INSERT IGNORE`。


---


## **CHAR 和 VARCHAR 有什么区别？**
- **CHAR** 是固定长度的字符串类型，存储时会在末尾补足空格，适用于存储固定长度的数据，如代码、状态等。
- **VARCHAR** 是可变长度的字符串类型，存储时根据实际长度占用存储空间，适用于存储长度可变的数据，如用户输入的文本、备注等。


---

## **Text 数据类型可以无限大吗？**

MySQL 提供了 3 种 TEXT 类型，最大长度如下：
- **TEXT**：65,535 bytes（~64KB）
- **MEDIUMTEXT**：16,777,215 bytes（~16MB）
- **LONGTEXT**：4,294,967,295 bytes（~4GB）


---


## **MySQL 的关键字 IN 和 EXISTS**

在 MySQL 中，`IN` 和 `EXISTS` 都是用来处理子查询的关键字，但它们在功能、性能和使用场景上有所不同。


### **IN 关键字**
`IN` 用于检查左侧的表达式是否存在于右侧的列表或子查询的结果集中。如果存在，则 `IN` 返回 `TRUE`，否则返回 `FALSE`。

#### **语法结构：**

```mysql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1, value2, ...);
```

或

```mysql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (SELECT column_name FROM another_table WHERE condition);
```

#### **示例：**

```mysql
SELECT * FROM Customers
WHERE Country IN ('Germany', 'France');
```


### **EXISTS 关键字**

`EXISTS` 用于判断子查询是否至少能返回一行数据。它不关心子查询返回什么数据，只关心是否有结果。
如果子查询有结果，则 `EXISTS` 返回 `TRUE`，否则返回 `FALSE`。
#### **语法结构：**
```mysql
SELECT column_name(s)
FROM table_name
WHERE EXISTS (SELECT column_name FROM another_table WHERE condition);
```

#### **示例：**

```mysql
SELECT * FROM Customers
WHERE EXISTS (SELECT 1 FROM Orders WHERE Orders.CustomerID = Customers.CustomerID);
```


### **区别与选择：**

1. **性能差异：**
    - 在大多数情况下，`EXISTS` 的性能优于 `IN`，尤其是当子查询的表很大时。
    - `EXISTS` **一旦找到匹配项就会立即停止查找**，而 `IN` 可能会扫描整个子查询结果集。
2. **使用场景：**
    - 如果子查询结果集较小且不频繁变动，`IN` 可能更直观易懂。
    - 当子查询涉及外部查询的每一行判断，并且子查询的效率较高时，`EXISTS` 更为合适。
3. **NULL 值处理：**
    - `IN` 能够正确处理子查询中包含 `NULL` 值的情况。
    - `EXISTS` 不受子查询结果集中 `NULL` 值的影响，因为它关注的是结果的存在性，而不是具体值。

### **总结：**

- **如果子查询返回的结果集较小，或者子查询的数据不经常更新，使用** `IN`。
- **如果子查询结果集很大，或者查询逻辑更复杂，使用** `EXISTS` **更高效。**


---

### **SQL 查询执行顺序**

![[sql_order.webp]]

所有的查询语句都是从 `FROM` 开始执行，在执行过程中，每个步骤都会生成一个虚拟表，这个虚拟表将作为下一个执行步骤的输入，最后一个步骤产生的虚拟表即为输出结果。
#### **执行顺序：**

```
(9)  SELECT
(10) DISTINCT <column>,
(6)  AGG_FUNC (<column> or <expression>), ...
(1)  FROM <left_table>
(3)  <join_type> JOIN <right_table>
(2)  ON <join_condition>
(4)  WHERE <where_condition>
(5)  GROUP BY <group_by_list>
(7)  WITH {CUBE|ROLLUP}
(8)  HAVING <having_condition>
(11) ORDER BY <order_by_list>
(12) LIMIT <limit_number>;
```


---


## WITH ROLLUP 与 WITH CUBE 的区别

在 MySQL 的 `GROUP BY` 语句中，`WITH ROLLUP` 和 `WITH CUBE` 都用于计算聚合数据的汇总，但它们的作用有所不同。

### 1. **WITH ROLLUP**
`WITH ROLLUP` 用于计算**逐级汇总**，即按照 `GROUP BY` 指定的列顺序，逐层进行小计，最终计算一个总计。

#### **示例数据表（sales）：**

|region|product|sales_amount|
|---|---|---|
|East|A|100|
|East|B|150|
|West|A|200|
|West|B|250|

#### **查询** `WITH ROLLUP`
```mysql
SELECT region, product, SUM(sales_amount) AS total_sales
FROM sales
GROUP BY region, product WITH ROLLUP;
```

#### **查询结果：**

| region | product | total_sales   |
| ------ | ------- | ------------- |
| East   | A       | 100           |
| East   | B       | 150           |
| East   | NULL    | 250 (East 总计) |
| West   | A       | 200           |
| West   | B       | 250           |
| West   | NULL    | 450 (West 总计) |
| NULL   | NULL    | 700 (所有总计)    |

**特点：**
- 计算 `region + product` 的销售额。
- **只计算逐层汇总**，不会计算交叉组合。
- 适用于**层级数据统计**，如部门 → 职位 → 总计。

### 2. **WITH CUBE**
`WITH CUBE` 计算的是**所有可能的组合**，包括各个维度的独立汇总，以及交叉组合。
#### **查询** `WITH CUBE`

```java
SELECT region, product, SUM(sales_amount) AS total_sales
FROM sales
GROUP BY region, product WITH CUBE;
```

#### **查询结果：**

| region | product | total_sales   |
| ------ | ------- | ------------- |
| East   | A       | 100           |
| East   | B       | 150           |
| East   | NULL    | 250 (East 总计) |
| West   | A       | 200           |
| West   | B       | 250           |
| West   | NULL    | 450 (West 总计) |
| NULL   | A       | 300 (A 产品总计)  |
| NULL   | B       | 400 (B 产品总计)  |
| NULL   | NULL    | 700 (所有总计)    |

**特点：**
- 计算 `region + product` 的销售额。
- **计算所有可能的分组组合**，包括交叉组合。
- 适用于**多维度数据分析**，如 `地区 ↔ 产品 ↔ 总计`。

### **WITH ROLLUP vs WITH CUBE 总结**

|         | **WITH ROLLUP**           | **WITH CUBE**            |
| ------- | ------------------------- | ------------------------ |
| 汇总方式    | **逐层汇总**（按 `GROUP BY` 顺序） | **计算所有可能的分组组合**          |
| 适用场景    | **层级数据统计**（如公司 → 部门 → 岗位） | **多维度分析**（如地区 ↔ 产品 ↔ 时间） |
| 计算的总计类型 | 只计算逐级总计，不计算交叉组合           | 计算所有组合，包括交叉组合            |


### **适用场景**

- `WITH ROLLUP` 适用于**层级数据统计**，如：
    - 统计每个 `region` 的总销售额，并计算全国总额。
    - 统计每个 `部门` 的 `岗位` 人数，并计算全公司总人数。
- `WITH CUBE` 适用于**多维度分析**，如：
    - 计算 `地区` 和 `产品` 的所有组合的销售数据。
    - 统计 `用户年龄` + `消费品类` 组合的所有销售数据。


---

## 场景

1. 给学生表、成绩表，求不存在01课程但存在02课程的学生的成绩
2. 给定一个学生表Student_score(stu_id, subject_id, score)，查询总分排名在5-10名的学生id及对应的总分