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

Web 安全 / 代码审计背景，正在转向 AI Agent 安全方向。主攻 Prompt Injection、Jailbreak、Agent 框架审计。记录学习笔记与攻防研究。
