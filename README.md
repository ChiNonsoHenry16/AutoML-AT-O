# Evasion Attacks on Cost-Utility-Based Adversarial Training for Online AutoML in IoT Networks

Research code and experimental materials accompanying our study of cost-constrained adversarial training for online intrusion detection using IoTID20.

## Project Status

**Finalized code is in progress.** This repository is under active preparation and revision. Scripts, notebooks, documentation, and experimental outputs may change as implementation checks and evaluation corrections are completed.

Materials currently available should be treated as preliminary. A versioned release will identify the finalized implementation, dependencies, configurations, and corresponding results. Until then, the repository should not be considered a complete reproduction package for the manuscript.

## Overview

The study examines whether cost-constrained feature-space augmentation improves online learners' performance against a greedy evasion attack.

The evaluated learner families include:

- Hoeffding Tree (HT)
- Leveraging Bagging (LB)
- Streaming Random Patches (SRP)
- Hoeffding Adaptive Tree (HAT)
- Adaptive Random Forest (ARF)

The experiments use a processed subset of **IoTID20**. Cross-dataset generalization has not been established.

## Implemented Attack

The current generator:

1. Assigns a modification cost of `1.0` to every predictor column.
2. Considers numeric-valued features in dictionary order.
3. Tests positive and negative changes equal to 20% of each feature's training-set range.
4. Clips each candidate to that feature's training-set minimum and maximum.
5. Queries the learner and computes binary misclassification loss using the true label.
6. Accepts a modification only when it strictly increases that loss.

Each accepted modification consumes one cost unit. Each affordable numeric feature requires three prediction queries during generation, giving an upper bound of `3 × d` queries per record, where `d` is the number of predictors. There is no separate query cap.

### Threat-Model Limitations

The implementation does **not** enforce:

- An immutable-feature mask.
- Categorical or integer validity.
- Dependencies among traffic features.
- Explicit utility or attack-functionality preservation.
- Realizability of perturbed records as network traffic.

Numerically encoded categorical variables may therefore be considered perturbable.

The method is best described as **cost-constrained feature-space augmentation**, not fully utility-preserving traffic manipulation. The attack uses true labels and seeks any misclassification; it is not restricted to malicious-to-benign evasion.

## Adversarial Training

The current training budget is:

`cost_budget = 0.2 × number_of_predictors`

For each initialization record, the learner receives:

1. A clean learning update.
2. Adversarial-example generation against the updated model.
3. A second update using the returned example and the original label.

During streaming, the clean prediction is recorded before these updates.

If generation returns an unchanged example, the second update repeats the clean record. A matched two-clean-update baseline is needed to distinguish augmentation effects from the effects of additional learning updates.

## Evaluation Protocols

### Clean Prequential Evaluation

Each streamed record is predicted before it is used for learning. The supplied training routines report cumulative clean prequential accuracy.

Rolling-window accuracy is computed separately for drift analysis.

### Preliminary Attack-Budget Evaluation

The current budget sweep uses:

`b = [0.00, 0.05, 0.10, 0.20, 0.40, 0.80, 1.00]`

The available cost is calculated as:

`B = b × number_of_predictors`

The preliminary implementation freezes each final model and evaluates clean and perturbed versions of the same stream records already used for learning.
