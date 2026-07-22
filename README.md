# Learning Hub Rules

The public rules repo for the **Learning Hub** — a private, browser-readable knowledge base of self-contained HTML "intuition blueprints" for a visual learner.

## What this repo is

Two repos make up the Learning Hub:

- **`learning-hub-rules`** (this repo, public) — the rules. Any AI agent fetches `CONVENTIONS.md` from here to learn how to build a blueprint.
- **`learning-hub`** (private) — the blueprints, shared CSS, scripts, and deploy pipeline. Deployed to Cloudflare Pages at `https://learning-hub-3qh.pages.dev/`.

This repo is public so that any agent — opencode, Claude Code, claude.ai, any future tool — can fetch the conventions via `raw.githubusercontent.com` without repo access.

## Files

| File | Purpose |
|------|---------|
| `CONVENTIONS.md` | **The single source of truth.** Self-contained: template skeleton, five patterns, quiz protocol, filing checklist. Everything an agent needs to build a blueprint. |
| `ARCHITECTURE.md` | The living design document. Full system context, design choices, lessons learned. Read this before making changes. |
| `recipes/blueprint.md` | Extended pedagogy notes (human reference; agents don't fetch this) |
| `recipes/quiz.md` | Full quiz protocol with calibration notes (human reference) |
| `recipes/lint.md` | Hub health checks: orphan blueprints, missing backlinks, contradictions (human reference) |
| `skills/learn/SKILL.md` | The `/learn` skill definition — thin wrapper that fetches `CONVENTIONS.md` |
| `template.html` | Full template with inlined CSS (human reference; agents build from the skeleton in `CONVENTIONS.md`) |

## Key URLs

- **Conventions (raw):** `https://raw.githubusercontent.com/ArturGR3/learning-hub-rules/main/CONVENTIONS.md`
- **Live hub:** `https://learning-hub-3qh.pages.dev/`
- **Manifest:** `https://learning-hub-3qh.pages.dev/manifest.json`
- **Shared CSS:** `https://learning-hub-3qh.pages.dev/assets/blueprint.css`

## For agents

Read `ARCHITECTURE.md` for full system context before making changes. Read `CONVENTIONS.md` for the operational rules agents follow. When you change how the system works, update `ARCHITECTURE.md` to record the design choice and rationale.
