---
title: 在树莓派上运行yolo v4
date: 2021-02-13 22:13:43
tags: arm Neural-Network
categories: 

- 嵌入式

---

### 引言

需要完成在树莓派上调用yolo v4的任务，共计耗时两周，3/4的时间在尝试如何在树莓派上运行基于PyTorch的yolo v4，最终使用基于OpenCV-Python调用yolo v4可行，推理时间4s左右。

### 环境

为最大化树莓派效能，我们为树莓派刷入了Ubuntu 20.04 Server的基于arm64的系统。由于特殊原因，我们也使用了Ubuntu官网提供的Ubuntu 18.04 Desktop系统。

### 基于PyTorch的yolo v4

在Github中搜索yolov4，down下几个Top基于PyTorch实现的yolo v4库。首先发现大部分的实现需要依赖大量的第三方库，为了提升速度我们首先[替换系统中的apt源和使用国内的pip源](https://zhaoyiming.github.io/2021/02/13/Speed%20up%20your%20runtime%20environment%20Deployment%20of%20arm-based%20devices/)。然而，我发现这些基于PyTorch的yolov4在推理照片的时候无法正常运行，所有数据都是错误的。我怀疑是树莓派内存太小导致模型无法正常加载，所以尝试使用darknet实现yolov4。

### 基于Darknet的yolov4

从Github上down下来官方的darknet库，按照要求进行make编译，结果发现并不能正常编译，原来是Ubuntu20.10系统的g++版本太高，导致无法正常进行编译，所以我们使用Ubuntu18.04系统继续进行编译，编译可以正常通过。yolov4的推理耗时在58s左右，这个数据并不能令我们满意，而且由于要使用flask，我们只得又使用Ubuntu20.10系统寻求新的方法。

### 基于OpenCV-python的yolov4

我们知道OpenCV4已经内置了yolov4的调用，所以我们使用pip安装`opencv-python==4.4.0`，使用内置函数进行调用，可以正常使用，推理时间为4s左右，满意。





