---
title: "Continuous Testing for Machine Learning Systems"
slug: "continuous-testing-for-machine-learning-systems"
date: 2021-07-25T00:00:00+10:00
draft: false
description: "A layered approach to testing machine-learning systems across code, data, models, pipelines, monitoring, and experiments."
tags: ["MLOps","testing","machine learning","CI/CD"]
canonicalURL: "https://medium.com/data-science/continuous-testing-for-machine-learning-systems-a8519eede545"
---

*Originally published on [Medium](https://medium.com/data-science/continuous-testing-for-machine-learning-systems-a8519eede545).*

## Table of contents

- Testing in machine learning systems

- The scope of ML testing

- When do we perform the different types of tests?

- Conclusion

## Testing in machine learning systems

Testing in the software industry is a well-researched and established area. The good practices which have been learned from the countless number of the failed projects help us to release frequently and have fewer opportunities to see defects in production. Industry common practices like CI, test coverage, and TDD are well adopted and tailored for every single project.

However, when we try to borrow the SWE testing philosophy to machine learning areas, we have to solve some unique issues. In this post, we’ll cover some common problems in the testing of ML models (systems) and discuss potential solutions.

The ML system here stands for a system (pipeline) that generates prediction (insights) which can be consumed by users. It may include a few machine learning models. For example, an OCR model(system) could include one ML model to detect text region, one ML model to tell which current text region class is ( car plate vs road sign), and one model to recognize the text from a picture.

## The scope of ML testing

A model is composed of the code (algorithm, pre-process, post-process, etc), data, and infrastructure which facilitates the runtime.

Different types of testings cover the quality assurance for different components of the system.

Data testing: ensuring new data satisfies your assumptions. This testing is needed before we train a model and make predictions. Before training the model, the X and y (labels)

Pipeline testing: ensuring your pipeline is set up correctly. It’s like the integration tests in SWE. For the ML system, it may measure consistency (reproducibility) as well.

Model evaluation: evaluating how good your ML pipeline is. Depends on the metrics and dataset set you’re using, it could refer to different things.

- Evaluation on holdout/cross-validation dataset.

- Evaluation of deployed pipelines and ground truth(continuous evaluation ).

- Evaluation based on the feedback of system users (the business-related metrics, not a measurable ML proxy)

There are a bunch of techniques that can be applied in the process, like slice-based evaluation, MVP(a critical subset of data) groups/samples analysis, ablation study, user subgroup-based experiments (like Beta testing, and A/B testing).

Model testing: involves explicit checks for behaviors that we expect our model to follow. This type of testing is not for telling us the accuracy-related indicators, but for preventing us from behaving badly in production. Common test types include, but are not limited to:

- Invariance(Perturbation) Tests: perturbations of the input without affecting the model’s output.

- Directional Expectation Tests: to achieve we should have a predictable effect on the model output. For example, if the loss of blood within a surgery goes up, the blood for transfusion should go up as well.

- Benchmark regression: use predefined samples and accuracy gate to ensure a version of the model won’t introduce insane issues.

- Overfitting (Memorisation) test: try to overfit the model with a small fraction of the full dataset, and confirm if the model is able to memorize the data.

## When do we perform the different types of tests?

Some people may ask why we need to use holdout evaluation and continuous evaluation to measure almost the same metrics in CI and serving time.

One reason is that we can’t fully estimate model performance by seeing metrics on a predefined holdout dataset is that the data leakage sometimes is hard to detect than it looks. For example, some features which were expected to exist in the serving time turn out to have high latency to acquire, so our trained models can’t get used to seeing this feature always being empty.

Sometimes model evaluation could be very expensive, so a full cycle holdout evaluation is not feasible integrated into CI. In this case, we can define a subset regression evaluation within the CI, and only do the full evaluation before important milestones.

Model testing is not a one-off step, instead, it should be a continuously integrated process with the automation setup. Some of the test cases can be performed with the CI process, so each code commit will trigger them and we can guarantee the code/model quality in the main branch of a repo. Others can be conducted in the serving environment, so we won’t be blind to how well our system performs, and we can have relatively sufficient time to fix issues when we have them. Sometimes the ongoing tests within the serving environment can be seen as a part of the monitoring component, and we can integrate with alerting tool to close the loop.

## Conclusion

Machine learning systems are not straightforward to test, not only because it includes more components (code + data) to verify, but also it has a dynamic nature. Although we didn’t change anything our models can be stale because of the data change (data drift) or the nature of things change (concept drift) over time.

Automated testing is an essential component in CI / CD to verify the correctness of pipelines with a low footprint. While manual tests and human-in-the-loop verification are still crucial steps before we say a new ML pipeline is production-ready. After a pipeline has been released to production, continuous monitoring and evaluation can ensure we’re not flying blind. Finally, customer feedback-based tests (.i.e A/B tests) are able to tell us if the problem we are trying to solve is actually getting better.

There is no silver bullet in the ML system testing, continuously trying to cover edge cases would help us have fewer opportunities to make mistakes. Hope one day we can figure out a simple metric like code coverage to tell if our system is good enough.

## Reference:

- Effective testing for machine learning systems.

- Using AntiPatterns to avoid MLOps Mistakes

- CS 329S: Machine Learning Systems Design
