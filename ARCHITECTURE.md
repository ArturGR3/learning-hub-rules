# Learning Hub — Architecture

**Last updated:** 2026-07-22
**Purpose:** The living design document for the Learning Hub. Read this before making changes. Update this when you change how the system works — record the design choice and rationale.

---

## 1. Purpose

The Learning Hub is a private, browser-readable knowledge base of self-contained HTML "intuition blueprints." The user is a visual learner who builds understanding with AI assistance. The goal is *understanding*, not artifacts — AI makes it too easy to skip the learning; this system keeps learning in the loop.

Every blueprint teaches a concept **intuition-up** (basic concepts → examples → intuition → precise terms) and is filed to a private GitHub repo that auto-deploys to Cloudflare Pages. The hub grows by agents writing to it across many sessions, from multiple environments. A spaced-retrieval nudge surfaces blueprints not quizzed in 2+ weeks.

The design principle: **everything built with AI also teaches.** When you hit a concept mid-build, you pause, learn it, generate a blueprint, file it, then resume.

---

## 2. Two repos

### `learning-hub-rules` (public)
- **GitHub:** `ArturGR3/learning-hub-rules`
- **Purpose:** The rules. Source of truth for how to build blueprints, quiz, and lint.
- **Served via:** `raw.githubusercontent.com/ArturGR3/learning-hub-rules/main/...` (no deploy needed — raw serves directly from the repo)
- **Files:** `CONVENTIONS.md` (the single source of truth), `recipes/` (human reference), `skills/learn/SKILL.md` (the `/learn` skill), `template.html` (human reference)

### `learning-hub` (private)
- **GitHub:** `ArturGR3/learning-hub` (private)
- **Purpose:** The actual blueprints and infrastructure.
- **Deployed to:** Cloudflare Pages at `https://learning-hub-3qh.pages.dev/`
- **Cloudflare project name:** `learning-hub` (subdomain has random suffix `3qh` for privacy-by-obscurity)
- **Cloudflare Account ID:** `7618e6ac94880318d4f14a1ddfbffc3c`
- **GitHub secrets:** `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`
- **Files:** `topics/*.html` (blueprints), `assets/blueprint.css` (shared styling), `scripts/build-index.py` (index + manifest generator), `.github/workflows/deploy.yml` (GitHub Action), `index.html` + `manifest.json` (generated), `log.md` (append-only log), `CLAUDE.md` + `AGENTS.md` (fetch pointers)

### Why the split
The rules are public so any agent can fetch them via `raw.githubusercontent.com` without repo access. The content is private so the blueprints (and the chat URLs in their metas) stay private. claude.ai can't fetch private repos — this split makes the rules reachable from any environment.

---

## 3. Memory mechanism

The system has no central API, no database, no plugin. Its memory is a single file fetched at the start of every session. This is the **fetch pointer** pattern: a one-line instruction that says "fetch this URL and follow it." That URL is `CONVENTIONS.md`, and it's self-contained — everything an agent needs to build a blueprint is in that one file.

The pointer appears in three places (one per entry point):

| Where | Loaded by | When |
|-------|-----------|------|
| `CLAUDE.md` / `AGENTS.md` in the private repo | Claude Code / opencode | When working inside the `learning-hub` repo |
| claude.ai "Learning Hub" Project instructions | claude.ai | Every chat in the Project |
| `/learn` skill (`SKILL.md`) | opencode / Claude Code | When the user types `/learn` in any directory |

All three contain the same one line:

```
Fetch https://raw.githubusercontent.com/ArturGR3/learning-hub-rules/main/CONVENTIONS.md and follow it before acting.
```

The claude.ai Project also includes guardrail instructions (use exact class names, don't inline CSS, don't link to non-existent blueprints) because CONVENTIONS.md alone isn't always followed strictly enough — the Project instructions have system-level authority.

### Why self-contained
Originally CONVENTIONS.md was tier-1 (short) with recipes as tier-2 (fetched on demand). This broke in claude.ai, which blocks constructed `raw.githubusercontent.com` URLs to non-indexed repos. The agent fetched CONVENTIONS.md (trusted, from Project instructions) but couldn't fetch anything it pointed to. The fix: inline everything into CONVENTIONS.md. One fetch = everything needed.

---

## 4. Blueprint structure

Every blueprint follows five patterns, defined in CONVENTIONS.md:

1. **Visual grammar legend** — one line in the header, establishes colors/symbols used by every diagram. Taught once, used N times.
2. **Modular sections** — each section has a chapter label (`.chno`) and a *claim-title* (`h2`), not a label-title. Sections pick from available blocks (prose, diagram, formula+where, key) as the topic needs.
3. **Formula + "reading it"** — every formula is followed by a plain-English breakdown (`.where`) defining every symbol, every time. The anti-false-aha mechanism.
4. **Per-section "Remember"** (`.key`) — one-sentence distillation at the end of each section. Not optional.
5. **"In one breath" closing** (`.closing` / `.breath`) — single-paragraph synthesis. The "do I actually understand this?" test.

### Required class names
Blueprints use these exact CSS class names because they map to `assets/blueprint.css`:
`header.title`, `.eyebrow`, `h1.subtitle`, `.meta`, `.tags`, `.legend`, `.swatch`, `.dot`, `.chno`, `h2`, `.diagram`, `.caption`, `.formula`, `.lab`, `.eq`, `.where`, `.wlab`, `.key`, `.closing`, `.breath`, `.cross-refs`, `footer`

Do not invent custom class names — the shared CSS will not style elements it doesn't recognize.

### Required `<meta>` tags
Every blueprint's `<head>` contains: `source-chat`, `last-quizzed`, `prerequisites`, `tags`, `created`, `last-updated`.

### CSS linking
Blueprints link the shared stylesheet via **absolute URL** (not relative, not inlined):
```html
<link rel="stylesheet" href="https://learning-hub-3qh.pages.dev/assets/blueprint.css">
```
This makes the blueprint render correctly in both artifact previews (claude.ai) and when deployed.

### Why absolute CSS URL
Relative `/assets/blueprint.css` doesn't render in claude.ai artifact previews (the file is in a private repo). Agents responded by inlining their own CSS, which created divergent designs that don't match the hub. The absolute URL (pointing to the deployed pages.dev) works in both preview and deployment — no inlining needed.

---

## 5. Deploy pipeline

Every push to `main` that touches `topics/**`, `assets/**`, or `scripts/**` triggers the GitHub Action (`.github/workflows/deploy.yml`):

1. **`validate.py` runs** — checks all `topics/*.html` against conventions (required metas, absolute CSS URL, required class names, no dangling cross-refs). Fails the build if violations found.
2. **`build-index.py` runs** — scans all `topics/*.html`, extracts `<meta>` tags and `<a href>` links between blueprints
3. **Generates two files:**
   - `index.html` — visual landing page (lists blueprints, shows quiz nudges, shows graph edges)
   - `manifest.json` — machine-readable metadata (filename, title, tags, prerequisites, last-quizzed, links-to, linked-by)
4. **Commits them back** with `[skip ci]` (no infinite loop)
5. **Deploys to Cloudflare Pages** via `cloudflare/pages-action@v1`

Total time: ~30 seconds.

### `validate.py` — mechanical convention enforcement
Replaces prompt-based "don't do X" instructions with deterministic CI checks. Catches: missing metas, relative CSS URLs, missing required class names, dangling cross-references to non-existent blueprints. Reference pages (`<meta name="blueprint-type" content="reference">`) are exempt from teaching-specific classes (legend, formula, where, key, closing, breath). Deploy fails on validation errors — immediate feedback.

### `file-blueprint.sh` — filing hygiene
A script agents run instead of doing 5 manual steps: `./scripts/file-blueprint.sh topics/<slug>.html "<log-message>"`. Validates, appends log.md, stages only source files (never generated index.html/manifest.json), commits, pushes. Prevents: running build-index.py locally, committing generated files, forgetting log.md.

### `manifest.json` — why it exists
claude.ai agents can't read the private repo's `topics/` directory. The manifest (deployed to the public pages.dev URL) lets agents that can fetch it see what blueprints already exist, for cross-referencing. If they can't fetch it, blueprints work standalone — cross-ref maintenance happens on laptop.

---

## 6. Cross-references (Karpathy model)

There is no `taxonomy.md` and no enforced hierarchy. Categories emerge as clusters of cross-links. The graph is the structure.

When filing a new or edited blueprint:
1. If you can see existing blueprints (via `topics/` or `manifest.json`), link to related ones in the body AND in the `cross-refs` block. Edit those blueprints to link back. One write touches multiple files.
2. If you can't see existing blueprints (phone session): produce a standalone blueprint. **Do not create `<a>` links to blueprints you cannot verify exist.** Use HTML comments (`<!-- future blueprint: dns.html -->`) instead.

### Why no dangling links
In claude.ai testing, an agent invented 4 cross-links to blueprints that didn't exist (dns.html, dhcp.html, etc.). These are dangling references that break the graph and would be flagged by a lint. The rule: only link what you can verify.

---

## 7. Design choices and rationale

### Why three modes (teach / crystallize / build)
The original skill assumed one flow: teach → build → file. Real usage revealed three distinct cases. **Teach** is a dedicated learning session ("teach me DNS"). **Crystallize** is capturing learnings from a work session ("summarize what we covered") — the teaching already happened embedded in the work; the agent outlines topics, checks understanding, fills gaps, then generates blueprints with "Remember" callouts from real misconceptions. **Build** is just generating an artifact. The mode is inferred from the user's request — no upfront questions. Each mode has its own natural gate (teaching confirmation, outline agreement, or direct generation).

### Why mechanical enforcement over prompts
Prompts are non-deterministic. Three claude.ai test runs showed agents violating explicit instructions (inlined CSS, custom class names, dangling cross-links). `validate.py` catches these mechanically in CI — deploy fails on violation. `file-blueprint.sh` handles filing hygiene mechanically. The SKILL.md gets simpler (fewer "don't do X" rules) because the scripts enforce them. Agents get more freedom (try anything, validate.py catches mistakes) rather than more rules.

### Why CONVENTIONS.md is self-contained
Originally tier-1 (short) with tier-2 recipes fetched on demand. Broke in claude.ai (blocks constructed raw URLs to non-indexed repos). Fix: inline everything. One fetch = everything. See section 3.

### Why no recipe fetches
Even mentioning "optional" recipes in CONVENTIONS.md caused agents to burn turns attempting blocked fetches and explaining why they couldn't. Removed all mentions. Recipes remain in the repo for human readers and for opencode/Claude Code agents that can browse the repo directly.

### Why absolute CSS URL
Relative path doesn't render in claude.ai artifact preview. Agents inlined CSS as a workaround, creating divergent designs. Absolute URL to the deployed pages.dev works in both preview and deployment. See section 4.

### Why manifest.json
claude.ai can't read the private repo. The manifest (on the public pages.dev URL) is a best-effort bridge for cross-referencing. If an agent can fetch it, it knows what blueprints exist. If not, the blueprint works standalone. See section 5.

### Why no Cloudflare Access (auth gate)
Cloudflare Access (Zero Trust free tier) would require a payment method. The URL is private-by-obscurity (random suffix `3qh`, not indexed). Auth was skipped to avoid the payment requirement. Can be added later if needed.

### Why the Karpathy cross-ref model
No taxonomy means no maintenance burden and no forced hierarchy. Categories emerge naturally as link clusters. Bidirectional links ensure the graph is traversable from any node. One write touches multiple files — this is by design.

### Why [skip ci] on Action commits
The Action commits `index.html` and `manifest.json` back to the repo after rebuilding them. Without `[skip ci]`, this commit would trigger the Action again, creating an infinite loop. The `[skip ci]` tag in the commit message tells GitHub to skip the workflow.

---

## 8. Lessons learned

### Test run 1 (claude.ai, pi-hole blueprint)
**Problem:** Agent couldn't fetch `recipes/blueprint.md` or `template.html` (claude.ai blocks constructed raw URLs to non-indexed repos). Built from CONVENTIONS.md alone but couldn't match the real template or do cross-reference maintenance.
**Fix:** Made CONVENTIONS.md fully self-contained. Added `manifest.json` for cross-referencing.

### Test run 2 (claude.ai, pi-hole blueprint, updated instructions)
**Problems:**
- Agent still tried to fetch recipes (the "Optional reference files" section invited it)
- Agent inlined CSS (relative path doesn't render in preview)
- Agent invented 4 cross-links to non-existent blueprints
- Agent used custom class names instead of the template's (CSS wouldn't style them)
- Agent skipped per-section "Remember" callouts
**Fixes:** Removed all recipe fetch suggestions. Switched to absolute CSS URL. Added rule: no links to non-existent blueprints. Prepared fuller Project instructions with guardrails.

### Test run 3 (claude.ai, pi-hole blueprint, final Project instructions — but only the one-line fetch pointer was pasted)
**Result:** Agent behavior improved (no rogue fetches, HTML comments for future blueprints). But still inlined CSS and used custom class names — because the full Project instructions (with guardrails) hadn't been pasted into the Project settings yet.
**Conclusion:** The full Project instructions are critical. CONVENTIONS.md alone isn't enforced strongly enough by claude.ai agents. The Project instructions have system-level authority.

### Recurring git friction
Every time `build-index.py` runs locally, it generates `index.html` and `manifest.json` as untracked files. When the bot commits its own version on GitHub, `git pull --rebase` fails because of the untracked files. Fix: `rm -f index.html manifest.json` before pulling, or don't run `build-index.py` locally unless testing the script.

---

## 9. File reference

### This repo (`learning-hub-rules`)

| File | What it does | Who uses it |
|------|-------------|-------------|
| `CONVENTIONS.md` | The single source of truth. Self-contained rules for building blueprints. | Fetched by every agent at the start of every session |
| `ARCHITECTURE.md` | This document. Living design doc with rationale. | Read by agents and humans before making changes |
| `README.md` | Brief front page for the GitHub repo | Shown on the GitHub repo page |
| `CLAUDE.md` | Agent instructions for working in this repo | Read by Claude Code when working in this repo |
| `AGENTS.md` | Same as CLAUDE.md | Read by opencode when working in this repo |
| `recipes/blueprint.md` | Extended pedagogy notes | Human reference (agents don't fetch this) |
| `recipes/quiz.md` | Full quiz protocol with calibration | Human reference |
| `recipes/lint.md` | Hub health checks | Human reference |
| `skills/learn/SKILL.md` | The `/learn` skill definition — thin wrapper that fetches CONVENTIONS.md | Installed to `~/.agents/skills/` and `~/.claude/skills/` |
| `template.html` | Full template with inlined CSS | Human reference (agents build from the skeleton in CONVENTIONS.md) |

### Private repo (`learning-hub`)

| File | What it does |
|------|-------------|
| `topics/*.html` | The blueprints |
| `assets/blueprint.css` | Shared stylesheet, linked by every blueprint via absolute URL |
| `scripts/validate.py` | Convention enforcement — checks metas, CSS URL, class names, cross-refs | Runs in CI before build-index.py |
| `scripts/file-blueprint.sh` | Filing script — validate → log → stage → commit → push | Agents run this instead of manual steps |
| `scripts/build-index.py` | Scans `topics/`, generates `index.html` + `manifest.json` | Runs in CI on push |
| `.github/workflows/deploy.yml` | Push → rebuild → deploy to Cloudflare Pages |
| `index.html` | Generated (not hand-maintained) |
| `manifest.json` | Generated (not hand-maintained) |
| `log.md` | Append-only chronological record |
| `CLAUDE.md` / `AGENTS.md` | One-line fetch pointers to CONVENTIONS.md |

### Installed skills

| Location | Used by |
|----------|---------|
| `~/.agents/skills/learn/SKILL.md` | opencode |
| `~/.claude/skills/learn/SKILL.md` | Claude Code |

When the rules repo copy (`skills/learn/SKILL.md`) is updated, the installed copies need manual re-sync:
```bash
cp learning-hub-rules/skills/learn/SKILL.md ~/.agents/skills/learn/SKILL.md
cp learning-hub-rules/skills/learn/SKILL.md ~/.claude/skills/learn/SKILL.md
```

---

## 10. Status

### Built and working
- Two-repo architecture (public rules, private content)
- `CONVENTIONS.md` — self-contained, fetched by agents
- `/learn` skill — three modes (teach / crystallize / build), installed for opencode and Claude Code
- GitHub Action — validate → rebuild index + manifest → deploy to Cloudflare Pages
- `validate.py` — mechanical convention enforcement (metas, CSS URL, class names, cross-refs)
- `file-blueprint.sh` — filing hygiene script (validate → log → stage → commit → push)
- `manifest.json` — public metadata for cross-referencing
- `build-index.py` — scans topics, generates index and manifest
- 3 blueprints in the hub: `causal-inference.html`, `simple-regression.html`, `system-architecture.html`
- Cloudflare Pages deploy verified (both via Action and manual wrangler)

### Tested
- **Laptop flow (opencode)** — end-to-end test completed: opencode + DeepSeek V4 Pro
  built an IPv4-vs-IPv6 blueprint, filed it, pushed, GitHub Action validated + built
  index + deployed to Cloudflare Pages. Full cycle: 2m 39s, $0.02. validate.py passed
  in CI. Blueprint live and in manifest. Test blueprint cleaned up after verification.
- **Laptop flow (Claude Code)** — partial test via Herdr. Skill loaded correctly,
  conventions fetched, mode inferred correctly. Permission prompts created friction
  in the Herdr-automated test setup but don't affect normal interactive use.
- **Parallel skill testing** — 5 agents tested simultaneously via Herdr (3 Claude Code,
  2 opencode). All agents that processed prompts followed the SKILL.md correctly:
  right mode inference, right flow steps, no premature questions, no plan mode friction.

### Not yet tested
- **Phone flow (claude.ai)** — not tested. Will test once there are more blueprints
  in the hub and the system has been used in real sessions. Don't over-engineer before
  real usage reveals what actually matters.
- **claude.ai Project instructions** — the full guardrail instructions have been
  prepared but not yet pasted into the Project settings (only the one-line fetch
  pointer is there). Paste when ready to test the phone flow.

### Known issues
- **Agent sometimes skips `file-blueprint.sh`** — in the e2e test, the agent used
  `git pull --rebase && git push` manually instead of the filing script. Outcome was
  correct (right commit message, log.md appended, only source files committed) but
  the script also runs `validate.py` before committing — the agent skipped local
  validation and relied on CI. SKILL.md updated to make the script instruction more
  prominent and explicit ("Always use this script. Do not run git commands manually.")
- **Lint** — `recipes/lint.md` describes health checks (orphan blueprints, missing
  backlinks, contradictions). Worth running once there are 4+ blueprints.
