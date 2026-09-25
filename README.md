<div align="center">

# 🛡️ Evasion Attacks on Cost-Utility-Based Adversarial Training

### Online AutoML in IoT Networks

Research code and experimental materials for **cost-constrained adversarial training** using **IoTID20**.

![Status](https://img.shields.io/badge/Status-In_Preparation-orange?style=for-the-badge)
![Evaluation](https://img.shields.io/badge/Results-Preliminary-yellow?style=for-the-badge)
![Dataset](https://img.shields.io/badge/Dataset-IoTID20-0078D4?style=for-the-badge)
![Learning](https://img.shields.io/badge/Learning-Online-6F42C1?style=for-the-badge)

**[Overview](#-overview) · [Attack](#-implemented-attack) · [Training](#-adversarial-training) · [Evaluation](#-evaluation-protocols)**

</div>

---

## 🚧 Project Status

> [!IMPORTANT]
> **Finalized code is in progress.**
> This repository is under active preparation and revision. Current materials are preliminary and do **not** constitute a complete reproduction package for the manuscript.

Scripts, notebooks, documentation, and outputs may change as implementation checks and evaluation corrections are completed.

A **versioned release** will identify the finalized implementation, dependencies, configurations, and corresponding results.

## 🔎 Overview

Does cost-constrained feature-space augmentation improve online learners’ performance against a greedy evasion attack?

The study evaluates five learner families on a **processed subset of IoTID20**:

| Abbreviation | Learner |
|:---:|---|
| **HT** | Hoeffding Tree |
| **LB** | Leveraging Bagging |
| **SRP** | Streaming Random Patches |
| **HAT** | Hoeffding Adaptive Tree |
| **ARF** | Adaptive Random Forest |

**Scope:** Cross-dataset generalization has not been established.

## ⚔️ Implemented Attack

The current generator applies a greedy search over numeric-valued features.

| Component | Current implementation |
|---|---|
| Modification cost | `1.0` per predictor column |
| Feature order | Dictionary order |
| Candidate changes | ±20% of each feature’s training-set range |
| Bounds | Training-set minimum and maximum |
| Objective | Binary misclassification loss using the true label |
| Acceptance rule | Accept only a strict increase in loss |
| Budget consumption | One cost unit per accepted modification |

Each affordable numeric feature requires **three prediction queries**, giving an upper bound of **`3 × d` queries per record**, where `d` is the number of predictors. There is **no separate query cap**.

### ⚠️ Threat-Model Limitations

The implementation does **not** enforce:

- An immutable-feature mask
- Categorical or integer validity
- Dependencies among traffic features
- Explicit utility or attack-functionality preservation
- Realizability of perturbed records as network traffic

Numerically encoded categorical variables may therefore be considered perturbable.

> [!WARNING]
> The implemented method is best described as **cost-constrained feature-space augmentation**, not fully utility-preserving traffic manipulation.
>
> It uses **true labels** and seeks **any misclassification**—not exclusively malicious-to-benign evasion.

## 🔄 Adversarial Training

The current training budget is:

```python
cost_budget = 0.2 * number_of_predictors
```

For each initialization record, the learner performs:

```text
Clean learning update
        ↓
Generate an adversarial example against the updated model
        ↓
Learn from the returned example using the original label
```

During streaming, the **clean prediction is recorded before these updates**.

> [!NOTE]
> If generation returns an unchanged example, the second update repeats the clean record. A **matched two-clean-update baseline is needed** to separate augmentation effects from the effects of additional learning updates.

## 📊 Evaluation Protocols

### Clean Prequential Evaluation

Each streamed record is **predicted before it is used for learning**.

- Training routines report **cumulative clean prequential accuracy**.
- **Rolling-window accuracy** is computed separately for drift analysis.

### Preliminary Attack-Budget Evaluation

The current budget sweep uses:

```python
b = [0.00, 0.05, 0.10, 0.20, 0.40, 0.80, 1.00]
B = b * number_of_predictors
```

Each final model is frozen and evaluated on clean and perturbed versions of the **same stream records already used for learning**.

> [!CAUTION]
> This is a **preliminary, seen-record evaluation**, not a held-out robustness assessment. Its results should not be interpreted as evidence of generalization to unseen traffic.

---

<div align="center">

**Research in progress · Preliminary implementation · Evaluation revisions ongoing**

</div>
