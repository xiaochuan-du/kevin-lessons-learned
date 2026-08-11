---
title: "Hard Negatives, Sample Weights, and a Model Cascade I Didn't Build"
date: 2026-08-10T00:00:00+10:00
draft: false
math: true
description: "A LightGBM experiment changed the evidence I require before moving from sample weighting to specialist models and cascades."
tags: ["machine-learning", "lightgbm", "model-architecture", "loss-functions"]
---

I expected the hardest negative examples to need more attention. In my existing evaluation, giving them half as much weight improved both precision and recall on validation and held-out test data.

A specialist trained only on the positive class and those hard negatives did not provide enough evidence to justify a second model. That result mattered because I had started with an architecture question: should I split the problem across two models?

The experiment did not prove that cascades are a bad idea. I did not evaluate a complete routed system. It did give me a more useful default: before adding a model, test whether the current objective is giving each part of the data the right amount of influence.

I will describe the classes as $P$, $N_1$, and $N_2$. The names and numerical results are omitted, but the experimental pattern is real:

| Group | Label | What made it different |
|---|---:|---|
| $P$ | 1 | The positive class |
| $N_1$ | 0 | Relatively easy to separate from $P$ |
| $N_2$ | 0 | Harder to separate because it looked more like $P$ |

The deployed target was binary, so both negative groups had the same label. Their difficulty was not the same.

## Why a second model looked reasonable

A single model had to separate $P$ from two different-looking negative populations. $N_1$ appeared to be the broad, easy case. $N_2$ occupied the ambiguous part of the feature space.

My first design idea was a cascade:

1. route obvious $N_1$ cases to a negative decision;
2. send the remaining $P/N_2$ cases to a specialist.

This can be a sensible production pattern. A cheap first stage may reject easy cases before an expensive second stage. Candidate generation and ranking may have different objectives. Distinct populations may also justify distinct models when specialists actually outperform a shared model.

None of those general patterns proved that my problem needed a cascade. I still had to show that isolating $P$ and $N_2$ made the hard boundary easier to learn.

## What I tested

The baseline used one LightGBM binary classifier:

    y(P) = 1; y(N_1) = y(N_2) = 0

I compared four ideas:

| Experiment | Training population | Relative per-row training weight |
|---|---|---|
| Global baseline | $P+N_1+N_2$ | $1,1,1$ |
| Emphasise hard negatives | $P+N_1+N_2$ | $1,1,alpha$, with $alpha>1$ |
| Reduce hard-negative influence | $P+N_1+N_2$ | $1,1,0.5$ |
| Specialist | $P+N_2$ | Equal weights |

Increasing the influence of $N_2$ did not improve the result. With $w(N_2)=0.5$, both precision and recall improved on validation and held-out test data. The reported metrics for the $P$-versus-$N_2$ specialist were worse than those for the weighted global model.

This is a case report, not a reproducible benchmark. I am not publishing the subgroup sizes, metric values, split design, threshold-selection procedure, repeated-seed results, or enough protocol detail to establish a like-for-like global-versus-specialist comparison. I therefore do not use the result to claim that one architecture beat the other. Precision and recall also depend on the operating threshold; a robust follow-up should compare both models on the identical evaluation rows using the same validation-only threshold policy, then report precision-recall curves, PR-AUC, and performance at a fixed operational constraint.

Even with that limitation, the available evidence was enough for a narrower decision: I did not yet have a reason to accept the cost of a specialist model.

## What the weight changed

LightGBM's binary objective is log loss, and its API accepts a non-negative training weight for each row. A simplified per-example objective is:

    L(theta)
      = sum[i in P] loss_i
      + sum[i in N_1] loss_i
      + alpha * sum[i in N_2] loss_i

Setting $alpha=0.5$ does not tell the model that $N_2$ is unimportant. It makes an $N_2$ row contribute half as much as a $P$ or $N_1$ row with the same unweighted loss. The subgroup's total influence still depends on how many rows it contains. In boosted trees, the row weights affect the gradients and Hessians used to choose splits and leaf values.

It is tempting to describe the result as regularisation. The effect resembled regularisation because held-out performance improved when the difficult group exerted less influence. Technically, sample weighting changes the empirical objective—the training distribution or error costs—not the model-capacity penalty in the usual L1/L2 sense.

That distinction also matters for probabilities. A score learned under subgroup weights is optimised for the weighted objective. It should not automatically be treated as a calibrated probability under the unweighted production population. I would select the operating threshold, and if necessary recalibrate the score, on an unweighted validation set representative of serving traffic.

The [LightGBM parameters documentation](https://lightgbm.readthedocs.io/en/stable/Parameters.html) identifies the binary objective as log loss, and the [Python API](https://lightgbm.readthedocs.io/en/stable/pythonapi/lightgbm.LGBMClassifier.html) documents per-row training weights. Those mechanics are established. Why $0.5$ worked in this dataset is not.

## The mechanism is still a hypothesis

The first draft of my explanation was too neat: $N_1$ teaches coarse boundaries, later tree splits focus on $P$ versus $N_2$, and removing $N_1$ destroys that hierarchy. A LightGBM ensemble can represent rules resembling that story, but I did not inspect paths or run an ablation that proves it happened.

Several explanations fit the observations:

- The specialist used a smaller and differently balanced training population; sample size, class prevalence, or a tuning budget suited to the global model could explain the gap.
- $N_2$ may contain noisier labels or ambiguous ground truth.
- $P$ and $N_2$ may overlap because important separating features are missing.
- The full weight of $N_2$ may encourage patterns that fit its training examples but do not repeat out of sample.
- $N_1$ may provide useful shared structure, so the global learner estimates a better overall boundary.
- The weighting may better match the population or error trade-off represented by the evaluation metrics.

These explanations imply different next actions. A fair specialist comparison needs size-matched controls where possible, equivalent tuning budgets, and evaluation of both models on the identical $P/N_2$ slice. Label noise calls for an audit. Missing features call for new information, not another loss function. A population mismatch calls for correcting sampling or evaluation. Shared structure supports retaining the global model.

I would separate the explanations with a weight curve and slice-level evaluation:

    alpha in {0, 0.1, 0.25, 0.5, 0.75, 1, 2}

For every value, I would report overall performance and the same metrics separately for $P$ versus $N_1$ and $P$ versus $N_2$. I would select $alpha$ only on validation data, open the test set once, repeat training across seeds, and use an out-of-time test if the data has temporal structure.

The pattern would answer a useful question. If overall performance improves while $N_2$-specific performance declines, the result is an explicit trade-off. If $N_2$-specific performance also improves at a lower weight, noise or overfitting becomes more plausible. If every model performs poorly on $P$ versus $N_2$, the bottleneck is more likely to be features or labels than architecture.

## A specialist is not yet a cascade

Training on $P+N_2$ tested whether isolation helped a specialist learn that boundary. It did not test the router, coverage, latency, or error propagation of a complete cascade.

That distinction prevents an overclaim. My specialist result lowered the case for decomposition; it did not measure the end-to-end alternative.

Before I would deploy a cascade now, I would want evidence for all of the following:

| Question | Evidence I would require |
|---|---|
| Does the problem genuinely decompose? | Different objectives, feature sets, compute budgets, or a specialist with repeatable slice lift |
| Can cases be routed reliably? | Router recall and coverage measured on the population the specialist must see |
| Does the specialist add value? | Comparison with the global model on the identical routed subset |
| Does the whole system win? | End-to-end metrics including first-stage false negatives and threshold policy |
| Is the lift worth operating? | Latency, failure modes, monitoring, retraining, and debugging costs |

A cascade is most compelling when it buys something a single objective cannot cheaply provide. Heterogeneous negatives alone are weak evidence. A hard subgroup may still share features and statistical structure with the rest of the population, as my specialist result suggested.

My working rule is therefore to start with one model when the target is the same and the global model remains competitive on every important slice. I consider decomposition after a specialist demonstrates repeatable lift, or when stages have clearly different jobs or compute constraints.

## What I would do with a neural network

The direct neural-network equivalent is weighted binary cross-entropy:

    L = sum[i] a(group_i) * BCE(y_i, p_i)

The same experiment can begin with:

    a(P) = 1; a(N_1) = 1; a(N_2) = alpha

Focal loss answers a different question. It multiplies cross-entropy by a factor based on the model's confidence:

    FL(p_t) = -(1 - p_t)^gamma * log(p_t)

As $gamma$ increases, well-classified examples contribute less relative loss. Lin and colleagues introduced focal loss for dense object detection, where a vast number of easy background examples overwhelmed training. The [original paper](https://openaccess.thecvf.com/content_ICCV_2017/papers/Lin_Focal_Loss_for_ICCV_2017_paper.pdf) does not establish that every difficult subgroup should receive more influence.

This is important for $N_2$. If its examples are hard because they are informative and underrepresented, focal modulation may help. If they are hard because labels are noisy, classes overlap, or features are missing, repeatedly concentrating relative training attention on them may hurt.

Group weighting and focal modulation can be combined:

    L_i = a(group_i) * (1 - p_t,i)^gamma * BCE(y_i, p_i)

I would treat both $alpha$ and $gamma$ as hypotheses. Starting with $gamma=0$ reproduces ordinary group weighting and gives a clean baseline. Any move toward focal loss should earn its place through unweighted validation, subgroup metrics, calibration checks, and repeated runs.

## The lesson I am keeping

The most useful result was not that $0.5$ is a good hard-negative weight. That value belongs to one dataset, model, and evaluation setup.

The result changed the order in which I investigate this class of problem. I now test the global model, inspect subgroup metrics, vary subgroup influence, audit labels and features, and train a specialist as an ablation. Only then do I accept the engineering cost of routing and operating another model.

A difficult example is not automatically an important example. Difficulty describes how the current model experiences a row. Training weight encodes how strongly that row should shape the objective. My experiment worked better once I stopped treating those two quantities as the same.
