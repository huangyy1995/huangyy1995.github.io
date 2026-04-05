---
layout: post
title: "用强化学习训练 AI 打砖块：我的项目设计思路"
date: 2026-04-05T21:00:00+08:00
categories:
  - project
tags:
  - AI
  - game
  - reinforcement learning
---

之前写了个 HTML5 的打砖块游戏（[breakout.xingyao.info](http://breakout.xingyao.info/)），最近在给它加 RL（强化学习）训练系统——让 AI 从零学会玩这个游戏。记录一下设计思路。

## 为什么选打砖块

打砖块是强化学习的经典实验场景，DeepMind 2013 年那篇开山之作就是用 DQN 打 Atari Breakout。不过原始的 Atari 实验用的是像素输入，我这个版本不一样：游戏是自己写的，所有内部状态都可以直接暴露给 AI，能做更多有趣的实验。

## 架构：浏览器游戏 + Python 训练

整体架构分两端：

```
浏览器（JS 游戏引擎）       Python（RL 训练）
     Game.js          ←WebSocket→    ws_env.py
     stepForAI()                     Gymnasium Env
     getState()                      DQN / PPO / GRPO Agent
```

游戏跑在浏览器里，通过 WebSocket 暴露一个 Gym 风格的接口（`reset()` / `step(action)`）。Python 端连上来后，每一步发一个动作，收到下一帧的状态和奖励。中间有个轻量的 WebSocket 中继服务器做转发。

**为什么不把游戏逻辑搬到 Python？** 因为游戏本身已经有完整的渲染、音效、50 个关卡——保持它在浏览器里运行意味着训练过程可以实时可视化，也能随时切换到人类手动玩。训练时关掉渲染，速度也够用。

## 观测空间：233 维向量

没有用图像输入，而是把游戏状态拍平成一个 233 维的 float32 向量：

| 区段 | 内容 | 维度 |
|------|------|------|
| 挡板 | x 坐标, 宽度 | 2 |
| 球 | x, y, vx, vy, 是否已发射 | 5 |
| 砖块网格 | 14×16 的 HP 值矩阵 | 224 |
| 元信息 | 剩余生命比例, 剩余砖块比例 | 2 |

所有值归一化到 [0, 1]。用固定 14×16 网格而不是变长砖块列表，是为了保持输入维度一致——不同关卡的砖块布局不同，但网格大小相同，空位填 0。

## 三个 Agent：DQN、PPO、GRPO

选了三个算法做对比实验，各有特点。

### DQN：经典离散控制

动作空间 4 个离散值：左移、不动、右移、发球。

网络结构就是一个简单的 MLP（233→256→256→128→4），输出四个动作的 Q 值。用 ε-greedy 探索，经验回放缓冲区 10 万条，每 1000 步同步一次 target network。

DQN 的好处是稳定、好调试，坏处是只能做离散控制——挡板只有「全速左/不动/全速右」三个选项，没有中间态。

### PPO：连续控制

PPO 用的是连续动作空间，输出一个 [-1, 1] 的浮点数表示挡板移动方向和力度。这比 DQN 的三档控制精细得多。

用了 Actor-Critic 架构，共享底层特征提取，Actor 输出 tanh 压缩的高斯分布，Critic 估算状态价值。GAE（λ=0.95）做优势估计，clip ratio 0.2。

### GRPO：无 Critic 的策略优化

这是最有意思的一个。GRPO（Group Relative Policy Optimization）来自 DeepSeek-R1 的思路——不要 Critic，而是对同一个起始状态采样 G 条完整轨迹，用组内均值和标准差来归一化优势。

为什么在打砖块上试 GRPO？几个原因：
- 打砖块的单局不长（200-2000 步），采样 G 条完整轨迹是可行的
- 没有 Critic 意味着更少的参数和更简单的代码
- 组内相对基线天然适应奖励尺度的变化，不需要额外做 reward normalization
- 训练早期 agent 很少打到砖块（稀疏奖励），GRPO 在这种情况下理论上比需要精确 value estimation 的 PPO 更鲁棒

## 奖励设计

基础奖励来自游戏本身：

| 事件 | 奖励 |
|------|------|
| 击碎砖块 | +1 |
| 通关 | +50 |
| 丢球 | -5 |

此外提供了可选的 reward shaping（默认关闭，用 `--reward-shaping` 开启）：

- **追球奖励**：挡板越接近球的 X 坐标，给 +0.01 × (1 - \|距离\|)。鼓励 agent 学会移到球下方
- **球向上奖励**：球向上运动时给微小正奖励（+0.001），抑制「让球掉下去」
- **超时惩罚**：连续 600 步没打到砖块就截断，给 -1。防止 agent 学会无限弹跳但不打砖块

Reward shaping 默认关闭是为了对比实验的纯净性——开了 shaping 训练更快，但不代表算法本身更好。

## 训练流程

```bash
# 启动 WebSocket 中继
python ws_server.py

# 开浏览器打开游戏（连接 WS）
npm run dev

# 开始训练（以 DQN 为例）
python train.py --agent dqn --episodes 5000 --headless

# 评估
python evaluate.py --agent dqn --checkpoint checkpoints/dqn_best.pt

# 看 AI 打游戏
python play.py --agent dqn --checkpoint checkpoints/dqn_best.pt
```

`--headless` 会关掉浏览器端的渲染，配合 frame skip=4（每个决策重复 4 帧），训练速度提升不少。日志同时写 TensorBoard 和 W&B。

## 目前的状态和下一步

游戏本体（Phase 1）已经完成：50 关、三种道具（多球/分裂/加命）、多球系统、程序化音效和 BGM、移动端适配。

RL 训练（Phase 2）的代码框架已经搭好，正在调参和跑实验。接下来想做的：
- 三个算法的学习曲线对比
- 开/关 reward shaping 的效果对比
- 看 agent 能打到第几关
- 可能后续加 CNN + 图像输入的版本做更公平的对比

---

项目地址：[breakout.xingyao.info](http://breakout.xingyao.info/)，可以先去玩几把再说。
