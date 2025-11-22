# Abstract

This project implements an end-to-end, production-style CTR prediction pipeline using the **AntMRC dataset**, a large-scale multi-scene recommendation dataset from Ant Group. AntMRC is especially valuable because it includes **rich textual attributes**, **dense user–item interaction logs**, and **multi-scene labels**, enabling experiments that resemble real-world ad ranking and feed recommendation systems. Although the full dataset contains millions of samples, this work uses a focused **10k-sample slice**, which still provides enough variety to build and evaluate a complete industrial workflow.

The project follows the same progression used in modern recommendation engineering:  
1) distributed ETL with Spark + HDFS,  
2) user/item/interaction feature construction,  
3) classical DNN baselines,  
4) multi-task architectures (MMoE, PLE),  
5) semantic enhancement via **BERT embeddings + Whitening**, and  
6) cold-start recovery using **LLM-based text fallback**.

Each component is added one step at a time, and each step produces measurable gains. Starting from a simple dense DNN with AUC ≈ **0.74**, the system gradually evolves into a multi-source, multi-task fusion model that achieves **0.9739 validation AUC** and **0.9717 test AUC**.  
The overall improvement—**+0.23 absolute AUC**—is consistent with what industrial CTR systems typically obtain when combining structured signals with semantic text understanding and cold-start logic.

Even though this project only uses a compact 10k slice, the modeling pipeline is designed to scale directly to the full AntMRC dataset. With millions of samples, longer historical windows, and richer text, the same architecture should achieve even better stability and generalization, making it suitable for real-world ranking, cold-start handling, and multi-scene optimization.

---

# Performance Summary

| Stage | Description | Val AUC | Δ vs Previous |
|-------|-------------|---------|----------------|
| 1 | Baseline DNN (dense features only) | **0.74** | — |
| 2 | Add engineered user/item/UI features | **≈0.80** | **+0.06** |
| 3 | Multi-task MMoE | **0.83–0.85** | **+0.03–0.05** |
| 4 | PLE (shared + task experts) | **0.86–0.87** | **+0.02** |
| 5 | BERT embeddings + Whitening | **0.87–0.90** | **+0.03** |
| 6 | Fusion Model (MMoE + Content Tower + α-Gate) | **0.92–0.93** | **+0.02–0.03** |
| 7 | LLM-based cold-start fallback | **0.9739** | **+0.04** |
| — | **Final Test AUC** | **0.9717** | — |

**Total improvement:** **0.74 → 0.97 AUC**  
A full **+0.23 absolute** gain across the pipeline.

---

