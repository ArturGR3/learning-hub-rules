# Recipe — Socratic Quiz

This is a tier-2 recipe. Load it when the user asks to be quizzed on a blueprint. The goal is to test the *model*, not fact recall.

## Why this exists

Reading a blueprint produces a feeling of understanding that may be false. The quiz forces the user to do the connecting themselves — to use the model, not just recognize it. This is where durable learning actually happens.

## The protocol

1. **Read the blueprint** the user named. Extract the load-bearing insights — the "Remember" callouts and the "In one breath" synthesis are the densest source.

2. **Ask one question at a time.** Wait for the user's answer before asking the next. Don't dump a list.

3. **Probe the model, not recall.** Good questions:
   - "Predict what happens if {{X is changed}}." — forces the user to use the model forward
   - "Explain why {{Y}} is the case, in your own words." — forces synthesis, not recognition
   - "Generalize: would this still hold if {{Z}}?" — tests the boundaries of the model
   - "Here's a scenario: {{...}}. What would you expect, and why?" — applies the model to a new case
   - "Compare {{A}} and {{B}} — what's the actual difference, and when does it matter?" — forces distinction, not conflation

   Bad questions (avoid):
   - "What is the definition of X?" — recall, not understanding
   - "True or false: ..." — binary, no model use
   - "List the steps in ..." — recall
   - Anything the user could answer by re-reading one sentence

4. **After each answer, respond with:**
   - Whether the model they used was right
   - *What specifically* was right or wrong (not just "correct" / "incorrect")
   - The next question, building on what they just showed they understood

5. **Use the blueprint's diagrams.** If the user is shaky, point them at a specific diagram and ask them to explain what it shows in their own words. Visual learners consolidate through the visuals.

6. **End the quiz with a synthesis question** — the kind of question the "In one breath" section answers. "Sum up the whole topic in a paragraph, as if explaining to someone who hasn't read this." Their answer tells both of you whether the model is integrated or just fragmentary.

## After the quiz

- Update `<meta name="last-quizzed">` in the blueprint to today's date.
- Push. The Action redeploys; the spaced-retrieval nudge in `index.html` resets for this blueprint.
- Append `log.md`: `## [YYYY-MM-DD] quiz | {{topic}} — {{one-line on how it went}}`.

## Calibration

If the user gets everything right, the next quiz on this blueprint should be harder — push toward edge cases, generalizations, "what if we relax this assumption?" If they struggle, the next quiz should anchor back to the intuition and the diagrams before pushing to precise terms.

The quiz is not an exam. It's a learning tool. The point is to surface where the model is still soft, so the next session knows what to refine.
