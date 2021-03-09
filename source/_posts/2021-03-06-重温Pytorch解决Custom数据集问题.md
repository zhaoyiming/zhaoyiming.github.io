---
title: 重温PyTorch解决Custom dataset问题
date: 2021-03-06 15:13:43
tags:
- NeuralNetwork
categories: 
- NeuralNetwork
---

### 主要问题

+ 如何解决自定义数据集构造和读取
+ 如何将整个数据集分成训练数据和测试数据

### PyTorch数据读取

PyTorch的数据加载模块，一共涉及到Dataset，Sampler，DataLoader三个类

三者的关系：

1. 设置Dataset，将数据data source包装成Dataset类，暴露提取接口。
2. 设置Sampler，决定采样方式。我们是能从Dataset中提取元素了，还是需要设置Sampler告诉程序提取Dataset的策略。
3. 将设置好的Dataset和Sampler传入DataLoader，同时可以设置shuffle，batch_size等参数。使用DataLoader对象可以快捷方便地在给定数据集上遍历。

总结来说，即Dataloader负责总的调度，命令Sampler定义遍历索引的方式，然后用索引去Dataset中提取元素。于是就实现了对给定数据集的遍历。

 在本文中，我们使用自定义的Dataset，不使用Sampler。

### 如何解决自定义数据集的构造和读取

关于这个问题，我们一般使用神经网络框架内置的数据集，非常方便地就可以获得构造的数据集，这里以PyTorch为例，可以使用`trainset = torchvision.datasets.CIFAR10(root='data/',train = True,download=True)`来获得torchvision内置的数据集，然后再通过DataLoader进行数据读取。

当我们使用自己的数据集时，我们的文件结构一般是按数据类别进行文件分类，所以我们需要读取所有的数据到集合中，然后再根据PyTorch提供的数据集构造函数来构造一个自定义Dataset类。

一个标准的Dataset类应该是：

```
class MyDataset(torch.utils.data.Dataset):#需要继承torch.utils.data.Dataset
    def __init__(self):
        # 初始化文件路径或文件名列表。
        # 初始化该类的一些基本参数。
    def __getitem__(self, index):
        ＃1。从文件中读取一个数据（例如，plt.imread）。
        ＃2。预处理数据（例如torchvision.Transform）。
        ＃3。返回单个数据对（例如图像和标签）。
    def __len__(self):
        # 返回数据集的总大小。
```

我们构造的Dataset：

```
class NotMinstDataset(Dataset):
    def __init__(self, file_dir):
        self.imgs = []
        labels = []
        # 从所有的文件夹中读取图片
        for root, sub_folders, files in os.walk(file_dir):
            for name in files:
                self.imgs.append(os.path.join(root, name))
                labels.append(root.split("\\")[1])
                
        # 将String类型的标签向量化
        le = preprocessing.LabelEncoder()
        # 变成array
        self.targets = le.fit_transform(labels)
        # 变为tensor
        self.targets = torch.as_tensor(self.targets)

    def __getitem__(self, index):
        img = self.imgs[index]
        label = self.targets[index]
        # 读取图片并转换为向量
        img = Image.open(img)
        img = self.img_transform(img)

        return img, label

    def __len__(self):
        return len(self.imgs)

    def img_transform(self, img):
        transform = transforms.Compose(
            [
                transforms.ToTensor(),
            ]
        )
        img = transform(img)
        return img
```

构造Dataset类后，我们直接使用Dataloader进行读取:

```
def data(train=True):
    dataset = NotMinstDataset("./notMNIST_small")
    train_size = int(0.8 * len(dataset))
    test_size = len(dataset) - train_size
    train_dataset, test_dataset = torch.utils.data.random_split(dataset, [train_size, test_size])
    if train:
        loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
    else:
        loader = DataLoader(test_dataset, batch_size=64, shuffle=True)
    return loader
```

[参考知乎链接](https://zhuanlan.zhihu.com/p/270028097)

###  如何将整个数据集分成训练数据和测试数据

思考，一般的数据集都会帮你划分好训练集和测试集，但是当你自己使用自定义数据集的时候，可能就需要自己进行分割。在哪一步对数据集进行划分是一个问题，以往我一般在构造Dataset类的准备函数中使用`sklearn.model_selection`的`train_test_split`函数进行数据集的划分，这样当我们使用dataloader进行数据集调用的时候，很可能遇到函数复用时实例化了两个分割的数据集。于是乎使用以下方法可以避免以上的问题。但是同时这个方法仍有问题，在每类样本不均衡的时候，很可能会造成抽样不均衡。

```
def data(train=True):
    dataset = NotMinstDataset("./notMNIST_small")
    train_size = int(0.8 * len(dataset))
    test_size = len(dataset) - train_size
    train_dataset, test_dataset = torch.utils.data.random_split(dataset, [train_size, test_size])
    if train:
        loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
    else:
        loader = DataLoader(test_dataset, batch_size=64, shuffle=True)
    return loader
```

[参考StackOverflow](https://stackoverflow.com/questions/50544730/how-do-i-split-a-custom-dataset-into-training-and-test-datasets)

### 完整代码

```
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import torchvision.transforms as transforms
from PIL import Image
from sklearn import preprocessing
import os
import time

device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
print("device:", device)


# Dataset
class NotMinstDataset(Dataset):
    def __init__(self, file_dir):
        self.imgs = []
        labels = []
        for root, sub_folders, files in os.walk(file_dir):
            for name in files:
                self.imgs.append(os.path.join(root, name))
                labels.append(root.split("\\")[1])
        le = preprocessing.LabelEncoder()
        self.targets = le.fit_transform(labels)
        self.targets = torch.as_tensor(self.targets)

    def __getitem__(self, index):
        img = self.imgs[index]
        label = self.targets[index]
        img = Image.open(img)
        img = self.img_transform(img)

        return img, label

    def __len__(self):
        return len(self.imgs)

    def img_transform(self, img):
        transform = transforms.Compose(
            [
                transforms.ToTensor(),
            ]
        )
        img = transform(img)
        return img


# getdata
def data(train=True):
    dataset = NotMinstDataset("./notMNIST_small")
    train_size = int(0.8 * len(dataset))
    test_size = len(dataset) - train_size
    train_dataset, test_dataset = torch.utils.data.random_split(dataset, [train_size, test_size])
    if train:
        loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
    else:
        loader = DataLoader(test_dataset, batch_size=64, shuffle=True)
    return loader


# Network
class Net(nn.Module):

    def __init__(self):
        super(Net, self).__init__()

        self.conv1 = nn.Conv2d(1, 6, 5, 1, padding=2)
        self.conv2 = nn.Conv2d(6, 16, 5, 1)

        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        x = F.max_pool2d(F.relu(self.conv2(x)), (2, 2))
        x = x.view(x.size(0), -1)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x



net = Net().to(device)
criterion = nn.CrossEntropyLoss().to(device)
optimizer = optim.Adam(net.parameters(), lr=0.001)

trainloader = data(True)
testloader = data(False)
t0 = time.time()
# training
for epoch in range(1):
    running_loss = 0.0
    for batch_idx, (input, label) in enumerate(trainloader):
        input, label = input.to(device), label.to(device).squeeze()
        optimizer.zero_grad()
        output = net(input)
        loss = criterion(output, label)
        loss.backward()
        optimizer.step()

        running_loss += loss.item()

        if batch_idx % 100 == 99:
            print('[%d, %5d] loss: %.10f' % (epoch + 1, batch_idx + 1, running_loss / 100))
            running_loss = 0.0
print('{} seconds'.format(time.time() - t0))

# 测试
correct = 0
total = 0
with torch.no_grad():
    for batch_idx, (input, label) in enumerate(testloader):
        input, label = input.to(device), label.to(device)
        outputs = net(input)
        _, predicted = torch.max(outputs.data, 1)
        total += label.size(0)
        correct += (predicted == label).sum().item()

print('Accuracy of the network on the test images: %.10f %%' % (100 * correct / total))
```



