# Eval-First for Claude Code

A lightweight Claude Code skill that bakes eval-first thinking into every model-output change — without ceremony, without refusing work, without asking the user to label cases.

## What it does

For any work involving prompts, agents, skills, judges, or model outputs, the skill nudges Claude to:

1. **Name what success looks like** — 1-2 sentences on the criterion
2. **Name where things can go wrong** — 2-4 specific failure modes, including negative cases for any constraint change
3. **Propose simple evals** — 3-5 concrete cases validating the key assumptions, generated from context

Run as part of the plan. Show verdict first, evidence second. Mechanical changes (typos, formatting) skip the protocol.

## Why

Most prompt/agent/skill changes ship without anyone asking "how would we know this worked?" Production failures are then caught by users instead of by the developer. This skill makes verification a natural part of doing the work — Claude generates the cases, picks the cheapest fitting check, and surfaces the comparison as the answer.

## Install

```
/plugin marketplace add robertnowell/eval-first
/plugin install eval-first@eval-first-marketplace
```

Or pinned to a release:

```
/plugin marketplace add robertnowell/eval-first@v1.0.0
```

## How it works

Three layers, classic progressive disclosure:

| Layer | What loads | When |
|---|---|---|
| **L1 — `description`** | The load-bearing sentence + skip rule | Always in context (every Claude Code session) |
| **L2 — `SKILL.md` body** | Full protocol + reference pointers | When the skill is invoked for a non-trivial change |
| **L3 — `references/`** | Anchor patterns, cheap-check snippets, comparison templates | When SKILL.md tells Claude to read them |

The ambient cost (L1 only) is ~200 tokens. The full skill (L1+L2) costs ~600 tokens. References (L3) load on demand at ~190 lines total.

## References

- [`skills/eval-first/references/anchor-patterns.md`](skills/eval-first/references/anchor-patterns.md) — 4 patterns for generating eval cases without asking the user (delta-driven, F×S×P tuples, synthetic-Q per chunk, 3-slot fallback).
- [`skills/eval-first/references/cheap-checks.md`](skills/eval-first/references/cheap-checks.md) — Failure-type → minimal-check decision table with concrete snippets.
- [`skills/eval-first/references/comparison-templates.md`](skills/eval-first/references/comparison-templates.md) — Verdict-first comparison templates A-E.

## Credits

Built by [@robertnowell](https://github.com/robertnowell) for the StartX founder community. Draws on canonical work by Anthropic (Demystifying Evals for AI Agents), Hamel Husain (Field Guide), Eugene Yan (Product Evals), Shreya Shankar (SPADE / EvalGen), Jason Liu (RAG flywheel), and the Promptfoo / Inspect AI / OpenAI Evals frameworks.

## License

MIT
