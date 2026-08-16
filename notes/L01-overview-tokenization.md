# L1 概览与分词 (Overview and Tokenization)

视频：https://www.youtube.com/watch?v=SQ3fZ1sAqXI

## 我认为重要的点

<!-- 在这里记录你看视频时觉得重要的内容 -->

## 我不明白的问题

- Efficiency lens 是什么？
- Scaling law 是什么？
- byte 和 token 之间怎么换算？

## 讨论总结

**Efficiency lens（效率视角）**：这门课评判"好坏"的标准不是方法优不优雅，而是"给定固定计算预算（FLOPs）和数据，谁能把 loss 压得更低"。后面讲的分词方式、架构选型、超参数、并行策略都要用这把尺子衡量——每一分算力换回多少模型质量。对 Agent 方向的意义：做 Agent 系统时的选型（大模型少调用 vs 小模型多轮迭代）本质也是同一个"给定预算，最大化有效产出"的框架。

**Scaling law（缩放定律）**：模型 loss 会随参数量 N、数据量 D、算力 C 增大按可预测的幂律曲线下降，因此可以用小规模实验拟合出曲线，外推预测大模型的表现，而不用真的先烧钱训出来。L1 只是预告概念，L9/L11 会展开数学细节。它是 efficiency lens 能落地的工具——把"模型开多大、数据喂多少"从拍脑袋变成可计算。

**byte ↔ token 换算**：没有固定公式，比例是 BPE 训练出来的经验压缩率，必须实际跑 encode 才知道确切数字。取决于：待编码文本和训练语料是否同分布、训练时的 vocab size、文本本身的重复度。经验数值（以英文语料训练的 byte-level BPE 为例）：

- 英文文本：大约 4 bytes ≈ 1 token（约等于 0.75 个单词 ≈ 1 token）
- 中文文本：情况更差——一个汉字在 UTF-8 里是 3 bytes，但因为训练语料里英文占比更高，词表里给汉字分配的完整 merge 往往不够，很多汉字会被切成 2 个甚至更多 token。结果是中文常出现 1 个汉字 ≈ 1.5～2 个 token，bytes/token 比英文低不少——同样语义内容，中文"更贵"，也是中文 Agent 应用更容易撑爆 context window 的原因之一。

这个比值本质上就是 BPE 训练时在优化的目标：给定 vocab size 这个"预算"，最大化压缩率（bytes/token）——和 efficiency lens 是同一套逻辑，只是预算从 FLOPs 换成了词表大小。

要查确切比值，最直接的办法是实际编码一段文本，比较 `len(text.encode('utf-8'))` 和 `len(tokenizer.encode(text))`。
