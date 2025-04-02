## **Transformer 架构的计算流程简述（适合有神经网络基础者）**

**Transformer 是一种完全基于注意力机制（Self-Attention）的序列建模架构，被广泛用于大语言模型（如 GPT、BERT）中。**

---

### **一、核心理念**

> 抛弃 RNN 和 CNN，完全依赖注意力机制来建模序列中词与词之间的关系，能够并行处理序列，训练效率高，效果强。

---

### 二、基本结构（以编码器为例）

```
输入 Embedding → 多层 Encoder（每层：Self-Attention + Feed Forward）→ 输出
```

每层 Encoder 的结构如下：

---

### 三、一层 Transformer Encoder 的计算流程
#### 1. 输入准备
- 输入句子（如：["I", "am", "Kevin"]）
- 每个词转换为词向量（embedding），并加入位置编码（Position Embedding），形成输入矩阵 X

#### 2. Self-Attention 机制
- 核心公式：
$Attention(Q, K, V) = softmax(QK^T / \sqrt{d_k}) * V$

- 从输入 X 得到 Q（Query）、K（Key）、V（Value）：
$Q = XW^Q,\quad K = XW^K,\quad V = XW^V$

- 每个词都对其他词计算注意力权重，然后加权平均 V 向量

> 本质：每个词“看”所有其它词，决定该关注谁

#### 3. 残差连接 + LayerNorm（第1次）
- 将 Attention 输出与原输入做残差相加，再做 LayerNorm：

$Z_1 = LayerNorm(X + Attention(Q, K, V))$

#### 4. Feed Forward Network（前馈网络）
- 对每个词单独做两层全连接网络（带激活函数 ReLU 或 GELU）

#### 5. 残差连接 + LayerNorm（第2次）
- 将前馈网络输出与 Z1 相加，再做 LayerNorm
$Z_2 = LayerNorm(Z_1 + FFN(Z_1))$


---

### **四、Transformer Decoder 计算流程详解**

Transformer 的 Decoder 负责**根据已生成的词，逐步预测下一个词**。它由多个 Decoder Layer 堆叠组成，每一层包含三大核心模块。


#### 📦 Decoder 的整体结构

```text
输入 Embedding（含位置编码）
↓
多层 Decoder（每层包含三部分）：
  1. Masked Self-Attention
  2. Encoder-Decoder Attention
  3. Feed Forward Network
↓
输出向量（最终接 softmax 预测词）
```

#### 🧱 每层 Decoder 的详细组成与流程

##### 1️⃣ Masked Multi-Head Self-Attention
- 与 Encoder 的 Self-Attention 结构相同，但在 **softmax 前加入了 mask**。
- Mask 是一个上三角矩阵，确保当前词**只能关注自己和它之前的词**，防止信息“泄露”未来内容。
- 适用于训练和生成阶段的自回归性质。
![[decoder_formula.png]]
- 每个词只能看到前面的词，符合生成式语言模型的性质。

上三角矩阵：

![[上三角.png]]

##### 2️⃣ Encoder-Decoder Attention（跨模块注意力）
- 让 Decoder 当前词去“看”Encoder 的所有输出（上下文表示）。
- 输入来自两边：
    - Query：来自 Decoder（上一步的输出）
    - Key 和 Value：来自 Encoder（编码后的上下文）
- 这样 Decoder 就能**结合输入序列的信息**来生成合理的输出。


##### 3️⃣ Feed Forward Network（前馈网络）
- 和 Encoder 中一致，每个位置上使用：
    - 两层线性变换，中间加激活函数（ReLU 或 GELU）

```text
FFN(x) = max(0, xW₁ + b₁)W₂ + b₂
```


##### 4️⃣ 残差连接 + LayerNorm（每步都有）
- 每个子模块都加了残差连接 + LayerNorm，提升训练稳定性：
```text
x = LayerNorm(x + SubLayer(x))
```


#### 🔁 多层堆叠
- Transformer Decoder 通常由 N 层组成（如 GPT-2 为 12 层，GPT-3 为 96 层）。
- 每一层都能提取更深层的语言模式和上下文关系。


#### 🔚 Decoder 最后的输出
- Decoder 最后一层输出的是每个位置的词向量表示。
- 最终接一个线性层 + softmax，输出一个词表分布，预测下一个词：

```text
P(word_t | word_1, ..., word_{t-1}) = softmax(W * output_t + b)
```


#### ✅ 总结一句话：
> Transformer 的 Decoder 就是在不断“看前文 + 看上下文”，然后一步步地预测出下一个词，从而完成生成。


---

### 五、多层堆叠
- 将上述结构堆叠 N 层（如 BERT-base 有 12 层）
- 每层可以学习更深层次、更长距离的依赖关系

---

### 六、总结一句话：

> Transformer 就是不断地让每个词“看”其它所有词，并学会该关注谁，然后通过前馈网络不断优化表示，堆叠多层之后就能很好地建模语言上下文。




----



## 纯Encoder、纯Decoder和Encoder-Decoder的一些示例
