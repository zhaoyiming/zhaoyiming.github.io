---
title: 5$首年付vps
date: 2018-03-01 12:01:26
tags:
catalogue:
- 服务器
---
## 准备工作：
+ 一个用于校园认证的大学邮箱。
+ 一个[Github账号](http://github.com)。
+ 一个paypal账号。
## 获取Github Student Developer Pack的优惠
打开[Github学生认证界面](https://education.github.com/pack/),点击Get Your Pack。登录Github账号，之前登录过的应该会直接进入认证。![提示认证][1]
提示认证，点击Yes按钮。
接着进入认证界面。根据页面提示填入相应内容，School name填写学校英文名就可以，最后一栏How do you plan to use Github？象征性写两句就可以。如果没有学校邮箱也可以，可以上传学生卡照片之类的。验证比较宽松，很容易就能通过，第一次没有申请成功可以修改一下再尝试。
验证成功后在My pack中可以找到DigitalOcean的50刀优惠码。![DigitalOcean优惠码][2]
## 获取vps
### 注册[DigitalOcean账号](https://m.do.co/c/7b48d52d438e)这里是我的邀请链接，可以额外获得10$，官网直接申请的话并没有orz。
### 激活优惠码
![billing][3]
在Setting界面找到Billing选项卡，在下拉的Promo Code中填入刚才在学生认证中看到的优惠码，如果出现错误可以找客服沟通一下。
![promo code][4]
### 验证账户
虽然DigitalOcean优惠码已经激活了，但是这时候还不能购买VPS，需要账户验证。账户验证有两种方式，一是绑定信用卡，二是通过paypal向账户充值5美元。根据提示付款解锁吧，这里就不再赘述了，paypal是可以绑定国内储蓄卡的。
### 购买VPS
点击首页的Create按钮，选择Droplets，进入购买。选择一个发行版Linux，这里可以选一个任意版本的Ubuntu，往下找到5美元月付的选项卡，再往下选择地区，用[测速网址](http://speedtest-sfo1.digitalocean.com/)找到一个网速最快，延迟最小的地区。本地联通一般是sfo圣弗兰西斯科机房较优。其他没有提到的选项不用管，最后点击Create完成创建。
## 酸酸乳的搭建
这里先放上[秋水逸冰的一键脚本](https://shadowsocks.be/9.html)。
从刚才创建的页面开始![服务器界面][5]
点击Access Console按钮，进入网页控制台，你邮箱之前应该会受到dg发来的服务器密码，输入，会提示你创建密码，需要注意的是unix特性输入的密码是不会显示的，直接输入即可。
成功进入服务器后，参见秋水大大的博客直接设置即可。设置完后，在相应系统的应用中输入设置的信息即可。


  [1]: http://imglf4.nosdn.127.net/img/WU1zZkNURmhGdm16M0pieWhqMDNIVkhETnFrV2o5MXVrQmhUMlUvN0FSTnFEeHhmV01MOTdnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0
  [2]: http://imglf5.nosdn.127.net/img/WU1zZkNURmhGdm16M0pieWhqMDNIZjBJQmQwcU1mS3dKbkFMZTl3VWU2QUJBdVdzc3FSaXV3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0
  [3]: http://imglf3.nosdn.127.net/img/WU1zZkNURmhGdm16M0pieWhqMDNIU0V3eHAvK3ZUSENJK0RDRjJjN1phdmluelNVWUtPbkx3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0
  [4]: http://imglf6.nosdn.127.net/img/WU1zZkNURmhGdm16M0pieWhqMDNIZG94MGR2d1ZDbzAyMDlQUzZFMDEzZXQ0MktRWTVOeCtRPT0.jpg?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg
  [5]: http://imglf6.nosdn.127.net/img/WU1zZkNURmhGdmxBa3hFSWFKeEdZZ1VrSFoxZkhYNGxCSmhzQ24vRGpvTmJuKzZQWXNQWWNRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0