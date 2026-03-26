---
layout: post
title: "多道程序并发 - 死锁，信号量，生产者消费者"
date: 2017-10-30T21:48:46+08:00
categories:
  - programming
tags:
  - operating system
---

# 临界区

是一段代码，在这段代码中进程将访问共享资源，当另外一个进程已经在这段代码中运行时，这个进程就不能在这段代码中运行

# 饥饿

指一个可运行的进程尽管能继续执行，但被调度器无限期忽视，而不能被调度器执行的情况

# 死锁

两个或者两个以上的进程因其中每个进程都在等待其他进程做完某些事情而不能继续执行

## 死锁的条件

  * 互斥
  * 占有且等待
  * 非抢占(可预防)



# 活锁

两个或者两个以上的进程为了响应其他进程中的变化而继续改变自己的状态但是不做有用的工作

# 竞争

多个线程或进程在读写一个共享数据时结果依赖于他们执行的相对时间

# 信号量

两个或以上的进程通过简单的信号进行合作，一个进程可以被迫在某一位置停止，直到他接受到一个特定的信号。

  * 一个信号量可以初始化乘非负数
  * semWait操作使信号量减1（占有一个信号量），若值变成负数，则执行semWait的进程被堵塞，否则进程继续执行
  * semSignal操作使信号量加1（释放一个信号量），若值变成正数或0，则被semWait堵塞的进程被解除堵塞
  * 二元信号量即为 _互斥_ ，所以说信号量是互斥锁的一种实现方式
  * 由具体的算法(例如FIFO)解决线程调度问题



# 条件变量

  * 与信号量的区别在于根本目的不同。条件变量用于管程(monitor)，目的是为了防止busy waiting。当一段程序所需资源在尝试访问时被锁住了，并不是无限次的重复检查是否解锁，而是使用条件变量，当资源可用时通知程序使用



# 生产者消费者问题

以下代码中，producer负责往buffer里写数据，consumer负责从buffer中取数据。需要保证数据的一致性。代码基本思想如下:  


  * 生产者
    * 锁lock_bp保证每次写入或读出buffer时可以正确操作
    * producer首先检测是否有空闲空间，若无，释放lock并等待直到有空buffer出现并且锁可用
    * 写数据
    * 获取锁后加入buffer数据
    * signal data信号量并释放锁
  * 消费者
    * 获取锁后检查是否有可用空间，若无，释放lock并等待直到有可取数据出现并且锁可用
    * 取出数据后释放锁
    * 取数据
    * 获取锁后将空间设置为空
    * signal free信号量并释放锁



Producer  

    
    
    lock(lock_bp)  
    while (free_bp.is_empty())  
        cond_wait(cond_freebp_empty, lock_bp)  
    buffer <- free_bp.get_buffer()  
    unlock(lock_bp)  
      
    ... produce data in buffer ...  
      
    lock(lock_bp)  
    data_bp.add_buffer(buffer)  
    cond_signal(cond_databp_empty)  
    unlock(lock_bp)  
      
  
---  
  
Consumer  

    
    
    lock(lock_bp)  
    while (data_bp.is_empty())  
        cond_wait(cond_databp_empty, lock_bp)  
    buffer <- data_bp.get_buffer()  
    unlock(lock_bp)  
      
    ... consume data in buffer ...  
      
    lock(lock_bp)  
    free_bp.add_buffer(buffer)  
    cond_signal(cond_freebp_empty)  
    unlock(lock_bp)  
      
  
---
