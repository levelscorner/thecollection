# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

`thecollection` is the **public output repo** for The Collector pipeline.

It contains AI research reports generated automatically by `thecollector` pipeline:

```
thecollector (private, pipeline code)
    └── runs daily via cron
    └── crawls RSS + web via Crawl4AI
    └── scores + compresses articles
    └── generates report with Claude Code
    └── git pushes report here → thecollection (public, this repo)
```

This repo is read-only from a human perspective — reports are written by the pipeline, not manually.

## Directory Structure

```
thecollection/
├── reports/
│   ├── daily/          — one .md file per day  (YYYY-MM-DD-weekday.md)
│   └── weekly/         — one .md file per week (YYYY-MM-DD-WNN.md)
└── CLAUDE.md
```

## Report Format

Each report is a Markdown file with YAML frontmatter:

```markdown
---
date: 2026-03-08
day: Sunday
type: daily
generated: 2026-03-08T02:31:00Z
pipeline: crawl4ai -> preprocess -> claude-code
---

# AI Research Digest ...
## TL;DR
## Top Stories
## Research & Papers
## Tools & Releases
## Quick Takes
```

Weekly reports follow the same structure with additional sections (Week in Review, Industry Moves, Trends to Watch, Next Week Preview).

## How Reports Get Here

The `thecollector` pipeline runs on a cron schedule and pushes directly to this repo:

- Daily: 2:30 AM UTC (8:00 AM IST) — `reports/daily/`
- Weekly: 3:30 AM UTC Monday (9:00 AM IST) — `reports/weekly/`

The push happens from `/home/level/ws/thecollector/scripts/pipeline.sh` on the server.

## Publishing Options

### Option A — GitHub Pages (active if `_config.yml` exists)

Reports are served as a static site at `https://levelscorner.github.io/thecollection`.

To enable: add `_config.yml` at the repo root:

```yaml
title: The Collection
description: Daily and weekly AI research reports, generated automatically.
url: "https://levelscorner.github.io"
baseurl: "/thecollection"
theme: minima

collections:
  reports:
    output: true
    permalink: /reports/:path/

defaults:
  - scope:
      path: "reports"
    values:
      layout: post
```

Then in GitHub: Settings → Pages → Source: `main` branch, `/ (root)`.

### Option B — Export to blog / newsletter

Reports are plain Markdown — copy-paste directly into Ghost, Substack, Medium, or any static site generator (Hugo, Astro, Jekyll). No report files need to change for Hugo/Astro — the existing `reports/` structure maps cleanly to a content collection.

## SSH Push Access (Server Setup)

The pipeline server needs a deploy key that can push to this repo:

```bash
# Generate a dedicated deploy key on the server
ssh-keygen -t ed25519 -C "thecollector-pipeline" -f ~/.ssh/thecollection_deploy -N ""

# Add the public key to: GitHub → thecollection → Settings → Deploy keys (Allow write access)
cat ~/.ssh/thecollection_deploy.pub

# Configure SSH to use this key for thecollection only
cat >> ~/.ssh/config << 'EOF'
Host github-thecollection
    HostName github.com
    User git
    IdentityFile ~/.ssh/thecollection_deploy
EOF

# Re-clone using the alias
git clone git@github-thecollection:levelscorner/thecollection.git thecollection
```

## Constraints

- Do NOT modify files in `reports/` — they are pipeline output
- Do NOT change the `reports/daily/` and `reports/weekly/` directory structure — the pipeline expects it
- Do NOT add code — this is a content repo; code lives in `thecollector`
- To change report templates, edit `thecollector/scripts/generate_report.sh` (private repo)
