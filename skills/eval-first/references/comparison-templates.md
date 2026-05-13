# Comparison templates — how to show verdict + evidence

**Verdict first** (one line). Evidence (diff/table) second. Match the diff format to the artifact's grain.

## Template A — Verdict-first summary (DEFAULT)

```
Verified: 4 of 5 claims match source.
1 differs — see below.

Claim 3: "Pipeline ships 50s vertical video"
  Source:  "Pipeline ships ~50s vertical video at 1080×1920"
  Draft:   "Pipeline ships 60-second portrait video"
  Delta:   duration 50s → 60s; "vertical" → "portrait"
```

Verdict line 1. Evidence is exactly the row that matters. No table headers eating space for a single failing row. **Use this by default.**

## Template B — Word-level prose diff (sentence edits)

```
Before: The pipeline ships ~50s vertical videos at 1080×1920.
After:  The pipeline ships ~50s vertical videos at 1080×1920 with burned-in captions.
Diff:   The pipeline ships ~50s vertical videos at 1080×1920{+ with burned-in captions+}.
```

Three lines: full before, full after, single annotated line showing the surgery. Use for prose paragraph edits where a line-diff would show the whole paragraph as "rewritten."

## Template C — Three-column table (atomic comparisons, ≥3 rows, short cells)

```
| Field          | Expected   | Got     | ✓/✗ |
|----------------|------------|---------|-----|
| episode_id     | 006        | 006     | ✓   |
| duration_s     | 50         | 62      | ✗   |
| has_captions   | true       | false   | ✗   |
```

Switch away the moment a cell needs to wrap. Tables for atomic values; prose for paragraphs.

## Template D — Side-by-side prose + "what changed" gloss

```
BEFORE                                  AFTER
───────────────────────────────────     ───────────────────────────────
The pipeline ships short vertical       The pipeline ships ~50s vertical
videos. They include captions.          videos at 1080×1920 with
                                        burned-in captions.

What changed: added duration (~50s), resolution (1080×1920), and
clarified that captions are burned in.
```

The **gloss** does the work — without it the reader has to do the diff in their head. Always include a one-line "what changed."

## Template E — Bless prompt (when user ratifies, doesn't specify)

```
Generated draft: drafts/006-thumbnail-v3.md
Previous:        drafts/006-thumbnail-v2.md

Diff (v2 → v3):
- bold red title, 3 words
+ bold red title, 4 words ("vs." inserted)

Accept v3? (y / show full diff / regenerate)
```

Name BOTH files so user can `git diff` later if they accept and regret.

## Rules

- **Verdict first, evidence second.** Single-line summary makes verification feel like *the answer*. Diff below is the proof.
- **Match the diff format to the artifact's grain.** Line diffs for code/JSON. Word diffs for prose. Three-column tables for atomic values.
- **Collapse the boring case.** If it passes, say "matches" in one line and stop. Don't print the full artifact twice.
- **Name what changed in words, not just symbols.** A one-line "what changed: duration 50s → 60s" converts the diff from raw evidence to interpreted evidence.
- **Inline for this-turn comparisons; file artifact for durable ones.** A claim-vs-source check this turn → render the diff in chat. A draft-vs-previous-draft check that will be re-run → write both versions to files, render the diff in chat AND tell the user the paths.

## Eval persistence format (for `evals/<feature>.md`)

```markdown
# Evals: <feature>

## Anchors

### Case 1: <one-line summary>
**Input:** <verbatim input>
**Expected behavior:** <criterion>
**Check type:** <regex / schema / subset-match / binary-judge / paired-refusal>

### Case 2: ...

### Case N (negative): <one-line summary>
**Input:** <input that should NOT trigger the new behavior>
**Expected:** <the unchanged behavior>
```

Keep cases short and concrete. The eval file is the contract for what this change preserves.
