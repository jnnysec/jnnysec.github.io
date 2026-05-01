<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>jnnysec — AI Agent Security Notes</title>
  <meta name="description" content="个人学习笔记，专注 AI Agent 安全领域。记录 LLM、Agent 架构的安全探索过程。" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
  <style>
    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
    html { font-size: 16px; -webkit-font-smoothing: antialiased; }
    body {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #ffffff;
      color: #1a1a1a;
      line-height: 1.6;
    }
    a { color: inherit; text-decoration: none; }
    ul, ol { list-style: none; }
    button { font-family: inherit; cursor: pointer; border: none; background: none; }

    .shell { display: flex; height: 100vh; overflow: hidden; }

    /* Sidebar */
    .sidebar {
      width: 240px; flex-shrink: 0;
      background: #f7f7f5; border-right: 1px solid #e8e8e6;
      display: flex; flex-direction: column;
      overflow-y: auto; padding: 8px 6px;
    }
    .sb-workspace {
      display: flex; align-items: center; gap: 8px;
      padding: 6px 10px; border-radius: 6px; margin-bottom: 4px;
      cursor: pointer; transition: background .15s;
    }
    .sb-workspace:hover { background: #ebebea; }
    .sb-ws-avatar {
      width: 24px; height: 24px; border-radius: 5px;
      background: #2563eb; display: flex; align-items: center;
      justify-content: center; font-size: 11px; font-weight: 700;
      color: #fff; flex-shrink: 0;
    }
    .sb-ws-name { font-size: 13px; font-weight: 600; color: #1a1a1a; }
    .sb-section-label {
      font-size: 11px; font-weight: 600; color: #999;
      letter-spacing: 0.05em; padding: 12px 10px 4px;
    }
    .sb-item {
      display: flex; align-items: center; gap: 7px;
      padding: 5px 10px; border-radius: 5px;
      font-size: 13.5px; color: #555; cursor: pointer;
      transition: background .12s; user-select: none;
      text-decoration: none;
    }
    .sb-item:hover { background: #ebebea; color: #1a1a1a; }
    .sb-item.active { background: #e8e8e6; color: #1a1a1a; font-weight: 500; }
    .sb-icon { font-size: 14px; width: 20px; text-align: center; flex-shrink: 0; }
    .sb-sub { padding-left: 30px; font-size: 12.5px; color: #888; }
    .sb-sub:hover { color: #1a1a1a; }
    .sb-divider { height: 1px; background: #e5e5e3; margin: 8px 6px; }
    .sb-add {
      display: flex; align-items: center; gap: 7px;
      padding: 5px 10px; border-radius: 5px;
      font-size: 12.5px; color: #bbb; cursor: pointer;
      transition: background .12s, color .12s;
    }
    .sb-add:hover { background: #ebebea; color: #888; }

    /* Main */
    .main { flex: 1; display: flex; flex-direction: column; overflow: hidden; min-width: 0; }
    .topbar {
      height: 44px; display: flex; align-items: center;
      justify-content: space-between; padding: 0 20px;
      border-bottom: 1px solid #ececea; background: #fff; flex-shrink: 0;
    }
    .breadcrumb { display: flex; align-items: center; gap: 5px; font-size: 13px; color: #999; }
    .crumb { color: #666; }
    .sep { color: #ccc; font-size: 12px; }
    .crumb-cur { color: #1a1a1a; font-weight: 500; }
    .topbar-actions { display: flex; align-items: center; gap: 4px; }
    .tb-btn { font-size: 12.5px; color: #888; padding: 4px 10px; border-radius: 5px; transition: background .12s; }
    .tb-btn:hover { background: #f0f0ee; color: #555; }

    .page-scroll { flex: 1; overflow-y: auto; }
    .page-cover {
      height: 160px; background: #f0f5fb;
      background-image: repeating-linear-gradient(
        45deg, transparent, transparent 24px,
        rgba(37,99,235,0.045) 24px, rgba(37,99,235,0.045) 48px
      );
    }
    .page-content { max-width: 720px; margin: 0 auto; padding: 44px 60px 80px; }

    .page-icon-wrap { font-size: 52px; line-height: 1; margin-bottom: 16px; display: block; }
    .page-title { font-size: 34px; font-weight: 700; color: #1a1a1a; line-height: 1.2; margin-bottom: 20px; letter-spacing: -0.02em; }

    /* Properties */
    .props-block { margin-bottom: 28px; padding-bottom: 20px; border-bottom: 1px solid #ececea; }
    .prop-row { display: flex; align-items: center; padding: 5px 0; font-size: 13.5px; }
    .prop-key { width: 120px; flex-shrink: 0; color: #999; display: flex; align-items: center; gap: 6px; font-size: 12.5px; }
    .prop-val { color: #555; font-size: 13px; }
    .prop-val a { color: #2563eb; }
    .prop-val a:hover { text-decoration: underline; }

    /* Pills */
    .pill { display: inline-flex; align-items: center; font-size: 11.5px; font-weight: 500; padding: 2px 8px; border-radius: 12px; margin-right: 4px; }
    .pill-blue   { background: #e8f0fe; color: #1a56db; }
    .pill-orange { background: #fff7ed; color: #b45309; }
    .pill-green  { background: #ecfdf5; color: #166534; }
    .pill-gray   { background: #f3f4f6; color: #6b7280; }
    .pill-purple { background: #ede9fe; color: #5b21b6; }
    .pill-red    { background: #fee2e2; color: #b91c1c; }

    /* Callout */
    .callout { background: #f8f8f7; border-radius: 6px; padding: 12px 16px; margin-bottom: 28px; display: flex; gap: 12px; align-items: flex-start; }
    .callout-icon { font-size: 18px; flex-shrink: 0; margin-top: 1px; }
    .callout-body { font-size: 14px; color: #555; line-height: 1.65; }

    /* Headings */
    .block-h2 { font-size: 18px; font-weight: 600; color: #1a1a1a; margin: 32px 0 12px; display: flex; align-items: center; gap: 8px; }
    .bh-icon { font-size: 16px; }

    /* Post list */
    .post-list { margin-bottom: 8px; }
    .post-row {
      display: flex; align-items: center; gap: 10px;
      padding: 7px 8px; border-radius: 5px; cursor: pointer;
      transition: background .12s; border-bottom: 1px solid #f3f3f2;
      text-decoration: none;
    }
    .post-row:last-child { border-bottom: none; }
    .post-row:hover { background: #f7f7f5; }
    .post-row:hover .post-title-text { color: #2563eb; }
    .post-dot { width: 5px; height: 5px; border-radius: 50%; flex-shrink: 0; margin-top: 1px; }
    .post-title-text { flex: 1; font-size: 13.5px; color: #1a1a1a; line-height: 1.45; transition: color .12s; min-width: 0; }
    .badge-new { background: #fee2e2; color: #b91c1c; font-size: 9.5px; font-weight: 600; padding: 1px 6px; border-radius: 10px; margin-left: 6px; vertical-align: middle; flex-shrink: 0; }
    .post-date { font-size: 11.5px; color: #c0c0c0; flex-shrink: 0; min-width: 72px; text-align: right; }

    /* Topics */
    .topic-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(128px, 1fr)); gap: 8px; margin-bottom: 8px; }
    .topic-chip {
      background: #fafafa; border: 1px solid #e8e8e6; border-radius: 7px;
      padding: 10px 12px; cursor: pointer; transition: border-color .15s, background .15s, transform .12s;
      text-decoration: none; display: block;
    }
    .topic-chip:hover { background: #f3f3f1; border-color: #d0d0ce; transform: translateY(-1px); }
    .chip-emoji { font-size: 16px; margin-bottom: 6px; display: block; }
    .chip-name { font-size: 12.5px; font-weight: 600; color: #333; margin-bottom: 2px; }
    .chip-count { font-size: 11px; color: #aaa; }

    /* About */
    .text-block { font-size: 14px; color: #555; line-height: 1.7; margin-bottom: 14px; }
    .about-block { display: flex; align-items: center; gap: 12px; padding: 12px 0 4px; }
    .about-avatar { width: 36px; height: 36px; border-radius: 7px; background: #e8f0fe; display: flex; align-items: center; justify-content: center; font-size: 14px; font-weight: 700; color: #1a56db; flex-shrink: 0; }
    .about-name { font-size: 13.5px; font-weight: 600; color: #1a1a1a; }
    .about-sub { font-size: 12px; color: #999; margin-top: 2px; }
    .about-links { display: flex; gap: 10px; margin-top: 6px; }
    .about-link { font-size: 12px; color: #2563eb; border-bottom: 1px solid #bdd1f8; padding-bottom: 1px; transition: border-color .12s; }
    .about-link:hover { border-color: #2563eb; }

    /* Divider & Footer */
    .block-divider { height: 1px; background: #ececea; margin: 28px 0; }
    .page-footer { display: flex; align-items: center; gap: 8px; margin-top: 40px; padding-top: 20px; border-top: 1px solid #ececea; font-size: 12px; color: #ccc; flex-wrap: wrap; }
    .page-footer a { color: #bbb; }
    .page-footer a:hover { color: #888; }
    .pf-dot { width: 3px; height: 3px; border-radius: 50%; background: #ddd; }

    /* Scrollbar */
    ::-webkit-scrollbar { width: 5px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: #ddd; border-radius: 10px; }
    ::-webkit-scrollbar-thumb:hover { background: #bbb; }

    /* Responsive */
    @media (max-width: 768px) {
      .sidebar { display: none; }
      .page-content { padding: 32px 22px 60px; }
      .page-title { font-size: 26px; }
      .topic-grid { grid-template-columns: repeat(2, 1fr); }
    }
  </style>
</head>
<body>

<div class="shell">

  <!-- Sidebar -->
  <aside class="sidebar" aria-label="侧边导航">
    <div class="sb-workspace">
      <div class="sb-ws-avatar">J</div>
      <div class="sb-ws-name">jnnysec</div>
    </div>
    <div class="sb-divider"></div>

    <nav>
      <a href="/"         class="sb-item active"><span class="sb-icon">🏠</span> 主页</a>
      <a href="/notes"    class="sb-item"><span class="sb-icon">📝</span> 学习笔记</a>
      <a href="/notes/prompt-injection" class="sb-item sb-sub"><span class="sb-icon" style="font-size:11px;">▸</span> Prompt Injection</a>
      <a href="/notes/jailbreak"        class="sb-item sb-sub"><span class="sb-icon" style="font-size:11px;">▸</span> Jailbreak</a>
      <a href="/notes/agent-security"   class="sb-item sb-sub"><span class="sb-icon" style="font-size:11px;">▸</span> Agent Security</a>
      <a href="/notes/rag-memory"       class="sb-item sb-sub"><span class="sb-icon" style="font-size:11px;">▸</span> RAG / Memory</a>
      <a href="/research" class="sb-item"><span class="sb-icon">🔬</span> 研究记录</a>
      <a href="/resources"class="sb-item"><span class="sb-icon">🔗</span> 参考资源</a>
      <a href="/about"    class="sb-item"><span class="sb-icon">👤</span> 关于</a>
    </nav>

    <div class="sb-divider"></div>
    <div class="sb-section-label">收藏</div>
    <a href="/starred" class="sb-item"><span class="sb-icon">⭐</span> 精选笔记</a>
    <div class="sb-divider"></div>
    <div class="sb-add"><span style="font-size:16px;line-height:1;">+</span> 新建页面</div>
  </aside>

  <!-- Main -->
  <div class="main">
    <header class="topbar">
      <div class="breadcrumb">
        <span class="crumb">jnnysec</span>
        <span class="sep">›</span>
        <span class="crumb">AI Agent Security</span>
        <span class="sep">›</span>
        <span class="crumb-cur">主页</span>
      </div>
      <div class="topbar-actions">
        <button class="tb-btn">分享</button>
        <button class="tb-btn">⋯</button>
      </div>
    </header>

    <main class="page-scroll">
      <div class="page-cover" role="img" aria-label="封面"></div>

      <article class="page-content">

        <span class="page-icon-wrap" aria-hidden="true">🛡️</span>
        <h1 class="page-title">AI Agent 安全学习笔记</h1>

        <div class="props-block">
          <div class="prop-row">
            <div class="prop-key"><span>👤</span> 作者</div>
            <div class="prop-val"><a href="https://github.com/jnnysec" target="_blank" rel="noopener">jnnysec</a></div>
          </div>
          <div class="prop-row">
            <div class="prop-key"><span>📅</span> 创建于</div>
            <div class="prop-val">2026 年 5 月</div>
          </div>
          <div class="prop-row">
            <div class="prop-key"><span>🔄</span> 最近更新</div>
            <div class="prop-val">2026-05-01</div>
          </div>
          <div class="prop-row">
            <div class="prop-key"><span>🏷️</span> 标签</div>
            <div class="prop-val">
              <span class="pill pill-blue">AI Security</span>
              <span class="pill pill-purple">LLM</span>
              <span class="pill pill-green">Agent</span>
            </div>
          </div>
          <div class="prop-row">
            <div class="prop-key"><span>📊</span> 笔记数</div>
            <div class="prop-val">1 篇，持续更新</div>
          </div>
        </div>

        <div class="callout">
          <span class="callout-icon">💡</span>
          <div class="callout-body">
            Web 安全 / 代码审计 → AI Agent 安全。主攻 <strong>Prompt Injection</strong>、<strong>Jailbreak</strong>、<strong>Agent 框架审计</strong>。这里记录学习笔记与攻防研究。
          </div>
        </div>

        <h2 class="block-h2"><span class="bh-icon">📋</span> 最新笔记</h2>
        <div class="post-list">
          <a class="post-row" href="/articles/llm-intro-from-security-perspective">
            <div class="post-dot" style="background:#2563eb;"></div>
            <div class="post-title-text">
              从「预测下一个词」到「攻击下一个词」——一个安全工程师看 Karpathy 的 LLM 入门课
              <span class="badge-new">NEW</span>
            </div>
            <span class="pill pill-orange">LLM 安全</span>
            <div class="post-date">2026-05-01</div>
          </a>

</div>
            <div class="post-title-text">
              Prompt Injection in Multi-Agent Systems：从理论到实战
              <span class="badge-new">NEW</span>
            </div>
            <span class="pill pill-orange">威胁模型</span>
            <div class="post-date">2026-05-01</div>
          </a>

          <a class="post-row" href="/notes/mcp-security-model">
            <div class="post-dot" style="background:#9ca3af;"></div>
            <div class="post-title-text">MCP 安全模型笔记：信任边界与权限设计</div>
            <span class="pill pill-blue">笔记</span>
            <div class="post-date">2025-04-15</div>
          </a>

          <a class="post-row" href="/notes/rag-data-poisoning">
            <div class="post-dot" style="background:#9ca3af;"></div>
            <div class="post-title-text">RAG 管道中的数据投毒：间接攻击向量</div>
            <span class="pill pill-green">研究</span>
            <div class="post-date">2025-04-02</div>
          </a>

          <a class="post-row" href="/notes/llm-jailbreak-basics">
            <div class="post-dot" style="background:#9ca3af;"></div>
            <div class="post-title-text">LLM 安全基础：越狱技术分类与防御思路</div>
            <span class="pill pill-gray">基础</span>
            <div class="post-date">2025-03-20</div>
          </a>

          <a class="post-row" href="/notes/agent-memory-risks">
            <div class="post-dot" style="background:#9ca3af;"></div>
            <div class="post-title-text">Agent 记忆机制的安全隐患：持久化与泄露风险</div>
            <span class="pill pill-purple">架构</span>
            <div class="post-date">2025-03-10</div>
          </a>

          <a class="post-row" href="/notes/agentbench-security">
            <div class="post-dot" style="background:#9ca3af;"></div>
            <div class="post-title-text">AgentBench 评测框架在安全场景中的使用</div>
            <span class="pill pill-blue">工具</span>
            <div class="post-date">2025-02-28</div>
          </a>

        </div>

        <div class="block-divider"></div>

        <h2 class="block-h2"><span class="bh-icon">🏷️</span> 话题分类</h2>
        <div class="topic-grid">
          <a href="/notes/prompt-injection" class="topic-chip">
            <span class="chip-emoji">💉</span>
            <div class="chip-name">Prompt Injection</div>
            <div class="chip-count">coming soon</div>
          </a>
          <a href="/notes/jailbreak" class="topic-chip">
            <span class="chip-emoji">🔓</span>
            <div class="chip-name">Jailbreak</div>
            <div class="chip-count">coming soon</div>
          </a>
          <a href="/notes/agent-security" class="topic-chip">
            <span class="chip-emoji">🤖</span>
            <div class="chip-name">Agent Security</div>
            <div class="chip-count">coming soon</div>
          </a>
          <a href="/notes/rag-memory" class="topic-chip">
            <span class="chip-emoji">🗄️</span>
            <div class="chip-name">RAG / Memory</div>
            <div class="chip-count">coming soon</div>
          </a>
          <a href="/notes/supply-chain" class="topic-chip">
            <span class="chip-emoji">📦</span>
            <div class="chip-name">Supply Chain</div>
            <div class="chip-count">coming soon</div>
          </a>
        </div>

        <div class="block-divider"></div>

        <h2 class="block-h2"><span class="bh-icon">👤</span> 关于作者</h2>
        <p class="text-block">Web 安全 / 代码审计背景，正在转向 AI Agent 安全方向。主攻 Prompt Injection、Jailbreak、Agent 框架审计。这里记录学习笔记与攻防研究。</p>

        <div class="about-block">
          <div class="about-avatar">J</div>
          <div>
            <div class="about-name">jnnysec</div>
            <div class="about-sub">AI Agent Security · Threat Research</div>
            <div class="about-links">
              <a href="https://github.com/jnnysec"  class="about-link" target="_blank" rel="noopener">GitHub</a>
              <a href="/feed.xml" class="about-link">RSS</a>
            </div>
          </div>
        </div>

        <footer class="page-footer">
          <span>© 2026 jnnysec</span>
          <span class="pf-dot"></span>
          <a href="https://jnnysec.github.io">jnnysec.github.io</a>
          <span class="pf-dot"></span>
          <span>个人学习记录，仅供参考</span>
        </footer>

      </article>
    </main>
  </div>
</div>

</body>
</html>
