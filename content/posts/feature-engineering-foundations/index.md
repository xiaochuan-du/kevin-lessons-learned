---
title: "An elegant way to do feature engineering — feature engineering foundations"
slug: "feature-engineering-foundations"
date: 2021-03-09T00:00:00+11:00
draft: false
description: "Foundational patterns for maintainable, model-agnostic feature engineering across numerical, categorical, and multivariate data."
tags: ["feature engineering","machine learning","data science"]
canonicalURL: "https://dxiaochuan.medium.com/an-elegant-way-to-do-feature-engineering-feature-engineering-foundations-a023f2921ed5"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/an-elegant-way-to-do-feature-engineering-feature-engineering-foundations-a023f2921ed5).*

After repeating creating use case-specific and business logic coupled feature engineering code for a couple of years, I am thinking if it is possible to have a model agnostic (both algorithms and frameworks ), production-ready, data scientist friendly way to do feature engineering. I would summarize what we did for the use cases in financial service and healthcare, and propose some best practices (this is not accurate, we’ve seen good and better, but never seen best) to handle common problems in the lifecycle of ML models.

The purpose of feature engineering is to prepare features for ML models to process, so we want the output of feature engineering to

- improve the accuracy of the models

- be easy to maintain

- be computed with low latency

- ease the posthoc explanation

- contain metadata for auditing purpose: feature/data linage

- be easy to change for debugging and experimental tests

The discussion would be divided into the following sections:

- Feature engineering foundations

- Feature generation pipeline

- Operate feature engineering in an ML workflow

This article is about the 1st section. A bunch of information in this blog was extracted from the book “[Machine learning design patterns](https://learning.oreilly.com/library/view/machine-learning-design/9781098115777/)”.

## Content

- Naming convention

- Univariate data representation methods: numerical inputs, an array of numbers, categorical inputs, An array of Categorical Inputs

- Multi-variate data representation methods: Feature Cross, concatenating representations

## Naming convention

- input: the real-world data fed to the model

- feature: the transformed data that the model actually operates on

- feature engineering: The process of creating features to represent the input data. We can think of feature engineering as a way of selecting the data representation. This process can be learned/fixed / hybrid.

- feature extraction: The process of learning features to represent the input data.

- target: the value we need to predict

## Univariate data representation methods

### Numerical Inputs

Need scale due to quickly converge for LR/NN & standardize magnitudes across features.

- Linear scaling

- Min-max scaling

- Clipping (in conjunction with min-max scaling)

- Z-score normalization

- Winsorizing: min-max with outliers removal

2. Nonlinear transformation

When

- Data is skewed and neither uniformly distributed nor distributed like a bell curve

- The relation of features and target is not linear

- Help to improve the interpretability of the system

Methods:

- bins/quantiles: choose buckets is to do histogram equalization, where the bins of the histogram are chosen based on quantiles of the raw distribution. Or the bu

- logarithm, sigmoid and polynomial expansions (square, square root, cube, cube root, and so on)

- parametric transformation: .i.e Box-Cox transform

### An Array of numbers

- Representing the input array in terms of its bulk statistics: average, median, minimum, maximum, and so forth.

- Empirical distribution — i.e., by the 10th/20th/… percentile, and so on.

- For ordered signals (for example, in order of time or by size), representing the input array by the last three or some other fixed number of items. For arrays of length less than three, the feature is padded to a length of three with missing values. This method can be taken as order weighted bulk statistics.

### Categorical Inputs

- one-hot encoding

- label encoding

- Hashed Feature: to address incomplete vocabulary, model size due to cardinality, and cold start. The hash process needs to be deterministic (no random seeds or salt) and portable. The num_buckets should be a tunable parameter and a good rule of thumb is each bucket has 5 entries in it. If the collision of entries leads to the skewness of the counts in each bucket, we can aggerate features to use the new columns to do the work. As to the hash algorithm, a fingerprint function is better than cryptographic algorithms (MD5, SHA1) due to its deterministic and simple nature.

- Embeddings: the low-dimensional representation of those feature values with respect to the learning task. The size of embedding should be tunable, and the rule of thumb is to use the fourth root or 1.6 times the square root of the total number of unique categorical elements.

### An array of Categorical Inputs

- counting/relative-frequency

- bulk statistics: mode, the median, the 10th/20th/… percentile, etc.

- Time-weighted statistics (last three items in a series)

### Other Inputs

- Datetime: Cyclical features encoding

- Unstructre data (image / biosignals/ video, etc): Autoencoding, CNN, etc.

## Multi-variate data representation methods

### Feature Cross

Concatenating two or more categorical features in order to capture the interaction between them. This would make models simpler and have a better performance.

We should preprocess numeric features into categorical features to enable feature cross.

And to handle high cardinality, we can play it with hashed or embedding methods. L1 and L2 regulation are useful to encourage sparsity of features and reduce overfitting.

### Concatenating representations

An input to a model can be represented as a number or as a category, an image, or free-form text. But in real-world use cases, the information for a machine learning problem could come from different sources, and we can enable models to make the decision for the multi-input problem. To achieve this, we can concatenate the encoded features before input them into models.

Reference:

- Machine learning design patterns
