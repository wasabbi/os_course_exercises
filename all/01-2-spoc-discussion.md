# 操作系统概述

---

## **提前准备**

（请在上课前完成）

* 完成lec1的视频学习和提交对应的在线练习
* git pull ucore\_os\_lab, ucore\_os\_docs, os\_tutorial\_lab, os\_course\_exercises in github repos。这样可以在本机上完成课堂练习。
* 知道OS课程的入口网址，会使用在线视频平台，在线练习/实验平台，在线提问平台\(piazza\)
  * [http://os.cs.tsinghua.edu.cn/oscourse/OS2018spring](http://os.cs.tsinghua.edu.cn/oscourse/OS2018spring)


* 会使用linux shell命令，如ls, rm, mkdir, cat, less, more, gcc等，也会使用linux系统的基本操作。
* 在piazza上就学习中不理解问题进行提问。



# 思考题

## 填空题

* 当前常见的操作系统主要用<u>C/C++, ASM</u>编程语言编写。
* "Operating system"这个单词起源于<u>OPERATOR</u>。
* 在计算机系统中，控制和管理<u>系统资源</u>、有效地组织<u>多道程序</u>运行的系统软件称作<u>操作系统</u>。
* 允许多用户将若干个作业提交给计算机系统集中处理的操作系统称为<u>批处理</u>操作系统
* 你了解的当前世界上使用最多的32bit CPU是<u>ARM</u>，其上运行最多的操作系统是<u>ANDROID</u>。
* 应用程序通过<u>系统调用</u>接口获得操作系统的服务。
* 现代操作系统的特征包括<u>并发</u>，<u>共享</u>，<u>虚拟</u>，<u>异步</u>。
* 操作系统内核的架构包括 <u>宏内核</u> ，<u>微内核</u> ，<u>外核</u>。

/* 宏内核（单体内核,monolithic kernel），微内核(microkernel)，外核（exokernel）*/

## 问答题

- 请总结你认为操作系统应该具有的特征有什么？并对其特征进行简要阐述。
并发:计算机系统中同时存在多个运行的程序, 需要OS管理和调度. 共享:同时访问(宏观上), 互斥共享(微观上) 虚拟:利用多道程序设计技术, 让每个用户都觉得有一个计算机专门为他服务. 异步:程序的执行不是一贯到底, 而是走走停停, 不可预知. 只要运行环境相同, OS保证程序运行的结果相同 (输入一致, 输出一致)


- 为什么现在的操作系统基本上用C语言来实现？为什么没有人用python，java来实现操作系统？c语言相对于python和java是比较低级的语言, 所以更容易使用比较基础的功能. 比如, c语言里面涉及到指针对应内存, 这样容易编写操作系统管理存储的功能. 但python和java根本就没有指针的概念,只有引用.不好访问内存.而且java语言和python分别一般是在已有jdk, python程序的环境下运行的, 所以不适合于编写操作系统.

---

## 可选练习题

---

- 请分析并理解[v9\-computer](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)以及模拟v9\-computer的em.c。理解：在v9\-computer中如何实现时钟中断的；v9 computer的CPU指令，关键变量描述有误或不全的情况；在v9\-computer中的跳转相关操作是如何实现的；在v9\-computer中如何设计相应指令，可有效实现函数调用与返回；OS程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9\-computer中如何完成一次内存地址的读写的；在v9\-computer中如何实现分页机制；


- 请编写一个小程序，在v9-cpu下，能够输出字符


- 输入的字符并输出你输入的字符


- 请编写一个小程序，在v9-cpu下，能够产生各种异常/中断


- 请编写一个小程序，在v9-cpu下，能够统计并显示内存大小

