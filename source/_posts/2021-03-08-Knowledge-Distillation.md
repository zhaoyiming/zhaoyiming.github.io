---
title: Knowledge Distillation
date: 2021-03-08 14:13:43
tags:
- NeuralNetwork
- 模型压缩
categories: 
- NeuralNetwork
---

## Introduction

经过两天的知识总结与代码梳理，终于完成了知识蒸馏相关内容的PyTorch实现。主要难度点在于，由于需要进行大量的验证，因此必须要实现模型验证的自动化。关于知识蒸馏的代码部分并不多。

## Implement

### 损失函数

本篇使用的损失函数分别是出自Hinton的[Distilling the Knowledge in a Neural Network](https://arxiv.org/pdf/1503.02531.pdf)和[FITNETS: HINTS FORTHINDEEPNETS](https://arxiv.org/pdf/1412.6550.pdf)

### 代码

[https://github.com/zhaoyiming/knowledge-distillation](https://github.com/zhaoyiming/knowledge-distillation)