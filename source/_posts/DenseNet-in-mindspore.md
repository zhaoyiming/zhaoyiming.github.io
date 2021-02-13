---
title: DenseNet in mindspore
date: 2020-09-02 10:11:26
tags: Neural-Network
---

# DenseNet in MindSpore

&emsp;关于DenseNet实现的文章有很多，但是MindSpore作为华为研发的新生框架，此类网络结构实现的还不够，所以我在这里使用MindSpore复现了一下DenseNet网络。

## 实验配置

+ MindSpore 0.5 version
+ 华为Ascend
+ Python 3.7

## 注意事项

&emsp;需要注意的是由于MindSpore初期算子支持不够,所以不能使用AvgPool2d，所以我们在这里用MaxPool2d代替实现。并且由于0.5版本并不支持全局池化，所以我们使用ReduceMean算子代替全局池化.经过实际测试，这种替换对最后的结果影响并不大。

## 具体实现

### 数据集

&emsp;根据原论文中提供的数据集，我决定采用最简单的CIFAR10作为本次的实验数据。该数据集提供50000个训练样本和10000个测试样本，样本尺寸为32*32。样本被标注为10类。

### 实验前准备

&emsp;由于我们使用的华为的Ascend平台，所以要事先配置好obs桶，将文件和数据集上传到obs桶中，数据集放在`cifar10`文件夹内。

&emsp;在我们开始训练时,需要新建一个训练任务，训练文件选择对应的`train.py`，数据集位置选择`cifar10`文件夹。具体的操作步骤见华为提供的Ascend平台使用教程。

### 网络结构

&emsp;根据论文指导，我们采用growth_rate为12、Depth为100的DenseNet-BC网络结构以及不带Bottleneck和Compression操作的普通DenseNet网络结构。除了ImageNet数据集外，其他数据集均使用3层Dense Block结构，所以我们使用一个`Conv+[DB(Dense Block)+TL(Transition Layer)+DB+TL+DB]+BN+ReLU+Pool+Dense`的网络结构。

&emsp;其中Dense Block由一个`BN层+ReLU层+3x3的卷积层`组成，如果使用DenseBlock-BC网络结构，则还需要加一个`BN层+ReLU层(Bottleneck Layer)+1x1的卷积层`放在其3x3卷积层前，可以提高运算效率和最后的精度。

&emsp;其中Transition Layer则由一个`BN层+ReLu层+1x1卷积层+AvgPool2d层`组成.用来减少特征映射的尺寸，提高计算效率。

&emsp;在网络的最后，我们使用一个全局池化层，将8x8的特征映射池化为1x1，然后将其展开，用来代替传统的全连接层。

&emsp;样本初始尺寸为32x32，经过两层Transition Layer，变为16x16和8x8。最后经过全局池化层，变为1x1。

### 训练

&emsp;在模型的训练过程中，我们使用动态学习率，初始学习率为0.1，达到50%的epoch时衰减为十分之一，达到75%的epoch时再次衰减为此时的十分之一。

&emsp;我们根据论文使用0.0001的weight_decay和0.9的momentum。优化器采用随机梯度下降(SGD)。损失函数使用交叉熵损失函数。使用一种简单的随机正态分布将每层的卷积层进行初始化，并使用镜像、裁剪等数据增强操作。

&emsp;使用的训练epoch为300，最终耗时6小时34分钟，训练loss降至0.001。

### 实验结果

&emsp;测试样本为CIFAR10中的10000个不重复测试样本。最终我们在DenseNet网路结构上实现了94.68%的准确率，在DenseNet-BC网络结构上实现了94.9%的准确率，并未达到论文中的得到的准确率，我猜想是因为Ascend平台的差异，以及我们之前使用的一些替代的算子造成的准确率差异。实验结果如下图所示。

![loss](https://github.com/zhaoyiming/image/raw/master/densenet/loss.jpg)

### 实现代码

`https://github.com/zhaoyiming/`



