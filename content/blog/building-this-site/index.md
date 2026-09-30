---
title: "How I Built This Site"
date: 2026-07-12
description: "Hugo + Hextra + Dev Containers + GitHub Pages + Claude Code."
tags: ["hugo", "github-pages", "claude-code", "devops", "containers", "github-actions", "web-development"]
categories: ["Engineering"]
---

## Stack

- **Hugo** — static site generator
- **Hextra** — theme ([imfing/hextra](https://github.com/imfing/hextra))
- **Dev Containers** — reproducible Hugo environment via Docker
- **GitHub Pages** — hosting, auto-deploys on push to `main`
- **Claude Code** — AI pair-programmer for templates and CSS

## Setup

```bash
mkdir my-personal-website && cd my-personal-website && git init
```

Created the project structure: `.devcontainer/devcontainer.json` for a containerized Hugo environment, `hugo.toml` for site config, and `content/` for pages. I followed [The Indie Coder's dev container setup guide](https://theindiecoder.cloud/posts/dev-container-setup-for-hugo/) for getting Hugo running inside VS Code with Docker; the container pulls in Hugo extended, Go, and Git so that the build environment is fully portable. Open the folder in your IDE, and it prompts "Reopen in Container" to run. You will need Docker Desktop running in the background for this to work.

Added Hextra as a git submodule:

```bash
git submodule add https://github.com/imfing/hextra.git themes/hextra
```

Note: some guides omit the full path. You need `/imfing/hextra.git`.

## Local Preview

```bash
hugo server --bind=0.0.0.0
```

Enter `http://localhost:1313/romina-yaps/` in your CLI to preview locally.

## Deployment

A GitHub Actions workflow builds and deploys on every push:

```yaml
on:
  push:
    branches:
      - main
```

The workflow checks out with `submodules: recursive`, runs `hugo --gc --minify`, and deploys to Pages.

Things I had to fix:

- **Pages source:** Go to repo **Settings > Pages** and switch source to **GitHub Actions**.
- **Deploy action:** The guide used `peaceiris/actions-gh-pages` (branch-based). I switched to `actions/deploy-pages` (native Pages). No extra branch is needed with this and works with the setting above.
- **Permissions:** The job needs `pages: write` and `id-token: write` to execute successfully.

## Customization

All overrides live in `layouts/` and `assets/css/custom.css` so as to never touch theme files directly. Hugo's template hierarchy makes this clean.

## Result

The site is live at [rominap22.github.io](https://rominap22.github.io).
