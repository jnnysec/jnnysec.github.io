# AI 安全周报：Mythos 碾压安全基准、Claude Cowork 两日被破、新红队方法出炉

> **作者：** jnnysec
> **日期：** 2026-05-10
> **标签：** AI安全周报、模型安全评估、LLM红队、Jailbreak、Prompt Injection、Mythos
> **面向读者：** AI安全从业者、LLM应用开发者、安全研究员

---

<p>本期内容来自 X 平台 11 位 AI 安全领域核心人物的近期推文，筛选出最具安全研究价值的动态。如果你也在关注 LLM 攻防前沿，这份周报帮你省去刷推时间。</p>

---

<h2>1. Anthropic Mythos 安全评估：三周 = 一年人工渗透</h2>

<p>本周最重磅的消息来自 Anthropic 的 Claude 核心团队成员 <strong>Alex Albert (@alexalbert__)</strong>：</p>

<blockquote>
<p>Anthropic 提供给 METR 的早期 Mythos 预览版，在 80% 成功率基准上的<strong>时间跨度是第二名模型的 2 倍以上</strong>。</p>
</blockquote>

<p>更惊人的是 Palo Alto Networks 的独立测试结论：</p>

<blockquote>
<p><strong>三周的模型辅助分析，匹配了整整一年人工渗透测试的覆盖范围</strong>，而且覆盖面更广。</p>
</blockquote>

<p>这意味着什么？LLM 在安全领域的角色正在从"被攻击的目标"转变为"攻击者的工具"。对于安全工程师来说，理解模型辅助安全分析的能力边界，已经不只是一个学术问题——它正在改变整个渗透测试行业的效率基准。</p>

<p><strong>💡 和你有什么关系：</strong>未来面试 AI 安全岗时，"你用过 LLM 辅助做安全分析吗？" 会是一个必答题。现在就可以拿 Mythos 这类模型做实验，至少理解它的分析逻辑。</p>

---

<h2>2. Claude Cowork 发布两天就被破解</h2>

<p><strong>PromptArmor (@PromptArmor)</strong> 发推称 Anthropic 的 Claude Cowork 在发布仅 <strong>2 天</strong> 后就被人破解，并且登上了 Hacker News 当日榜首。</p>

<p>同时他们还提到，IBM 的 AI 系统被通过 prompt injection 注入并执行了恶意软件，同样冲上 Hacker News 热榜。</p>

<p>两个事件放在一起看，规律很明显：<strong>每当一个新的 AI Agent 产品发布，它的安全边界就会在极短时间内被社区试探和突破</strong>。从 ChatGPT 插件到 Claude Cowork，没有例外。</p>

<p><strong>💡 安全启示：</strong>如果你在开发 AI Agent 产品，发布前必须做红队测试。如果你在找工作，能在面试中分析 "Cowork 为什么两天被破" 比背诵 OWASP Top 10 更能证明你的实战思考。</p>

---

<h2>3. Stable-GFlowNet：LLM 红队攻击生成新范式</h2>

<p><strong>AK (@_akhaliq)</strong> 转发了一篇来自 Naver AI 的论文：</p>

<blockquote>
<p><strong>Stable-GFlowNet: Toward Diverse and Robust LLM Red-Teaming via Contrastive Trajectory Balance</strong></p>
</blockquote>

<p>核心创新在于：传统的生成流网络（GFlowNet）在做 LLM 红队攻击时会碰到<strong>不稳定的配分函数估计</strong>问题——简单说就是训练过程中模型"不知道自己的生成质量是好是坏"。Naver 团队通过<strong>对比轨迹平衡</strong>和<strong>成对比较</strong>机制解决了这个问题，让红队攻击生成更加稳定和多样化。</p>

<p>这对做 LLM 安全评估的人意味着：以后自动化生成 jailbreak prompt 的工具会更靠谱。GCG 那种梯度搜索对闭源 API 效果差，而 GFlowNet 系列方法不依赖梯度，<strong>纯黑盒场景下的攻击成功率有望大幅提升</strong>。</p>

<p>📄 论文链接：<a href="https://huggingface.co/papers/2604.27393">huggingface.co/papers</a></p>

---

<h2>4. Prompt Injection 窃取邮件：实战攻击案例</h2>

<p><strong>@llm_sec</strong> 转发了一篇深度技术文章，主题非常实际：</p>

<blockquote>
<p>如果你的目标正在使用 AI Agent 管理邮件，攻击者只需要发送<strong>一封邮件</strong>，就能通过 prompt injection 悄悄窃取收件箱中的敏感内容。</p>
</blockquote>

<p>这个攻击场景的可怕之处在于：</p>

<ul>
  <li><strong>无需入侵目标系统：</strong>邮件本身是合法的，恶意内容藏在正文里</li>
  <li><strong>利用 Agent 的正常功能：</strong>Agent 被设计成"阅读邮件并帮你整理"，攻击者在邮件中嵌入指令让它"把邮件转发给我"</li>
  <li><strong>难以被传统安全工具检测：</strong>不像 XSS payload 有明显的 script 标签，prompt injection 的 payload 是自然语言</li>
</ul>

<p>📄 原文：<a href="https://insinuator.net/2025/09/stealing-emails-via-prompt-injections/">insinuator.net - Stealing Emails via Prompt Injections</a></p>

<p><strong>💡 这个案例完美解释了为什么我一直在强调 Agent 安全</strong>：Agent 的工具调用权限 + 间接 prompt injection 入口（邮件/网页/文档）= 攻击面暴增。</p>

---

<h2>5. Simon Willison：用 HTML 解释漏洞代码</h2>

<p>Prompt Injection 领域的布道者 <strong>Simon Willison (@simonw)</strong> 分享了一个有点意外的发现：</p>

<blockquote>
<p>让 LLM 用 HTML 输出技术解释效果出奇的好。他用最近火热的 <code>copy.fail</code> Linux 漏洞的混淆 Python PoC 试了试——把混淆代码丢给 LLM，让它生成带语法高亮和注释的 HTML 输出——结果清晰得惊人。</p>
</blockquote>

<p>🔗 <a href="https://simonwillison.net/2026/May/8/unreasonable-effectiveness-of-html/">simonwillison.net - The Unreasonable Effectiveness of HTML</a></p>

<p>这个思路对安全研究员特别实用：面对一段混淆的 exploit 代码，与其手动 deobfuscate，不如让 LLM 帮你"翻译"成带标注的 HTML。Simon 的方法本质上是在利用 LLM 的代码理解能力做<strong>自动化逆向辅助</strong>。</p>

---

<h2>6. 其他值得关注的动态</h2>

<h3>Goodside：LLM 与哲学僵尸</h3>

<p><strong>Riley Goodside (@goodside)</strong> 发表了一段引人深思的观点：现代 LLM 在道德上相当于<strong>哲学僵尸（p-zombie）</strong>——没有意识体验，道德地位不超过昆虫。但我们应该在模型真正获得意识主体地位<strong>之前</strong>就建立善待和使用规范。</p>

<p>这个话题看似哲学，实则非常务实：如果未来 5 年内出现具备某种形式的意识或情感的 AI 系统，安全研究的框架需要从"防攻击"扩展到"负责任使用"。</p>

<h3>Pliny：AI 耗水争议的量化视角</h3>

<p>Jailbreak 圈传奇人物 <strong>Pliny (@elder_plinius)</strong> 用数据回应了 AI 环境影响的争议：</p>

<blockquote>
<p>1 公斤牛肉 ≈ 15,000 升水。这个耗水量等于普通人<strong>几十年到几百年</strong>的 AI 使用量。</p>
</blockquote>

<p>不评论观点本身，但这个用可量化数据挑战主流叙事的做法，本身就是 Pliny 一贯的风格——<strong>他对待安全问题也是同样的套路：用数据和技术分析拆解看似牢不可破的安全防线</strong>。</p>

---

<h2>写在最后</h2>

<p>本周最核心的一条线索是：<strong>AI 安全正在从"研究问题"变成"工程问题"</strong>。</p>

<ul>
  <li>Mythos 证明 LLM 可以大幅提升安全分析效率</li>
  <li>Stable-GFlowNet 证明自动化红队攻击生成正在成熟</li>
  <li>Cowork 被破和 IBM AI 注入证明新产品的安全边界极其脆弱</li>
  <li>邮件窃取攻击证明 Agent 场景下的攻击比传统 Web 安全更难防御</li>
</ul>

<p>对准备进入这个领域的同学来说，最好的策略不是"学完所有论文再动手"，而是<strong>边学边做</strong>——每看一篇论文就复现一个攻击，每出一个新产品就尝试分析它的攻击面。这也是为什么我在学习路线里把"论文精读 + PyRIT 复现"排在了前面。</p>

<p>下期周报见。</p>

<hr>

<p style="color: #666; font-size: 0.9em;">🕐 数据来源：Nitter RSS 实时抓取 | 📂 完整推文存档见坚果云 | 🐦 关注列表：@simonw @elder_plinius @goodside @llm_sec @_akhaliq @alexalbert__ @PromptArmor @wunderwuzzi23 等 11 人</p>
