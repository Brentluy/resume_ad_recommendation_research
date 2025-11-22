# 📌 Project Introduction — Building an Industrial-Grade CTR System from the AntMRC Dataset

When I started preparing for large-scale recommendation roles (ByteDance Global CRM, Google Personalization, Rokt MLE), one thing became very clear: **every strong candidate needs at least one end-to-end ranking project that truly “feels” industrial** — distributed computing, feature engineering, multi-task modeling, semantic embeddings, and cold-start handling. I didn’t want another toy dataset or a shallow Kaggle notebook. I wanted something that could actually demonstrate my ability to work at production scale.

That’s why I chose the **AntMRC dataset** — a real-world, multimodal dataset from Ant Financial, containing **mixed structured logs + rich text fields + multi-scene CTR labels**. Even though the raw data reaches the *million/10-million level*, I intentionally restricted myself to **only 10k samples** for the public Colab version. My goal was simple:  
➡️ *If a system is well-designed, even a 10k subset should reveal the same modeling patterns you’d need at full scale.*

Over the next few weeks, I treated this like a real company project. I wrote the Spark + HDFS ETL pipeline myself, designed the feature store layout, built user/item/UI features, gradually increased model complexity, added semantic towers, and finally solved cold-start with an LLM fallback. I didn’t just “stack models”; every step was motivated by a concrete issue I observed from the data.

The result surprised me even more:  
| Stage | Model | Overall AUC |
|------|-------|-------------|
| Baseline | Simple DNN | **0.74–0.75** |
| Multi-task | MMoE | **0.80–0.83** |
| Expert routing | PLE | **0.82–0.86** |
| Semantic features | MMoE + BERT-Whitening | **0.86–0.87** |
| Cold-start enhancement | + LLM Fallback | **0.87–0.93** |
| **Final fusion system** | **MMoE + Content Tower + α-Gate + BERT + LLM** | **0.9739 (val) / 0.9717 (test)** |

A total lift of **+24 percentage points**, achieved through systematic engineering — not guesswork.

Looking back, this project reflects who I am as an engineer:  
- I start from **data reliability** (Spark ETL) before touching any model.  
- I take **feature design seriously** — user, item, cross-features, rolling windows, scene-pivot stats.  
- I build models the way real companies do: **MMoE → PLE → fusion gates → semantic towers → LLM fallback**.  
- I debug like someone who has been burned by real systems — always watching distribution drift, cold-start performance, input schema stability.  
- And I constantly balance **accuracy vs cost**, because no ranking system runs without constraints.

Even though this public repo only trains on 10k rows, the full pipeline is purposely designed so it can scale to **tens of millions** immediately. The architecture (Spark → ORC → feature store → PyTorch MMoE/PLE → semantic tower → LLM fallback → evaluation suite) mirrors what major tech companies use internally. If given the full AntMRC corpus or an actual production dataset, I am confident this system would continue to push AUC even higher.

More importantly, this project convinced me that I genuinely enjoy this type of work — debugging ETL jobs, designing expert networks, understanding scene behavior, improving long-tail recall, and making cold users “come alive” through text signals. This is exactly the kind of end-to-end ownership expected from ML Engineers in ByteDance Ads, Google Recsys, or Rokt’s Ranking team.

This repository is my way of showing that — even as a student — **I can already think, structure, and deliver like a real recommender-system engineer**.


## CHINESE 中文
# 📌 项目介绍 — 用 AntMRC 数据集从零构建工业级 CTR 推荐系统

在准备字节跳动、Google、Rokt 这类一线大厂的推荐算法 / MLE / Ranking 岗位时，我很快意识到：**想让人相信你的能力，必须拿出一个真正“工业味”很强的端到端推荐系统项目**。  
不是 Kaggle 那种练习性质的数据，也不是几十行代码的玩具模型，而是要覆盖——分布式数据处理、特征工程、深度 CTR 架构、多任务学习、文本语义融合、冷启动策略、线上可复现的评估体系。

这也是我选择 **AntMRC 数据集** 的原因。  
AntMRC 是蚂蚁金服开源的大规模推荐场景数据集，特点非常“真实”：

- 同时包含 **结构化行为日志 + 文本实体序列**（多模态）  
- 标签是多场景 CTR（A/B/C/D/E 五个场景）  
- 原始规模可以达到**百万级甚至上千万**  
- 数据分布复杂，非常接近真实广告/推荐流量

为了能开源、能在 Colab 上跑，我只抽取了 **1 万行样本**。  
但我给自己定下的目标是：  
➡️ **即使只有 10k 行，也要把整个系统搭成“能在公司上线”的标准。**

所以整个项目我完全按“工程项目”来推进，而不是按“学术实验”推进。

---

## 🚀 从 0 到 1 的系统搭建之旅（像在真实公司做项目）

我花了两周，从数据清洗开始，一步一步往前推。  
虽然借助了 ChatGPT 来写代码，但**每一步的方向、取舍、架构设计都是我自己做决策的**。

整个指标的提升路径非常清晰：

| 模型阶段 | 方法 | AUC（Val） | 提升 |
|---------|------|------------|-------|
| Step 1 | 最简单 DNN（全结构化特征） | **0.73** | — |
| Step 2 | MMoE 多任务学习 | ~0.80+ | **+7pts** |
| Step 3 | PLE 结构化分层专家网络 | ~0.86–0.87 | **+6pts** |
| Step 4 | BERT-Whitening 文本语义塔（128维） | ~0.90+ | **+4pts** |
| Step 5 | 场景级 Content Tower（文本分场景纠偏） | ~0.93 | **+3pts** |
| Step 6 | LLM Fallback（冷启动文本补充） | **0.9739** | **+4pts** |
| Step 7 | 最终端到端评估（Val/Test） | **0.97+** | — |

➡️ **总提升：从 0.73 → 0.97，整整 +24 个百分点。**

指标能爬这么高，不是“堆模型”堆出来的，而是——  
**每一步都是为了解决上一阶段出现的问题。**

比如：

- 用户/物品稀疏 → 做 7 日窗口、场景 pivot  
- 多场景之间互相干扰 → MMoE / PLE  
- 文本字段有价值但维度噪声大 → BERT → Whitening  
- 某些场景天然数据极少（典型冷启动） → 文本塔 → LLM fallback  
- 长尾 user/item 没历史 → LLM embedding 做兜底  
- 模型知道结构特征，但不知道语义 → 融合 gate α  

整个系统不是“实验堆叠”，而是一个非常标准的工业链路：

**Spark ETL → 特征库 → 多模态融合 → 多任务建模 → 文本增强 → 冷启动诊断 → 完整模型产物（artifact）**

---

## 📦 AntMRC 数据集：为什么适合做工业级项目？

AntMRC 最吸引我的地方有几点：

- 数据体量大，能让学生也体验“准工业级”的难度  
- 同时包含 **query 实体序列、服务实体序列、item_title、item_entities** 等大量文本字段  
- 分场景（多任务）CTR，本身非常适合作 MMoE/PLE  
- 数据中天然存在冷启动（尤其是 item）  
- 标签真实，分布不平衡，需要真正的工程思考

说直白点：  
**这是一个学生能拿到，且和字节 / 阿里 / 美团真实广告数据最接近的公开数据集之一。**

---

## 🧩 拿到数据之后，我的整体思路是什么？

我给自己设定的约束是：  
> “把自己当成公司里唯一负责推荐系统的 ML Engineer，从0把系统搭出来。”

所以整个推进顺序完全模仿一线公司内部流程：

1. **先把 ETL 建扎实**：Spark + HDFS，统一 ORC，保证 schema 干净  
2. **构建特征仓库**：用户 / 物品 / 交互 → 7日窗口 + 长期累计 + 场景分布  
3. **构建训练表**：把特征按 date join 回主表  
4. **搭 baseline**：DNN，先看模型能否“吃进去”分布  
5. **处理多场景歧义**：MMoE / PLE 多任务  
6. **注入语义感知能力**：BERT→Whitening  
7. **处理场景差异和文本偏置**：Content Tower  
8. **处理冷启动问题**：LLM fallback（只对 cold/few-shot 计算 embedding，性价比极高）  
9. **完整的冷启动评估体系**：user_lvl × item_lvl × scene  
10. **保存可复现模型产物（artifacts）**：超参、Whitening 矩阵、特征 meta、模型权重、报告

这种流程不是“学生作业”，而是公司里 ML engineer 的实际工作方式。

---

## 🧠 我从这个项目里得到的思考与成长

做完之后，我觉得最重要的收获有三点：

### ① 推荐系统不是“调模型”，而是“调信息流”
我从一开始就不是在 stack 网络结构，而是在不断回答：

- 模型缺什么信息？  
- 用户/物品的行为如何跨场景迁移？  
- 哪些维度缺失？（冷启动）  
- 文本怎么和结构特征更好地融合？  
- 哪些部分可以让模型“更懂业务”？  

当我加上正确的信息（比如文本语义塔、冷启动LLM向量）时，AUC 立刻暴涨。

### ② 工程化能力比模型本身更重要  
所有真实推荐组招人，都更看重这些：

- 是否能保证数据层可靠  
- 能不能做多天级特征  
- 能不能设计可迭代的 feature pipeline  
- 是否知道如何做 production-style evaluation  
- 是否能用成本合理的方式提升冷启动效果  

这个项目让我把这些能力都串在一起打磨了一遍。

### ③ 这就是我想做的工作  
越做越觉得：  
**我就是喜欢把复杂的系统从混乱变成可控的感觉。**  
尤其是：

- 分析流量分布  
- 看 AUC 在不同 user_lvl / item_lvl 下的表现  
- 思考如何让模型“更聪明”  
- 用 LLM 做语义增强  
- 设计 gate、expert、tower 的协同逻辑  

我发现这就是我未来工作里最享受的一部分。

---

## 🎯 为什么它和 ByteDance / Google / Rokt 非常对口？

因为这三类公司都做同一件事：

### ✔ 复杂、多场景、多模态的推荐  
（和 AntMRC 一模一样）

### ✔ 多任务结构（MMoE / PLE）是标配  
（字节跳动广告、Google Play、YouTube 都在用）

### ✔ 文本 + 结构化的混合特征  
（搜索/广告的核心）

### ✔ 冷启动永远是痛点  
（LLM fallback 是最前沿方向）

### ✔ 端到端工程能力远胜“单点调参”  
（这正是本项目最强的部分）

---

## ⭐ 总结：这是一个用来证明“我能胜任一线推荐系统岗位”的项目

它包含：

- 分布式 ETL  
- 特征仓库  
- 深度 CTR 模型  
- 多任务学习（MMoE / PLE）  
- 文本语义塔（BERT-Whitening）  
- LLM Fallback 冷启动  
- 完整冷启动分析体系  
- 模型产物（artifacts）  
- 可扩展到千万级流量的架构  

更重要的是，它体现了我作为一个 MLE 的完整思考过程。  
不仅能“写模型”，也能**理解数据、业务、成本、分布、稀疏性、冷启动、可部署性**。

如果让我用全量数据（百万~千万级）继续训练，我对把 AUC 推到 **0.98+** 充满信心。

这个项目，就是我向一线公司证明：  
➡️ **我已经具备真正的推荐系统工程能力，随时可以接入你们的生产系统。**
