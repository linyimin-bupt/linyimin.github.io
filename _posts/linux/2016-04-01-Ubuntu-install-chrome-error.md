---
layout: post
title: 安装chrome,使用apt-get update报错
categories: [Linux]
---

> 管它什么真理无穷,进一寸有一寸的欢喜

<!-- TOC -->

- [错误提示](#%E9%94%99%E8%AF%AF%E6%8F%90%E7%A4%BA)
- [解决方法](#%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95)
  - [原因](#%E5%8E%9F%E5%9B%A0)

<!-- /TOC -->


## 错误提示

```
apt-get update
```

```
无法下载
http://dl.google.com/linux/chrome/deb/dists/stable/Release  Unable to find expected entry 'main/binary-i386/Packages' in Release file (Wrong sources.list entry or malformed file)
```

## 解决方法

```
vim /etc/apt/sources.list.d/google-chrome.list
```

### 原因

出现这种情况的原因是：因为官方的Google Chrome库不再提供32位包
  

所以把`deb http://dl.google.com/linux/chrome/deb/ stable main` 改成`deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main` 之后再执行以下命令:

```shell
$ sudo apt-get update
```

就可以更新Linux源了
