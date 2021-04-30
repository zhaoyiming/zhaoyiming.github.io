# 在Nvidia TX2(arm)上搭建YOLOv4环境

## 总结

结合之前的经验，总共耗时两天。由于我的TX2系统为3.x版本，没有自带cuda，要安装非常麻烦，所以进行刷机。如果自带cuda请略过这一部。

主要问题集中在Jetpack系统的刷入以及Pytorch cuda加速上。

## 安装Jetpack

### 正常流程

按照网上的教程：

1. 注册Nvidia账号
2. 安装VMware虚拟机安装Ubuntu系统，开启桥接模式连接网络，安装sdkmanger。
3. 按照流程打开sdkmanger下载安装。
4. 中途提示安装，按照晚上的教程进行恢复模式。
5. 刷入后，需要先安装ssh工具，请自行查阅。
6. 安装ssh后，在host端继续安装cuda等工具。
7. 成功安装。

### 注意事项

+ 如果usb和tx2的corp无法显示在虚拟机中，请打开usb3.0兼容。
+ sdkmanger登录以及下载几乎都需要fq，请打开sdkmanger的内置proxy设置代理(找到宿主机host的ip，以及ssr开放的端口，使用http和https)。
+ 线材用盒子自带的usb即可。
+ 空间需要50g以上。

## 安装Pytorch以及Torchvision

### 环境配置

最好是用Miniforge或者其他Conda环境来构建虚拟环境。

### 开始安装

踩过许多坑，最终发现以下配合最为简洁。

[Pytorch下载地址](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-8-0-now-available/72048),最好使用该地址下Nvidia官方编译好的带cuda的Pytorch，否则不能开启cuda加速。

[Torchvision下载地址](https://github.com/KumaTea/pytorch-aarch64/releases)前者Nvidia官方只提供了本地编译torchvision，我实测编译失败且慢，不使用。(想要编译0.9.0版本的除外，见注意事项3)

## 注意事项

+ 需要注意的Jetpack自带的cuda10.2的话，可以使用的pytorch版本为1.6.0及以上
+ 如果遇到mish-cuda找不到，请`pip install git+https://github.com/thomasbrandon/mish-cuda/`
+ 实测Pytorch1.8.0+torchvision0.9.0以下的版本在使用torchvision的nms算子的时候会报错，无解。
+ 如果出现核心已转储的问题考虑安装`numpy==1.19.4`