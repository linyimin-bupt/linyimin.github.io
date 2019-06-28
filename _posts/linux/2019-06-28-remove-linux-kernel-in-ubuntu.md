---
layout: post
title: 删除Ubuntu下多余的内核
categories: [Linux]
---

> 破山中贼易, 破心中贼难

如果升级到了一个新的内核，并且还比较稳定，那么老的内核就可以清理了，放在电脑里也占位置。方法（命令行比较通用）如下：

## 查看系统内存在的内核版本列表

```shell
sudo dpkg --get-selections |grep linux
```

## 查看当前Ubuntu系统使用的内核版本

```shell
uname -a
```

## 删除多余内核

```shell
sudo apt-get purge linux-headers-3.0.0-12 linux-image-3.0.0-12-generic 
```

## 更新grup

```shell
sudo update-grub
```

## 参考链接

[删除多余的ubuntu内核](https://blog.csdn.net/guang11cheng/article/details/50577951)