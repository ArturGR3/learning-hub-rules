# Learning Hub — Conventions

This is the tier-1 rules file. Every session in the hub loads it. Keep it short — it costs context every message. Recipes (tier-2) load on demand.

## Who the user is

A visual learner building understanding with AI assistance. The goal is *understanding*, not artifacts. AI makes it too easy to skip the learning; this system keeps learning in the loop.

Teach intuition-up: basic concepts → examples → intuition → precise terms. Simple English where it works, but *correct technical vocabulary* — simple ≠ dumbed-down. The user wants to grow their vocabulary of precise terms.

**Diagrams matter — the user is a visual learner.** Use them for relationships, spatial structures, transformations. Don't use them for definitions or linear prose. One per section unless the section is fundamentally visual. Every diagram uses the visual grammar established in the blueprint's legend.

## The hub

Two repos:

- **`learning-hub-rules`** (public, this repo) — `CONVENTIONS.md`, `recipes/`, `skills/`. Fetchable by any agent from `https://raw.githubusercontent.com/<you>/learning-hub-rules/main/...`.
- **`learning-hub`** (private) — `topics/*.html` (the blueprints), `assets/blueprint.css`, `log.md`, `index.html` (generated), `CLAUDE.md`, `AGENTS.md`, `scripts/`, `.github/workflows/`.

A blueprint is a self-contained HTML file at `topics/<name>.html`. Each blueprint teaches one topic, intuition-up, and links to its prerequisites and dependents. The substance is HTML — visuals, diagrams, worked examples — not markdown notes.

## When building or refining a blueprint

1. Fetch `recipes/blueprint.md` from this repo and follow it.
2. Use the template (linked from the recipe) as the structural starting point.
3. Fill every `<meta>` in the head: `source-chat`, `last-quizzed` (empty if new), `prerequisites`, `tags`, `created`, `last-updated`.
4. **Maintain cross-references (Karpathy model).** Before filing, scan `topics/`. If the new/edited blueprint relates to existing ones, link to them in the body *and edit those pages to link back*. One write touches multiple files. The graph is the structure — no `taxonomy.md`, no enforced hierarchy. Categories emerge as clusters of cross-links.
5. Append `log.md` in the private repo: `## [YYYY-MM-DD] {{ingest|refine|lint}} | {{topic}} — {{one-line summary}}`.
6. Push. The GitHub Action rebuilds `index.html` and deploys.

## When quizzing

Fetch `recipes/quiz.md`. The Socratic protocol probes the user's *model* — asks them to predict, explain, generalize — not fact recall. After a quiz, update `<meta name="last-quizzed">` in the blueprint and push. The index Action surfaces blueprints not quizzed in 2+ weeks as a spaced-retrieval nudge.

## When linting

Fetch `recipes/lint.md`. Look for: orphan blueprints (no inbound links), missing backlinks (A links to B but B doesn't link back), concepts mentioned in prose without their own blueprint, contradictions between pages. Append `log.md` with what was found and fixed.

## The metas

Every blueprint carries these in `<head>`:

- `source-chat` — URL of the chat that produced the blueprint (for reopening the session)
- `last-quizzed` — date of last Socratic quiz, empty if never
- `prerequisites` — comma-separated filenames of blueprints this one builds on
- `tags` — comma-separated cross-cutting labels
- `created`, `last-updated` — dates

## Recipes

- `recipes/blueprint.md` — how to build a good learning blueprint (template patterns, visual-learning constraint, pedagogy)
- `recipes/quiz.md` — Socratic quiz protocol
- `recipes/lint.md` — hub health checks

## Filing from a phone

Phone sessions in the claude.ai "Learning Hub" Project can generate artifacts but can't push to the repo. To file: copy the HTML out of the artifact, paste as a new file on github.com mobile, commit. The Action rebuilds and deploys. Cross-reference maintenance happens on the next laptop session.
