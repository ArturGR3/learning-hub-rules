# Recipe — Building a Learning Blueprint

This is a tier-2 recipe. Load it when building or refining a blueprint. It describes the pedagogy, the template patterns, and the visual-learning constraint.

## The goal

A blueprint teaches intuition-up so the user actually understands, not just feels they understand. The user is a visual learner. The substance is HTML — visuals, diagrams, worked examples — not markdown notes.

## Intuition-up ordering

Teach in this order, adapted to what the topic needs:

1. **The intuition first** — plain English, the metaphor or mental model, no jargon. The "why this exists" — what problem does the concept solve?
2. **Examples** — concrete before abstract. The most familiar case first, then a second from a different angle if the concept has multiple modes.
3. **The precise terms** — introduce technical vocabulary, anchored to the intuition above. Simple English where it works, but *correct technical vocabulary*. Simple ≠ dumbed-down. The user wants to grow their vocabulary of precise terms.
4. **Why it's shaped this way** — the structural insight, the trade-off the shape embodies, what breaks if you do it differently.

Not every section needs all four. Pick what the topic needs. The order is a default, not a law.

## The visual-learning constraint

**Diagrams are for relationships, spatial structures, transformations.** A diagram earns its place when the concept is fundamentally visual — when seeing it once replaces reading it three times.

**Don't use a diagram for:**
- Definitions (a sentence is sharper)
- Linear prose (a diagram of "step 1 → step 2 → step 3" is decoration)
- Anything you could explain equally well in one sentence

**Do use a diagram for:**
- How parts relate (the lookup chain, the dependency graph, the data flow)
- Spatial structures (a tree, a hierarchy, overlapping distributions)
- Transformations (one state becoming another, a quantity moving through a function)
- Comparisons where shape matters (two distributions, before/after)

One per section unless the section is fundamentally visual. Every diagram uses the visual grammar established in the blueprint's legend (see pattern 1 below).

## The five template patterns

Every blueprint uses the template at `learning-hub-rules/template.html` (or the equivalent path in your hub). The template bakes in these five patterns. Follow them.

### 1. Visual grammar legend

One line in the header. Establishes the colors and symbols every diagram in the page will use.

```html
<div class="legend">
  <span class="label">visual grammar</span>
  <span class="swatch"><span class="dot" style="background:#1F7A6D;"></span><b>{{concept A}}</b></span>
  <span class="swatch"><span class="dot" style="background:#B23A6E;"></span><b>{{concept B}}</b></span>
  <span class="swatch"><span class="dot dashed"></span><b>{{counterfactual / imputed}}</b></span>
</div>
```

The legend is taught once, used N times. A reader who has learned the grammar can scan every diagram in the page without re-reading captions. Pick colors that carry meaning — `treated` vs `control`, `observed` vs `counterfactual`, `input` vs `output` — not arbitrary decoration.

### 2. Modular sections

Sections are modular — no fixed order. Each section picks from the available blocks (prose, diagram, formula+where, key) as the topic needs. Not all sections need all blocks. A definition section might be one paragraph of prose. A technique section might have prose + diagram + formula + key. Order is topic-driven.

Each section opens with a chapter label and a title that is a *claim*, not a label:

- ✅ "Why the average difference between groups lies"
- ✅ "Compare like with like"
- ❌ "Introduction to matching"
- ❌ "Background"

The title tells the reader what they'll understand by the end of the section.

### 3. Formula + "reading it"

Every formula is immediately followed by a plain-English breakdown that defines **every symbol, every time**. Vocabulary is re-stated, never assumed from earlier sections.

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

This is the single highest-value pattern. The anti-false-aha mechanism. A reader who skips the formula can still read the breakdown; a reader who reads both actually understands the formula.

### 4. Per-section "Remember"

At the end of each section, a one-sentence distillation of the load-bearing insight. Often encodes the corrected misconception.

```html
<div class="key">
  <span class="lab">Remember</span>
  <p>{{one sentence — the thing you'd whisper to a friend.}}</p>
</div>
```

This is the misconception-fighting mechanism, distributed per-section rather than as one global block. "I thought X, but actually Y" lives here when the section is specifically about a misconception the user held.

### 5. "In one breath" closing

A single-paragraph synthesis of the whole topic. Each section's load-bearing insight strung together into one coherent statement.

```html
<section class="closing">
  <div class="chno">In one breath</div>
  <p class="breath">{{one paragraph — the whole topic, synthesized.}}</p>
</section>
```

This is the "do I actually understand this?" test. If you can write it, you understand it. If you can't, the blueprint isn't done.

## Cross-references (Karpathy model)

The graph is the structure. Before filing a new or edited blueprint:

1. Scan `topics/` (or read `index.html`).
2. If the topic relates to existing blueprints, link to them in the body using inline `<a class="xref" href="#secNN">§NN</a>` references and in the `cross-refs` block at the bottom.
3. **Edit the related blueprints to link back.** One write touches multiple files. If `aipw.html` links to `matching.html`, then `matching.html` should link to `aipw.html` in its "leads to" section.
4. Categories emerge as clusters of cross-links. There is no `taxonomy.md` and no enforced hierarchy.

## The metas (every blueprint's `<head>`)

Fill all of these:

- `source-chat` — URL of the chat that produced the blueprint
- `last-quizzed` — date of last Socratic quiz, empty if new
- `prerequisites` — comma-separated filenames this builds on
- `tags` — cross-cutting labels
- `created`, `last-updated` — dates

## What one blueprint covers

A blueprint mirrors what one learning session produced. A focused session on DNS → one focused blueprint. A weeks-long session on causal inference → one field-through-line with multiple sections. The blueprint mirrors the session; the graph handles connections to the rest of the hub. Don't split a session's output artificially; don't merge unrelated sessions either.

## Filing checklist

- [ ] Template used, all `<meta>` filled
- [ ] Visual grammar legend present
- [ ] Each section has a claim-title, not a label-title
- [ ] Every formula has a "reading it" block defining every symbol
- [ ] Each section has a "Remember" callout
- [ ] "In one breath" closing written (the understanding test)
- [ ] Diagrams used only where they earn their place (visual-learning constraint)
- [ ] Cross-references to related blueprints added in both directions
- [ ] `log.md` appended: `## [YYYY-MM-DD] ingest | {{topic}} — {{one-line summary}}`
- [ ] Push → Action rebuilds index and deploys
