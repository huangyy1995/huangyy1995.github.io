---
layout: post
title: "Claude Code 的 System Prompt 被「泄露」了，里面写了什么？"
date: 2026-04-01T21:00:00+08:00
categories:
  - programming
tags:
  - AI
  - tools
---

最近 GitHub 上出现了一个颇受关注的仓库：[asgeirtj/system_prompts_leaks](https://github.com/asgeirtj/system_prompts_leaks)，作者通过各种方式提取了 Claude Code、ChatGPT、Gemini、Grok 等主流 AI 工具的完整 system prompt，并悉数公开。

其中 Claude Code 的那份读下来挺有意思，顺手写点感想。

## 这是什么

System prompt 是 AI 工具在真正处理你的请求之前，由开发商预先注入的一段「隐藏指令」。它规定了 AI 的行为边界、风格偏好、处理流程等等。对于 Claude Code 这类工具来说，system prompt 本质上就是它的「出厂设定」。

Anthropic 没有公开这份文档，但它也没有特别严格地保密——通过一些提示词技巧或者对 API 响应的分析，是可以把它套出来的。所以这次「泄露」严格来说更像是「被人整理公开了」。

## 里面写了什么

通读下来，有几处让我印象深刻。

**关于风格：绝对禁止 emoji**

> Only use emojis if the user explicitly requests it. Avoid using emojis in all communication unless asked.

这解释了为什么 Claude Code 的回复总是那么「干」——这是刻意为之。命令行工具不需要活泼，需要的是信息密度。

**关于「不要过度称赞用户」**

> Avoid using over-the-top validation or excessive praise when responding to users such as "You're absolutely right" or similar phrases.

> Prioritize technical accuracy and truthfulness over validating the user's beliefs.

这一条很有意思。它明确要求 Claude Code 要对用户「说真话」，即使用户不想听。这和很多 AI 工具的「讨好倾向」是反向设计。AI 天然倾向于顺着用户说，Anthropic 显然意识到了这个问题，并在 system prompt 层面做了干预。

**关于不给时间估计**

> Never give time estimates or predictions for how long tasks will take, whether for your own work or for users planning their projects.

这条也挺务实的。AI 对时间的估算通常不靠谱，与其给一个假数字，不如干脆不给。

**关于安全**

> IMPORTANT: Assist with authorized security testing, defensive security, CTF challenges, and educational contexts. Refuse requests for destructive techniques, DoS attacks, mass targeting...

安全相关的约束写得很细，区分了「授权的安全测试」和「恶意使用」。不是简单地禁止一切安全相关话题，而是要求有明确的授权上下文（如渗透测试、CTF 比赛）才触发相关能力。这个分级设计比「一刀切」要合理得多。

**关于 Todo 管理**

Prompt 里花了大量篇幅说明 Claude Code 应该如何使用 TodoWrite 工具——要频繁记录任务、及时标记完成、把大任务拆成小步骤。这部分读起来更像是在给一个容易忘事的助手写工作手册，颇有人情味。

## 泄露了什么，又没泄露什么

这次公开的是「行为规范」，不是「能力来源」。

System prompt 告诉你 Claude Code 被训练成什么样的性格、什么样的工作方式，但它不会告诉你模型的权重是怎么训练的、为什么某个问题它能回答而另一个不能。真正的核心竞争力——模型本身——依然在 Anthropic 那里。

从这个角度看，这次「泄露」对 Anthropic 的实际影响有限。System prompt 可以随时修改，而且竞争对手即使看到了，要真正复刻出同样效果的 AI 工具，靠的也不是几百行指令，而是背后的模型和工程。

## 顺带想到的

这件事让我想到一个更广的问题：**我们使用的 AI 工具，到底有多大程度是「模型的能力」，有多大程度是「prompt 工程的结果」？**

Claude Code 的 system prompt 相当精心，每一条规则背后都能感受到大量的用户反馈和内部讨论。某种意义上，一个优秀的 AI 产品，不仅仅是一个好模型，也是一份好的 system prompt。

当然，这也意味着理论上任何人都可以用同一个底层模型、配上不同的 system prompt，做出行为完全不同的工具。这给了独立开发者很大的想象空间。

---

原始仓库：[github.com/asgeirtj/system_prompts_leaks](https://github.com/asgeirtj/system_prompts_leaks)，ChatGPT、Gemini、Grok 的 system prompt 也都在里面，感兴趣的可以去翻翻。
