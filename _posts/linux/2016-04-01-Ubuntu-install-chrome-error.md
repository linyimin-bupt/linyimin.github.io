---
layout: post
title: 安装chrome,使用apt-get update报错
categories: [Linux]
---

> 破山中贼易,破心中贼难
> 管它什么真理无穷,进一寸有一寸的欢喜


```
apt-get update
```

无法下载
 http://dl.google.com/linux/chrome/deb/dists/stable/Release  Unable to find expected entry 'main/binary-i386/Packages' in Release file (Wrong sources.list entry or malformed file)

解决方法：

```
vim /etc/apt/sources.list.d/google-chrome.list
```


出现这种情况的原因是：因为官方的Google Chrome库不再提供32位包
  

所以把
deb http://dl.google.com/linux/chrome/deb/ stable main 改成
deb [arch=amd64] http://dl.google.com/linux/chrome/deb/      stable main
即可之后再执行

```
apt-get update
```
就可以更新Linux源了
