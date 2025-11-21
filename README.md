# Ad Recommendation on AntM2C Logs (Spark + PyTorch MMoE)

## Summary

This project is an end-to-end ad recommendation pipeline built entirely from scratch, designed to demonstrate my ability to work with large-scale log data, Spark + HDFS ETL, feature engineering, and modern deep CTR models (MMoE / PLE + text embeddings).

The data used is a public/synthetic version of AntM2C click logs. I use it to recreate the full workflow of a real-world recommendation system:

- Distributed ETL with Spark + HDFS
- Convert CSV → ORC → Parquet (partitioned by date/scene)
- Generate rich user / item / user-item features
- Train CTR models:
  - Baseline DNN
  - MMoE
  - PLE
- Integrate text embeddings (BERT / m3e + Whitening) for cold-start improvement
- Evaluate cold-start performance (user zero/few/warm, item zero/few/warm, per-scene)

This project demonstrates production-grade recommendation engineering from data → feature store → deep model → evaluation.

---

## 1. Project Overview

This repository implements a multi-scenario CTR prediction system on AntM2C-format ad logs:

1. Read raw CSV logs from HDFS  
2. Convert logs to ORC partitions  
3. Generate user, item, and user-item features  
4. Join features into a training table (ORC/Parquet)  
5. Train CTR models (DNN, MMoE, PLE)  
6. Add text embeddings using BERT/m3e + Whitening  
7. Perform cold-start evaluation  

---

## 2. Dataset

Raw CSV files stored in:

hdfs://namenode:9000/data/antm2c/raw_10k/date=*/part-*.csv

Columns:
- user_id, item_id
- scene
- log_time
- label
- optional text fields: item_title, item_entity_names, query_entity_seq

Tech stack:
- Spark + HDFS
- ORC / Parquet
- PyTorch
- transformers / sentence-transformers

---

## 3. Spark ETL

### 3.1 CSV load

Basic HDFS sanity check, row count.

### 3.2 CSV → ORC

Steps:
- parse timestamp to log_ts
- extract date
- cast label, scene
- drop corrupted rows
- write ORC partitioned by (date, scene)

---

## 4. Feature Engineering

### 4.1 User Features

Includes:
- u_imp_day, u_clk_day
- 7-day rolling sums (Window.rowsBetween(-6, 0))
- cumulative impressions/clicks/CTR
- per-scene cumulative stats (pivoted): u_imp_s0..s4, u_clk_s0..s4

### 4.2 Item Features

Same structure as user:
- daily
- 7-day
- cumulative CTR

### 4.3 User–Item Interaction Features (no leakage)

Use cumulative window ending at previous day:
Window.unboundedPreceding → -1

Outputs:
- ui_imp_hist
- ui_clk_hist

These ensure label leakage is prevented.

### 4.4 Save Feature Tables

Each feature block saved as ORC partitioned by date:
- features_user_10k
- features_item_10k
- features_ui_10k

---

## 5. Training Table Join

Base logs + user features + item features + ui_hist are joined into:

/data/antm2c/train_10k_parquet/

Then loaded into Pandas for modeling.

---

## 6. Baseline DNN

Steps:
- Encode user_id, item_id to categorical indices
- Build simple MLP: Linear → ReLU → Dropout → Linear → Sigmoid
- Train with BCE loss
- Evaluate ROC-AUC

Serves as baseline.

---

## 7. MMoE (Multi-gate Mixture-of-Experts)

Scenes are treated as tasks (task_id = scene).

Architecture:
- K experts shared globally
- Each task has its own gate producing softmax weights
- Task tower produces final output

Benefits:
- Captures shared and task-specific patterns simultaneously.

---

## 8. PLE (Progressive Layered Extraction)

Decomposes representation into:
- shared experts
- task-specific experts

Each task gate mixes:
(shared_experts + task_specific_experts)

More expressive and common in production systems.

---

## 9. Text Features (BERT / m3e + Whitening)

Construct combined text from:
- item_title
- item_entity_names
- query_entity_seq

Embed using:
- bert-base-chinese, or
- moka-ai/m3e-base

Whiten text embeddings to 128-dim:
- compute covariance SVD
- W = U[:, :128] / sqrt(S[:128])
- EW = (E - mean) @ W

Append to numeric features.

---

## 10. MMoE + Content Tower + α-Gate

Add a second branch (content tower) that sees only text embeddings.

Final prediction:
final = (1 − α) * mmoe + α * content

Where α is learned from content embedding per-task.

This significantly improves cold-start user/item performance.

---

## 11. Cold-Start Evaluation

User/item are classified as:
- zero (unseen)
- few (rare)
- warm (frequent)

AUC is reported for:
- overall
- user zero/few/warm
- item zero/few/warm
- each scene
- (scene × user level)
- (scene × item level)

This mimics industry ad-ranking dashboards.

---

## 12. Saved Artifacts

Each run outputs to:

artifacts_mmoe_run_YYYYMMDD-HHMMSS/

Files:
- mmoe_state.pt  
- mmoe_config.json  
- whiten_mu.npy, whiten_W.npy  
- feature_meta.json  
- cold_report_long.csv  
- cold_report_pivot.csv  
- val_metrics.json  

---

## 13. How to Run

1. Start Spark + HDFS  
2. Put raw logs under /data/antm2c/raw_10k/  
3. Run ETL:
   - CSV → ORC  
   - user/item/ui features  
   - join → ORC/Parquet  
4. Run modeling:
   - DNN → MMoE → PLE  
   - BERT/m3e embedding  
   - Whitening  
   - MMoE + Content Tower  
   - cold-start evaluation  
   - save artifacts  

---

## 14. Skills Demonstrated

- Spark + HDFS ETL
- Partitioned ORC/Parquet data engineering
- User / item / interaction feature design (rolling + cumulative)
- Avoiding label leakage
- Implementing MMoE / PLE models
- Text embedding integration (BERT, m3e, Whitening)
- Cold-start analysis
- Repeatable experiment artifact management

A full demonstration of real-world ad recommendation engineering.
