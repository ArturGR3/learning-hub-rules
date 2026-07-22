---
name: learn
description: Trigger a learning session. Fetches the Learning Hub conventions and turns the current session into a learning session — teach a concept intuition-up, generate a blueprint, file it into the hub.
---

# /learn

Invoke this skill mid-build when you hit a concept you want to understand, not just use.

## What it does

1. Fetch `https://raw.githubusercontent.com/ArturGR3/learning-hub-rules/main/CONVENTIONS.md` and read it.
2. The current session is now a learning session. Apply the conventions: who the user is, how to teach, the hub rules.
3. CONVENTIONS.md is fully self-contained — it has the five template patterns, the HTML template skeleton, the quiz protocol, and the filing checklist inline. No follow-up fetches needed. (If your environment allows fetching from the rules repo, `recipes/` and `template.html` are optional supplementary detail.)
4. When the user asks for a blueprint on a concept, build it per CONVENTIONS.md. Use the template skeleton. Fill all metas. Maintain cross-references if you can see existing blueprints.
5. **If you have repo access** (opencode, Claude Code): file the blueprint into the private `learning-hub` repo — save `topics/<name>.html`, update cross-refs in related blueprints, append `log.md`, push. The GitHub Action rebuilds the index and deploys.
6. **If you don't have repo access** (claude.ai phone session): output the complete HTML for the user to copy into GitHub manually. Note any cross-refs that need laptop follow-up.
7. Offer to quiz the user on the blueprint (Socratic protocol is in CONVENTIONS.md).
8. Offer to resume the original build.

## When invoked mid-build

Pause the build. Don't lose context — note where you were. Teach the concept. Generate the blueprint. File it. Then offer to resume.

## Manifest (best-effort)

`https://learning-hub-3qh.pages.dev/manifest.json` lists all existing blueprints with metadata. Fetch it if you can — it tells you what blueprints already exist so you can cross-reference. If your environment blocks it, skip; the blueprint works standalone.

## One-line pointer

This skill is a thin wrapper. The actual rules live in CONVENTIONS.md at the URL above. CONVENTIONS.md is self-contained — everything needed to build a blueprint is there. Content evolves underneath; this skill rarely needs to change.
