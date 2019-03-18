---
layout: post
title: 远程连接Linux虚拟机
categories: [Linux]
tags: 学习
---

> 破山中贼易，破心中贼难

1. 查看虚拟机Linux是否安装ssh

```bash
  $ ps -e | grep ssh
```
如果显示如下结果：
```bash
  linyimin@ubuntu:/etc/ssh$ ps -e | grep ssh
    1380 ?        00:00:00 ssh-agent
    2466 ?        00:00:00 ssh-agent
```
说明没有安装ssh server,这时如果试图通过ssh连接该机器会出现：
```bash
  Connection closed by remote host 
  ```

2. 安装ssh-server服务

```bash
$ sudo apt-get install openssh-server
```
3. VMware NAT网络配置(Edit -> VMware Network Editor)

- 在VMware Network Editor对话框中设置，先设置画红线部分，具体如下图:


  ![在这里插入图片描述](https://img-blog.csdn.net/20181023160224202?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x5bTE1Mjg5OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

- 根据NAT设置，使用ifconfig查看当前IP，然后设置成静态ip，具体如下图所示：


![在这里插入图片描述](https://img-blog.csdn.net/20181023160301382?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x5bTE1Mjg5OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后点击应用即可。