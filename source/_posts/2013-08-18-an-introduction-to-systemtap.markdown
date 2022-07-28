---
layout: post
title: "an introduction to systemtap"
keywords: "systemtap introduction"
description: "an simple introduction to systemtap"
date: 2013-08-18 10:07
comments: true
categories: [devops]
tags: [systemtap]
---
就像它的名字system（系统）tap（窃听），systemtap提供了底层的支持，以简化系统在运行时的信息收集（包括用户态和内核态的信息）。通过对收集到的信息进行分析，可以帮忙我们识别潜在的性能原因和功能问题。
<!-- more -->

    SystemTap is a tool for the Linux Operating System
    that allows developers and system administrators to deeply investigate
    the behavior of the kernel and even user space applications
    in order to discover error conditions, performance issues,
    or just to understand how the system works, similarly to DTrace.

    systemtap不会直接告诉你程序哪里出了问题。但是在你对代码熟悉的情况下，可以用一成的功力使出七八成的效果

以往在收集这些信息时，我们需要添加检测点，重新编译，安装，重启使之生效。
挺没意思的。。。

使用systemtap就不需要了，我们只要编写相关脚本，并让systemtap的命令
行接口stap（systemtap script translator/driver）进行调用就可以收集到用
户空间和内核空间的相关信息。

通过编写SystemTap脚本，我们就可以很容易地收集甚至篡改系统数据，这是普
通的linux系统工具所做不到的。

### 相关背景知识 ###
在Solaris系统上，有一个大名鼎鼎的动态跟踪工具
[Dtrace](http://en.wikipedia.org/wiki/DTrace)，这一个相当棒的工具（从
2001年10月开始开发，在2005年1月首次发布），曾
荣获《华尔街杂志》
[2006技术创新大奖中的金奖](https://blogs.oracle.com/swan/date/20070307)
，和ZFS文件系统一样，DTrace一直都因版权问题而无法移植到Linux上，但
Oracle（SUN公司被Oracle收购）在2012年2月宣布发布DTrace for Linux beta
版，即将Solaris操作系统的动态跟踪工具移植到他们的Unbreakable
Enterprise Kernel(2.6.39)内，也就是说Linux人员终于也可以使用DTrace了。

在2005年1月开始，开源社区为linux操作系统开发了基于GPL许可的systemtap。

目前一般的Linux发行版，比如Fedora、OpenSuse、CentOS等，已经包含有systemtap的完整支持了。

### 与其他trace工具比较 ###
* Dtrace：
  dtrace不用多说了，systemtap的开发，就是用来替代dtrace的，但是他们还
  是有以下几个明确的差异：
  1. Dtrace不允许你任意地注入c代码，即在运行时修改系统数据，通过-g选项
  2. Dtrace脚本是以虚拟机字节码的形式在内核中进行解释，而systemtap脚本
     则是以本地二进制代码的形式被进行加载
* strace：
  strace工作于用户态，只能用于处理系统调用
* ltrace：
  ltrace工作于用户态，只能用于处理用户态的函数
* gdb：
  gdb也工作于用户态，它的目标是进行交互式的调试

更多信息请参见：[SystemtapDtraceComparison](http://sourceware.org/systemtap/wiki/SystemtapDtraceComparison)

而systemtap作为一个通用工具，通过追踪（tracing），可以了解：

    在一段时间内，某进程执行了哪些系统调用以及次数
    一个函数执行了多长时间
    进程函数调用栈并使用相关工具生成火焰图 (Flame Graph)
    甚至可以使用systemtap脚本进行任意发挥


reference:
http://sourceware.org/systemtap/langref/SystemTap_overview.html#SECTION00021000000000000000
http://raisama.net/talks/fisl10/kernel-hacking/stap.pdf
http://cheeselee.fedorapeople.org/systemtap-robin-20130817.pdf
http://agentzh.org/misc/slides/yapc-na-2013-flame-graphs.pdf
http://www.linuxfoundation.jp/jp_uploads/seminar20090225/obata.pdf
http://dtrace.org/blogs/brendan/2011/10/15/using-systemtap/
http://lenky.info/2013/02/04/systemtap%E5%88%9D%E8%AF%95%E7%94%A8/
