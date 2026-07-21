---
name: learn
description: Trigger a learning session. Fetches the Learning Hub conventions and turns the current session into a learning session — teach a concept intuition-up, generate a blueprint, file it into the hub.
---

# /learn

Invoke this skill mid-build when you hit a concept you want to understand, not just use.

## What it does

1. Fetch `https://raw.githubusercontent.com/{{you}}/learning-hub-rules/main/CONVENTIONS.md` and read it.
2. The current session is now a learning session. Apply the conventions: who the user is, how to teach, the hub rules.
3. When the user asks for a blueprint on a concept, also fetch `https://raw.githubusercontent.com/{{you}}/learning-hub-rules/main/recipes/blueprint.md` and follow it.
4. Build the blueprint per the recipe. Use the template. Fill all metas. Maintain cross-references (Karpathy model).
5. File the blueprint into the private `learning-hub` repo: clone/pull, add `topics/<name>.html`, update cross-refs in related blueprints, append `log.md`, push. The GitHub Action rebuilds the index and deploys.
6. Offer to resume the original build.

## When invoked mid-build

Pause the build. Don't lose context — note where you were. Teach the concept. Generate the blueprint. File it. Then offer to resume.

## One-line pointer

This skill is a thin wrapper. The actual rules live at the URL above. Content evolves underneath; this skill rarely needs to change.
