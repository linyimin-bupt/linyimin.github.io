---
layout: post
title: Ubuntu apt-get update和apt-get upgrade的区别
categories: [Linux]
---

> 破山中贼易,破心中贼难
> 管它什么真理无穷,进一寸有一寸的欢喜

<!-- TOC -->

- [`apt-get update`](#apt-get-update)
- [`apt-get upgrade`](#apt-get-upgrade)

<!-- /TOC -->


## `apt-get update`

```
$ sudo apt-get update
```

   同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包。

## `apt-get upgrade`

```
sudo apt-get upgrade
```
upgrade 是升级已安装的所有软件包，升级之后的版本就是本地索引里的，因此，在执行 upgrade 之前一定要执行 update, 这样才能是最新的。