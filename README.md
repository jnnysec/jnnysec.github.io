# jnnysec.github.io

AI Agent security notes published with GitHub Pages and Jekyll.

## Repository

- Site: <https://jnnysec.github.io/>
- GitHub Pages source: `new-jekyll-style` branch, repository root
- Main content: Markdown posts in `_posts/`
- Homepage: `index.md`
- Shared layout and CSS: `_layouts/default.html`
- Site metadata: `_config.yml`

## Local Preview

Install the GitHub Pages Jekyll stack:

```bash
bundle install
```

Run the site locally:

```bash
bundle exec jekyll serve --livereload
```

Open <http://127.0.0.1:4000/>.

## Writing Posts

Create new posts in `_posts/` with this filename format:

```text
YYYY-MM-DD-post-slug.md
```

Each post should start with Jekyll front matter:

```markdown
---
layout: default
title: Article title
tags: [AI安全, Prompt Injection]
---
```

The homepage automatically lists files from `_posts/`.

## Publish Flow

```bash
git status
git add .
git commit -m "Add new post"
git push origin new-jekyll-style
```

GitHub Pages builds from the root of `new-jekyll-style`.
