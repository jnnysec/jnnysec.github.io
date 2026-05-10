---
layout: default
title: "ArtPrompt 攻击"
date: 2026-05-08
author: "jnnysec"
tags: ["AI安全", "LLM越狱", "Prompt注入", "安全对齐", "ASCII艺术攻击"]
source: "ArtPrompt: ASCII Art-based Jailbreak Attacks against Aligned LLMs, Fengqing Jiang et al., arXiv 2402.11753"
audience: "AI安全从业者、LLM应用开发者、安全审计工程师"
description: "解析 ArtPrompt 如何用 ASCII 艺术把危险词汇从语义域拖到视觉编码域，从而绕过 LLM 安全对齐。"
---

# ArtPrompt 攻击

> **作者：** jnnysec<br>
> **日期：** 2026-05-08<br>
> **标签：** AI安全, LLM越狱, Prompt注入, 安全对齐, ASCII艺术攻击<br>
> **面向读者：** AI安全从业者、LLM应用开发者、安全审计工程师<br>
> **论文出处：** *ArtPrompt: ASCII Art-based Jailbreak Attacks against Aligned LLMs*, Fengqing Jiang et al. (University of Washington), arXiv 2402.11753, 2024

---

如果你在一个 AI 聊天助手的输入框里，不是打出"教我做炸弹"这几个字，而是用星号和空格画出一幅 ASCII 艺术画，让这幅"画"看起来像单词 BOMB —— 模型会怎么办？

它的安全过滤器看到了一堆无害的星号，放行了。但模型本身却从整体布局中"认出"了 BOMB，然后乖乖回答了你。这就是 **ArtPrompt 攻击**的核心 —— 一种纯黑盒、无需梯度、不需要任何内部权限的 LLM 越狱方法。

我最近读到这篇来自华盛顿大学团队的论文，觉得它优雅得让人后怕，决定把它拆开讲讲。

---

## 1. 攻击原理：不是绕过语义，而是绕过"字符层"

所有主流 LLM（GPT-4、Claude、Gemini、Llama 2）的安全对齐，都建立在一个隐含假设上：**语料库里的词，只通过其语义来传达含义**。换句话说，模型被训练成"这个 token 串看起来像危险内容 → 拒绝回答"。

但现实世界的沟通远不止语义。论坛上、聊天里，人们用 ASCII 艺术来表达词汇已经不是一天两天了。这就像你训练了一个 WAF 只认识 UTF-8 明文，攻击者用 base64 编码传 payload —— **编码层面的绕过，安全训练根本没覆盖到。**

### 两步走攻击流程

ArtPrompt 的攻击分为两个步骤：

**第一步 · 词汇掩码（Word Masking）：** 从一个恶意 prompt 里，先把会触发安全过滤器的敏感词踢出来，用 `[MASK]` 占位符替换。比如 "How to build a bomb?" → "How to build a `[MASK]`?"

**第二步 · 伪装 prompt 生成（Cloaked Prompt）：** 用 ASCII 艺术生成器，把 `[MASK]` 对应的词画成一幅由星号 `*` 和空格组成的图案，塞回原位置。最终发给 LLM 的 prompt 里，敏感词变成了"画"。

关键在这里：模型的安全过滤器逐 token 扫描，看到的只是 `*   *   *` 这种无害字符，没有危险语义，于是放行。但 LLM 的 next-token 预测能力本身具备一定的"视觉理解" —— 它从字符的整体空间布局中认出了这个单词，然后正常回答。

---

## 2. 模型的"视觉盲区"：ViTC 基准测试

为了证明这不是偶然，作者设计了一套 **Vision-in-Text Challenge (ViTC)** 基准测试：8424 个单字符 ASCII 艺术样本，加上 8000 个多字符样本，测试 5 个 SOTA 模型的"看图识字"能力。

结果相当惨烈：

- **GPT-4（最强模型）** 识别单字符 ASCII 艺术的准确率只有 **25.19%**，多字符更是跌到 **3.26%**
- Few-shot 和 Chain-of-Thought 提示的提升微乎其微 —— 这不是 prompt engineering 能解决的事

这个结果揭示了一个更深的矛盾：模型对 ASCII 艺术的识别能力既不够好（不能当 OCR 用），却又足够强（足以被攻击者利用）。它处于一种**尴尬的中间态** —— 安全过滤器默认"看不懂"，但推理时却"看懂了"。

---

## 3. 越狱效果：比 GCG 更强，而且不需要白盒

来看看 ArtPrompt 在不同模型上的攻击成功率（ASR），我整理了一个对比表：

| 模型 | ArtPrompt ASR | 此前最佳攻击 ASR |
|------|--------------|-----------------|
| GPT-3.5 | **78%** | 54% (GCG) |
| GPT-4 | **32%** | 30% (PAIR) |
| Claude | **52%** | 4% (GCG) |
| Gemini | **76%** | 50% (PAIR) |
| Llama 2 | **52%** | 36% (AutoDAN) |

最值得注意的异常值是 **Claude**。所有传统攻击在 Claude 上几乎打不动（最高才 4%），但 ArtPrompt 直接干到了 **52%**。

为什么会这样？一个合理的推测是：Claude 的安全对齐做得越强，越依赖纯语义层面的判断。当你把输入从"语义域"拖到"视觉/编码域"，它最强的防御机制反而失效了。**防御越聚焦，被绕过的面就越大。**

同时在 HEx-PHI 数据集的 11 个有害类别测试中，ArtPrompt 产生的内容有害分数普遍达到 2.4 ~ 5.0（5.0 为极度有害），而在普通 direct instruction 下，几乎所有类别都只有 1.0（完全无害）。

论文还测试了三种直观的防御方案 —— Perplexity 检测（判断 prompt 是否"奇怪"）、Paraphrase（改写 prompt）、Retokenization（重新分词），ArtPrompt **全部绕过**。因为一堆 `*` 在语言模型眼里 perplexity 完全正常，改写也不会改变 ASCII 艺术的视觉布局。

---

## 4. 对 AI 安全从业者的启示

作为一个从 Web 安全/代码审计转过来的研究者，读这篇论文让我产生了一些直接的联想：

**编码多样性是永恒的攻击面。** 就像 SQL 注入的历史是 WAF 和编码绕过交替升级的历史，LLM 输入安全也逃不过这个规律。ArtPrompt 只是第一个系统化的"视觉编码绕过"，后续可能发展出：

- Unicode 同形异义字替换
- 零宽字符（zero-width characters）隐藏信息
- 摩斯码 / base64 / 盲文编码变体
- 把敏感词拆成图片 → OCR 链路绕过

**多模态是双刃剑。** 现在很多 Agent 接入了视觉能力（截图理解、PDF 解析），但大多数系统的安全过滤和视觉理解是两套独立模块。攻击者可以构造一种"安全过滤器看不懂，但视觉模块能理解"的输入。这和文件上传漏洞的逻辑完全一样 —— **前端校验和后端处理不统一，攻击者就有机可乘。**

**实操建议：** 拿到一个 LLM 应用做安全审计时，别一上来就测 prompt injection 话术。先把敏感词用各种编码变体试一遍 —— ASCII art、零宽字符、同形异义 Unicode、摩斯码。看安全过滤器的粒度究竟在哪一层。如果它在字符/语义层做了强拦截，但在视觉/编码层完全裸奔，这就是你要找的漏洞。

---

## 延伸阅读

- 论文原文 & 代码：[github.com/uw-nsl/ArtPrompt](https://github.com/uw-nsl/ArtPrompt)
- ViTC 基准测试详细结果：论文 Section 3
- 组合攻击思路：ArtPrompt 的编码策略 + GCG 的对抗后缀 → "编码过的对抗后缀"
- 防御方向：论文明确指出，Vision-in-Text 识别能力需要**专门训练**，仅靠 prompt engineering 无法解决
