# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Hexo 7.1.1 static blog (Chinese language). Deployed to GitHub Pages at xobo.org.

## Commands

```bash
npm run server   # Local dev server
npm run build    # Generate static site (hexo generate)
npm run clean    # Remove generated files
npm run deploy   # Deploy to GitHub Pages
```

## Architecture

- **`_config.yml`** — Main Hexo config (theme: Hacker)
- **`_config.butterfly.yml`** — Butterfly theme config (alternate theme, extensive customization)
- **`source/_posts/`** — Blog posts organized in subdirectories by topic (Java/, ops/, database/, etc.)
- **`themes/Hacker/`** — Active theme
- **`scaffolds/post.md`** — Template for new posts
- **`draft/`** — Draft posts (not published)

## Post Frontmatter Format

```yaml
---
title: Post Title
date: YYYY-MM-DD HH:mm:ss
tags: []
categories:
  - Category Name
---
```

Some older posts include an `id` field. Use `<!-- more -->` for excerpt breaks.

## CI/CD

GitHub Actions (`.github/workflows/pages.yml`) builds and deploys on push to `master` using Node.js 20.
