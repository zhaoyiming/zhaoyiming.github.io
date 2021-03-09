---
title: 为你的博客添加Fork me on Github
date: 2018-02-05 10:23:53
tags: 
- 博客
- hexo
categories: 
- 学外拓展
---

# 为你的博客添加Fork me on Github

利用作者写好的代码很容易实现。
### 复制源码
从[Github地址](https://github.com/blog/273-github-ribbons)中复制一个你喜欢的样式，修改替换`<a href="xxx">`中的Github地址为你自己的Github地址。
### 修改主题文件
复制代码到博客目录下(以material主题为例)`themes\material\layout\layout.ejs`。其他主题大体相同，不同主题`layout`文件类型可能不同。
修改后如图
![layout文件截图][1]
添加如图位置即可，不同主题可能代码不相同，如果添加后未生效，尽量在主函数体中添加。

[1]: http://imglf3.nosdn.127.net/img/WU1zZkNURmhGdmtOV1FwRGUrL3BReUYxeDVTMVk5SVlNQ1BBdDBoWk00QW40U1JVM0NRVDhnPT0.png?imageView&thumbnail=1894y859&type=png&quality=96&stripmeta=0