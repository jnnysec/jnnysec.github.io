---
layout: default
title: AI Agent 安全学习笔记
---

# 🛡️ AI Agent 安全学习笔记

<small>Web 安全 → AI Agent 安全 · 2026 年 5 月起</small>

---

## 📋 文章

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small style="color:#999;font-size:.98em;">{{ post.date | date: "%Y-%m-%d" }}</small>
      {% if post.tags contains "LLM安全" %}<span style="color:#fff;background:#2563eb;padding:1px 8px;border-radius:7px;font-size:.88em;margin-left:4px;">LLM 安全</span>{% endif %}
      {% if post.new %}<span style="background:#fee2e2;color:#b91c1c;font-size:.89em;border-radius:7px;padding:1px 7px;margin-left:4px;">NEW</span>{% endif %}
    </li>
  {% endfor %}
</ul>

---

## 🏷️ 话题分类

<div style="display:flex;flex-wrap:wrap;gap:11px; margin:12px 0;">
  <span style="background:#f2f7fa;color:#2563eb;padding:6px 16px;border-radius:8px;font-size:.98em;font-weight:600;">💉 Prompt Injection</span>
  <span style="background:#f2f7fa;color:#2563eb;padding:6px 16px;border-radius:8px;font-size:.98em;font-weight:600;">🔓 Jailbreak</span>
  <span style="background:#f2f7fa;color:#2563eb;padding:6px 16px;border-radius:8px;font-size:.98em;font-weight:600;">🤖 Agent Security</span>
  <span style="background:#f2f7fa;color:#2563eb;padding:6px 16px;border-radius:8px;font-size:.98em;font-weight:600;">🗄️ RAG / Memory</span>
  <span style="background:#f2f7fa;color:#2563eb;padding:6px 16px;border-radius:8px;font-size:.98em;font-weight:600;">📦 Supply Chain</span>
</div>

---

## 👤 关于

**jnnysec** — 安全工程师，正在从 Web 安全 / 代码审计转向 AI Agent 安全方向。

之前在 Web 安全领域做漏洞挖掘和代码审计，SQL 注入、反序列化、SSRF 那套非常熟。2026 年开始系统学习 AI 安全，发现底层逻辑惊人地相通——Prompt 注入就是 LLM 世界的 SQL 注入，Agent Tool Calling 就是新的命令执行。

当前主攻 Prompt Injection（直接注入与间接注入）、Jailbreak（越狱技术）、Agent 框架审计（Dify / LangChain 等）。博客记录学习笔记和攻防研究，所有内容均为个人理解，欢迎指正交流。
