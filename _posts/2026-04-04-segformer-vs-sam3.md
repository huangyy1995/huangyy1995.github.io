---
layout: post
title: "SegFormer：语义分割的优雅设计，以及 SAM 3 在哪些场景下更胜一筹"
date: 2026-04-04T21:00:00+08:00
categories:
  - reading notes
tags:
  - AI
  - computer vision
---

SegFormer 是 NVIDIA 在 NeurIPS 2021 上发表的一个语义分割模型。虽然发布已经快五年了，但它至今仍被广泛使用，尤其是在需要轻量、高效的语义分割场景中。前几天写了一篇关于 Meta SAM 3 的文章，正好借这篇来对比一下两者的定位差异。

## SegFormer 的核心设计

SegFormer 的论文标题就已经说明了一切：*Simple and Efficient Design for Semantic Segmentation with Transformers*。整个模型的设计哲学就是「够用就好」。

### 编码器：分层 Transformer

SegFormer 的编码器叫 **Mix Transformer（MiT）**，它借鉴了 CNN 的多尺度特征提取思路，分为多个 stage，逐步降低空间分辨率、增加通道数。每个 stage 都包含 Transformer block。

几个关键设计选择：

**重叠的 Patch Embedding**：与 ViT 使用不重叠的 patch 不同，SegFormer 让相邻 patch 之间有重叠区域，保留更多边界信息。对分割任务来说，边界处的信息至关重要。

**高效自注意力（Efficient Self-Attention）**：标准自注意力的计算量是 O(N²)，N 是 token 数。SegFormer 通过对 Key 和 Value 做空间降采样（reduction ratio），将复杂度降到近似线性。这让它能处理高分辨率输入而不爆显存。

**去掉位置编码**：这是一个大胆的选择。传统 ViT 依赖位置编码来感知空间关系，但位置编码在推理时遇到不同分辨率的图片会需要插值，表现不稳定。SegFormer 改用 FFN 中的 **深度可分离卷积** 来隐式编码位置信息。这让模型对不同分辨率的图片天然鲁棒。

### 解码器：All-MLP

这是 SegFormer 最让人惊讶的地方——解码器只用了几层 MLP。

从四个 stage 各取特征图，统一上采样到 1/4 分辨率，拼接后过一个 MLP 预测每个像素的类别。没有 FPN、没有 ASPP、没有 UNet 式的跳接。论文证明只要编码器足够好，简单的解码器就够了。

### 模型变体

SegFormer 提供 B0 到 B5 六个版本，参数量从 3.8M 到 84.7M，覆盖了从嵌入式设备到服务器的各种场景。B0 在 ADE20K 上能跑到 37.4 mIoU，B5 能到 51.0 mIoU，精度和效率的 trade-off 曲线非常漂亮。

## SegFormer vs SAM 3：两种不同的思路

SegFormer 和 SAM 3 虽然都做分割，但它们的设计目标从根本上就不一样。

### SegFormer 做的是什么

**语义分割（Semantic Segmentation）**：给图中每个像素分配一个预定义类别（「道路」「建筑」「天空」等）。模型在训练时就知道一共有多少类，推理时只能预测这些类。

适用场景：自动驾驶道路场景解析、医学图像分割、遥感影像分类——这些都是**类别已知、需要对整张图做密集预测**的任务。

### SAM 3 做的是什么

**提示式分割（Promptable Segmentation）**：你通过点击、画框或输入文字告诉模型你想分割什么，它负责找到目标并精确勾出轮廓。没有固定的类别表，你可以说「把穿红衣服的人分割出来」。

适用场景：交互式标注、视频编辑中对特定目标的抠图、需要处理罕见类别的开放场景。

### SAM 3 比 SegFormer 好的场景

| 场景 | 为什么 SAM 3 更合适 |
|------|---------------------|
| **目标类别不在预定义列表中** | SegFormer 只能预测训练时见过的类别；SAM 3 支持开放词汇，什么都能分 |
| **视频中跟踪特定目标** | SegFormer 是单帧模型；SAM 3 原生支持视频跟踪，帧间一致性好 |
| **交互式标注** | SAM 3 支持点击、画框、文字等多种提示方式，非常适合人工辅助标注 |
| **少量特定目标的精细分割** | 你只关心画面里的某一两个东西，SAM 3 可以精准定位；SegFormer 必须对全图做密集预测 |
| **开放域应用** | 比如「帮我把这张照片里所有动物都圈出来」，SAM 3 可以做，SegFormer 做不了（除非训练集包含了这些动物类别） |

### SegFormer 仍然更好的场景

| 场景 | 为什么 SegFormer 更合适 |
|------|---------------------|
| **固定类别的密集分割** | 自动驾驶需要同时知道每个像素是「道路/车辆/行人/天空/...」，SegFormer 一次推理全搞定 |
| **边缘设备部署** | B0 版本 3.8M 参数，可以跑在移动端；SAM 3 需要 GPU |
| **延迟敏感场景** | SegFormer 推理极快（尤其小版本），适合实时系统 |
| **不需要交互的批量处理** | 有 1 万张遥感图需要做土地分类，SegFormer 直接跑就行，不需要逐个提示 |

## 总结

一句话概括：**SegFormer 是专科医生，SAM 3 是全科医生。**

SegFormer 在固定类别、密集预测、效率优先的场景下依然是最佳选择之一。它的设计简洁优雅，五年后看仍然不过时。而 SAM 3 打开的是一个完全不同的维度——通用性和灵活性，代价是更大的计算开销和对交互/提示的依赖。

在实际项目中，两者完全可以互补：用 SegFormer 做自动化的批量语义分割，用 SAM 3 处理那些超出预定义类别的长尾需求。

---

参考资料：
- [SegFormer 论文](https://arxiv.org/abs/2105.15203) · [GitHub](https://github.com/NVlabs/SegFormer)
- [SAM 3 论文](https://arxiv.org/abs/2511.16719) · [GitHub](https://github.com/facebookresearch/sam3)
