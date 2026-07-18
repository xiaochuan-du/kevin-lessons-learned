---
title: "Exploring services for hosting your ML models on AWS"
slug: "hosting-ml-models-on-aws"
date: 2023-06-14T00:00:00+10:00
draft: false
description: "A comparison of AWS options for hosting machine-learning models, from managed APIs to custom infrastructure."
tags: ["AWS","MLOps","machine learning","deployment"]
canonicalURL: "https://dxiaochuan.medium.com/exploring-services-for-hosting-your-ml-models-on-aws-9dda4f56a3b0"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/exploring-services-for-hosting-your-ml-models-on-aws-9dda4f56a3b0).*

## Table of contents

- Introduction

- SageMaker hosting VS homegrown hosting

- What are the appropriate use cases for each?

- Conclusion

## Introduction

Speaking of hosting ML models in AWS, one of the most painful points is that you have to choose from many options. Each option has pros and cons; no universal solution fits all use cases[1]. The latest innovations from AWS make the selection even more complicated. For example, the powerful [MME models](https://docs.aws.amazon.com/sagemaker/latest/dg/multi-model-endpoints.html) may provide a more cost-efficient deployment opinion for medium size neural network models, but it could make the deployment process over-complicated for a small-size engineering team. In the blog post, I will capture some facts and arguments you can consider when selecting an ML model host service.

## SageMaker hosting VS homegrown hosting

We are focusing on real-time and batch inference for simplicity; Asynchronous and serverless inference won’t be covered.

[Amazon SageMaker](https://aws.amazon.com/sagemaker/) is a fully managed machine learning (ML) service. It provides a range of features that can be used in real-time inference. We can call this opinion SageMaker hosting.

ML is part of the furniture for many companies, and one of the common practices is not treating ML workflow as special. Existing CICD tools and deployment services could also be used for ML models but require some adaption functions and unique feature development and maintenance. Let’s name this method homegrown hosting.

### Real-time inference

Some possible hosting services are ECS, EKS, and EC2.

Performance: In terms of network, security, and compute, both opinions are similar because, under the hood, they are empowered by the same AWS infrastructure. I benchmarked single node performance following [this blog](https://aws.amazon.com/blogs/machine-learning/best-practices-for-load-testing-amazon-sagemaker-real-time-inference-endpoints/) for a `distilbert-base-uncased`HuggingFace model using m5.large EC2 instances and ml.m5.large SageMaker endpoint, the QPS are 22 in the two setups. Then I re-ran the test for [an Xgboost model](https://github.com/aws/amazon-sagemaker-examples/blob/main/advanced_functionality/xgboost_bring_your_own_model/xgboost_bring_your_own_model.ipynb), and the QPS are ~ 430 for both of them. I don’t see any performance difference for the same infrastructure setting. The latency is also very similar (~3 s for the 1st and 200 ms for the 2nd test). It’s worth mentioning that the performance varies when you host different ML models, and communication overhead is significant between upstream services and the hosting environment.

Development cost: SageMaker hosting provides a wide range of features like [shadow variants](https://docs.aws.amazon.com/sagemaker/latest/dg/model-shadow-deployment.html), [monitoring](https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor.html), [A/B Testing](https://aws.amazon.com/blogs/machine-learning/a-b-testing-ml-models-in-production-using-amazon-sagemaker/), and native integration with other AWS services like [Amazon CloudWatch](http://aws.amazon.com/cloudwatch). However, not all these features are mandatory for an ML use case. Moreover, some alternatives could provide more rich features for a single function and align better with existing software in production.

Deployment cost: you can use the existing CICD workflow to deploy both opinions. And the cost difference is minimal. SageMaker hosting provides more built-in features like [Blue/Green Deployments](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails-blue-green.html), [Auto-Rollback](https://docs.aws.amazon.com/sagemaker/latest/dg/deployment-guardrails-configuration.html), and [Shadow tests](https://docs.aws.amazon.com/sagemaker/latest/dg/shadow-tests.html). However, a full-fledged deployment tool may provide similar features as well.

Running cost: the difference is noticeable. SageMaker hosting ([pricing](https://aws.amazon.com/sagemaker/pricing/)) is generally 20% more expensive than home-grown hosting ([pricing](https://aws.amazon.com/ec2/pricing/on-demand/)) for the same resources. You can use [savings Plans](https://aws.amazon.com/savingsplans/) to make both options more affordable.

Scalability: both opinions support enormous scaling, but there is an important fact that may influence your decision-making. SageMaker hosting is based on EC2 instances. Thus scaling out may be slower than container-based scaling like ECS / EKS. For a warm cluster, the difference could be 5 mins vs 20 seconds. Homegrown hosting could fit better if you seek a swift cluster autoscaling solution.

Big models: serving big models like LLM and generative CV models like stable diffusion are more common nowadays. This creates a new type of challenge in model hosting. For example, an ML model may require multi-GPU and multi instances to run inference like [Falcon-40B](https://aws.amazon.com/blogs/machine-learning/deploy-falcon-40b-with-large-model-inference-dlcs-on-amazon-sagemaker/). Using SageMaker hosts may make life earlier because it has more pre-built environments like [DJL](https://aws.amazon.com/blogs/opensource/how-netflix-uses-deep-java-library-djl-for-distributed-deep-learning-inference-in-real-time/), [Hugging Face LLM Inference containers](https://aws.amazon.com/blogs/machine-learning/announcing-the-launch-of-new-hugging-face-llm-inference-containers-on-amazon-sagemaker/), [Triton Inference Server](https://docs.aws.amazon.com/sagemaker/latest/dg/triton.html). For homegrown solutions, you must keep up with the latest upgrades (like drives and acceleration software) and solve library conflicts yourself.

### Batch inference

Due to its simplicity, many companies start their ML productionization journey by adopting batch inference. Some options are EC2, AWS Batch, EMR, Glue, and SageMaker Batch transform. Most of the discussions we had for the real-time inference apply to batch inference, we’d only highlight some unique parts.

Performance: SageMaker hosting provides a consistent inference for real-time and batch inference. This simplifies the usage but impacts the batch transform performance. [Using EC2 with Ray](https://www.anyscale.com/blog/offline-batch-inference-comparing-ray-apache-spark-and-sagemaker) can perform better than Spark-based solutions (Glue & EMR) and SageMaker hosting.

Deployment cost: deploying batch inference requires orchestration with upstream and downstream pipelines. Both opinions can work with the Step function, Airflow, and 3rd party orchestration tools. SageMaker hosting supports SageMaker Pipeline, which has better visualization if integrated with SageMaker Studio.

## What are the appropriate use cases for each?

SageMaker hosting:

- You don’t want to distract from your business-related feature development.

- New to ML productionalization (< 10 ML models in production)

- You don’t have a dedicated ML engineering team to work on foundation ML services like shadow testing, monitoring, etc.

- Host big models

homegrown hosting:

- You want to maximize reusing foundation services like clusters (ECS, EKS), CICD, observability, tracing, and other tooling systems.

- You have a mature ML engineering team; the 20% running cost-saving is greater than homegrown tools development & maintenance costs.

- Require low latency autoscaling (< 1 min)

- You have a restricted SLA for batch inference.

- You have special requirements like over 60 seconds latency for a real-time endpoint ( using async workflow would make better sense)

## Conclusion

In the blog, we discussed factors you could consider to select a service to host your ML models. If you modulize your infrastructure and ML code from the beginning, you can convert this selection from a [one-way door to a two-way decision](https://shit.management/one-way-and-two-way-door-decisions/). It is not rare to start with SageMaker hosting and then migrate to homegrown hosting if there are clear cost, performance, and flexibility benefits. The more important thing is to avoid spending too much energy building wheels, which slow down your business value delivery.

## Reference

[1] Model hosting patterns in Amazon SageMaker, Part 1: Common design patterns for building ML applications on Amazon SageMaker | AWS Machine Learning Blog: [https://aws.amazon.com/blogs/machine-learning/model-hosting-patterns-in-amazon-sagemaker-part-1-common-design-patterns-for-building-ml-applications-on-amazon-sagemaker/](https://aws.amazon.com/blogs/machine-learning/model-hosting-patterns-in-amazon-sagemaker-part-1-common-design-patterns-for-building-ml-applications-on-amazon-sagemaker/)

[2] Model hosting patterns in Amazon SageMaker, Part 6: Best practices in testing and updating models on SageMaker | AWS Machine Learning Blog: [https://aws.amazon.com/blogs/machine-learning/part-6-model-hosting-patterns-in-amazon-sagemaker-best-practices-in-testing-and-updating-models-on-sagemaker/](https://aws.amazon.com/blogs/machine-learning/part-6-model-hosting-patterns-in-amazon-sagemaker-best-practices-in-testing-and-updating-models-on-sagemaker/)

[3] How Forethought saves over 66% in costs for generative AI models using Amazon SageMaker | AWS Machine Learning Blog: [https://aws.amazon.com/blogs/machine-learning/how-forethought-saves-over-66-in-costs-for-generative-ai-models-using-amazon-sagemaker/](https://aws.amazon.com/blogs/machine-learning/how-forethought-saves-over-66-in-costs-for-generative-ai-models-using-amazon-sagemaker/)

[4] Best practices for load testing Amazon SageMaker real-time inference endpoints | AWS Machine Learning Blog: [https://aws.amazon.com/blogs/machine-learning/best-practices-for-load-testing-amazon-sagemaker-real-time-inference-endpoints/](https://aws.amazon.com/blogs/machine-learning/best-practices-for-load-testing-amazon-sagemaker-real-time-inference-endpoints/)

[5] Choosing the right GPU for deep learning on AWS | by Shashank Prasanna | Towards Data Science: [https://medium.com/towards-data-science/choosing-the-right-gpu-for-deep-learning-on-aws-d69c157d8c86](https://medium.com/towards-data-science/choosing-the-right-gpu-for-deep-learning-on-aws-d69c157d8c86)
