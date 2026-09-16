# ax-dx — always-on rules

Full canonical rules: `rules/00-triage-and-tags.md` and
`rules/01-active-mode.md`. This file is a condensed summary for runtimes
that auto-load `AGENTS.md`; read the files above for anything not fully
covered here, especially before an active-mode audit.

## Applies to every edit you make in this project

Before adding documentation or comments to a function or module,
classify it:

- **Tier 0** (trivial, local, already tested) → write clean code, no
  tags, leave it alone.
- **Tier 1** (some callers, one or two non-obvious behaviors) → tag only
  those specific claims inline.
- **Tier 2** (public API, multiple callers, shared state, hidden side
  effects) → tag every docstring claim, and add a file-level evidence
  header.

Tag format (any comment syntax, identical tag text):

```
AX-VERIFIED: <claim> — source: <test | call site | assertion>
AX-UNVERIFIED: <claim that could not be confirmed — flag, don't silently fix>
```

Tier 2 file header:

```
# ===== AX-EVIDENCE =====
# Blast radius: ...
# Verified invariants: ...
# Known concealment risks: ... (or "none found")
# Last checked against implementation: ...
# ========================
```

Never fabricate a source for a `AX-VERIFIED` tag. When a claim can't be
confirmed, tag it `AX-UNVERIFIED` and surface it — don't silently trust
it or silently rewrite it.

Default to Tier 0 when uncertain. Most code should receive none of this
— over-tagging is a failure mode, not thoroughness.

## On-demand audit

If explicitly asked to audit, or invoked as `/ax-dx` or `/ax-dx <path>`,
follow `rules/01-active-mode.md`: read-only, same triage and
verification standard applied in bulk, output a report — never edit
files unless separately asked to apply the tags afterward.
