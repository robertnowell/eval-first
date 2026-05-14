---
name: eval-first
description: "Bakes evaluation thinking into model output work. Every plan names what success looks like, where things can go wrong, and proposes simple checks validating key assumptions."
allowed-tools: Read, Grep, Glob, Write, Edit, Bash
---

# Eval-First — full protocol

You invoked this skill because the change is non-trivial. The description's guidance applies but you also want concrete patterns. Here's the full protocol.

## Plan must include

1. **Success looks like:** 1-2 sentences naming the criterion. What does a correct output of this changed system look like?
2. **Things go wrong when:** 2-4 specific failure modes. For any constraint change (do X / don't Y), include at least one negative case (the over-refusal / drift guard).
3. **Simple evals validating these assumptions:** 3-5 concrete cases (input + expected behavior + cheapest fitting check). Generate them yourself from context. Never ask the user to produce them.

Run the evals as part of doing the work. Show **verdict first** (one line: "3 of 3 cases match, 1 differs"), evidence (diff or table) second.

## Pick the right pattern for each step

- Generating anchor cases → [references/anchor-patterns.md](references/anchor-patterns.md)
- Cheapest fitting check per failure type → [references/cheap-checks.md](references/cheap-checks.md)
- Verdict-first comparison templates → [references/comparison-templates.md](references/comparison-templates.md)

## Two-phase for high-stakes judgments

When grading subjective output (tone, coherence, "did the change land?"), run two passes:

1. **Cold-read** — answer 3-5 naive questions about the output WITHOUT the rubric in context. *(What's it claiming? Did it confuse you? Could you summarize the argument?)*
2. **Apply rubric** — score using the cold-read answers as evidence.

Single-pass judges confabulate — they pattern-match the rubric instead of reading the content.

## Persist when load-bearing

If the eval cases would be useful for the next change to the same code, save them to `evals/<feature>.md` in the current project. Format in [references/comparison-templates.md](references/comparison-templates.md). Opportunistic, not mandatory.

## Anti-rules

- Never ask the user to label or produce test cases — generate from context.
- Never output Phase 0 / Phase 1 / Phase 2 headers — that's ceremony.
- Never refuse the work because evals aren't set up.
- For constraint changes (do X / don't Y), always include the negative case.
- Deterministic before model-based: a regex beats an LLM judge call.
- Binary pass/fail, not 1-5 scalars.
