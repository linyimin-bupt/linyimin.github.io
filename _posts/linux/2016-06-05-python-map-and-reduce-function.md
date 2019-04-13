---
layout: post
title: 使用Python下的map和reducce函数
categories: [Linux]
---

> 破山中贼易,破心中贼难
> 管它什么真理无穷,进一寸有一寸的欢喜

<!-- TOC -->

- [map()函数](#map%E5%87%BD%E6%95%B0)
- [reduce()函数:](#reduce%E5%87%BD%E6%95%B0)
- [Example](#example)

<!-- /TOC -->

## map()函数

接收两个参数:一个是函数,一个是序列,map函数将传入的函数一次作用到序列的每个元素,若传入的函数有返回则把结果作为新的序列返回.反之,返回空序列(字符串也是序列)


## reduce()函数:

接受两个参数:一个是函数,一个是序列,reduce函数将传入的函数(必须两个参数)作用到序列上,输出结果继续和序列的下一个元素做运算,最终reduce()函数的返回结果,由传入的函数返回结果决定.

## Example

下面看一个例子:通过调用map()函数和reduce()函数,求一个整数的组成数字及其数字之和

```
  #_*_coding:UTF-8_*_
  
  """
  2016-06-05
  程序通过调用map()函数和reduce()函数,
  求输入一个整数输出组成该整数的数字及其和
  
  """
  num = input('输入一个整数:')
  #将整数转换成字符串
  s = str(num)
  
  #定义map参数函数
  def f(s):
      #字符与数字字典
      dic = {'1':1,'2':2,'3':3,'4':4,'5':5,"6":6,'7':7,'8':8,'9':9,'0':0}
      return dic[s]
      
  #定义reduce参数函数
  def add(x,y):
      return x + y
      
  #调用map()函数,将字符串转换成对应数字序列,并打印
  s = map(f,s)
  print "输入整数%d的组成数字为%s"%(num,s),
  
  #调用reduce函数,对数字序列求和,并打印
  Sum = reduce(add,s)
  print "其和为:%d"%Sum
```

程序执行结果:
![这里写图片描述](http://img.blog.csdn.net/20160605175406976)