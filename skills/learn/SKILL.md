---
name: learn
description: Trigger a learning session. Crystallize understanding into a blueprint — teach a concept from scratch, capture learnings from a work session, or generate a blueprint directly.
---

# /learn

Invoke this skill when you want to crystallize understanding into a blueprint.

## Step 1: Fetch conventions

Fetch `https://raw.githubusercontent.com/ArturGR3/learning-hub-rules/main/CONVENTIONS.md` and read it. Then fetch `https://learning-hub-3qh.pages.dev/manifest.json` (best-effort — skip if blocked; it lists existing blueprints for cross-referencing).

CONVENTIONS.md is fully self-contained — template skeleton, five patterns, quiz protocol, filing checklist. No other fetches needed. Do not fetch recipes, template.html, the CSS file, or build scripts.

## Step 2: Determine the mode

Infer the mode from the user's request. Don't ask upfront — start with the natural first action.

### TEACH — "teach me X" / "I want to understand X"

The user wants to learn something new. This is a dedicated learning session.

1. **Diagnose**: Figure out where the user's understanding currently is. Ask what they know, probe gently. Have a conversation, not a questionnaire.
2. **Teach**: Build up from the user's current level. Intuition first, then examples, then precise terms. Use the conventions' intuition-up ordering. Diagrams where they earn their place. This is a conversation, not a monologue — check understanding as you go.
3. **Crystallize**: After teaching, ask "Want me to crystallize this into a blueprint?" If yes → generate the blueprint from the teaching → file it (step 3). If no → offer a quiz, done.

### CRYSTALLIZE — "summarize what we learned" / "make blueprints for what we covered"

The user has been working (with you or another agent) and wants to capture learnings. The teaching already happened, embedded in the work.

1. **Outline**: Review the conversation for concepts worth capturing. Propose an outline: "Here are N topics: 1. X, 2. Y, 3. Z." Let the user modify the list.
2. **Check understanding**: For each topic, ask the user if it's clear. Ask clarifying questions. Fill gaps with mini-teaching where needed. The "Remember" callouts in the blueprint should come from real gaps or misconceptions discovered here — not generic statements.
3. **Crystallize**: Generate each blueprint from the conversation + the clarification feedback. File each (step 3).

### BUILD — "build a blueprint for X" / "generate a blueprint"

The user knows what they want and just wants the artifact.

1. **Generate**: Build the blueprint per CONVENTIONS.md. Use the template skeleton. Fill all metas. Maintain cross-references.
2. **File** (step 3).

## Step 3: File the blueprint

**If you have repo access** (opencode, Claude Code):

1. Find or clone the `learning-hub` repo. **Do not guess a path** — search for it: check the current directory, its parent, and `~/playground/` for an existing `learning-hub` directory containing a `.git` folder. If found, use it. Only if none exists, clone `ArturGR3/learning-hub` into `~/playground/learning-hub`. Never create a second clone when one already exists on disk.
2. Save the blueprint to `topics/<slug>.html`.
3. Maintain cross-references: scan `topics/`, link related blueprints both directions. Edit linked blueprints to add back-links.
4. **File using the script — do not run git commands manually:**

   ```bash
   ./scripts/file-blueprint.sh topics/<slug>.html "{{topic}} — {{one-line summary}}"
   ```

   This script validates the blueprint, appends log.md, stages only source files, commits, and pushes. **Always use this script.** Do not run `git add`, `git commit`, `git push`, or `build-index.py` yourself — the script handles all of that, and the GitHub Action regenerates `index.html` and `manifest.json` on push.

**If you don't have repo access** (claude.ai phone session): output the complete HTML for the user to copy into GitHub manually. Note cross-refs for laptop follow-up.

## Plan mode

If in plan mode: teach (text output), outline (text output), or generate HTML (text output). All are allowed. Then ask the user to exit plan mode for filing — writing files requires build mode. Don't present a "plan" of blueprint sections — the conventions define the structure. Just teach / outline / build.

## After filing

Offer to quiz the user (Socratic protocol is in CONVENTIONS.md). One question at a time, probe the model not recall, end with synthesis.
