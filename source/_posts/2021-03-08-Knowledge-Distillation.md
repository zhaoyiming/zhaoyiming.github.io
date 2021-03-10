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

经过两天的知识总结与代码梳理，终于完成了知识蒸馏相关内容的PyTorch实现。主要难度点在于：由于需要进行大量的精度验证，因此必须要实现模型精度验证的自动化，根据Hinton的论文，知识蒸馏相关的代码部分并不多。

## Implement

### Dataset

数据统一使用CIFAR10，后续可以进行CIFAR100数据集的实验。

使用torchvision自带的dataset类进行调用，然后进行数据增强、归一化和向量化。

### Model

我们的学生网络最开始使用传统的LeNet-5 CNN网络，在训练CIFAR10的过程中，我发现精度只有不到80%，分析其网络深度不够，不足以拟合CIFAR10数据集。于是加深网络到32、64、128个卷积核，最终可以实现86%的精度。

我们教师网络采用比较熟悉的DenseNet，版本为growth=12和Depth=100的DenseNet-BC。最终训练精度为94.7%，距离论文提到的95.5%仍有一定差距。

### Train

关于模型蒸馏实验的方法主要分为以下几个步骤：

1. 首先训练基本的CNN作为BaseLine。
2. 然后训练300个epoch的DenseNet作为Teacher model。
3. 最后使用训练好的DenseNet模型来进行推理得到一个样本后，将t-model和s-model的softmaxT结果进行KL散度拟合，加上s-model和True Label的交叉熵，形成此次推理的Loss Function。

### Loss Function

本篇使用的损失函数分别是出自Hinton的[Distilling the Knowledge in a Neural Network](https://arxiv.org/pdf/1503.02531.pdf)和[FITNETS: HINTS FORTHINDEEPNETS](https://arxiv.org/pdf/1412.6550.pdf)。

具体实现代码为：

```
def loss_fn_kd(outputs, labels, teacher_outputs, params):
    alpha = params.alpha
    T = params.temperature
    KD_loss = nn.KLDivLoss()(F.log_softmax(outputs/T, dim=1),
                             F.softmax(teacher_outputs/T, dim=1)) * (alpha * T * T) + \
              F.cross_entropy(outputs, labels) * (1. - alpha)

    return KD_loss
```



### Comments

+ 需要关注一下Relational Network Knowledge Distillation。
+ 根据在知乎看到的一些内容，相同的网络结构可以取得更好的效果，后续将进行尝试。

### Code

[https://github.com/zhaoyiming/knowledge-distillation](https://github.com/zhaoyiming/knowledge-distillation)

## Experiment

| TimeLine    | student net | student acc | teacher net | teacher acc | kd acc | loss function | epoch | Comments                         |
| ----------- | ----------- | ----------- | ----------- | ----------- | ------ | ------------- | ----- | -------------------------------- |
| 2021.3.8.11 | CNN         | 0.7511      | DenseNet    | 0.8194      | 0.7511 | fitnet        | 30    | Inital version                   |
| 2021.3.8.13 | CNN         | 0.8412      | DenseNet    | 0.9273      | 0.8600 | fitnet        | 30    | common version, overfit DenseNet |
| 2021.3.9.09 | CNN         | 0.8412      | DenseNet    | 0.9470      | 0.8667 | fitnet        | 30    | common DenseNet                  |
| 2021.3.9.10 | CNN         | 0.8412      | DenseNet    | 0.9470      | 0.8831 | softmaxT      | 100   | softmaxT loss function           |
| 2021.3.9.15 | CNN         | 0.8412      | DenseNet    | 0.9470      | 0.8805 | fitnet        | 100   | enlarge epoch number             |
| 2021.3.9.17 | CNN         | 0.8650      | DenseNet    | 0.9470      | 0.8841 | fitnet        | 100   | improve CNN acc                  |
| 2021.3.9.18 | CNN         | 0.8650      | DenseNet    | 0.9470      | 0.8754 | fitnet        | 100   | change T from 20 to 4            |