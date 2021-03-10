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

## Experiment

| 实验时间    | student | student acc | teacher  | teacher acc | kd acc | loss function | epoch | 注释                             |
| ----------- | ------- | ----------- | -------- | ----------- | ------ | ------------- | ----- | -------------------------------- |
| 2021.3.8.11 | cnn     | 0.7511      | densenet | 0.8194      | 0.7511 | fitnet        | 30    | init version                     |
| 2021.3.8.13 | cnn     | 0.8412      | densenet | 0.9273      | 0.8600 | fitnet        | 30    | common version, overfit densenet |
| 2021.3.9.09 | cnn     | 0.8412      | densenet | 0.9470      | 0.8667 | fitnet        | 30    | common densenet                  |
| 2021.3.9.10 | cnn     | 0.8412      | densenet | 0.9470      | 0.8831 | softmaxT      | 100   | softmaxT loss function           |
| 2021.3.9.15 | cnn     | 0.8412      | densenet | 0.9470      | 0.8805 | fitnet        | 100   | enlarge epoch number             |
| 2021.3.9.17 | cnn     | 0.8650      | densenet | 0.9470      | 0.8841 | fitnet        | 100   | improve cnn acc                  |
| 2021.3.9.18 | cnn     | 0.8650      | densenet | 0.9470      | 0.8754 | fitnet        | 100   | change T from 20 to 4            |