---
layout: default
title: AI Agent 安全学习笔记
---

# 🛡️ AI Agent 安全学习笔记

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

## 👤 关于

**jnnysec** · CISSP / PMP /CISP / CISP-PTE / CAISP / ISO 27001 · ISC2 & CSA 会员

Web 安全出身，现转向 AI Agent 安全。主攻 Prompt Injection、Jailbreak、Agent 框架审计。博客记录学习与攻防研究。
