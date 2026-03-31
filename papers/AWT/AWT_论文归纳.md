Adversarial Watermarking Transformer
Towards Tracing Text Provenance with Data Hiding

论文阅读归纳笔记

Sahar Abdelnabi, Mario Fritz  |  CISPA Helmholtz Center for Information Security  |  IEEE S&P 2021

## 一、背景与问题
### 1.1 核心问题
近年来，基于 Transformer 的大型语言模型（LLM）在文本生成任务上取得了突破性进展，但随之而来的是严重的滥用风险：虚假信息传播、学术不诚信、深度伪造文本等。如何追踪机器生成文本的来源（Provenance Tracing）成为亟待解决的安全问题。

本文研究自然语言水印（Natural Language Watermarking）作为一种可持续的防御手段，目标是在文本中自动、无感知地嵌入可验证的水印信息，从而实现对机器生成文本的溯源与检测。
### 1.2 现有方法对比
### 1.3 本文方法的三个核心属性
## 二、核心方法
### 2.1 整体架构
AWT（Adversarial Watermarking Transformer）是首个端到端的文本数据隐藏模型，无需配对训练数据，自动学习词汇替换位置和替换词。整体架构由三个核心组件构成：

### 2.2 Hiding Network 详解
#### 2.2.1 Encoder（编码器）
Encoder 是一个标准的 Transformer Encoder 模块，由多层 Transformer Encoder Layer 堆叠而成，每层包含自注意力（Self-Attention）块和全连接层。
输入：句子 S = {W₀, W₁, ..., Wₙ}，词汇经 One-hot 编码后映射到 embedding 空间，加入正弦位置编码（Sinusoidal Embeddings）。
输出：对最后一层 Encoder 输出在时间维度上做平均池化，得到固定长度的句子向量表示。
#### 2.2.2 Message 处理
输入消息 M = {b₀, b₁, ..., b_q}（q 位随机二进制比特），经全连接层（FC Layer）映射到与 embedding 相同的维度，然后与 Encoder 输出的句子编码相加，形成句子与消息的共享 embedding，在 Decoder 每个时间步都加入此共享 embedding。
#### 2.2.3 Decoder（解码器）
Decoder 架构与 Encoder 类似，额外增加了对 Encoder 输出的 Attention 层（类似机器翻译中的 Cross-Attention）。Decoder 以自回归方式生成输出文本序列。
关键技术：使用 Gumbel Softmax 处理文本的离散性，使梯度可以通过离散的词汇选择反向传播，实现端到端训练。
权重共享：为减少参数量并加速收敛，embedding 权重在整个网络中共享（text autoencoder、message decoder、discriminator、pre-softmax 层）。
### 2.3 Revealing Network（揭示网络）
Revealing Network 是一个独立的 Transformer Encoder，只接收输出文本 S'（不接收原始输入文本），负责从水印文本中重建原始二进制消息 M̂。
与 Hiding Network 的 Encoder 共享 embedding 权重，进一步减少参数量。
### 2.4 Discriminator 与对抗训练
Discriminator 是一个 Transformer Encoder 分类器，任务是区分原始输入文本（真实）和水印输出文本（伪造）。
对抗训练目标：Hiding Network 试图生成让 Discriminator 无法区分的输出文本；Discriminator 试图正确区分两者。这种对抗训练迫使水印编码更加隐蔽，减少可检测的统计特征。
实验验证：加入 Discriminator 后，SBERT Distance 显著降低，说明输出文本语义质量更好；同时分类器 F1 更接近 0.5，说明保密性更强。
### 2.5 训练策略（两阶段）
两阶段训练的原因：第一阶段让模型先学会基本功能（warm start），第二阶段再引入语义约束，避免从头训练时收敛困难。
数据集：WikiText-2（WT2），训练时随机采样 4-bit 消息，每段 80 词（N(80,5) 变长）。
### 2.6 推理策略
### 2.7 文档级水印验证
长文本处理：将文档分段（每段 80 词），每段嵌入 4-bit 消息，拼接成长水印序列。
零假设检验：H₀ = 匹配比特数是随机产生的。在 H₀ 下，匹配比特数 X ~ Binomial(n, 0.5)。
验证公式：Pr(X > k | H₀) = Σᵢ₌ₖⁿ C(n,i) × 0.5ⁿ，若 p-value < T（T=0.05 或 0.01）则验证通过。
文本越长，可积累的观测越多，验证置信度越高；在较低 Bit Accuracy 的操作点下，也可通过增加文本长度来保证验证成功率。
## 三、实验设置
### 3.1 数据集
### 3.2 评估指标
## 四、实验结果
### 4.1 有效性评估
消融实验：对比完整 AWT、无 Fine-tuning 的 AWT、无 Discriminator 且无 Fine-tuning 的 AWT 三个变体。
推理策略效果：Best-of-many 中 n 从 1 增加到 40 时，METEOR 持续提升，SBERT Distance 持续降低，但 Bit Accuracy 略有下降（可通过 Averaging 策略补偿）。
### 4.2 保密性评估
实验设置：在非配对的水印/非水印文本上训练三种分类器（Transformer、LSTM、BoW-SVM），评估其区分能力（F1 越接近 0.5 越好）。
### 4.3 鲁棒性评估
核心结论：水印与文本语义深度绑定。攻击者若想彻底去除水印，必须大规模修改文本，这与其保留原文语义的目标相矛盾。
### 4.4 用户研究
## 五、与基线对比
## 六、一页速查
