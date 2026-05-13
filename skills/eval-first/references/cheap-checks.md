# Cheap checks — failure type → minimal verification

Pick the cheapest fitting check. Promote to LLM-judge only when the property can't be expressed deterministically.

## Decision table

| Failure type | Cheap check | Snippet |
|---|---|---|
| **Structured output** (JSON, code, file) | parse-and-assert + schema | `json.loads()` then `jsonschema.validate()`; for code: `python -c "import ast; ast.parse(open(f).read())"` |
| **Tool-call correctness** | subset match on args | `assert call["name"] == X and call["args"].get(k) == v` |
| **Prose tone/quality** | side-by-side regen on 3 anchors | run before/after on representative inputs, eyeball or binary judge |
| **Refusal calibration** | paired test (must-refuse + must-not-refuse) | always TWO assertions: `is-refusal` AND `not-is-refusal` on a twin input |
| **Multi-step trajectory** | subset match `in_order` mode | `it = iter(actual); assert all(r in it for r in required)` — extra steps OK |
| **Hallucination / citations** | extract URLs, set-membership against source | `re.findall(URL_RE, output) - {c["url"] for c in context}` must be empty |
| **Skill behavior change** | diff against prior on 2 anchor invocations | `diff <(./skill@HEAD < a.json) <(./skill@WORKING < a.json)` |
| **Regression vs prior known-good** | snapshot of invariants, not bytes | store `{keys, n_items, contains_phrases, max_len}` not raw text |

## Binary judge prompt (when deterministic doesn't fit)

Hamel-canonical shape — binary outcome, free-text critique:

```
You are a [domain] evaluator.
<input>{input}</input>
<output>{output}</output>
Write a detailed critique, then return JSON:
{"critique": "...", "outcome": "good" | "bad"}
```

Why binary: 1-5 Likert scales force vagueness. Binary forces precision and forces you to define what makes something "bad."

## Rules

- **Deterministic before model-based.** If `parse-or-die`, `regex`, `set membership`, or `subset match` can express the property, use that. Never burn an LLM call on a property a 5-line script could decide.
- **Binary, not scalar.** Every cheap check returns pass/fail. No "looks good." This forces the check to be precise enough to fire.
- **Negative case is mandatory for refusal / safety / "should not" properties.** A `not-` assertion alone is half a test; pair it with a positive twin.
- **Subset match, not exact match, for sequences and args.** Tool trajectories and tool args almost always want partial / `in_order` semantics — exact match is brittle and trains the team to ignore failures.
- **Snapshot invariants, not bytes, for prose.** Capture what must remain true (keys present, citations resolve, length bounded, refusal-or-not) — not the literal string.
- **Cost gradient governs cadence.** Cheap deterministic checks run on every output; judges run on changes; humans/A-B run on releases.

## Hallucination check — concrete snippets

For citation hallucination (RAG, retrieved files, conversation transcripts):

```python
import re
URL_RE = re.compile(r"https?://[^\s)\]}>'\"]+")
cited = set(URL_RE.findall(output))
retrieved = {chunk["url"] for chunk in context_chunks}
fabricated = cited - retrieved
assert not fabricated, f"Fabricated URLs: {fabricated}"
```

For file/line citations (code agents):

```python
CITE = re.compile(r"\[(?P<f>[^\]:]+):(?P<a>\d+)-(?P<b>\d+)\]")
for m in CITE.finditer(output):
    f, a, b = m["f"], int(m["a"]), int(m["b"])
    assert os.path.exists(f) and a <= b <= line_count(f), m.group(0)
```

For unsourced numbers in summaries:

```python
for num in re.findall(r"\b\d{2,}(?:[.,]\d+)*\b", output):
    assert num in source_text, f"Unsourced number: {num}"
```
