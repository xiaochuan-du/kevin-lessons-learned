---
title: "Demystify Feature Store"
slug: "demystify-feature-store"
date: 2020-03-25T00:00:00+11:00
draft: false
description: "A practical breakdown of feature-store capabilities, from lineage and point-in-time extraction to online serving."
tags: ["feature store","MLOps","machine learning"]
canonicalURL: "https://dxiaochuan.medium.com/demystify-feature-store-cb474709450e"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/demystify-feature-store-cb474709450e).*

In this day and age, more and more organizations would like to have a uniform tool to deal with their tons of features. For a simple POC product, feature governance and linage seem to be overkill, but when it comes to big, complex, and continuously evolving projects, they do prompt high quality of feature and execution efficiency in the following data science projects.

To summarize, why do we need a feature store?

- Standardizing the feature definitions. It should be an open, extensible, unified platform for feature storage.

- Promoting feature reusability. It facilitates discovery and feature reuse across machine learning projects.

- Providing point-in-time accuracy for feature extraction. It is important for Reproducibility.

- Feature quality checking. Data scientists / Machine learning engineers define the acceptance criteria for the feature values, and these rules will be used to check feature value during data ingestion.

- Remove train/ prediction data skewness. Data scientists do not need to worry about features for prediction and model training that come from different distributions or go across different transform process.

Sometimes, feature store means different things to different people or companies because they have their own unique requirements and want those requirements to become a part of the feature store implement.

In this article, I am going to divide the requirements for common a feature store into groups, so that you can pick some or all of them into your feature store implementation road map.

## Group 1: Vanilla feature store

- Feature lineage: versioning, description, owner, time, provenance. It helps to answer the data scientists’ following questions. Where did these features come from? How did they get calculated? who is the owner of these features? Are they still valid?

- The point-in-time feature extraction. This is used to prevent data leakage. When training models, some features are only available after a certain time, users should take that into consideration.

- High throughput data ingestion. Batch data loading performance is one of the critical metrics for the feature store.

- Low latency online feature processing. For online prediction, features should be prepared in a timely manner, this contributes to the end-to-end prediction performance.

- SDK. Generally speaking, data engineers, machine learning engineers, and data scientists are the main users of the feature store. They may have their own language preference to communicate with the feature store. Python and java might be the most popular choices for the SDK.

- Feature discovery functions. Help users to explore the feature store, this is for feature reusability.

## Group 2: Uniform Offline & online process

- Offline & online logic sharing. Although nearly all of us think it is a bad thing to have separate software to deal with offline and online data, we have to implement it in this way to meet online and offline own requirements. It is extremely helpful to share the same logic for the online and offline processes, and only turn into a different process when necessary.

- Online & offline data sync. Sometimes we use a data warehouse as offline data storage (for uber Hive, for feast Bigquery), and use a key-value database as online storage (for uber Cassandra, for feast Redis), do we need to synchronize data across these data storage? Some features might only be available in offline storage, how could we let online prediction use the feature. Lambda architecture might be one option, could we do it in a more elegant way?

## Group 2: Automatic feature generation

- DSL for feature generation. Some feature stores consider feature generation as one part of their function, they use `yaml` or other config files to define how features are generated. One good example is zipline.

- Schedule feature generation. Scheduling feature generation jobs can also be a part of the feature store, Airflow seems to be the default choice for implementation.

- Backfilling. When new feature generation rules are created, the historical data will be used to calculate the new feature.

## Group 3: Advanced requirements

- No storage lock-in. For online and offline data storage, it will be great if we can migrate from one to another in order to prevent vendor lock-in.

- Uniform batch and Streaming process. Use the same framework to deal with online or offline data process.

## Reference:

- Rethinking Feature Stores

- Michelangelo PyML: Introducing Uber’s Platform for Rapid Python ML Model Development

- Introducing Feast: an open source feature store for machine learning

- Meet Michelangelo: Uber’s Machine Learning Platform
