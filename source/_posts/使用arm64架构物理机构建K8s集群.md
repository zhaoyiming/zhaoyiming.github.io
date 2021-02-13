---
title: 使用arm64架构物理机构建K8s(Kubernetes)集群
date: 2020-12-02 20:11:26
tags: arm 环境部署
categories: 
- 嵌入式
---

在Windows下安装K8s集群并不算很难，在arm64系统下安装会出现很多问题，如果有代理，建议全程代理。

## 准备环境

本文使用的集群由一个Nvidia Tx2当作master节点，系统为JetPack 3.3(Ubuntu 18.04)。

三个树莓派刷成ubuntu系统作为node节点，系统具体为arm64的18.04.5的[Ubuntu Server](https://ubuntu.com/download/raspberry-pi)。所有物理机处于同一个局域网络环境下。

## 安装步骤：

### master篇

#### 关闭swap

如果不关闭swap，K8s会出现各种报错。

```
# 手动关闭swap
    swapoff -a
# 关闭swap开机自启，注释掉swap字样的行。不过该行可能不存在，就需要每次手动关闭swap。
    nano /etc/fstab
```

#### 安装kubeadm kubectl kubelet docker-ce

由于我个人无法找到正确的kubernetes源来安装三个kubeadm组件，所以这里我推荐大家去清华源自的[kubernetes仓库](https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt/pool/)中寻找离线包进行下载安装。我们需要安装的是[kubectl_1.15.2](https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt/pool/kubectl_1.15.2-00_arm64_590728548106979631ff013af47054895222f1ab25674aed5a6e6c11460648d1.deb)、[kubelet_1.15.2](https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt/pool/kubelet_1.15.2-00_arm64_b3e642cecc9f8b162da843d2b125dae20415840122fa7bee396fe5eaea8cac81.deb)、[kubeadm_1.15.2](https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt/pool/kubeadm_1.15.2-00_arm64_6808e06f1d0e6a24b04b22b701cdbeb12fb0a19c51c5948a2f7ac29c6fdefce7.deb)和[docker-ce_18.06.2](https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/dists/xenial/pool/stable/arm64/docker-ce_18.06.2~ce~3-0~ubuntu_arm64.deb)这四个包。下载后使用以下命令安装：

```
    sudo chmod +x xxx.deb
    sudo dpkg -i xxx.deb
```



但是在安装kubelet的过程中可能需要安装如socat之类的依赖，如果添加的源中无法找到合适的依赖，可以使用华为源，以下命令直接替换：

```
	wget -O /etc/apt/sources.list https://repo.huaweicloud.com/repository/conf/Ubuntu-Ports-bionic.list
    apt-get update
```

或者清华源，需要执行添加到/etc/apt/sources.list文件中：

```
# 理论为16.04的源，但是我没能成功找到18.04的arm64的清华源地址。
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial main multiverse restricted universe
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security main multiverse restricted universe
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates main multiverse restricted universe
    deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-backports main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates main multiverse restricted universe
    deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-backports main multiverse restricted universe
```



需要注意的是，在安装过程中可能会需要一些apt无法自己下载的依赖，我们可以在上述kubernetes仓库中下载解决，我需要的是[kubernetes-cni_0.7.5](https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt/pool/kubernetes-cni_0.7.5-00_arm64_16f686a176ee62fc4f960fd4b272e5e26c73fcced8bd1f8ce9a68a54b2b07e28.deb)和[cri-tools_1.13.0](https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt/pool/cri-tools_1.13.0-00_arm64_551fb3bc0ac49efe6a2fd37e0c3c081290c661353055d5c933f41d440ca0c7bd.deb)这两个包。

创建 `/etc/docker/daemon.json` 使 docker 的 cgroupdriver 为 systemd ，并重启服务。

```
# daemon.json内容
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2"
    }
# 重启docker服务
    sudo systemctl daemon-reload
    sudo systemctl restart docker
```



#### Pull Docker Images

由于K8s默认使用的`.gcr.io`官方镜像被墙掉，所以我们需要找到一些替代方法来下载。我们从**[mirrorgooglecontainers](https://hub.docker.com/u/mirrorgooglecontainers/)**仓库来手动下载镜像，并更改tag来达到离线下载的目的。

```
# Pull mirrorgooglecontainers的images
    docker pull mirrorgooglecontainers/kube-apiserver-arm64:v1.15.2
    docker pull mirrorgooglecontainers/kube-controller-manager-arm64:v1.15.2
    docker pull mirrorgooglecontainers/kube-scheduler-arm64:v1.15.2
    docker pull mirrorgooglecontainers/kube-proxy-arm64:v1.15.2
    docker pull mirrorgooglecontainers/pause-arm64:3.1
    docker pull mirrorgooglecontainers/etcd-arm64:3.3.10
    docker pull coredns/coredns:coredns-arm64

# 将mirrorgooglecontainers 的镜像打上k8s.gcr.io 的 tag
    docker tag mirrorgooglecontainers/kube-apiserver-arm64:v1.15.2 k8s.gcr.io/kube-apiserver:v1.15.2
    docker tag mirrorgooglecontainers/kube-controller-manager-arm64:v1.15.2 k8s.gcr.io/kube-controller-manager:v1.15.2
    docker tag mirrorgooglecontainers/kube-scheduler-arm64:v1.15.2 k8s.gcr.io/kube-scheduler:v1.15.2
    docker tag mirrorgooglecontainers/kube-proxy-arm64:v1.15.2 k8s.gcr.io/kube-proxy:v1.15.2
    docker tag mirrorgooglecontainers/pause-arm64:3.1 k8s.gcr.io/pause:3.1
    docker tag mirrorgooglecontainers/etcd-arm64:3.3.10 k8s.gcr.io/etcd:3.3.10
    docker tag coredns/coredns:coredns-arm64 k8s.gcr.io/coredns:1.3.1

# 删除 mirrorgooglecontainers 的镜像，空间够大不建议，否则出错要重新pull
    docker rmi mirrorgooglecontainers/kube-apiserver-arm64:v1.15.2
    docker rmi mirrorgooglecontainers/kube-controller-manager-arm64:v1.15.2
    docker rmi mirrorgooglecontainers/kube-scheduler-arm64:v1.15.2
    docker rmi mirrorgooglecontainers/kube-proxy-arm64:v1.15.2
    docker rmi mirrorgooglecontainers/pause-arm64:3.1
    docker rmi mirrorgooglecontainers/etcd-arm64:3.3.10
    docker rmi coredns/coredns:coredns-arm64
```

#### 初始化kubernetes

```
# 初始化命令 --apiserver-advertise-address选用。--pod-network-cidr根据后续安装flannel或calico而定，若安装calico请更改为192.168.0.0/16，本文使用flannel。--ignore-preflight-errors Swap选用。
	kubeadm init --apiserver-advertise-address=<本机ip> --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors Swap 
```

安装成功会出现successfully字样，并提示你进行以下操作：

```
# 替换$HOME为你的用户地址执行即可
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

并记得要记住`kubeadm join 192.168.31.73:6443 --token 9ovifx.j5g38h7v8jdieb9z \
    --discovery-token-ca-cert-hash sha256:6bf8f538c06613f52c728c80e3eb1c6a4c873d0c6f506aab2b8c785ea228fc36 `类似信息，以供后续node加入K8s网络，此处注意密钥有效期仅为一天，过期请执行Google解决。

执行以下操作确定初始化成功。

```
# 查看集群信息
    $ kubectl cluster-info
    Kubernetes master is running at https://192.168.31.73:6443
    KubeDNS is running at https://192.168.31.73:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
# 查看所有node
    $ kubectl get nodes
    NAME                STATUS     ROLES    AGE     VERSION
    ming-master   NotReady   master   3m50s   v1.15.2
# 查看所有Pod
    $ kubectl get pods --all-namespaces
```

#### 安装Flannel网络插件

我们需要选择Flannel或者Calico网络插件来驱动K8s。

```
# 此处可能会遇到无法解析raw.githubusercontent.com的问题，请自行搜索修改host结局。
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 此处应该可以看到所有 pods为Running状态
    kubectl get pods --all-namespaces -o wide
```

所有配置结束，但是一般配置下来都不会这么顺利，所以请使用`kubectl describe pod xxx（pod的id）-n kube-system `自行分析、谷歌来进行debug。 

#### 注意事项

要保证足够的剩余硬盘空间，否则可能会导致proxy pod创建失败。

### Node篇

#### 安装步骤

在树莓派机器上重复Master篇中的**关闭swap、安装kubeadm kubectl kubelet docker-ce以及Pull Docker Images**章节。然后根据前文的密钥加入K8s网络:

```
kubeadm join 192.168.31.73:6443 --token 9ovifx.j5g38h7v8jdieb9z \
    --discovery-token-ca-cert-hash sha256:6bf8f538c06613f52c728c80e3eb1c6a4c873d0c6f506aab2b8c785ea228fc36 
```

如果执行成功，请在master节点上执行`kubectl get nodes`查看成功加入的节点。

#### 注意事项

有些成功加入的树莓派节点会非常卡顿，不知道我这几台是不是特殊情况。

### 参考文章

+ https://sbilly.github.io/post/howto-setup-kubernetes-cluster-on-armbian-linux-based-on-phicomm-n1-arm64-sbc/
+ https://www.edureka.co/blog/install-kubernetes-on-ubuntu

