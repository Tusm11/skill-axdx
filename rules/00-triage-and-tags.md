# ax-dx — Triage and Verified-Claim Tags (canonical)

This file is the single source of truth for ax-dx's core mechanism. Every
per-agent adapter (SKILL.md, AGENTS.md, .cursor/rules, .agent/rules,
.kiro/steering) points here rather than duplicating this content — edit it
once, every adapter inherits the change.

## The core idea

Documentation is a claim, not a fact, until it's checked against the
implementation, a test, or a call site. Write code with a clean,
idiomatic human-facing surface (DX), and attach a separate, checkable
trail of verified claims for whatever reads the code next — including an
AI agent that has no other source of context (AX). Spend that
verification effort only where it's actually worth it, calibrated by
blast radius — not uniformly on every line.

## Mechanism 1 — Triage before writing anything extra

Classify every function or module touched:

| Tier | When it applies | What to do |
|---|---|---|
| **Tier 0 — skip** | Pure, local-only, trivial, or fully exercised by an adjacent test in the same file. No external callers. | Write clean, idiomatic code. No evidence layer. |
| **Tier 1 — light** | Some internal callers, moderate complexity, or a non-obvious-but-local behavior. | Add inline verified-claim tags only for the specific non-obvious claims. |
| **Tier 2 — full** | Public/exported API, multiple callers across modules, shared/mutable state, or any side effect not visible from the signature. | Add the full file-level evidence header plus inline tags for every docstring claim. |

Judge the tier by blast radius (what breaks if this is wrong),
non-obviousness (would a competent reader miss this unassisted), and
distance from a tested boundary (is there already a test that would
catch a violation). When in doubt, pick the lower tier — over-tagging
erodes DX and trains readers to skim past the tags entirely.

## Mechanism 2 — Verified-claim tags (inline, Tier 1+)

Tag any claim that isn't self-evident from reading the code:

```
AX-VERIFIED: <the claim> — source: <test name | call site | assertion in code>
AX-UNVERIFIED: <a claim from surrounding docs/comments that could not be
  confirmed against the implementation — flag it, don't silently trust
  or silently delete it>
```

Use whatever comment syntax the language requires (`#`, `//`, `--`), but
keep the tag text (`AX-VERIFIED`, `AX-UNVERIFIED`) identical so it stays
grep-able across a codebase and across languages.

Example:

```python
def merge_sorted(a: list[int], b: list[int]) -> list[int]:
    """Merge two sorted lists into one sorted list."""
    # AX-VERIFIED: inputs must already be sorted ascending — caller
    # contract, confirmed by test_merge_sorted_rejects_unsorted_input;
    # not checked at runtime.
    ...
```

An `AX-UNVERIFIED` tag is a finding, not a failure to silently fix.
Surface it to the user rather than either blindly trusting the original
claim or unilaterally rewriting it — the correct resolution may depend
on intent only the human has.

Do not re-verify a claim from scratch every time. A claim already
carrying `AX-VERIFIED` and pointing at a still-existing test or call
site remains valid evidence; re-derive it only when the code it
describes has actually changed.

## Mechanism 3 — File-level evidence header (Tier 2 only)

Add one block near the top of the file (after imports, before the first
definition):

```
# ===== AX-EVIDENCE =====
# Blast radius: <who calls this>
# Verified invariants: <bullet list, each with a source>
# Known concealment risks: <side effects, shared state, or ordering
#   dependencies NOT obvious from function signatures alone>
# Last checked against implementation: <what was actually re-verified>
# ========================
```

Keep it short and honest. If nothing concealed exists, write "none
found" rather than omitting the line — an absence that was actively
checked for is more useful than a silence that might just mean nobody
looked.

## The two-sided test

Before adding any convention beyond these three mechanisms, check it
against both audiences:

- Does it cost a human reader something (clutter, another thing to keep
  in sync)?
- Does it save an agent something real (avoids re-deriving context,
  prevents acting on a false claim)?

If a proposed convention only helps one side, it doesn't belong in this
skill. This is what keeps ax-dx from sprawling into generic "add more
comments" advice.
