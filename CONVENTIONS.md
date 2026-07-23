# Learning Hub — Conventions

This file is the single source of truth. Everything you need to build a blueprint is below — the template skeleton, the five patterns, the quiz protocol, the filing checklist. **Do not attempt to fetch any other URL from this repo** (recipes, template.html). If a fetch fails, you already have what you need. If you cannot fetch this file at all, ask the user to paste it.

## Who the user is

A visual learner building understanding with AI assistance. The goal is *understanding*, not artifacts. AI makes it too easy to skip the learning; this system keeps learning in the loop.

Teach intuition-up: basic concepts → examples → intuition → precise terms. Simple English where it works, but *correct technical vocabulary* — simple ≠ dumbed-down. The user wants to grow their vocabulary of precise terms.

**Diagrams matter — the user is a visual learner.** Use them for relationships, spatial structures, transformations. Don't use them for definitions or linear prose. One per section unless the section is fundamentally visual. Use consistent, meaningful colors across all diagrams in a blueprint — `treated` vs `control`, `observed` vs `counterfactual`, `evidence` vs `inference` — not arbitrary decoration. The same color should mean the same thing in every diagram on the page.

## The hub

- **`learning-hub`** (private) — `topics/*.html` (the blueprints), `assets/blueprint.css` (shared styling), `log.md`, `index.html` (generated). Deployed to Cloudflare Pages at `https://learning-hub-3qh.pages.dev/`.
- **`learning-hub-rules`** (public, this repo) — this file. (recipes/ and template.html also exist for human readers; agents don't need them.)

A blueprint is a self-contained HTML file at `topics/<name>.html`. Each teaches one topic, intuition-up, and links to its prerequisites and dependents. The substance is HTML — visuals, diagrams, worked examples — not markdown notes.

## How to build a blueprint

### Intuition-up ordering

Teach in this order, adapted to what the topic needs:

1. **The intuition first** — plain English, the metaphor or mental model, no jargon. The "why this exists" — what problem does the concept solve?
2. **Examples** — concrete before abstract. The most familiar case first, then a second from a different angle if the concept has multiple modes.
3. **The precise terms** — technical vocabulary, anchored to the intuition above.
4. **Why it's shaped this way** — the structural insight, the trade-off the shape embodies.

Not every section needs all four. The order is a default, not a law.

### The visual-learning constraint

**Use a diagram for:** how parts relate (dependency graph, data flow), spatial structures (trees, hierarchies, overlapping distributions), transformations (one state becoming another), comparisons where shape matters (two distributions, before/after).

**Don't use a diagram for:** definitions (a sentence is sharper), linear prose (a diagram of "step 1 → step 2" is decoration), anything you could explain equally well in one sentence.

One per section unless the section is fundamentally visual. Use consistent colors across all diagrams — the same color means the same thing throughout the page.

**SVG text must fit inside its containing box.** JetBrains Mono advance width is ~0.6em — at 12.5px that's ~7.5px/char, at 10.5px it's ~6.3px/char. Leave ~8px padding each side. When a label is too long for its box, shorten the label or use the smaller font size — don't widen the box without checking downstream layout.

### The four template patterns

Every blueprint follows these. They are the load-bearing structure.

**1. Modular sections** — each section opens with a chapter label and a title that is a *claim*, not a label:

- ✅ "Why the average difference between groups lies"
- ✅ "Compare like with like"
- ❌ "Introduction to matching"
- ❌ "Background"

The title tells the reader what they'll understand by the end of the section. Sections are modular — no fixed order. Each picks from available blocks (prose, diagram, formula+where, key) as the topic needs.

**2. Formula + "reading it"** — every formula is immediately followed by a plain-English breakdown that defines **every symbol, every time**. Vocabulary is re-stated, never assumed from earlier sections:

```html
<div class="formula">
  <span class="lab">{{formula label}}</span>
  <div class="eq">{{$$ formula $$}}</div>
</div>
<div class="where">
  <span class="wlab">reading it</span>
  <ul>
    <li><b>{{symbol}}</b> — {{plain-English definition, one sentence}}</li>
    <li><b>{{symbol}}</b> — {{plain-English definition, one sentence}}</li>
    <li><b>{{whole expression}}</b> — {{what it says in words}}</li>
  </ul>
</div>
```

This is the single highest-value pattern — the anti-false-aha mechanism. A reader who skips the formula can still read the breakdown; a reader who reads both actually understands the formula.

**3. Per-section "Remember"** — at the end of each section, a one-sentence distillation of the load-bearing insight. Often encodes the corrected misconception:

```html
<div class="key">
  <span class="lab">Remember</span>
  <p>{{one sentence — the thing you'd whisper to a friend.}}</p>
</div>
```

"I thought X, but actually Y" lives here when the section is about a misconception the user held.

**4. "In one breath" closing** — a single-paragraph synthesis of the whole topic. Each section's load-bearing insight strung together into one coherent statement:

```html
<section class="closing">
  <div class="chno">In one breath</div>
  <p class="breath">{{one paragraph — the whole topic, synthesized.}}</p>
</section>
```

This is the "do I actually understand this?" test. If you can write it, you understand it. If you can't, the blueprint isn't done.

### HTML template skeleton

Copy this structure. CSS uses an absolute URL so it renders in artifact preview AND when deployed:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="source-chat" content="">
<meta name="last-quizzed" content="">
<meta name="prerequisites" content="">
<meta name="tags" content="">
<meta name="created" content="">
<meta name="last-updated" content="">
<title>{{TOPIC}} — intuition blueprint</title>
<link rel="stylesheet" href="https://learning-hub-3qh.pages.dev/assets/blueprint.css">
<!-- For math, add MathJax: -->
<script>
window.MathJax = { tex: { inlineMath: [['\\(','\\)']], displayMath: [['$$','$$']] } };
</script>
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js" async></script>
</head>
<body>
<div class="page">

  <header class="title">
    <div class="eyebrow">Intuition Blueprint</div>
    <h1>{{central question}}<span class="subtitle">{{topic}}</span></h1>
    <div class="meta">
      <span>created: {{DATE}}</span>
      <span>last-updated: {{DATE}}</span>
      <span class="tags">{{tags}}</span>
    </div>
  </header>

  <section>
    <div class="chno">{{01 — framing}}</div>
    <h2>{{claim-title — what they'll understand by the end}}</h2>
    <p>{{Plain-English intuition. The metaphor. The "why this exists".}}</p>
    <div class="diagram">
      <!-- <svg viewBox="0 0 540 120"> ... </svg> -->
      <div class="caption">{{Fig. N — what to look at}}</div>
    </div>
    <div class="formula">
      <span class="lab">{{formula label}}</span>
      <div class="eq">{{$$ formula $$}}</div>
    </div>
    <div class="where">
      <span class="wlab">reading it</span>
      <ul>
        <li><b>{{symbol}}</b> — {{plain-English definition}}</li>
      </ul>
    </div>
    <div class="key">
      <span class="lab">Remember</span>
      <p>{{one-sentence distillation}}</p>
    </div>
  </section>

  <!-- More sections follow the same pattern.
       Inline cross-references: <a class="xref" href="#secNN">§NN</a> -->

  <section class="closing">
    <div class="chno">In one breath</div>
    <p class="breath">{{one-paragraph synthesis — each section's insight strung together}}</p>
  </section>

  <div class="cross-refs">
    <span class="label">prereqs</span>
    <a href="{{prerequisite.html}}">{{prerequisite}}</a>
    <span class="label" style="margin-left:20px;">leads to</span>
    <a href="{{dependent.html}}">{{dependent}}</a>
  </div>

  <footer>
    <span>blueprint v1 · last-quizzed: {{date or "—"}}</span>
    <span>source: <a href="{{chat-url}}">{{chat}}</a></span>
  </footer>

</div>
</body>
</html>
```

### The metas

Fill all of these in `<head>`:

- `source-chat` — URL of the chat that produced the blueprint (for reopening the session)
- `last-quizzed` — date of last Socratic quiz, empty if never
- `prerequisites` — comma-separated filenames of blueprints this one builds on
- `tags` — comma-separated cross-cutting labels
- `created`, `last-updated` — dates

### Cross-references (Karpathy model)

The graph is the structure. Before filing a new or edited blueprint:

1. If you can see existing blueprints, link to related ones in the body AND in the `cross-refs` block. Edit those blueprints to link back. One write touches multiple files.
2. Categories emerge as clusters of cross-links. There is no `taxonomy.md` and no enforced hierarchy.
3. **If you can't see existing blueprints** (phone session, private repo unreachable): produce a standalone blueprint. **Do not create `<a>` links to blueprints you cannot verify exist** — a link to a non-existent file is a dangling reference that breaks the graph. Instead, list related concepts as HTML comments (`<!-- future blueprint: dns.html -->`) for the user to wire up from laptop. The user maintains the graph from laptop.

**Manifest (best-effort):** `https://learning-hub-3qh.pages.dev/manifest.json` lists all blueprints with metadata (filename, title, tags, prerequisites, last-quizzed). If you can fetch it, use it for cross-referencing. If your environment blocks it, skip — the blueprint works standalone.

### What one blueprint covers

A blueprint mirrors what one learning session produced. A focused session on DNS → one focused blueprint. A weeks-long session on causal inference → one field-through-line with multiple sections. The blueprint mirrors the session; the graph handles connections to the rest of the hub. Don't split a session's output artificially; don't merge unrelated sessions.

### Filing checklist

- [ ] Template used, all `<meta>` filled
- [ ] Each section has a claim-title, not a label-title
- [ ] Every formula has a "reading it" block defining every symbol
- [ ] Each section has a "Remember" callout
- [ ] "In one breath" closing written (the understanding test)
- [ ] Diagrams used only where they earn their place (visual-learning constraint)
- [ ] Cross-references added (or noted as needed for laptop follow-up)

## How to quiz

The Socratic protocol probes the *model*, not fact recall. Reading a blueprint produces a feeling of understanding that may be false; the quiz forces the user to do the connecting themselves.

1. **Read the blueprint** the user names. The "Remember" callouts and "In one breath" are the densest source of load-bearing insights.
2. **Ask one question at a time.** Wait for the answer before asking the next. Don't dump a list.
3. **Probe the model, not recall.** Good questions:
   - "Predict what happens if {{X is changed}}." — forces the user to use the model forward
   - "Explain why {{Y}} is the case, in your own words." — forces synthesis
   - "Generalize: would this still hold if {{Z}}?" — tests the boundaries
   - "Here's a scenario: {{...}}. What would you expect, and why?" — applies to a new case
   - "Compare {{A}} and {{B}} — what's the actual difference, and when does it matter?" — forces distinction

   Bad questions (avoid): "What is the definition of X?", "True or false: ...", "List the steps in ..." — these test recall, not understanding.

4. **After each answer:** say *what specifically* was right or wrong (not just "correct" / "incorrect"), then build the next question on what they showed they understood.
5. **Use the blueprint's diagrams.** If the user is shaky, point them at a specific diagram and ask them to explain what it shows in their own words. Visual learners consolidate through the visuals.
6. **End with a synthesis question** — "Sum up the whole topic in a paragraph, as if explaining to someone who hasn't read this." Their answer tells both of you whether the model is integrated or fragmentary.

After the quiz: update `<meta name="last-quizzed">` in the blueprint and push. The index resets the spaced-retrieval nudge. If you can't push (phone session), tell the user the date to update.

**Calibration:** if the user gets everything right, the next quiz should push toward edge cases and generalizations. If they struggle, anchor back to the intuition and diagrams before pushing to precise terms. The quiz is a learning tool, not an exam.

## Filing from a phone (claude.ai)

Phone sessions can generate blueprints but can't push to the private repo. The flow:

1. Build the blueprint HTML following this file.
2. Output the complete HTML for the user to copy.
3. User goes to github.com mobile → `ArturGR3/learning-hub` repo → `topics/` → Add file → Create new file → name it `<slug>.html` → paste → commit.
4. The GitHub Action rebuilds `index.html` and deploys to Cloudflare Pages.
5. Cross-reference maintenance, `log.md` append, and `<meta name="last-quizzed">` updates happen on the next laptop session.

The artifact preview should show correct styling via the absolute CSS URL. If it doesn't render, the deployed version will — the stylesheet loads from the hub's Pages URL.

## Filing from a laptop (opencode, Claude Code)

1. Build the blueprint following this file.
2. Save to `topics/<slug>.html` in the `learning-hub` repo.
3. Maintain cross-references: scan `topics/`, link related blueprints both directions. Edit linked blueprints to add back-links.
4. Run `scripts/file-blueprint.sh topics/<slug>.html "{{topic}} — {{one-line summary}}"` from the repo root. This validates the blueprint, appends `log.md`, stages source files, commits, and pushes. Don't run `build-index.py` locally — the GitHub Action regenerates `index.html` and `manifest.json` on push.
5. The Action rebuilds and deploys (~30s).


