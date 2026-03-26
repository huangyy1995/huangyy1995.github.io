---
layout: post
title: "LOL中的Determinism（一）"
date: 2018-07-05T11:24:22+08:00
categories:
  - programming
tags:
  - game
---

本文原作者[Rick Hoskinson(链接为原文通道)](<https://engineering.riotgames.com/news/determinism-league-legends-introduction>),仅供学习用途

# 介绍

这篇博客系列将介绍英雄联盟中的Determinism故障恢复，这个功能的目的是将游戏的进度随时保存，也可以回到过去的某一个时间点继续开始游戏。在比赛时难免会有突发状况，甚至还可能会触发了游戏的某个bug。在这个时候如果不想重赛，determinism就发挥作用了。可以读取游戏在异常状态之前的进度然后从该进度开始玩，避免了彻底重赛。这个博客系列将分为4个部分，此篇为第一部分：

  * [第一部分：介绍](<http://xingyaohuang.com/2018/07/05/determinism1>)
  * [第二部分：实现](<http://xingyaohuang.com/2018/07/05/determinism1>)
  * [第三部分：同步时钟](<http://xingyaohuang.com/2018/07/05/determinism1>)
  * [第四部分：修复Divergences](<http://xingyaohuang.com/2018/07/05/determinism1>)



负责这个功能的项目叫做ChronoBreak，但是determinism的灵感来自于另一款测试工具 Delta Checker，Delta Checker可以通过一系列重复的输入值对游戏进行测试。

# 从 Delta Checker 到 ChronoBreak

一些游戏的bug是否应该影响比赛，甚至导致重赛呢？LOL团队16年仔细分析了这个问题。毫无疑问，重赛对于观众和电竞选手都是不好的体验。LOL团队高度重视这个问题并开始寻求解决方案。

在determinism出现之前，电竞团队对这个问题有过一些尝试：定时存档，重新操作游戏到达指定时间，甚至用虚拟机模拟真实游戏。然而这些都不能绝对准确的恢复游戏进度，这些解决方案也没有被官方承认。

于是ChronoBreak团队就出现了，它继承了Delta Checker团队的工具，开始开发Deterministic故障恢复工具。要求如下：

  1. 可以随时记录系统变化，并带有验证功能，实现服务端determinism
  2. 实现pipeline式的服务端determinism，一边测试/修复bug
  3. 开发电竞主办方可用来恢复比赛进度的工具



该工具于16年底开发完成，并在17年7月上线了GUI工具。

# ChronoBreak的结构

在电竞主办方能够使用ChronoBreak之前，我们必须让服务器(SNR, Server Network Recording)能够记录每场比赛，记录的内容包括输入和各种设定。然后我们就可以通过这些记录来恢复比赛了，恢复过程将很快，比如恢复到比赛的第40分钟通常只需要不到3分钟。接着玩家们可以断开有bug的服务器并重新连接到恢复好的服务器即可继续游戏。

流程图如下：

![determinism0](/content/images/2018/07/Determinism_intro_workflow.png)

更多内容请见下一篇。
