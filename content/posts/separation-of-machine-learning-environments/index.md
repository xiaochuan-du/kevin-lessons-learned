---
title: "Separation of machine learning environments"
slug: "separation-of-machine-learning-environments"
date: 2021-10-17T00:00:00+11:00
draft: false
description: "How to separate machine-learning environments so experimentation stays flexible while production remains reliable."
tags: ["MLOps","machine learning","environments"]
canonicalURL: "https://dxiaochuan.medium.com/separation-of-machine-learning-environments-b5ed9f8e17ab"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/separation-of-machine-learning-environments-b5ed9f8e17ab).*

## Introduction

Let’s say we’ve identified a high-impact business problem at our company, then we trained a STOA model in our laptops. The performance on the test data set is amazing, and we can’t wait to tell the boss our solution is production-ready. It seems that the reasonable next step is to see happy faces😀 from customers.

But it turns out there are a bunch of stakeholders waiting at the gate of productionization. They are the Prod gatekeepers. They come from the Architect, SDE, security, Data governance, QA, team, etc.

There are many steps and processes we should address before go alive, and environment separation is one of them.

## Different roles, different worlds

Data scientists and ML engineers are following a highly iterative process to produce accurate and robust algorithms and models. As they progress through the ML lifecycle, they’ll find themselves iterating on a section until reaching a satisfactory level of performance, then proceeding forward to the next task (which may be circling back to an even earlier step). Moreover, a project isn’t complete after you ship the first version; you get feedback from real-world interactions and redefine the goals for the next iteration of deployment. [cited from [Organizing machine learning projects: project management guidelines.](https://www.jeremyjordan.me/ml-projects-guide/)]

At the same time, SDE, SA, and DevOps are following a different practice. They collect requirements from customers at first, then develop features in local dev and dev environment, run integration tests in NonProd or SIT (system integration test) environments, and then deploy the workload to production. There might be more environments get involved as well, for example, performance test environment, cost estimation environment, staging environment, alpha environment, etc.

More often than not, the ML service the Data scientists and ML engineers are working on is one component of the entire software system. So they need to figure out a way to adjust their development practice to fit into the organization’s broad development practices. There some a lot of common practices between these two groups of people, but DS / MLE might have to create some unique steps to make it work.

## Why is environment separation for ML special?

- Data is involved across the lifecycle. During the model building phase, DS / MLE need to have access to “real” data, the environment could be an exploration environment, but the data commonly is production data. For the dev/test/alpha environment, we need data to perform integration tests or performance tests, but the data could be synthetic because we might need high volume, or want to cover certain test cases. This is not the same as a normal software product. For software development, mocked data and stub services could be good enough to validate the system.

- Security. It will be great if DS / MLE can have access to production data all the time. This enables them to perform model monitoring, and online debugging. Only relying on logging and error tracing for ML products to fix issues is extremely hard, because silent failure ( 0 warning 0 errors) is the most common error type for ML projects. However, our data always come with sensitive and confidential attributes, thus we should act Least Privilege Access if possible. The permissions should vary from one environment to another.

- An ML automation environment. In some software development practices, we have a separate CI & CD to execute the CI & CD process and store artifacts for deployment. This environment would have an automated process to deploy the artifacts to other environments (dev/test/staging/prod/etc). In the ML world, ML models will be one type of these artifact. Moreover, we would count on automated ML pipelines to train models and store metadata for future analysis.

## A possible solution to separate environments

- Exploration environment: used for EDA and code deployment. It needs to read production data from the historical store and push code changes to repos.

- CICD environment: used for automation and building artifacts (PyPI libs, docker images, etc).

- ML CICD environment: used to perform whole dataset model training (long time model training), scheduled model retraining, and ML artifacts (models, encoding lookup, training data statistics, etc) saving. It could contain a service to manage ML metadata (.i.e MLflow, Metaflow, Metadata store).

- Runtime environments: These environments should follow the same setup as your software team. So you from network or IAM level, environments can go in pairs. For example, ml-dev can only talk with swe-dev environment.

In dev/staging and other non-production environments, we can use mocked data to validate the system. So these environments don’t need to have prod data access.

## Conclusion

No matter how do you handle the environment separation in your own ML projects, the KPI is cost-effectiveness, a developer-friendly development process, and sufficient access control. Some projects choose to combine exploration and ML CICD into a single environment to reduce the DevOps footprint, but this would increase the possibility that developers introduce errors in the ML artifact store by accident. There is no silver bullet in this topic, taking team, engineering practices, and time into consideration might help you to find the most suitable way for your team. See you in prod.

## Reference:

- Organizing machine learning projects: project management guidelines.

- Continuous Delivery for Machine Learning

- The path to production: how and where to segregate test environments

- Managing your machine learning lifecycle with MLflow and Amazon SageMaker

- Architect and build the full machine learning lifecycle with AWS: An end-to-end Amazon SageMaker demo

- Enterprise ML — Why getting your model to production takes longer than building it

- The Meaning of Production in the Data World

- Reaching MLE (machine learning enlightenment)

- What is a CI/CD Environment?
