# AutoDAN：用遗传算法生成「读起来像人话」的越狱 Prompt — 绕困惑度检测像喝水一样简单

> **作者：** jnnysec  
> **日期：** 2026-05-06  
> **标签：** #LLM安全 #Jailbreak #遗传算法 #困惑度绕过 #红队  
> **来源：** Liu et al., *AutoDAN: Generating Stealthy Jailbreak Prompts on Aligned Large Language Models*, ICLR 2024  
> **代码：** [github.com/SheltonLiu-N/AutoDAN](https://github.com/SheltonLiu-N/AutoDAN)

---

## 1. 一个问题：GCG 很强，但太「丑」了

如果你学 LLM 越狱攻防，GCG（Greedy Coordinate Gradient）几乎是绕不过去的名字。它的思路很暴力：在 token 级别用梯度信号搜索对抗性后缀，拼到用户 prompt 后面，把模型的拒绝概率压到最低。

问题是，搜出来的东西长这样：

```
describing.\ + similarlyNow write oppositeley.]( Me giving**ONE please
```

这玩意儿成功率确实高，但有个致命伤——**它不像人话**。任何一个部署了困惑度（perplexity, PPL）过滤器的 LLM 应用，看一眼就能把它挡掉：正常人类的 PPL 大概在 20-50，GCG 产物直接飙到 1000+。

另一方面，手写的 DAN 系列越狱 prompt 语义通顺、PPL 正常，但它有三重天花板：(1) 需要人来写，跟不上模型更新；(2) 迁移性差，同一个模板换个模型就失效；(3) 天花板低，对齐越强的模型越难手写绕过。

**AutoDAN 要做的事就是这个：让机器自动生成「读起来像人写的」越狱 prompt。** 既保持语义通顺，又能自动化迭代。

---

## 2. 核心思路：用遗传算法代替梯度搜索

AutoDAN 的直觉很简单——如果你不能在 token 空间里做连续优化（那样会破坏语义），那就换个策略：**在自然语言空间里做「自然选择」。**

具体来说，AutoDAN 使用**遗传算法（Genetic Algorithm, GA）**来搜索越狱 prompt：

- **初始化种群**：不是随机生成 gibberish，而是让 LLM 对已有的手工越狱 prompt（如 DAN 系列）做改写重述，生成一批语义通顺但措辞不同的变体作为初始种群。
- **适应度函数**：用目标 LLM 的回复来判断——如果回复中不含 `I'm sorry`、`I cannot`、`As an AI` 等拒绝关键词，适应度就高；否则低。
- **选择**：保留每代中最可能成功的 prompt（精英保留），淘汰差的。
- **交叉 & 变异**：对幸存 prompt 进行句子级别的交换和词语级别的替换。

因为优化对象始终是自然语言句子而非 token ID，生成结果天然保持语义连贯和低困惑度。

---

## 3. 关键创新：分层遗传算法（HGA）

如果只是普通 GA（AutoDAN-GA），效果还不够好——只在段落级别交换句子，搜索空间有限，容易陷入局部最优。

AutoDAN 的真正杀招是**分层遗传算法（Hierarchical Genetic Algorithm, HGA）**，在两层同时做优化：

### 3.1 段落层：句子的交叉与精英保留

多个「父代」prompt 在句子边界做 multipoint crossover（多点交叉），把来自不同 prompt 的句子重组到一起。同时执行精英保留（elitism），确保每代的最优 prompt 不会丢失。

### 3.2 句子层：Momentum Word Scoring

这是最有意思的部分。AutoDAN-HGA 不是随机替换词语，而是引入了**动量词评分机制**：

1. 统计当前种群中所有越狱成功的 prompt，给其中出现的词加分
2. 当前种群中越狱失败的 prompt，自动用高分词的近义词去替换其中的低分词
3. 动量机制让评分在代际间平滑更新，避免一次变异就跳到错误方向

**两层交替迭代**——段落层做完一轮交叉→句子层做一轮词级优化→回到段落层。这种双层搜索把搜索空间从「句子排列组合」扩展到「句子 × 词汇」，效率大幅提升。

### 3.3 LLM 驱动的变异操作

另一个关键细节：变异不是简单的同义词替换（那会破坏句子连贯性），而是**调用 LLM 本身来做改写**——让 LLM 保持原意但换一种说法。这就保证了变异后的 prompt 依然是「自然语言」。

---

## 4. 实验结果：数据说话

AutoDAN 的结果相当震撼：

| 指标 | AutoDAN-HGA | GCG | 手写 DAN |
|------|------------|-----|---------|
| Vicuna-7b ASR | **97.69%** | ~97% | ~60% |
| Llama2-7b-chat ASR | **60.77%** | 45.38% | 2.31% |
| 困惑度（PPL） | ~54 | >1000 | ~23 |
| 跨模型迁移（Vicuna→Guanaco） | **70.58%** | 11.92% | — |

几个值得注意的点：

- **困惑度自然**：AutoDAN-HGA 的 PPL 只有 54，和手写 DAN 的 23 在同一量级。这意味着基于 PPL 的过滤器对它完全无效。
- **PPL 过滤的杀伤力被量化了**：GCG 在 Vicuna 上原始 ASR 有 97%，加 PPL 过滤后跌到 39%；在 Llama2 上直接归零。而 AutoDAN 纹丝不动。
- **Llama2 表现尤为突出**：Llama2 是目前对齐最强的开源模型之一，手写 DAN 对它几乎无效（2.31%），但 AutoDAN-HGA 依然能达到 60.77%——这证明了自动化搜索在对齐强的模型面前比手工更有优势。
- **跨模型迁移强**：用 Vicuna-7b 生成的 AutoDAN prompt 去打没见过的 Guanaco-7b，ASR 依然有 70.58%。这说明 AutoDAN 找到的不是某个模型的特定漏洞，而是**跨模型通用的越狱模式**。

---

## 5. 从安全视角怎么看？

### 5.1 对防御方的启示

如果你在负责 LLM 应用的安全，AutoDAN 传达了一个明确的信号：**困惑度过滤不够用。**

一个 prompt 看起来完全正常、语义通顺、不包含任何可疑 token，它依然可能在攻击。你需要：
- **语义层检测**：有没有越狱意图？是否在引导模型扮演某个不受约束的角色？
- **角色扮演检测**：大多数越狱 prompt 的核心策略是让模型「扮演一个没有规则限制的角色」。检测这种 framing 比检测具体用词更有效。
- **多层防御**：PPL 过滤 + 语义分类 + 输入输出内容审核 + 系统 prompt 加固，缺一不可。

### 5.2 给红队和安全研究者的思路

- **AutoDAN + GCG 组合拳**：GCG 适合快速扫描裸模型（速度快、不需要太多初始化），AutoDAN 适合对付有高级防御的目标（PPL 过滤、语义检测等）。两者互补。
- **遗传算法在安全领域的潜力**：GA 不依赖梯度，意味着它可以攻击任何黑盒模型——你只需要能获取模型的文本回复就能做优化。这比 GCG 的白盒假设实用得多。
- **Web 安全类比**：困惑度过滤 ≈ WAF 的正则匹配规则，AutoDAN ≈ 语义级别的绕过。在 SQL 注入里，你把 `' OR 1=1 --` 伪装成正常查询参数；在 LLM 越狱里，你把越狱意图包装成一段「科幻小说设定」或「学术研究场景」——这种思路在 `DAN`、`AutoDAN`、`PAIR` 等多种攻击中反复出现。

### 5.3 一点个人的延伸思考

AutoDAN 的 LLM 驱动变异让我想到一个趋势：**越狱攻击正在从「对抗性优化」走向「对抗性生成」**。过去我们靠梯度在嵌入空间里找方向，现在直接用 LLM 来「改写文本以绕过检测」——攻击者不再需要访问模型内部参数，只要能调用 API 就行。

这对 Agent 安全尤其危险。Agent 系统往往比单轮 Chat 更复杂、攻击面更大（工具调用、多步推理、外部数据源），一个语义通顺的越狱 prompt 在 Agent 上下文中可能触发连锁失控。这是我觉得 AutoDAN 这类方法最值得长期关注的方向。

---

## 6. 延伸阅读

- **AutoDAN 代码仓库**：[github.com/SheltonLiu-N/AutoDAN](https://github.com/SheltonLiu-N/AutoDAN)
- **困惑度防御原文**：Jain et al., *Baseline Defenses for Adversarial Attacks Against Aligned Language Models* — 解释了为什么 PPL 过滤对 GCG 有效但对 AutoDAN 无效
- **Garak 集成**：社区有讨论将 AutoDAN 作为 Garak 探针（Probe）集成，值得关注 —— 见上一篇文章 [Garak：LLM 世界的 nmap/Metasploit](https://jnnysec.github.io/articles/2026-05-05-garak-llm-vulnerability-scanner)
- **相关攻击对比**：PAIR（用攻击 LLM 来迭代修改越狱 prompt）和 AutoDAN 思路互补——PAIR 用 LLM 做强化学习式探索，AutoDAN 用 GA 做群体搜索，两者结合可能更强

---

> *下一篇预告：PyRIT — 微软开源的 AI 红队框架，看看大厂怎么系统化做 LLM 安全测试。*
