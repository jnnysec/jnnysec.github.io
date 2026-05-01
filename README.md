<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>jnnysec — AI Agent Security</title>
  <meta name="description" content="Web 安全 → AI Agent 安全。主攻 Prompt Injection、Jailbreak、Agent 框架审计。" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
  <style>
    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
    html { font-size: 16px; -webkit-font-smoothing: antialiased; }
    body {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #ffffff; color: #1a1a1a; line-height: 1.6;
    }
    a { color: inherit; text-decoration: none; }
    ul, ol { list-style: none; }

    .shell { display: flex; height: 100vh; overflow: hidden; }

    /* ── Sidebar ── */
    .sidebar {
      width: 240px; flex-shrink: 0;
      background: #f7f7f5; border-right: 1px solid #e8e8e6;
      display: flex; flex-direction: column;
      overflow-y: auto; padding: 8px 6px;
    }
    .sb-brand {
      display: flex; align-items: center; gap: 8px;
      padding: 8px 10px; border-radius: 6px;
    }
    .sb-avatar {
      width: 26px; height: 26px; border-radius: 6px;
      background: #2563eb; display: flex; align-items: center;
      justify-content: center; font-size: 12px; font-weight: 700;
      color: #fff; flex-shrink: 0;
    }
    .sb-name { font-size: 13px; font-weight: 600; color: #1a1a1a; }

    .sb-divider { height: 1px; background: #e5e5e3; margin: 8px 6px; }

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

    /* ── Main ── */
    .main { flex: 1; display: flex; flex-direction: column; overflow: hidden; min-width: 0; }
    .topbar {
      height: 44px; display: flex; align-items: center;
      padding: 0 20px; border-bottom: 1px solid #ececea;
      background: #fff; flex-shrink: 0;
    }
    .breadcrumb { display: flex; align-items: center; gap: 5px; font-size: 13px; }
    .crumb { color: #666; }
    .sep { color: #ccc; font-size: 10px; }
    .crumb-cur { color: #1a1a1a; font-weight: 500; }

    .page-scroll { flex: 1; overflow-y: auto; }
    .page-cover {
      height: 120px; background: #f0f5fb;
      background-image: repeating-linear-gradient(
        45deg, transparent, transparent 24px,
        rgba(37,99,235,0.045) 24px, rgba(37,99,235,0.045) 48px
      );
    }
    .page-content { max-width: 720px; margin: 0 auto; padding: 44px 40px 80px; }

    .page-icon-wrap { font-size: 48px; line-height: 1; margin-bottom: 14px; display: block; }
    .page-title { font-size: 32px; font-weight: 700; color: #1a1a1a; line-height: 1.2; margin-bottom: 8px; letter-spacing: -0.02em; }
    .page-sub { font-size: 14px; color: #999; margin-bottom: 28px; }

    /* ── Pills ── */
    .pill { display: inline-flex; align-items: center; font-size: 11px; font-weight: 500; padding: 2px 8px; border-radius: 12px; margin-right: 4px; }
    .pill-blue   { background: #e8f0fe; color: #1a56db; }
    .pill-orange { background: #fff7ed; color: #b45309; }
    .pill-green  { background: #ecfdf5; color: #166534; }
    .pill-purple { background: #ede9fe; color: #5b21b6; }

    /* ── Callout ── */
    .callout { background: #f8f8f7; border-radius: 6px; padding: 12px 16px; margin-bottom: 28px; display: flex; gap: 12px; align-items: flex-start; }
    .callout-icon { font-size: 18px; flex-shrink: 0; margin-top: 1px; }
    .callout-body { font-size: 14px; color: #555; line-height: 1.65; }

    /* ── Section headings ── */
    .block-h2 { font-size: 16px; font-weight: 600; color: #1a1a1a; margin: 32px 0 10px; display: flex; align-items: center; gap: 7px; }

    /* ── Post list ── */
    .post-row {
      display: flex; align-items: center; gap: 10px;
      padding: 8px 10px; border-radius: 5px;
      transition: background .12s; border-bottom: 1px solid #f3f3f2;
      text-decoration: none; cursor: pointer;
    }
    .post-row:hover { background: #f7f7f5; }
    .post-row:hover .post-title-text { color: #2563eb; }
    .post-title-text { flex: 1; font-size: 13.5px; color: #1a1a1a; line-height: 1.45; transition: color .12s; min-width: 0; }
    .badge-new { background: #fee2e2; color: #b91c1c; font-size: 9.5px; font-weight: 600; padding: 1px 6px; border-radius: 10px; margin-left: 6px; vertical-align: middle; flex-shrink: 0; }
    .post-date { font-size: 11.5px; color: #c0c0c0; flex-shrink: 0; min-width: 72px; text-align: right; }

    /* ── Topics ── */
    .topic-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(128px, 1fr)); gap: 8px; margin-bottom: 8px; }
    .topic-chip {
      background: #fafafa; border: 1px solid #e8e8e6; border-radius: 7px;
      padding: 10px 12px; transition: border-color .15s, background .15s, transform .12s;
    }
    .topic-chip:hover { background: #f3f3f1; border-color: #d0d0ce; transform: translateY(-1px); }
    .chip-emoji { font-size: 16px; margin-bottom: 6px; display: block; }
    .chip-name { font-size: 12.5px; font-weight: 600; color: #333; margin-bottom: 2px; }
    .chip-count { font-size: 11px; color: #aaa; }

    /* ── About ── */
    .text-block { font-size: 14px; color: #555; line-height: 1.7; margin-bottom: 12px; }
    .about-avatar { width: 36px; height: 36px; border-radius: 7px; background: #e8f0fe; display: inline-flex; align-items: center; justify-content: center; font-size: 14px; font-weight: 700; color: #1a56db; vertical-align: middle; margin-right: 10px; }
    .about-name { font-size: 13.5px; font-weight: 600; color: #1a1a1a; display: inline; }
    .about-sub { font-size: 12px; color: #999; margin: 4px 0 6px; }
    .about-link { font-size: 12px; color: #2563eb; border-bottom: 1px solid #bdd1f8; padding-bottom: 1px; margin-right: 12px; transition: border-color .12s; }
    .about-link:hover { border-color: #2563eb; }

    /* ── Divider & Footer ── */
    .block-divider { height: 1px; background: #ececea; margin: 28px 0; }
    .page-footer { margin-top: 32px; padding-top: 16px; border-top: 1px solid #ececea; font-size: 12px; color: #ccc; display: flex; align-items: center; gap: 6px; flex-wrap: wrap; }
    .pf-dot { width: 3px; height: 3px; border-radius: 50%; background: #ddd; }

    /* ── Responsive ── */
    @media (max-width: 768px) {
      .sidebar { display: none; }
      .page-content { padding: 32px 22px 60px; }
      .page-title { font-size: 24px; }
      .topic-grid { grid-template-columns: repeat(2, 1fr); }
    }

    ::-webkit-scrollbar { width: 5px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: #ddd; border-radius: 10px; }
  </style>
</head>
<body>

<div class="shell">

  <!-- Sidebar -->
  <aside class="sidebar">
    <div class="sb-brand">
      <div class="sb-avatar">J</div>
      <div class="sb-name">jnnysec</div>
    </div>
    <div class="sb-divider"></div>
    <nav>
      <a href="/" class="sb-item active"><span class="sb-icon">🏠</span> 主页</a>
    </nav>
    <div class="sb-divider"></div>
    <div class="sb-item sb-sub" style="color:#aaa;cursor:default;">文章</div>
    <a href="/articles/llm-intro-from-security-perspective" class="sb-item sb-sub">Karpathy LLM 安全解读</a>
    <div class="sb-divider"></div>
    <div class="sb-item sb-sub" style="color:#aaa;cursor:default;">话题</div>
    <div class="sb-item sb-sub">Prompt Injection</div>
    <div class="sb-item sb-sub">Jailbreak</div>
    <div class="sb-item sb-sub">Agent Security</div>
    <div class="sb-item sb-sub">RAG / Memory</div>
    <div class="sb-item sb-sub">Supply Chain</div>
  </aside>

  <!-- Main -->
  <div class="main">
    <header class="topbar">
      <div class="breadcrumb">
        <span class="crumb">jnnysec</span>
        <span class="sep">/</span>
        <span class="crumb-cur">主页</span>
      </div>
    </header>

    <main class="page-scroll">
      <div class="page-cover" role="img" aria-label="封面"></div>

      <article class="page-content">

        <span class="page-icon-wrap">🛡️</span>
        <h1 class="page-title">AI Agent 安全学习笔记</h1>
        <p class="page-sub">Web 安全 → AI Agent 安全 · 2026 年 5 月起</p>

        <div class="callout">
          <span class="callout-icon">💡</span>
          <div class="callout-body">
            <strong>jnnysec</strong> — Web 安全 / 代码审计背景，正在转向 AI Agent 安全方向。主攻 <strong>Prompt Injection</strong>、<strong>Jailbreak</strong>、<strong>Agent 框架审计</strong>。这里记录我的学习笔记与攻防研究。
          </div>
        </div>

        <h2 class="block-h2">📋 文章</h2>

        <a class="post-row" href="/articles/llm-intro-from-security-perspective">
          <div class="post-title-text">
            从「预测下一个词」到「攻击下一个词」——一个安全工程师看 Karpathy 的 LLM 入门课
            <span class="badge-new">NEW</span>
          </div>
          <span class="pill pill-orange">LLM 安全</span>
          <div class="post-date">2026-05-01</div>
        </a>

        <div class="block-divider"></div>

        <h2 class="block-h2">🏷️ 话题分类</h2>
        <div class="topic-grid">
          <div class="topic-chip">
            <span class="chip-emoji">💉</span>
            <div class="chip-name">Prompt Injection</div>
            <div class="chip-count">coming soon</div>
          </div>
          <div class="topic-chip">
            <span class="chip-emoji">🔓</span>
            <div class="chip-name">Jailbreak</div>
            <div class="chip-count">coming soon</div>
          </div>
          <div class="topic-chip">
            <span class="chip-emoji">🤖</span>
            <div class="chip-name">Agent Security</div>
            <div class="chip-count">coming soon</div>
          </div>
          <div class="topic-chip">
            <span class="chip-emoji">🗄️</span>
            <div class="chip-name">RAG / Memory</div>
            <div class="chip-count">coming soon</div>
          </div>
          <div class="topic-chip">
            <span class="chip-emoji">📦</span>
            <div class="chip-name">Supply Chain</div>
            <div class="chip-count">coming soon</div>
          </div>
        </div>

        <div class="block-divider"></div>

        <h2 class="block-h2">👤 关于</h2>
        <p class="text-block">Web 安全 / 代码审计背景，正在转向 AI Agent 安全方向。主攻 Prompt Injection、Jailbreak、Agent 框架审计。这里记录学习笔记与攻防研究。</p>

        <div>
          <span class="about-avatar">J</span>
          <span class="about-name">jnnysec</span>
          <div class="about-sub">AI Agent Security · Threat Research</div>
          <a href="https://github.com/jnnysec" class="about-link" target="_blank" rel="noopener">GitHub</a>
        </div>

        <footer class="page-footer">
          <span>jnnysec.github.io</span>
          <span class="pf-dot"></span>
          <span>个人学习记录</span>
        </footer>

      </article>
    </main>
  </div>
</div>

</body>
</html>
