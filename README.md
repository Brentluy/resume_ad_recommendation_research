# 📌 Project Introduction — Building an Industrial-Grade CTR System from the AntMRC Dataset

When I started preparing for large-scale recommendation roles (ByteDance Global CRM, Google Personalization, Rokt MLE), one thing became very clear: **every strong candidate needs at least one end-to-end ranking project that truly “feels” industrial** — distributed computing, feature engineering, multi-task modeling, semantic embeddings, and cold-start handling. I didn’t want another toy dataset or a shallow Kaggle notebook. I wanted something that could actually demonstrate my ability to work at production scale.

That’s why I chose the **AntMRC dataset** — a real-world, multimodal dataset from Ant Financial, containing **mixed structured logs + rich text fields + multi-scene CTR labels**. Even though the raw data reaches the *million/10-million level*, I intentionally restricted myself to **only 10k samples** for the public Colab version. My goal was simple:  
➡️ *If a system is well-designed, even a 10k subset should reveal the same modeling patterns you’d need at full scale.*

Over the next few weeks, I treated this like a real company project. I wrote the Spark + HDFS ETL pipeline myself, designed the feature store layout, built user/item/UI features, gradually increased model complexity, added semantic towers, and finally solved cold-start with an LLM fallback. I didn’t just “stack models”; every step was motivated by a concrete issue I observed from the data.

The result surprised me even more:  
➡️ **Baseline DNN AUC: 0.73**  
➡️ **MMoE multi-task AUC: ~0.80+**  
➡️ **PLE expert routing AUC: ~0.86–0.87**  
➡️ **Adding BERT-Whitening embeddings: ~0.90+**  
➡️ **Introducing task-level content towers: ~0.93**  
➡️ **Final Fusion + LLM cold-start fallback: 0.9739 (Val), 0.9717 (Test)**  
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
