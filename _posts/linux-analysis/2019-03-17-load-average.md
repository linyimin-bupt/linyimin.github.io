---
layout: post
title: 平均负载
categories: [Linux]
---

> 破山中贼易，破心中贼难

<!-- TOC -->

- [平均负载的定义](#平均负载的定义)
- [查看平均负载](#查看平均负载)
  - [使用`uptime`命令查看](#使用uptime命令查看)
- [平均负载为多少时合理](#平均负载为多少时合理)
  - [查看CPU信息](#查看cpu信息)
- [平均负载和CPU使用率](#平均负载和cpu使用率)
- [平均负载案例分析](#平均负载案例分析)
  - [常用工具](#常用工具)
  - [使用stress模拟相关场景](#使用stress模拟相关场景)
- [总结](#总结)
- [参考链接](#参考链接)

<!-- /TOC -->

## 平均负载的定义

单位时间内， 系统处于**可运行状态**和**不可中断状态**的进程数。也可以理解成单位时间内，活跃的进程数。

可运行状态： 正在使用或者等待使用CPU的进程，使用top命令查看时，状态S为R的进程

不可中断状态： 正处在内核态关键流程中的进程。最常见的是等待硬件设备的IO响应。不可中断状态是**系统对进程和硬件设备的一种保护机制**。

例如， 当一个进程向磁盘读写数据时，为了保证数据的一致性，在得到磁盘回复前，是不能被其他进程或者中断打断的，这时候的进程就处于不可中断状态，如果此时的进程被打断了，就容易出现磁盘数据与进程数据不一致的问题。

Stack Overflow上对不可中断状态进程的解释

```
An uniterruptable process is a process which happens to be in a system call(kernel function) that can't be interrupted by a signal.
```

也就是说，处于不可中断状态的进程，在执行系统调用之后会被阻塞，而且不能被中断(杀掉)，直到系统调用完成。这系统系统调用实际上都是瞬时完成的，通过`ps`命令一般看不到这些进程，如果`ps`命令能观察到，可能是IO出现了问题，因为程序之所以进入不可中断状态，就是因为得不到相关IO响应(磁盘IO、网络IO、其他外设IO)。所以要想进程退出不可中断状态，就得使进程等待的IO恢复。

## 查看平均负载

### 使用`uptime`命令查看

使用命令`man uptime`查看`uptime`的作用：

```
Gives  a  one  line display of the following information. The current time, how long the system has been running, how many users are currently logged on, and the system load averages for the past 1, 5, and 15 minutes.
```

即查看当前时间，系统已经运行的时间、当前登录的用户数以及过去1、5、15分钟系统的平均负载。

```shell
uptime
```


![系统平均负载](http://blog.linyimin.club/images/posts/linux-analysis/uptime.png)


可以看到，过去1、5、15分钟，本系统的负载分别是1.05，1.27和1.31， 充分利用好这三个值，可以让我们更全面、更立体的理解目前系统的负载状态。

如果1、 5、 15分钟的平均负载：

- 基本相同，说明系统的负载很稳定；
- 如果1分钟前的负载远大于15分钟前的负载，说明最近1分钟的负载在增加。这种情况可能是临时性的，需要持续的观察。一旦1分钟的平均负载过大，说明系统发生了过载，需要想办法分析优化了；
- 如果1分钟前的负载远小于15钟前的负载，说明系统的负载在下降。

根据上图可以知道，整体来说系统的负载很稳定。那平均负载为多少时，才能保证系统的运行效率呢？

## 平均负载为多少时合理

最理想的情况下，平均负载和系统CPU的逻辑核数一致。所以我们需要先知道系统中CPU的逻辑核数。

通过读取/proc/cpuinfo文件获取系统CPU的信息。Linux系统贯彻“一切都是文件”的思想，所以系统相关的很多信息，进程状态都可以通过读取相关文件来获取。

Linux系统上的/proc文件目录是一种文件系统，及proc文件系统。与其他常见的文件系统不同，proc文件系统是一种伪文件系统（虚拟文件系统），所有的文件都存储在内存中，不占用任何的磁盘空间，存储的是当前内核运行状态的一系列文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息。可以理解为Linux为用户提供一种以文件系统的方式实现进程和内核的通信接口。

### 查看CPU信息

```shell
cat /proc/cpuinfo
```

![cpu information](http://blog.linyimin.club/images/posts/linux-analysis/cpuinfo.png)

相关字段说明

- processor: 每个逻辑处理器的id
- physical id: 物理封装的处理器id
- CPU cores： 一个物理封装的处理器中内核的数量
- siblings: 一个物理封装的处理器中逻辑处理器的数量

CPU的逻辑核数 = physical id * siblings = processor的个数

也可以直接计算processor的个数

```shell
cat /proc/cpuinfo | grep 'processor' | wc -l
```

一般来说，当平均负载高于CPU数量的70%时，就应该分析排查负载高的问题了，因为负载过高，就有可能导致进程的响应过慢，从而影响服务的正常功能。

最推荐的方法是： 监控系统的平均负载，根据相关的历史数据，判断负载的变化趋势，当发现负载有明显的升高趋势时，再去做相关的分析和调查。

## 平均负载和CPU使用率

平均负载和CPU使用率并不存在并不是完全对应的，因为平均负载表示的是单位时间内，系统中处于**可运行状态**和**不可中断状态**下的进程数，而CPU使用率表示的是单位时间下CPU的繁忙程度。所以应该具体情况具体分析：

- 对于大量CPU密集型进程： 平均负载会升高， 同时会使用大量的CPU，所以CPU的使用率也会升高；
- 对于大量IO密集型进程： 平均负载会升高，但是大部分的进程都在等待IO响应，CPU处于空闲状态，CPU使用率可能不会升高； 
- 对于大量处于等待CPU的进程， 平均负载会升高，而由于进程的调度也会使用CPU，所以CPU使用率也会升高。

## 平均负载案例分析

在进行案例分析之前，先介绍三个常用的工具

### 常用工具

- stress

> Impose certain types of compute stress on your system

`stress` 是一个系统压力测试工具，在这里用作异常进程模拟平均进程升高的场景。

|参数选项|解释说明|
|:---|:---|
|--timeout N|timeout after N seconds|
|--cpu N|spawn N workers spining on sqrt()|
|--io N|spawn N workers spining on sync()|
|--vm N|spawn N workers spining on malloc()/free()|

- mpstat

`mpstat` 是一个常用的多核CPU性能分析工具，用来实时查看每个CPU的性能指标及平均指标

> Report processors related statistics.

|参数选项|解释说明|
|:---|:---|
|-I|Report interrupts statistics|
|-P ALL|Report all processors statistics|
|-u|Report CPU utilization|
|N|Diaplay reports at N seconds internal|
|M|Diaplay M reports at N seconds internal, if M is 0 or omiting, will report continuously.|

**字段说明**

- %usr: 用户空间使用CPU的时间占比
- %nice: 优先级为nice的进程使用CPU时间占比
- %sys: 系统内核使用CPU时间占比
- %iowait: 系统处于IO时，CPU空闲时间占比
- %irq: 处理硬件中断的CPU时间占比
- %soft: 处理软中断的CPU时间占比
- %idle: 没有IO时，CPU空闲时间占比

  
- pidstat

`pidstat` 是一个常用的进程性能分析工具，用来实时查看进程的CPU、内存、I/O及上下文 切换等指标。

> Report statistics for Linux tasks.

|参数选项|解释说明|
|:---|:---|
|-d|Report I/O statistics|
|-l|Display the process command name and all its arguments|
|-p pid|Select process for which statistics is to be reported|
|-r|Report page faults and memory utilization|
|-s|Report stack utilization|
|-t|Display statistics for threads associated with selected tasks|
|-v|Display number of threads and file descriptors associated with current task|
|-w|Report task swiching activity|


**字段说明**

- KB_rd/s: Number of kilobytes the task has caused to be read from disk per second.
- KB_wr/s: Number of kilobytes the task has caused, or shall cause to be written to disk per second.
- minflt/s: Total number of minor faults the task has made per  second, those which have not required loading a memory page from disk.
- majfflt/s: Total number of major faults the task has made  per second, those which have required a memory page from disk.
- VSZ: Virtual Size
- RSS: Resident  Set Size
- threads: Number of threads associated with current task.
- fd-nr: Number of file descriptors associated with current task.
- cswch/s: Total number of voluntary context switches the task made per second.  A voluntary context switch occurs when a task blocks because it requires a resource that is unavailable.
- nvcswch/s: otal number of non voluntary context switches the task made per second.  A involuntary context switch takes place when a task executes for the duration of its time slice and then is forced to relinquish the processor.

### 使用stress模拟相关场景

1. CPU密集型进程

> 模拟一个CPU使用率100%的场景，时间为5分钟

```shell
$ stress --cpu 1 --timeout 600
```

使用`watch -d uptime` 查看系统的负载

![系统负载](http://blog.linyimin.club/images/posts/linux-analysis/stress-cpu-1-uptime.png)

**可以发现平均负载是在逐步下降的，最后会趋于稳定，如果不在运行其他程序**

使用`mpstat -P ALL 5` 查看CPU使用率(每隔5秒输出一次)

![CPU使用率](http://blog.linyimin.club/images/posts/linux-analysis/stress-cpu-1-utilization.png)

可以发现CPU 2的使用率接近100%，而iowait为0%，说明平均负载的升高是由于CPU密集型进程引起的。

**注意**： 上面的截图呈现平均负载逐渐下降的原因是之前启动了大量的进程，关闭之后直接运行的`stress`，如果之前系统没有运行其他进程的话，就会看到平均负载是逐渐升高的。

2. IO密集型进程

> 模拟IO，即不断的执行sync()

```shell
$ stress --io 1 --timeout 600
```

使用`watch -d uptime`查看系统的平均负载

- 运行前的系统平均负载

![运行前系统平均负载](http://blog.linyimin.club/images/posts/linux-analysis/stress-io-1-before-uptime.png)

- 运行后的平均负载

![运行后系统平均负载](http://blog.linyimin.club/images/posts/linux-analysis/stress-io-1-after-uptime.png)

使用`mpstat -P ALL 5`查看CPU情况

![CPU使用情况](http://blog.linyimin.club/images/posts/linux-analysis/stress-io-1-cpuinfo.png)

**可以发现， 系统的平均负载生高了，其中的一个CPU的系统使用率达到了30%， 而iowait有62%， 说明IO密集型进程也有可能使平均负载升高**

> 产生64个进程进行sync(IO)操作

```shell
$ stress --io 64 --timeout 600
```

使用`mpstat -P ALL 5`查看CPU使用情况

![密集型IO进程CPU使用情况](http://blog.linyimin.club/images/posts/linux-analysis/stress-io-64-cpuinfo.png)

与前面相比， 每个CPU的系统使用率升高了，iowait却只保持在2,30%, 在使用`pidstat -d -C stress`查看磁盘IO的具体情况：

![磁盘IO的具体情况](http://blog.linyimin.club/images/posts/linux-analysis/stress-io-64-pidio.png)

发现根本就没有想磁盘中写入任何数据，可能是缓冲区没有任何数据，无法产生大的IO压力，CPU都被大量的系统调用消耗了。

> 使用`stress --hdd 64 --timeout 600`模拟大量的IO进程（hdd表示将使用write()函数将临时文件写入磁盘）

使用`mpstat -P ALL 5`查看CPU使用情况

![密集型IO进程CPU使用情况](http://blog.linyimin.club/images/posts/linux-analysis/stress-hdd-64-cpuinfo.png)

与前面的sync相比， iowait提高到了70%多, 在使用`pidstat -d -C stress`查看磁盘IO的具体情况：

![磁盘IO的具体情况](http://blog.linyimin.club/images/posts/linux-analysis/stress-hdd-64-pidio.png)

发现根本进程有向磁盘中写入数据，产生了大的IO压力，CPU大部分被等待IO消耗，但也有一部分由系统调用消耗。



1. 大量进程场景

> 模拟64个进程

```shell
$ stress --cpu 64 --timeout 600
```

使用`watch -d uptime` 查看系统的负载

![系统负载](http://blog.linyimin.club/images/posts/linux-analysis/stress-cpu-64-uptime.png)

**可以发现平均负载是在逐步下降的，最后会趋于稳定，如果不在运行其他程序**

使用`mpstat -P ALL 5` 查看CPU使用率(每隔5秒输出一次)

![CPU使用率](http://blog.linyimin.club/images/posts/linux-analysis/stress-cpu-64-utilization.png)

可以发现所有CPU的使用率接近100%，而iowait为0%，说明平均负载的升高是由于CPU密集型进程引起的。

## 总结

平均负载提供了一个快速查看系统整体性能的手段反映了整体的负载情况，但是并不能直接发现哪里出现了性能瓶颈。因为平均负载高有可能是以下原因引起的：

- CPU密集型进程导致的
- I/O密集型导致的

所以，当发现负载高时，可以使用mpstat和pidstat工具来辅助分析负载的来源。

[我的手写笔记](http://blog.linyimin.club/notes/linux-performance-optimization/平均负载.pdf)

## 参考链接

[Linux性能优化实战-基础篇：到底应该怎么理解“平均负载”？](https://time.geekbang.org/column/article/69618)