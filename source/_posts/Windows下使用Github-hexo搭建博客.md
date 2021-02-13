---
title: Windows下使用Github+hexo搭建博客
date: 2018-02-01 22:53:16
tags: 博客 hexo
categories: 
- 学外拓展
---

# Windows下在Github上搭建hexo博客

## 关于第一篇技术博客，介绍如何在Windows环境下通过Github pages搭建hexo，包括如何从WordPress转移博客到hexo上。

## hexo搭建

### 搭建博客前小小的工作

#### 1.Github注册

在此不再赘述，在[Github官网](https://github.com/)注册即可。

#### 2.安装node.js

[node.js官网](https://nodejs.org/en/)下载安装。安装完毕后打开命令行界面输入`node -v`如图。
![node.js成功安装截图][1]

#### 3.安装git

[Git官网](https://git-scm.com/)安装提示默认选项安装即可。安装完毕后打开命令行输入`git --version`成功安装如图所示。
![git成功安装截图][2]
git安装成功后需要添加邮箱和用户名，将代码中双引号中的部分替换成你的用户名，范例邮箱替换成你的邮箱。

```
$ git config --global user.name "John Doe" 
$ git config --global user.email johndoe@example.com
```

添加完后我们需要添加SSH Key到你的Github账户，添加SSH Key是为了让你的Github账户可以绑定你的机器。
打开Git Bash执行代码：

```
ssh-keygen -t rsa -C "zym736531683@gmail.com"
```

Git Bash一般可以在左下角开始菜单栏中找到，同样将邮箱替换为你的邮箱。成功之后会在用户的.ssh目录下生成两个文件，如下图。.shh目录一般在:c盘 -> 用户（user）-> 用户名（我的为鸣）->.ssh
![.ssh目录][3]
使用[Notepad++](https://notepad-plus-plus.org/)或其他应用打开`id rsa.pub`文件复制所有内容
打开Github-> 点击头像 -> Settings -> SSH and GPG keys ->New SSH keys -> 粘贴刚才复制的内容至Key框中

### 搭建博客

#### hexo本地设置

在硬盘中新建一个文件夹用来存放你的博客数据，右键文件夹点击`Git Bash Here`
在弹出的CLI中输入：

```
npm install -g hexo-cli
```

安装完毕后输入hexo检测是否安装成功
![hexo安装成功截图][4]
需要注意的是，在输入命令后CLI可能没有立即反应，耐心等待即可。
初始化hexo,后一个hexo为创建文件夹名，可修改

```
hexo init hexo
```

初始化成功后显示`Start blogging with Hexo!`
接着依次输入：

```
cd hexo
npm install     // 安装依赖文件
hexo generate   //生成静态文件
```

完成后启动服务器

```
hexo server
```

在浏览器中输入`http://localhost:4000/`即可看到默认的网站。假如没有成功打开请检查前面的步骤。关于安装福昕阅读器会占用4000端口的问题，请输入`hexo server -p 5000`进行尝试。

#### Github Page配置

在Github首页点击`New repository`
在新的页面中`Repository name`输入

```
xxx.github.io
```

xxx最好为Github用户名

#### 部署到Github

用刚才下载的Notepad++打开hexo目录下的配置文件`_config.yml`在文件最后面的deploy属性中加入代码修改为：

```
type: git
repository: git@github.com:WX-JIN/WX-JIN.github.io.git
branch: master 
```

千万注意每个冒号后面要加空格，否则hexo g会失败。详情配置见[官方文档](https://hexo.io/zh-cn/docs/configuration.html)。
通过CLI在hexo目录下输入命令安装hexo-deployer-git插件：

```
npm install hexo-deployer-git --save
```

重要的三个命令：

```
hexo clean //若变更主题后不生效，需执行
hexo generate  //简写为hexo g ，生成静态文件，变更文章后执行
hexo deploy //简写为hexo d 部署至Github   
```

一般变更文章需执行后两个，可结合为`hexo d -g`，至此已经大工告成。

## 从其他博客如Wordpress转移到hexo

```
hexo migrate <文件目录>
```

从wordpress导出文件，如何导出不再赘述，文件目录为绝对路径，可将文件导出后放入hexo文件夹根目录`hexo migrate <文件>`

[1]: http://imglf4.nosdn.127.net/img/WU1zZkNURmhGdm5la21NN1A5OVhFZ2N4SVFVWmpzRlRnVXpqTDBaV3ZrM21DSGIxck42Q3FBPT0.png?imageView&amp;amp;amp;amp;amp;amp;thumbnail=500x0&amp;amp;amp;amp;amp;amp;quality=96&amp;amp;amp;amp;amp;amp;stripmeta=0
[2]: http://imglf5.nosdn.127.net/img/WU1zZkNURmhGdm4wbHJBR3d5TGtRRUgydHZyOFllY1FqR1RYS212aDJsMnlDMHAvODVxWjhRPT0.png?imageView&amp;amp;amp;amp;amp;amp;thumbnail=500x0&amp;amp;amp;amp;amp;amp;quality=96&amp;amp;amp;amp;amp;amp;stripmeta=0
[3]: http://imglf3.nosdn.127.net/img/WU1zZkNURmhGdms3ZmpBQldTYTJBNnZRWWJpZFlCLyticHFSdnlIQ0lwZktWaERwSmlOZm9BPT0.png?imageView&amp;amp;amp;amp;amp;amp;thumbnail=500x0&amp;amp;amp;amp;amp;amp;quality=96&amp;amp;amp;amp;amp;amp;stripmeta=0
[4]: http://imglf4.nosdn.127.net/img/WU1zZkNURmhGdms3ZmpBQldTYTJBMEg3NWlFSzlpbWpmRlNoMld2OTBDZU8rVnEvYW12bVFnPT0.png?imageView&amp;amp;amp;amp;amp;amp;thumbnail=500x0&amp;amp;amp;amp;amp;amp;quality=96&amp;amp;amp;amp;amp;amp;stripmeta=0