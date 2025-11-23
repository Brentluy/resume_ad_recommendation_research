# CTR Prediction with Multimodal Fusion & Cold-Start Modeling  
### **Project Introduction | 项目介绍**

### **English**

When I started this project, my goal was not simply to “run a model” — I wanted to prove that I can design and execute a **full, industry-grade recommendation pipeline** in a way that reflects how teams at Google, ByteDance, and ROKT actually work.  
As a student, I don’t yet have years of production experience, but I wanted to show that I *think* like someone who does:  
curious about failure modes, careful with data, and always pushing for robustness rather than chasing a single AUC number.

That mindset shaped the entire project.

I chose the **AntM2C (蚂蚁 MRC)** dataset because it is one of the few publicly available datasets that contain:  
- multi-scene business labels,  
- user history, item history,  
- rich text attributes (titles, entities),  
- and real cold-start patterns.

Even when I used only **10k samples**, the dataset already exposed the core problems of modern CTR prediction — multi-task conflict, sparse interactions, and unstable performance on new users/items.  
This forced me to use the exact techniques real companies rely on.

Across this project, every modeling step brought a meaningful and explainable improvement:

| Stage | Model | Overall AUC |
|------|-------|-------------|
| Baseline | Simple DNN | **0.74–0.75** |
| Multi-task | MMoE | **0.80–0.83** |
| Expert routing | PLE | **0.82–0.86** |
| Semantic features | MMoE + BERT-Whitening | **0.86–0.87** |
| Cold-start enhancement | + LLM Fallback | **0.87–0.93** |
| **Final fusion system** | **MMoE + Content Tower + α-Gate + BERT + LLM** | **0.9739 (val) / 0.9717 (test)** |

These improvements were not “accidental”—each was motivated by a specific business intuition:

- “Scenes behave differently → multi-task.”  
- “Titles/entities contain real intent → BERT semantic features.”  
- “History is missing for new IDs → LLM fallback as a safety net.”  
- “The model should know *when* to trust text → learnable α-gating.”  

The final pipeline resembles the architecture used in many large-scale recommender systems today:  
a **hybrid fusion model** that dynamically balances structural features, BERT embeddings, and LLM-based signals.

If trained on the **full AntM2C dataset** (millions of samples), the same architecture could likely reach **production-grade AUC**, matching what modern ranking teams see internally.

More importantly, this project demonstrates the way I approach problems:  
structured, curious, engineering-driven, and always thinking about robustness and business impact.

---

### **中文**

一开始做这个项目，我的目标并不是“跑通一个模型”，  
而是想证明——即使我还是学生，我也能够用**大厂 MLE 的方式**去思考、拆解、构建一个完整的推荐系统。

我没有多年上线经验，但我可以展现出一种成熟的思路：  
**关注冷启动、关注多场景差异、关注多模态融合、关注可复现性，而不是单纯堆模型。**

这套思路贯穿了整个项目。

我选用 **AntM2C（蚂蚁 MRC）** 数据集，是因为它非常贴近工业环境：  
既有业务场景（A–E），又有用户/物品历史，还有丰富的文本信号，同时具备真实的冷启动形态。  
即便我只用其中 **1 万条数据**，它依然完整暴露了推荐系统最关键的问题：

- 多任务冲突  
- 稀疏交互  
- 用户/物品冷启动  
- 文本语义与结构行为如何融合  

正因为这些挑战存在，我必须采用真实公司里常用的方案：


  | Stage | Model | Overall AUC |
|------|-------|-------------|
| Baseline | Simple DNN | **0.74–0.75** |
| Multi-task | MMoE | **0.80–0.83** |
| Expert routing | PLE | **0.82–0.86** |
| Semantic features | MMoE + BERT-Whitening | **0.86–0.87** |
| Cold-start enhancement | + LLM Fallback | **0.87–0.93** |
| **Final fusion system** | **MMoE + Content Tower + α-Gate + BERT + LLM** | **0.9739 (val) / 0.9717 (test)** |


- **DNN 基线**只有 ~0.74 AUC；  
- 换成 **MMoE/PLE**，提升到 **0.83–0.87**；  
- 引入 **BERT（Whitening）**后语义能力显著增强；  
- 加上 **LLM 冷启动兜底向量**，冷启动表现大幅稳定；  
- 最终的 **MMoE + 文本塔 + α-门控** 能动态决定“何时该信结构、何时该信文本”，  
  全局 AUC 达到 **0.9739（val）/0.9717（test）**。

  

每一步提升都有明确的业务逻辑支撑：

- “不同业务场景 → 必须多任务”；  
- “标题/实体是用户真实意图 → 必须语义向量”；  
- “没有历史的用户物品必须有补救方案 → LLM 兜底”；  
- “模型要自己学会取舍 → α-门控融合”。  

最后的系统，与 Google、字节跳动、ROKT 内部的 CTR 框架在理念上高度一致：  
是一个**结构化特征 + BERT + 大模型文本**的混合式推荐架构。

如果未来用全量百万级数据训练，我相信这套体系可以达到真正的线上效果。

更重要的是，这个项目展示了我处理问题的方式：  
工程化、系统化、有业务感、有好奇心、也愿意做困难但正确的事情。

