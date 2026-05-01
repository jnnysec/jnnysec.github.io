<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>jnnysec — AI Agent Security Notes</title>
  <meta name="description" content="个人学习笔记，专注 AI Agent 安全领域。记录 LLM、Agent 架构的安全探索过程。" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;700&family=Space+Grotesk:wght@300;400;500;600&display=swap" rel="stylesheet" />
  <style>
    :root {
      --bg:       #0a0d0f;
      --surface:  #111518;
      --surface2: #161c20;
      --border:   #1e2830;
      --accent:   #00d4a1;
      --accent2:  #0088ff;
      --accent3:  #ff6b35;
      --text:     #c8d8e0;
      --text2:    #5a7a88;
      --text3:    #3a5060;
      --mono:     'JetBrains Mono', monospace;
      --sans:     'Space Grotesk', sans-serif;
    }

    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

    html { scroll-behavior: smooth; }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--sans);
      min-height: 100vh;
      line-height: 1.6;
      overflow-x: hidden;
    }

    /* ── Background Effects ── */
    .scanlines {
      position: fixed; inset: 0; pointer-events: none; z-index: 0;
      background: repeating-linear-gradient(
        0deg,
        transparent, transparent 2px,
        rgba(0, 212, 161, 0.012) 2px, rgba(0, 212, 161, 0.012) 4px
      );
    }
    .grid-bg {
      position: fixed; inset: 0; pointer-events: none; z-index: 0;
      background-image:
        linear-gradient(rgba(0, 136, 255, 0.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(0, 136, 255, 0.03) 1px, transparent 1px);
      background-size: 40px 40px;
    }

    /* ── Layout ── */
    .container {
      max-width: 900px;
      margin: 0 auto;
      padding: 0 28px;
      position: relative;
      z-index: 1;
    }

    /* ── NAV ── */
    nav {
      padding: 22px 0;
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      justify-content: space-between;
      position: sticky;
      top: 0;
      background: rgba(10, 13, 15, 0.92);
      backdrop-filter: blur(12px);
      z-index: 100;
    }
    .nav-inner {
      max-width: 900px;
      margin: 0 auto;
      padding: 0 28px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      width: 100%;
    }
    .logo {
      font-family: var(--mono);
      font-size: 14px;
      color: var(--accent);
      display: flex;
      align-items: center;
      gap: 8px;
      text-decoration: none;
    }
    .logo-bracket { color: var(--text3); }
    .logo-domain  { font-size: 11px; color: var(--text3); }
    .status-dot {
      width: 6px; height: 6px;
      border-radius: 50%;
      background: var(--accent);
      animation: pulse 2.5s ease-in-out infinite;
      flex-shrink: 0;
    }
    @keyframes pulse { 0%,100%{opacity:1;box-shadow:0 0 0 0 rgba(0,212,161,0.4);} 50%{opacity:0.5;box-shadow:0 0 0 4px rgba(0,212,161,0);} }

    .nav-links {
      display: flex;
      gap: 24px;
      list-style: none;
    }
    .nav-links a {
      font-family: var(--mono);
      font-size: 12px;
      color: var(--text2);
      text-decoration: none;
      transition: color 0.2s;
    }
    .nav-links a:hover,
    .nav-links a.active { color: var(--accent); }

    /* ── HERO ── */
    .hero {
      padding: 64px 0 52px;
    }
    .hero-eyebrow {
      font-family: var(--mono);
      font-size: 11px;
      color: var(--accent2);
      letter-spacing: 0.2em;
      margin-bottom: 20px;
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .hero-eyebrow::before {
      content: '';
      display: inline-block;
      width: 28px; height: 1px;
      background: var(--accent2);
    }
    .hero h1 {
      font-family: var(--mono);
      font-size: clamp(28px, 5vw, 42px);
      font-weight: 700;
      line-height: 1.15;
      margin-bottom: 16px;
    }
    .hl-green  { color: var(--accent); }
    .hl-blue   { color: var(--accent2); }
    .hl-orange { color: var(--accent3); }

    .hero-sub {
      font-size: 15px;
      color: var(--text2);
      max-width: 560px;
      line-height: 1.8;
      margin-bottom: 36px;
    }

    .terminal-prompt {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 14px 20px;
      font-family: var(--mono);
      font-size: 13px;
      display: inline-flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 36px;
    }
    .prompt-sym  { color: var(--accent); }
    .prompt-text { color: var(--text2); }
    .cursor {
      display: inline-block;
      width: 8px; height: 15px;
      background: var(--accent);
      animation: blink 1s step-end infinite;
      margin-left: 2px;
      border-radius: 1px;
    }
    @keyframes blink { 0%,100%{opacity:1;} 50%{opacity:0;} }

    .hero-stats {
      display: flex;
      gap: 36px;
    }
    .stat-num {
      font-family: var(--mono);
      font-size: 24px;
      font-weight: 700;
      color: var(--accent);
    }
    .stat-label {
      font-family: var(--mono);
      font-size: 10px;
      color: var(--text3);
      letter-spacing: 0.15em;
      margin-top: 3px;
    }

    /* ── DIVIDER ── */
    .section-divider {
      height: 1px;
      background: var(--border);
      margin: 52px 0 36px;
      position: relative;
    }
    .section-divider::after {
      content: attr(data-label);
      font-family: var(--mono);
      font-size: 10px;
      color: var(--text3);
      letter-spacing: 0.2em;
      position: absolute;
      right: 0; top: -8px;
      padding-left: 14px;
      background: var(--bg);
    }

    /* ── POSTS ── */
    .posts-list {
      display: flex;
      flex-direction: column;
      gap: 2px;
      margin-bottom: 52px;
    }
    .post-item {
      border: 1px solid transparent;
      border-radius: 7px;
      padding: 18px 20px;
      cursor: pointer;
      transition: background 0.18s, border-color 0.18s;
      display: grid;
      grid-template-columns: 1fr auto;
      align-items: start;
      gap: 16px;
    }
    .post-item:hover {
      background: var(--surface);
      border-color: var(--border);
    }
    .post-item:hover .post-title { color: var(--accent); }

    .post-meta {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-bottom: 7px;
    }
    .tag {
      font-family: var(--mono);
      font-size: 10px;
      padding: 2px 8px;
      border-radius: 3px;
      letter-spacing: 0.1em;
    }
    .tag-red    { background: rgba(255,107,53,0.12); color: var(--accent3); border: 1px solid rgba(255,107,53,0.3); }
    .tag-blue   { background: rgba(0,136,255,0.12);  color: var(--accent2); border: 1px solid rgba(0,136,255,0.3); }
    .tag-green  { background: rgba(0,212,161,0.12);  color: var(--accent);  border: 1px solid rgba(0,212,161,0.3); }
    .tag-gray   { background: rgba(90,122,136,0.15); color: var(--text2);   border: 1px solid var(--border); }
    .tag-purple { background: rgba(167,139,250,0.12);color: #a78bfa;        border: 1px solid rgba(167,139,250,0.3); }

    .badge-new {
      background: rgba(255,107,53,0.15);
      color: var(--accent3);
      font-family: var(--mono);
      font-size: 9px;
      padding: 1px 6px;
      border-radius: 2px;
      letter-spacing: 0.1em;
      border: 1px solid rgba(255,107,53,0.3);
      vertical-align: middle;
      margin-left: 8px;
    }
    .post-title {
      font-size: 14px;
      font-weight: 500;
      color: var(--text);
      margin-bottom: 5px;
      line-height: 1.45;
      transition: color 0.18s;
    }
    .post-excerpt {
      font-size: 12px;
      color: var(--text3);
      line-height: 1.6;
    }
    .post-date {
      font-family: var(--mono);
      font-size: 11px;
      color: var(--text3);
      white-space: nowrap;
      padding-top: 3px;
    }

    /* ── TOPICS ── */
    .section-label {
      font-family: var(--mono);
      font-size: 10px;
      color: var(--text3);
      letter-spacing: 0.2em;
      margin-bottom: 16px;
      display: flex;
      align-items: center;
      gap: 12px;
    }
    .section-label::after {
      content: '';
      flex: 1;
      height: 1px;
      background: var(--border);
    }

    .topics-grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 8px;
      margin-bottom: 52px;
    }
    .topic-card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 16px 14px;
      cursor: pointer;
      transition: border-color 0.2s, background 0.2s, transform 0.15s;
    }
    .topic-card:hover {
      border-color: var(--accent2);
      background: var(--surface2);
      transform: translateY(-2px);
    }
    .topic-icon {
      font-size: 14px;
      margin-bottom: 10px;
      display: block;
    }
    .topic-name {
      font-family: var(--mono);
      font-size: 11px;
      color: var(--text);
      margin-bottom: 4px;
      line-height: 1.3;
    }
    .topic-count {
      font-family: var(--mono);
      font-size: 10px;
      color: var(--text3);
    }

    /* ── ABOUT ── */
    .about-block {
      background: var(--surface);
      border: 1px solid var(--border);
      border-left: 3px solid var(--accent);
      border-radius: 0 8px 8px 0;
      padding: 22px 26px;
      margin-bottom: 52px;
      display: flex;
      gap: 20px;
      align-items: center;
    }
    .avatar {
      width: 52px; height: 52px;
      border-radius: 50%;
      background: linear-gradient(135deg, rgba(0,212,161,0.25), rgba(0,136,255,0.25));
      border: 1px solid rgba(0,212,161,0.4);
      display: flex; align-items: center; justify-content: center;
      font-family: var(--mono);
      font-size: 18px;
      color: var(--accent);
      flex-shrink: 0;
    }
    .about-text h3 {
      font-size: 14px;
      font-weight: 500;
      color: var(--text);
      margin-bottom: 5px;
    }
    .about-text p {
      font-size: 13px;
      color: var(--text2);
      line-height: 1.65;
    }

    /* ── FOOTER ── */
    footer {
      border-top: 1px solid var(--border);
      padding: 26px 0 32px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .footer-left {
      font-family: var(--mono);
      font-size: 11px;
      color: var(--text3);
    }
    .footer-right {
      display: flex;
      gap: 20px;
    }
    .footer-link {
      font-family: var(--mono);
      font-size: 11px;
      color: var(--text3);
      text-decoration: none;
      transition: color 0.2s;
    }
    .footer-link:hover { color: var(--accent); }

    /* ── RESPONSIVE ── */
    @media (max-width: 640px) {
      .hero h1 { font-size: 26px; }
      .hero-stats { gap: 24px; }
      .topics-grid { grid-template-columns: repeat(3, 1fr); }
      .about-block { flex-direction: column; text-align: center; }
      footer { flex-direction: column; gap: 14px; text-align: center; }
      .footer-left { font-size: 10px; }
      .post-item { grid-template-columns: 1fr; }
      .post-date { margin-top: 6px; }
      .nav-links { gap: 16px; }
    }
    @media (max-width: 420px) {
      .topics-grid { grid-template-columns: repeat(2, 1fr); }
      .container { padding: 0 18px; }
    }
  </style>
</head>
<body>

  <div class="scanlines" aria-hidden="true"></div>
  <div class="grid-bg"   aria-hidden="true"></div>

  <!-- NAV -->
  <nav aria-label="主导航">
    <div class="nav-inner">
      <a href="/" class="logo" aria-label="jnnysec 首页">
        <span class="status-dot" aria-hidden="true"></span>
        <span class="logo-bracket">[</span>jnnysec<span class="logo-bracket">]</span>
        <span class="logo-domain">.github.io</span>
      </a>
      <ul class="nav-links">
        <li><a href="/"        class="active">~/home</a></li>
        <li><a href="/notes">~/notes</a></li>
        <li><a href="/research">~/research</a></li>
        <li><a href="/about">~/about</a></li>
      </ul>
    </div>
  </nav>

  <!-- MAIN -->
  <main>
    <div class="container">

      <!-- HERO -->
      <section class="hero" aria-labelledby="hero-title">
        <p class="hero-eyebrow" aria-hidden="true">AI_AGENT_SECURITY_BLOG</p>
        <h1 id="hero-title">
          Learning<br>
          <span class="hl-blue">AI Agent</span> <span class="hl-green">Security</span><br>
          <span class="hl-orange">in the open</span>
        </h1>
        <p class="hero-sub">
          个人学习笔记 · 威胁模型分析 · 攻防研究。<br>
          记录 LLM、Agent 架构的安全探索过程，从零构建 AI 安全知识体系。
        </p>
        <div class="terminal-prompt" aria-hidden="true">
          <span class="prompt-sym">$</span>
          <span class="prompt-text">cat /notes/latest.md</span>
          <span class="cursor"></span>
        </div>
        <div class="hero-stats" aria-label="博客统计">
          <div class="stat" aria-label="1 篇文章">
            <div class="stat-num">1</div>
            <div class="stat-label">ARTICLE</div>
          </div>
          <div class="stat" aria-label="5 个主题">
            <div class="stat-num">5</div>
            <div class="stat-label">TOPICS</div>
          </div>
          <div class="stat" aria-label="2026 年开始">
            <div class="stat-num">2026</div>
            <div class="stat-label">SINCE</div>
          </div>
        </div>
      </section>

      <!-- POSTS -->
      <div class="section-divider" data-label="LATEST_NOTES" aria-hidden="true"></div>

      <section aria-label="最新笔记">
                <ol class="posts-list" style="list-style:none;">

          <li>
            <a href="/articles/llm-intro-from-security-perspective" style="text-decoration:none;color:inherit;">
            <article class="post-item">
              <div>
                <div class="post-meta">
                  <span class="tag tag-red">LLM_SECURITY</span>
                  <span class="badge-new">NEW</span>
                </div>
                <h2 class="post-title">
                  从「预测下一个词」到「攻击下一个词」——一个安全工程师看 Karpathy 的 LLM 入门课
                </h2>
                <p class="post-excerpt">
                  基于 Karpathy 经典演讲的安全视角解读：LLM 的训练两阶段如何产生攻击面、Prompt Injection / Jailbreak / 数据投毒三类核心攻击、LLM OS 类比作为安全审计框架、Web 安全到 LLM 安全的技能平移地图。
                </p>
              </div>
              <time class="post-date" datetime="2026-05-01">2026-05-01</time>
            </article>
            </a>
          </li>

        </ol>
      </section>

      <!-- TOPICS -->
      <div class="section-label" aria-hidden="true">TOPICS</div>
      <section aria-label="话题分类">
                <div class="topics-grid">
          <div class="topic-card" role="link" tabindex="0">
            <span class="topic-icon" style="color:var(--accent3);">&#9670;</span>
            <div class="topic-name">Prompt Injection</div>
            <div class="topic-count">coming soon</div>
          </div>
          <div class="topic-card" role="link" tabindex="0">
            <span class="topic-icon" style="color:var(--accent2);">&#9632;</span>
            <div class="topic-name">Jailbreak</div>
            <div class="topic-count">coming soon</div>
          </div>
          <div class="topic-card" role="link" tabindex="0">
            <span class="topic-icon" style="color:var(--accent);">&#9650;</span>
            <div class="topic-name">Agent Security</div>
            <div class="topic-count">coming soon</div>
          </div>
          <div class="topic-card" role="link" tabindex="0">
            <span class="topic-icon" style="color:#a78bfa;">&#9679;</span>
            <div class="topic-name">RAG / Memory</div>
            <div class="topic-count">coming soon</div>
          </div>
          <div class="topic-card" role="link" tabindex="0">
            <span class="topic-icon" style="color:#fbbf24;">&#9671;</span>
            <div class="topic-name">Supply Chain</div>
            <div class="topic-count">coming soon</div>
          </div>
        </div>
      </section>

      <!-- ABOUT -->
      <section class="about-block" aria-label="关于作者">
        <div class="avatar" aria-hidden="true">J</div>
        <div class="about-text">
          <h3>jnnysec</h3>
          <p>Web 安全 / 代码审计背景，转向 AI Agent 安全方向。主攻 Prompt Injection、Jailbreak、Agent 框架审计。这里记录学习笔记和攻防研究。</p>
        </div>
      </section>

    </div><!-- /container -->
  </main>

  <!-- FOOTER -->
  <footer>
    <div class="container" style="display:flex;justify-content:space-between;align-items:center;width:100%;">
      <p class="footer-left">// jnnysec.github.io — personal learning records on AI security</p>
      <nav class="footer-right" aria-label="外部链接">
        <a href="https://github.com/jnnysec"  class="footer-link" target="_blank" rel="noopener noreferrer">GitHub</a>
        <a href="/feed.xml"                    class="footer-link">RSS</a>
      </nav>
    </div>
  </footer>

  <script>
    // Highlight active nav link based on current path
    const links = document.querySelectorAll('.nav-links a');
    links.forEach(a => {
      a.classList.remove('active');
      if (a.getAttribute('href') === window.location.pathname) {
        a.classList.add('active');
      }
    });
    if (window.location.pathname === '/') {
      document.querySelector('.nav-links a[href="/"]')?.classList.add('active');
    }

    // Post item click — placeholder navigation
    document.querySelectorAll('.post-item').forEach(item => {
      item.addEventListener('click', () => {
        const title = item.querySelector('.post-title').textContent.trim();
        const slug  = title
          .replace(/[^a-zA-Z0-9\u4e00-\u9fa5]+/g, '-')
          .replace(/^-+|-+$/g, '')
          .toLowerCase();
        window.location.href = '/articles/llm-intro-from-security-perspective';
      });
      item.setAttribute('tabindex', '0');
      item.addEventListener('keydown', e => {
        if (e.key === 'Enter' || e.key === ' ') item.click();
      });
    });

    // Fade-in on load
    document.body.style.opacity = '0';
    document.body.style.transition = 'opacity 0.4s ease';
    window.addEventListener('DOMContentLoaded', () => {
      requestAnimationFrame(() => { document.body.style.opacity = '1'; });
    });
  </script>

</body>
</html>
