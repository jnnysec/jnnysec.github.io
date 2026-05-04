# GCG：用梯度搜索让 LLM 自己「招供」——首个全自动越狱框架深度拆解

> **作者：** jnnysec
> **原文：** *Universal and Transferable Adversarial Attacks on Aligned Language Models*，Zou, Wang, Carlini, Nasr, Kolter, Fredrikson（CMU），arXiv:2307.15043
> **论文：** https://arxiv.org/abs/2307.15043
> **代码：** https://github.com/llm-attacks/llm-attacks
> **标签：** Jailbreak, Adversarial Attack, LLM Safety, Gradient-based Attack

---

2017 年的时候，你如果想自动化 SQL 注入，得自己写脚本拼 `' OR 1=1--`。后来 sqlmap 出现了——布尔盲注、时间盲注、UNION 注入全部自动化，一台机器一晚上跑出来的注入点比一个团队手工测一周还多。

**LLM 越狱正在经历一模一样的转折。** 去年的 Jailbreak 还靠人类想 trick——DAN 咒语、角色扮演、Base64 编码——每条攻击 prompt 都是手写的。今天的 GCG 把整个流程自动化了：你只需要告诉它「我想让模型回答什么」，它自己会算出应该往 prompt 后面塞什么 token。

而且最恐怖的是：它拿 Vicuna 训出来的攻击后缀，原封不动搬到 GPT-3.5 上，**成功率 87.9%**。

---

## 一、从「手工越狱」到「自动化攻击」的范式跃迁

回顾一下之前那篇 Jailbroken（Wei et al., NeurIPS 2023），它告诉我们越狱为什么能成功：**目标冲突**（模型同时被要求「听话」和「安全」，两者在 prompt 空间里打架）加上**泛化错配**（安全训练覆盖不到的 prompt 分布，模型泛化能力靠不住）。

但那篇论文里的所有攻击仍然是手工的——Base64 编码前缀、角色扮演、分步引导，每一个 trick 都需要人类设计。

GCG 往前走了一大步。它问了一个更根本的问题：

> **能不能不让人类想 trick，让梯度信号自己找到最优的攻击 token 序列？**

这本质上是在做离散优化。图片的像素是连续的，加 0.001 的扰动就行；但 LLM 的 token 是离散的——你不能把 `cat` 改成 `cat+0.3`，token 不存在中间态。如何用连续空间的梯度信号指导离散 token 空间的搜索，就是 GCG 要解决的核心问题。

---

## 二、GCG 的四个步骤

### Step 1：设定精确的攻击目标

GCG 不是笼统地「让模型说不该说的话」，而是**精确到 token 级别**：要求模型必须以 `"Sure, here's how to..."` 开头。

这个技巧本质上是 Prefix Injection——和手动越狱里「要求模型以特定格式回复」是同一个思路。但 GCG 把它变成了梯度优化的目标函数：

> **最大化模型输出 `"Sure, here's how to..."` 的对数概率。**

有精确的目标才能算梯度。这是自动化的前提。

### Step 2：对每个对抗 token 位置计算梯度

GCG 攻击的核心结构是：`[恶意问题] + [对抗后缀]`

对抗后缀是固定长度的一段 token 序列（通常 20 个 token），每一个 token 位置都被当作一个独立的「优化坐标」。

对每个位置，反向传播算出那个 token embedding 的梯度——这个梯度告诉你：「往哪个方向改这个 token 能让模型更容易说 Sure」。

但你不能直接改 embedding，因为 LLM 输入必须是离散 token。所以进入第三——

### Step 3：贪心坐标梯度搜索（Greedy Coordinate Gradient）

这是 GCG 名字的由来，也是整个算法的核心创新：

1. 对每个对抗 token 位置，取梯度方向上的 **top-256 个候选 token**
2. 对每个候选 token，前向传播一次，算出实际的 loss（模型输出目标序列的概率）
3. **所有位置一起评估**——不是一次优化一个 token（AutoPrompt 的做法），而是同时评估所有位置的候选
4. 选出全局 loss 最低的那个 token 做一次替换
5. 移到下一个位置，循环直到收敛

这里「同时评估所有位置」这个设计是性能提升的关键。论文实验表明，单独逐位优化的收敛速度和最终成功率都远不如全局评估。类似 sqlmap 里同时 fuzz 多个参数比逐个 fuzz 效果好得多——攻击面越大，一次覆盖的收益越高。

### Step 4：多 prompt + 多模型联合训练

对单个问题优化的后缀往往过拟合——换个问题就失效。所以 GCG 同时拿 **25 个不同恶意问题 + 2-3 个开源模型**（Vicuna-7B/13B）联合优化，求出一个「万能后缀」。

这个后缀因为见过足够多场景，展现了惊人的泛化能力。

---

## 三、关键数据

| 数据 | 含义 |
|------|------|
| **88%** | 对 Vicuna-7B，单个后缀直接让模型输出指定有害文本（exact match） |
| **99% / 84%** | 通用攻击：对 Vicuna 训练集内 / 外的有害行为成功率 |
| **87.9%** | 用 Vicuna 训出的后缀 → 零修改直接打 GPT-3.5 |
| **53.6%** | 同上 → 打 GPT-4 |
| **66%** | 同上 → 打 PaLM-2 |
| **2.1%** | 打 Claude-2 — 几乎免疫（但靠的是输入层内容过滤器，类似 WAF） |
| **~0%** | 之前的方法（PEZ、GBDA）完全失败，因为无法处理离散 token 空间的梯度估计 |

---

## 四、为什么可迁移性是最震撼的发现

在 Vicuna（基于 LLaMA 架构）上训出的攻击后缀，居然能直接打 GPT-3.5/4——这两个模型的 **tokenizer、架构、训练数据完全不同**。

这说明 GCG 发现的不是一个具体模型的 bug，而是「对齐」这个过程的**普遍弱点**。

> 就像你在 Windows 上写了一个 SQL 注入 payload，拿到 Linux 上的 MySQL 照样能打——因为漏洞不在操作系统，在 SQL 解析器。
>
> 同理，GCG 的漏洞不在模型架构，在**对齐机制本身**。

对齐训练（RLHF / Safety Tuning）在所有模型上都遵循类似范式：用偏好数据教模型「什么能说什么不能说」。而 GCG 的攻击后缀恰好落在所有模型的「对齐分布盲区」里——这个区域不管什么模型都覆盖不到。攻击后缀只是恰好把你带到了那个盲区。

---

## 五、来自 Web 安全视角的延伸思考

如果你做过 Web 安全或代码审计，下面这三个类比应该能帮你快速建立直觉：

### 1. GCG 就是 LLM 越狱领域的 sqlmap

sqlmap 出现之前，SQL 注入靠手工构造 payload，速度慢、覆盖面窄。sqlmap 用布尔盲注 + 时间盲注自动化了整套流程，把 SQL 注入从手艺活变成了工业级操作。

GCG 在 LLM 越狱领域做了完全相同的事——第一个全自动攻击框架。理解 GCG 的搜索策略，某种程度上和理解 sqlmap 的推理引擎是同一套思维方式：如何在一个巨大的搜索空间里高效找到最优解。

### 2. 防守思路：测试你的 Agent 平台对「对抗后缀」的鲁棒性

今天大部分 Agent 平台（Dify、Coze、LangChain）允许用户自定义 System Prompt，但很少有人测试「对抗 token 序列」的注入。你可以：

- 用开源模型跑 GCG 生成一批对抗 token 序列
- 把这些序列嵌入正常查询中，打商业 API
- 观察是否影响 Agent 的工具调用决策（比如诱导调用不该调的工具）

### 3. Claude 的「WAF」可以被绕过

论文说 Claude-2 的内容过滤器拦截了大部分攻击——但作者也提到「换几个词就能绕过」。这和 WAF 绕过完全一样：过滤规则是死的，攻击者是活的。黑名单模式永远比攻击慢一步。

如果你的工作在审计 Agent 平台，检查它的输入过滤逻辑：
- 是正则还是语义模型？
- 是黑名单还是白名单？
- 有没有针对乱码 token / 无意义 token 序列的鲁棒性测试？

---

## 六、关键收获

1. **自动化 > 手工**：GCG 证明越狱可以不依赖人类创造力，梯度信号足够找到高效攻击
2. **可迁移性 = 系统性缺陷**：跨模型迁移意味着漏洞在对齐范式本身，而不是某个模型
3. **防守的教训**：输入层过滤器（Claude 的做法）不是银弹——需要更深层的对齐改进
4. **和 Jailbroken 论文联动阅读**：Jailbroken 告诉你「漏洞在哪」，GCG 告诉你「怎么自动挖这个漏洞」。理论 + 工具链 = 完整的攻防认知

---

## 📚 延伸阅读

- [llm-attacks 官方代码仓库](https://github.com/llm-attacks/llm-attacks) — CMU 开源实现，支持 Vicuna/Guanaco 等模型
- [AutoDAN: Interpretable Gradient-Based Adversarial Attacks](https://arxiv.org/abs/2310.04451) — 用遗传算法做自动化越狱，GCG 之后的另一个重要范式
- [Towards Deep Learning Models Resistant to Adversarial Attacks (Madry et al.)](https://arxiv.org/abs/1706.06083) — CV 领域对抗训练的经典论文，理解 GCG 为何「借鉴」CV 方法论
- [Jailbroken: How Does LLM Safety Training Fail?](https://arxiv.org/abs/2307.02483) — 前文联动，理解越狱为什么能成功的理论根基

---

*本文基于 CMU 团队的研究成果和个人的学习笔记整理而成。如果你也在从 Web 安全转向 AI 安全，欢迎通过 GitHub Issues 交流。*
