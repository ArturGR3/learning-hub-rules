# Recipe — Lint the Hub

This is a tier-2 recipe. Load it when the user asks to lint the hub, or periodically as a health check. The goal is to find where the graph has decayed.

## Why this exists

The hub grows by agents writing to it across many sessions. Bookkeeping slips. Cross-references go stale. Concepts get mentioned in prose without their own blueprint. Lint catches this before it compounds.

## What to check

Run all of these. Report findings in a list before fixing anything, so the user can prioritize.

### 1. Orphan blueprints

A blueprint with no inbound links — nothing in the hub links to it.

- For each `topics/*.html`, search all *other* blueprints for `<a href="{{filename}}">` or `<a class="xref" href="#{{slug}}">`.
- If zero inbound links, the blueprint is an orphan. Either it's genuinely new (file it and link from related blueprints) or it's been disconnected (add a link from the most related blueprint).

### 2. Missing backlinks

A → B but B doesn't link back to A. The Karpathy model requires bidirectional links.

- For each pair of blueprints, check that if A's `cross-refs` block lists B in "prereqs" or "leads to", then B's `cross-refs` block lists A in the complementary direction.
- If A lists B as a prerequisite, B should list A in "leads to". If they're peers, both should list each other.
- Fix by editing B to add the backlink.

### 3. Concepts mentioned without their own blueprint

A blueprint's prose mentions a concept that doesn't have its own page yet.

- Scan each blueprint's prose for technical terms that appear in `<strong>`, `<em>`, or as the subject of a sentence.
- For each, check whether a `topics/<slug>.html` exists for it.
- If not, flag it. The user decides whether to write a new blueprint for it (most concepts don't need their own page — only the ones that deserve their own intuition-up treatment).

### 4. Contradictions

Two blueprints state something incompatible.

- This is hardest to detect mechanically. Look for: the same term defined differently in two places, the same claim with different sign or direction, "Remember" callouts that would contradict each other if both were true.
- Report with citations to both pages. The user resolves which is correct (or whether they're actually talking about different cases).

### 5. Stale "last-quizzed"

A blueprint with `last-quizzed` more than 2 weeks ago, or empty.

- This is already surfaced by the generated `index.html`'s spaced-retrieval nudge, but lint should call it out explicitly so the user can decide to quiz or mark as "won't quiz."
- Don't auto-quiz. Just report.

### 6. Broken metas

Any blueprint missing required `<meta>`: `source-chat`, `last-quizzed`, `prerequisites`, `tags`, `created`, `last-updated`.

- Report which file is missing which meta. Fix by filling it in.

## After linting

- Append `log.md`: `## [YYYY-MM-DD] lint | — {{N issues found, M fixed}}`.
- For fixes that touched blueprints, push. The Action rebuilds the index.
- For issues deferred (user chose not to fix), note them in `log.md` so the next lint can see if they've compounded.
