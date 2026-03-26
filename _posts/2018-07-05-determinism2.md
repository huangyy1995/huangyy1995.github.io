---
layout: post
title: "LOL中的Determinism（二）"
date: 2018-07-05T12:35:49+08:00
categories:
  - programming
tags:
  - game
---

本文原作者[Rick Hoskinson(链接为原文通道)](<https://engineering.riotgames.com/news/determinism-league-legends-implementation>),仅供学习用途

实现Determinism恢复工具花了拳头公司的工程师近一年的时间。这个工具能够记录游戏状态，回放游戏，在任意时间点重玩游戏。如果没有看过本系列的第一篇，请移步：

  * [第一部分：介绍](<http://xingyaohuang.com/2018/07/05/determinism1>)



# 定义

当输入相同时，游戏一定会生成同样的结果，这就是determinism。比如说我们定义  
`f(x) = x + 1`  
只要我们的输入固定，函数的结果也是固定的。这就是Determinism。不是Deterministic的过程称为Divergent的过程。

现在编程语言基本都是Deterministic的，所以我们要实现游戏的Determinism，最重要的一点就是如何对待未知的输入。

# 证明Determinism

为了证明系统是Deterministic的，我们需要在多次相同的输入下比较得到的结果。游戏中时常会出现Divergent，因为游戏是帧循环的，如果一旦出现很小的Divergence，这个Divergence将会越来越大，最后无法控制。所以我们要找到最开始小Divergence出现的根源所在。

将游戏的所有信息都记录下来是不现实的，所以我们将只记录关键的部分，即玩家和服务端的互动。

# 客户端输入

前面已经提到我们的目的是防止未知的输入，所以接下来我们先来明确一下游戏的输入。

我们将输入分为可控制的输入(键鼠操作)和不受控制的输入(随机数，由玩家操作产生的一些输入)：  
![determinism1](/content/images/2018/07/deteriminism_implementation_inputs.png)

我们想记录的是可控制的输入，通过之前的描述，我们很容易得到一种最简单的方法：记录下每一帧客户端向服务端传递的数据，在重放时，即可快速使用这些数据重新构建游戏的状态。而且此方法也相对简单。  
![determinism2](/content/images/2018/07/determinism_implementation_recording_flow.png)

# 控制输入

为了实现Determinism,首先要对原有的代码进行重构，比如把当前的多个时间api统一。此外，由于lol的帧数不是固定的，所以在记录每一帧信息的时候还要记录当前的帧速。

# 通信方式

为了让系统更加有Determinitic的性质，我们记录的内容的网络层级将尽可能低，仅仅在网络层之上。  
![determinism3](/content/images/2018/07/determinism_implementation_recording_layer_2.png)

还需要考虑的事情包括：

  * 游戏的初始值
  * Determinism的伪随机数生成器
  * 未初始化的值
  * 异步请求的处理
  * 关于内存的non-determinism
