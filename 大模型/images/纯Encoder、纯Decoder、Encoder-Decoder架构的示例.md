

## ✅ 1. **纯 Encoder 模型**（只用 Encoder）

### 🧠 特点：
- 处理的是**理解类任务**，比如分类、问答、文本匹配、摘要等。
- 输入一次性全部送进 Encoder，输出一个整体语义表示。

### 📌 代表模型：

| 模型名                                                               | 简介                               |
| ----------------------------------------------------------------- | -------------------------------- |
| **BERT**（Bidirectional Encoder Representations from Transformers） | 使用双向 Encoder，适合文本理解              |
| **RoBERTa**                                                       | 更大数据+更久训练版 BERT                  |
| **ALBERT**                                                        | 参数共享+分解，轻量版 BERT                 |
| **DeBERTa**                                                       | 加入相对位置编码和 disentangled attention |
| **ELECTRA**                                                       | 用判别器替代 MLM 训练，效率高                |


## ✅ 2. **纯 Decoder 模型**（只用 Decoder）
### 🧠 特点：
- 处理的是**生成类任务**，比如写文章、对话生成、代码生成。
- 自回归方式，一个词接一个词生成。

### 📌 代表模型：

|模型名|简介|
|---|---|
|**GPT 系列（GPT-1、GPT-2、GPT-3、GPT-4）**|基于纯 Decoder 的自回归语言模型|
|**GPT-Neo / GPT-J / GPT-NeoX**|开源社区复刻 GPT 的纯 Decoder 模型|
|**LLaMA 系列（Meta）**|更高效的轻量化纯 Decoder 模型，适合推理|
|**CodeGen / StarCoder**|专注于代码生成的纯 Decoder 模型|


## ✅ 3. **Encoder-Decoder 模型**（经典结构）
### 🧠 特点：
- 输入 → 编码为上下文 → Decoder 解码生成输出
- 适合需要“翻译”的任务，比如翻译、多轮问答、摘要等

### 📌 代表模型：

|模型名|简介|
|---|---|
|**原始 Transformer（Vaswani 2017）**|最早提出 Encoder-Decoder 架构|
|**T5（Text-to-Text Transfer Transformer）**|把所有任务转换为“文本输入 → 文本输出”，效果强|
|**BART**|BERT + GPT 的组合结构，支持 Encoder-Decoder|
|**mT5 / mBART**|多语言版本的 T5/BART，用于跨语言翻译|


## ✅ 总结对比表

|架构类型|代表模型|常用于任务类型|
|---|---|---|
|纯 Encoder|BERT, RoBERTa, DeBERTa|文本分类、句子理解、抽取|
|纯 Decoder|GPT, LLaMA, StarCoder|文本生成、对话、代码生成|
|Encoder-Decoder|Transformer, T5, BART|翻译、摘要、问答、对话|