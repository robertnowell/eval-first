# Anchor patterns — how to generate eval cases without asking the user

Decision tree: **user gave you a failing example?** → use it. **User edited a prompt?** → delta-driven. **User gave you a doc?** → synthetic-Q-per-chunk. **None of those?** → 3-slot fallback.

## Pattern 1 — Delta-driven (default for prompt edits)

For each instruction added/changed in a prompt, generate **two anchors**: one input where the change *should* fire, one where it *shouldn't*. Source: Shankar SPADE.

**Example.** User adds `"Don't speculate about pricing"` to a prompt:
- (a) "What does the Pro plan cost?" → should refuse/redirect (positive)
- (b) "How does Pro compare to Free in features?" → must not over-refuse (negative; mentions plans but isn't a price question)
- (c) "What integrations do you support?" → sanity baseline (unchanged)

## Pattern 2 — F×S×P tuple sampling (greenfield, no prior version)

Pick one Feature × 2-3 Scenarios × one Persona. Convert each tuple to a natural-language query in a **separate prompt** (not one shot — single-prompt synthesis collapses onto one style). Source: Hamel Husain.

**Example.** Customer support agent:
- Features: `order_status, refund, account_change, complaint`
- Scenarios: `valid_request, missing_data, conflicting_data, edge_case`
- Personas: `frustrated_repeat, first_time, power_user`
- Sample `(refund, missing_data, frustrated_repeat)` → "Generate a user input from someone irritated and impatient, demanding a refund for order #1234567890 but won't share their email. Hint at previous bad experiences." (#1234567890 deliberately doesn't exist.)

## Pattern 3 — Synthetic-Q per chunk (user gave you a doc)

For each retrievable chunk: `f"Generate 5 questions answerable by this text: {chunk.text}"`. The (question → chunk) pair is a labeled retrieval test by construction. Source: Jason Liu.

Empirical baseline: expect ≥80% recall on synthetic questions. If your system can't hit that on questions written *from* the doc, real users have no chance.

## Pattern 4 — The 3-slot fallback (always available)

When no other pattern fits, fill three slots:
- **Representative** — the median input the change is meant to improve
- **Adversarial / edge** — pushes the boundary (ambiguity, missing data, conflicting intent, typos, long context, non-English)
- **Negative / control** — input where the new behavior should NOT fire

This is the floor, not the ceiling. Expand if the task warrants it.

## Rules

- **Generate from delta, not scratch.** If a user is editing an existing prompt, the diff IS the test plan.
- **Two-step beats one-step.** Pick the (feature/scenario/persona) tuple first; *then* write the input in a separate call. Single-prompt synthesis collapses.
- **Ground in real artifacts when possible.** If the user gave Claude a doc, generate questions *from* the doc — not generic placeholders.
- **Prefer mining real failures over synthesizing.** If a trace or example exists in the conversation, it's a better anchor than anything Claude invents.
- **For vague criteria, generate multiple operationalizations.** "Be concise" is three different tests (sentence count, word count, LLM-judged). Pick one explicitly rather than letting the criterion stay fuzzy.
