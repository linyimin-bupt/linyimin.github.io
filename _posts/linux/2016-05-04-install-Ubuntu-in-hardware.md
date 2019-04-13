---
layout: post
title: 在移动硬盘里装多个linux系统
categories: [Linux]
---

> 破山中贼易,破心中贼难

1.下载Universal-USB-Installer制作usb启动盘

2.准备好需要安装的操作系统，本次说明使用ubuntu-14.04.4-desktop-i386.deb版本。

3.插上一个优盘，双击Universal USB Installer软件，此软件是绿色软件，看到如下界面。
![这里写图片描述](http://blog.linyimin.club/images/2016-05-04-install-Ubuntu-in-hardware/USB-Installer-setup.png)

这是一个许可协议，遵行GPL协议，点击“I Agree”按钮，看到下一个界面。

![这里写图片描述](http://blog.linyimin.club/images/GPL.png)
             
在Step1中选择要安装的操作系统类型----ubuntu

Step2中选择要安装的系统ISO文件

Step3选择要安装的优盘盘符,若找不到你想制定的移动硬盘，选择show all device。

Step4不用理会

   ![这里写图片描述](http://blog.linyimin.club/images/system-type.png)
   
以上步骤都完成后，点击“Create”按钮。

之后根据提示选择确定即可。

![这里写图片描述](http://blog.linyimin.club/images/create.png)

制作usb启动盘成功的截图

4.重启电脑，在刚进入界面时多次按下F12键，进入一个选择界面，选择USB启动。

5.选择使用ubuntu，进入如下界面

![这里写图片描述](http://blog.linyimin.club/images/ubuntu-desktop.jpeg)

6.插入移动硬盘（最好备份数据后格式化且不用分区）我之前没有格式化，安装的时候一直停留在探测文件界面，无法进入安装界面，找了网上很多教程都无法解决。

7.剩下的安装步骤查看以下的链接（很详细）

[怎样把Ubuntu装到移动硬盘里](http://jingyan.baidu.com/article/624e74598306c334e8ba5a00.html)

8.安装完第一个系统之后重复以上步奏安装其他系统。“安装启动加载器”，“**安装启动引导器的设备**”全都选择**移动硬盘**。而且不需要再讲硬盘格式化。

9.重启电脑，进入第一个系统，在文件系统中分别找出各个系统/boot/grub/grub.cfg或者/boot/grub/grub.conf文件分别分别复制到此时打开系统的/boot/grub/grub.cfg或者/boot/grub/grub.conf文件之后。

10.重启电脑，按F12键，选择USB启动

11.此时，可看到多个系统选项，选择你需要的系统进入即可。
