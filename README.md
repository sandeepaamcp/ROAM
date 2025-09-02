# ROAM & TRACE: Relevance-Oriented Attacks and LRP-guided Defence for Decentralised Federated Learning

## Background (brief)
- **Decentralised FL (DFL) risk:** Without a central coordinator, peers **exchange models directly**. A malicious insider (or Sybil group) can **intercept, modify, and return** poisoned models, making **stealthy, targeted manipulation** and **privacy leakage** easier to sustain across rounds.
- **Privacy issue:** Carefully engineered backdoors can leak whether a **specific record is in a peer’s dataset** (membership inference) or skew the decision boundary for a **targeted class/property**—while keeping updates **statistically close to benign**.
- **ROAM idea:** Use **Layer-wise Relevance Propagation (LRP)** to locate a **small set of most critical neurons** for a target behaviour and **perturb only those**, creating **relevance-oriented (minimal-footprint) backdoors** that evade many robust detectors.
- **TRACE idea:** Before sharing a model, **identify the sensitive/unique neurons via LRP** and **apply controlled perturbations** to them. This **breaks backdoor-based membership inference** while preserving the model’s utility better than coarse global noise.

---

##  ROAM & TRACE Summary
1. **ROAM (Attack):**  
   - Compute **LRP** on a victim/peer model for a **target sample/property**.  
   - **Select k critical neurons** (per chosen layer) and **inject small, directed perturbations** → targeted misclassification or backdoor that enables **membership inference via boundary correction**.  
   - Share the **poisoned** model back to the peer; repeat for persistence/propagation.
2. **TRACE (Defence):**  
   - On the sender side, run **LRP** to **find sensitive/unique neurons** likely to be exploited.  
   - **Perturb only those neurons** with **small, randomized corrections** before sharing → **reduces attack success** (esp. membership inference via backdoor) with **limited utility loss**.
3. **Why it works:** both attack and defence operate at the **neuron-importance level**; TRACE targets the **same high-leverage region** ROAM relies on, neutralising its advantage while avoiding whole-model noising.

---

## Features
- **DFL threat model**: insider + optional **Sybil** adversaries; peer-to-peer model exchange.  
- **ROAM modules**: LRP scoring, neuron selection, targeted perturbation/backdoor injection, MI probing.  
- **TRACE modules**: LRP scoring, unique-neuron detection, selective perturbation, pre-share sanitisation.  
- **Utilities**: batching, logging, visualisation (optionally t-SNE of relevance patterns), metrics for attack/defence.  
- **Baselines** (optional): compare against global DP noise or classic robust aggregation on top of TRACE.

---

## 📂 Repository Layout

```text
ROAM/                                      # Root of the ROAM & TRACE project
├─ roam_trace/                             # Notebook-style experiments & demos for ROAM/TRACE
│  ├─ dfl-system/                          # DFL orchestration / system-level evaluations
│  │  ├─ DM4-voila-connectivity-with-FL-system.ipynb   # Connectivity + end-to-end FL/DFL demo
│  │  └─ DM5-test-with-sota-poisoning-attacks.ipynb    # Benchmarks vs SOTA poisoning/defences
│  └─ impl/
│     └─ roam/                             # Core attack/defence notebooks
│        ├─ dfl-impl.ipynb                 # DFL training loop integrating ROAM & TRACE
│        ├─ mia-backdoor.ipynb             # Membership inference via backdoor correction
│        ├─ mia-defence.ipynb              # TRACE defence to mitigate MI/backdoor
│        └─ roam-poisoning.ipynb           # Relevance-oriented poisoning (ROAM)
│
├─ src/                                    # Reusable library code (datasets, models, FL, XAI)
│  ├─ dataset/                             # Dataset loaders/wrappers/utilities
│  │  ├─ customDatasets/                   # Dataset-specific adapters
│  │  │  ├─ CelebA.py                      # CelebA adapter
│  │  │  ├─ CustomDataset.py               # Base dataset abstraction
│  │  │  ├─ NIDD_5G.py                     # 5G-NIDD adapter
│  │  │  └─ NSL_KDD.py                     # NSL-KDD adapter
│  │  ├─ dataLoaderFactory.py              # Factory to build PyTorch DataLoaders
│  │  ├─ datasetHandler.py                 # Splits/transforms/normalisation
│  │  ├─ datasetStrategy.py                # Dataset selection/partitioning strategies
│  │  └─ poisoning.py                      # Data-poisoning helpers (if used)
│  │
│  ├─ FLProcess/                           # Federated-learning orchestration
│  │  ├─ CustomAggregate.py                # Aggregation entry + hooks
│  │  ├─ CustomFedAvg.py                   # FedAvg implementation
│  │  ├─ CustomFedAvgBase.py               # Shared FedAvg helpers
│  │  ├─ CustomFLAME.py                    # FLAME robust baseline (optional)
│  │  ├─ CustomKrum.py                     # Krum robust baseline
│  │  ├─ DPFlowerClient.py                 # Flower client with DP (optional)
│  │  ├─ FlowerClient.py                   # Base Flower client abstraction
│  │  └─ FLUtil.py                         # Sampling, metrics, common FL utilities
│  │
│  ├─ NN/                                  # Models and training utilities
│  │  ├─ MdlTraining.py                    # Train/eval loops
│  │  ├─ NNConfig.py                       # Hyperparameters & config
│  │  ├─ NNUtil.py                         # Save/load, seeding, helpers
│  │  └─ ResNet.py                         # Example model definitions
│  │
│  ├─ poisonDetection/                     # XAI/analysis used by ROAM/TRACE
│  │  ├─ clientAnalysis/                   # Strategy scaffolds & baselines
│  │  │  ├─ strategyFnDebugging.py         # Debug helpers
│  │  │  ├─ strategyFnGeneralAlg.py        # General poisoning baseline
│  │  │  └─ strategyFnRandomPoison.py      # Random poison baseline
│  │  ├─ clusteringHDBSCAN.py              # Density clustering of relevance/SHAP vectors
│  │  ├─ tsneVisualisation.py              # t-SNE plotting helpers
│  │  └─ xaiMetrics.py                     # LRP/SHAP computation utilities
│  │
│  └─ util/                                # Project-wide constants/helpers
│     └─ constants.py                      # Paths, seeds, defaults
│
├─ main.py                                 # Optional CLI entry for attack/defence pipelines
├─ .gitignore
└─ README.md

