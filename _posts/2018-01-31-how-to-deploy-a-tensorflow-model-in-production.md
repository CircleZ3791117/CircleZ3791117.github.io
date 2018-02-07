---
layout: post
title: 如何将Tensorflow模型部署到生产环境
date: 2018-01-31 10:13:00 -0000
tags: [匠心独运集]
image: tensorflow-serving.jpg
---

> 网上有关如何构造、训练和测试机器学习模型的教程数不胜数，那么如何在生产环境中部署我们训练和测试好的模型呢？
> 本文基于Tensorflow Serving构造了一个简单的webapp，并部署一个图像分类模型。

## Tensorflow Serving

**TensorFlow Serving 流程图：** ![Tensorflow workflow](/assets/img/2018_01_31/tensorflow-serving-arc.png)

**Deep learning：** ![Deep learning](/assets/img/2018_01_31/deep-learning-process.png)

Tensorflow Serving 特点：
- 可以说是谷歌为TensorFlow开发的”配套服务“
- 可以同时提供多个模型的服务（Great for A/B测试）
- 可以提供同一个模型多个不同版本的服务
- 基于C++实现


