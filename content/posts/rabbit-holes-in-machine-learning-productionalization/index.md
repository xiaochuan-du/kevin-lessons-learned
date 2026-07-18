---
title: "Rabbit holes in Machine Learning Productionlization"
slug: "rabbit-holes-in-machine-learning-productionalization"
date: 2020-09-20T00:00:00+10:00
draft: false
description: "An overview of the challenges, design patterns, and trade-offs involved in taking machine learning into production."
tags: ["MLOps","machine learning","production"]
canonicalURL: "https://dxiaochuan.medium.com/rabbit-holes-in-machine-learning-productionlization-d1fd61cb2094"
cover:
  image: "cover.jpeg"
  alt: "An overview of machine-learning production challenges"
  relative: true
---

*Originally published on [Medium](https://dxiaochuan.medium.com/rabbit-holes-in-machine-learning-productionlization-d1fd61cb2094).*

![An overview of machine-learning production challenges](cover.jpeg)

Machine learning (ML) is the study of computer algorithms that improve automatically through experience. It has been an important part of computer science for decades. Recently years, with the development of better algorithms, plenty of data, and more powerful computing power, the performance of ML models has been improved in a significant amount, thus ML started more and more contributing to business and industry use cases.

Different from research practice, ML in the industry requires a more standard processing pipeline, more robust experiment analysis, and more affordable deployment, leading to the creation of tools that help companies bring theoretical ML concepts into workable software/hardware.

In this article series, I would like to discuss the challenges in ML application and how could we leverage the existing framework and design patterns to tackle these problems.

Before diving deep into the contents, it worth mention that in many scenarios that general-purpose and fancy tools may be overkill for your use cases, hence always embracing the simple solutions can give you the agility and speed to deliver value for the business.

## Overview

When talking about ML application challenges these years, a list of Google papers appears again and again. As the forerunner of ML application, Google helps us to explore rabbit holes and its experience can be extremely beneficial to other companies. But some of the problems seem to be unique to the Google scale companies, thus if we simply copy their practices, it may cause an unnecessary burden for development, thus reducing effectiveness.

Papers discuss challenges in ML:

- Machine learning: the high-interest credit card of technical debt

- TFX: A TensorFlow-Based Production-Scale Machine Learning Platform

- Data Management Challenges in Production Machine Learning

- Rules of Machine Learning: Best Practices for ML Engineering

- Continuous Training for Production ML in the TensorFlow Extended (TFX) Platform

- DATA VALIDATION FOR MACHINE LEARNING

In the following series, I would like to summarize the common challenges we as data scientists or ML engineers may face in day-to-day work and try to propose solutions from ML tools, software design principles, and development practices perspective.

- ML code design patterns

- Development practice

- Continuous improvement

## Reference:

- Wikipedia: Machine learning
