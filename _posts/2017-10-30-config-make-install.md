---
layout: post
title: "Configure, make, make install背后的原理"
date: 2017-10-30T10:49:51+08:00
categories:
  - programming
tags:
  - random
---

如果用过Unix系统，相信对下面这三个命令绝对不陌生  

    
    
    ./configure  
    make  
    make install  
      
  
---  
  
这三条命令通常用于使用源代码安装一个软件，但是这三条命令的执行过程背后到底是什么呢？本文将讨论这一问题。

# 调试环境 - `./configure`

这个脚本通常是用来调试环境，使得当前的操作系统满足安装软件的基本条件。脚本的成功运行保证了所有的依赖环境或包都是可用的，或者会提示用户哪些东西是需要但是系统中没有的。

Unix程序通常是用C写的，所以我们通常会用C编译器构建程序。在这种情况下，configure脚本将会确认系统中是有C编译器的并找出在哪里调用它。

# 构造程序 - `make`

当configure过程完成后，我们便将进入构造程序的过程。主要内容其实是执行Makefile里的内容。关于Makefile本文不详细介绍，可参考[教程](<http://www.ruanyifeng.com/blog/2015/02/make.html>)。

# 安装程序 - `make install`

此时程序已经被构造好可以运行了，这一步需要做的是把可执行文件，library以及文档复制到最后的位置例如($PATH),此步骤完成后程序就算安装成功了。
