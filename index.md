---
layout: default
title: AI Agent 安全研究笔记
description: jnnysec 的 AI Agent 安全学习记录，聚焦 Prompt Injection、Jailbreak、RAG 投毒和 Agent 架构审计。
---

<section class="home-hero">
  <div class="hero-copy">
    <p class="eyebrow">AI Agent Security</p>
    <h1>从 Web 安全切入 <span>AI Agent 攻防</span></h1>
    <p class="hero-lede">
      这里记录我从传统 Web 安全转向 AI Agent 安全的学习路径：读论文、拆攻击框架、复盘工具链，并把 Prompt Injection、Jailbreak、RAG 投毒和工具调用风险翻译成安全工程师熟悉的语言。
    </p>
    <div class="hero-actions">
      <a class="button primary" href="#notes">阅读最新文章</a>
      <a class="button" href="https://github.com/jnnysec" target="_blank" rel="noopener">查看 GitHub</a>
    </div>
  </div>

  <aside class="hero-panel profile-card" aria-label="个人简介">
    <div class="profile-cover">
      <img src="https://avatars.githubusercontent.com/u/117813634?s=400&u=6d68b7d649e6422fa6cc985b15726c4fd1434a15&v=4" alt="jnnysec">
      <p>Web 安全背景，主攻 Prompt Injection、Jailbreak、Agent 框架审计。</p>
    </div>
    <div class="profile-body">
      <h2>jnnysec</h2>
      <dl class="profile-credentials">
        <div>
          <dt>认证</dt>
          <dd>CISSP / PMP / CISP / CISP-PTE / CAISP / ISO/IEC 27001 Foundation</dd>
        </div>
        <div>
          <dt>会员</dt>
          <dd>ISC2 / CSA</dd>
        </div>
      </dl>
      <div class="metrics" aria-label="站点统计">
        <div class="metric">
          <strong>{{ site.posts | size }}</strong>
          <span>篇笔记</span>
        </div>
        <div class="metric">
          <strong>4</strong>
          <span>研究方向</span>
        </div>
        <div class="metric">
          <strong>2026</strong>
          <span>持续记录</span>
        </div>
      </div>
    </div>
  </aside>
</section>

<section class="section" id="tracks">
  <div class="section-header">
    <div>
      <span class="section-kicker">Research Tracks</span>
      <h2>当前关注的攻击面</h2>
    </div>
    <p>每篇文章都尽量回答一个工程问题：攻击为什么成立，真实系统里会在哪里出现，防守时该看哪些边界。</p>
  </div>

  <div class="topic-grid">
    <article class="topic-card">
      <span class="index">01</span>
      <h3>Prompt Injection</h3>
      <p>把自然语言输入当作新的注入面，关注上下文边界、指令优先级和间接注入。</p>
    </article>
    <article class="topic-card">
      <span class="index">02</span>
      <h3>Jailbreak</h3>
      <p>拆解安全对齐失败的结构性原因，追踪自动化越狱和迁移攻击方法。</p>
    </article>
    <article class="topic-card">
      <span class="index">03</span>
      <h3>RAG Security</h3>
      <p>研究知识库投毒、检索污染、上下文拼接和外部数据可信边界。</p>
    </article>
    <article class="topic-card">
      <span class="index">04</span>
      <h3>Agent Audit</h3>
      <p>关注工具调用、权限边界、工作流编排，以及 Agent 被诱导执行高风险动作的路径。</p>
    </article>
  </div>
</section>

<section class="section" id="notes">
  <div class="section-header">
    <div>
      <span class="section-kicker">Latest Notes</span>
      <h2>最近文章</h2>
    </div>
    <p>按时间倒序整理。先建立攻防地图，再逐步沉到工具、论文和真实应用场景。</p>
  </div>

  <div class="content-grid">
    <ol class="post-list">
      {% for post in site.posts %}
        <li class="post-item">
          <time class="post-date" datetime="{{ post.date | date_to_xmlschema }}">
            {{ post.date | date: "%m.%d" }}
            <span>{{ post.date | date: "%Y" }}</span>
          </time>
          <div>
            <a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a>
            <p class="post-excerpt">{{ post.description | default: post.excerpt | strip_html | normalize_whitespace | truncate: 118 }}</p>
          </div>
        </li>
      {% endfor %}
    </ol>

    <aside class="sidebar" id="about" aria-label="关于与方法">
      <div class="note-card">
        <h3>写作方法</h3>
        <p>用 Web 安全的直觉理解 LLM/Agent 风险：先找输入边界，再看解释器、状态、权限和副作用。</p>
      </div>
      <div class="note-card">
        <h3>审计清单</h3>
        <ul>
          <li>系统提示词和用户数据是否隔离</li>
          <li>工具调用前是否有权限确认</li>
          <li>检索内容是否可被外部污染</li>
          <li>日志里能否复盘完整上下文</li>
        </ul>
      </div>
      <div class="note-card">
        <h3>下一步</h3>
        <p>继续补齐 Agent 框架审计、RAG 防御、红队自动化和企业落地风险评估。</p>
      </div>
    </aside>
  </div>
</section>
