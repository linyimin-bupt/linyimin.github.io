---
layout: post
title: /etc/sudoer修改错误，且进不了root修复
categories: [Linux]
---

> 破山中贼易, 破心中贼难
> 管它什么真理无穷, 进一寸有一寸的欢喜

<!-- TOC -->

- [sudo error](#sudo-error)
- [修改方法](#%E4%BF%AE%E6%94%B9%E6%96%B9%E6%B3%95)

<!-- /TOC -->

## sudo error

``` shell
~$ sudo
```

```
sudo： >>> /etc/sudoers：syntax error 在行 21 附近<<<
sudo： /etc/sudoers 中第 21 行附近有解析错误
sudo： 没有找到有效的 sudoers 资源，退出
sudo： 无法初始化策略插件
```

## 修改方法

1.重启ubuntu，在重启界面选择ubuntu高级选项

2.选择root进入root界面

3.输入

```
chmod 777 /dev/null 
mount -o remount rw /
vi /etc/sudoers
```

之后修改soduers文件并保存

4.退出root

```
exit
```
5.选择resume重新进入系统即可
