# ax-dx

A portable AI-agent skill for writing and reviewing code that's equally legible to the human maintaining it and the AI agent that will later modify it. Works natively in Claude Code, and via `AGENTS.md`/runtime-native rule files in Codex, Cursor, OpenCode, Antigravity, and Kiro.

## Why this exists

Most "write clean code" advice was written with one reader in mind: a human skimming in an IDE. That reader can infer intent from surrounding context, ask a teammate what a weird comment means, or just notice something looks off. Increasingly, code has a second reader, an AI agent that shows up later to extend or fix something, with none of the original context, and no one to ask. That agent doesn't skim for vibes. It reads the docstring, trusts it, and acts on it. If the docstring is stale, aspirational, or was never quite true, the agent is wrong in exactly the way the documentation was wrong.

Most attempts to fix this just mean "write more comments," which usually makes things worse, more prose for a human to wade through, with no guarantee any of it is actually accurate. ax-dx takes a different angle: treat documentation the way an investigative reporter treats a source. A docstring that claims "this list is always sorted" is a lead, not a fact, the evidence is the code that maintains the invariant, or a test that checks it. Nothing gets tagged as reliable until it's actually been checked against something real.

The goal isn't "more documentation." It's a small, clearly separated layer of *verified* claims sitting next to code that otherwise stays exactly as clean and idiomatic as it would without this skill at all.

## How it works

There are three mechanisms, applied together:

**1. Triage first.** Not every function deserves this treatment, most code in most files doesn't. Before adding anything, the skill classifies what it's looking at:

- **Tier 0** = trivial, local, already covered by an adjacent test. Left alone entirely.
- **Tier 1** = some callers, a bit of non-obvious behavior. Gets targeted inline tags only for the specific claims that actually need them.
- **Tier 2** = public API, multiple callers, shared state, anything with a side effect that isn't visible from the signature. Gets the full treatment.

The judgment call is based on blast radius (what breaks if this is wrong), how non-obvious the behavior is, and how far it sits from something already covered by a test. When it's ambiguous, it defaults to the lighter tier — the failure mode this guards against is bureaucratic over-tagging that just becomes noise nobody reads.

**2. Verified-claim tags.** For anything above Tier 0, claims that aren't self-evident from reading the code get a tag naming what actually backs them up:

```python
def merge_sorted(a: list[int], b: list[int]) -> list[int]:
    """Merge two sorted lists into one sorted list."""
    # AX-VERIFIED: inputs must already be sorted ascending — caller contract,
    # confirmed by test_merge_sorted_rejects_unsorted_input; not checked at runtime.
    ...
```

If a claim exists but can't actually be confirmed against the implementation, it gets tagged `AX-UNVERIFIED` instead — flagged as a discrepancy, not silently trusted and not silently rewritten. That distinction (rather than quietly "fixing" it) is deliberate: the correct resolution often depends on intent only a human has.

**3. A file-level evidence header, for the highest-stakes files only.** Tier 2 files get a short block near the top summarizing blast radius, verified invariants, and any concealed side effects, the things an agent would otherwise have to reconstruct by reading the whole file before touching it.

Together, the effect is a split: the code itself stays exactly as clean as good code always was, that's for the human. The evidence sits alongside it, tagged and sourced — that's for the agent. Neither side is asked to read the other's layer to get what it needs.

## Two modes

**Passive mode** is the default and needs no invocation. It applies automatically whenever code is being written, edited, or reviewed — triage happens quietly, tags get added inline as the code is produced.

**Active mode** is an on-demand audit, triggered explicitly:

```
/ax-dx              # audit the whole current repo
/ax-dx src/          # audit just this folder
/ax-dx src/api/users.ts   # audit just this file
```

Active mode is **read-only**. It walks the target, applies the same triage and verification standard as passive mode, and produces a report, but it never edits a file on its own. The report shows what would be tagged, what checked out, what didn't, and where it found something concealed that nothing currently documents. If you want those tags actually written into the code afterward, that's a separate, explicit ask — not something the audit does by default.

A typical report looks like:

```
# AX-DX Audit Report

## Summary
Files scanned: 14
Tier 0 (skipped): 41 units
Tier 1 (light):   9 units
Tier 2 (full):    3 units
Claims checked:   22
  Verified:       17
  Unverified:     5

## Findings by file

### src/api/users.ts
Tier: 2
- VERIFIED: idempotent on retry — confirmed by test_user_create_retry_safe
- UNVERIFIED: "safe to call concurrently" — no test asserts this, and the
  function writes to a shared counter with no lock
Concealment risks found: shared mutable counter not mentioned in docstring
```

## What this deliberately doesn't do

This isn't a general production-readiness tool, it doesn't audit error handling, security, logging conventions, or system-design capacity. Those are real concerns, but folding them in here would dilute the one thing this skill is actually for. Anything added to it has to pass a simple test: does it cost a human reader something, and does it save an agent something real? If a proposed convention only helps one side, it doesn't belong here.

It also isn't trying to verify everything, everywhere, all the time. The triage step exists specifically so this doesn't turn into ceremony — most functions should come out of this untouched, and that's working as intended, not a gap.

## Portability

Skills are meant to be portable, so the actual rules don't live only in
`SKILL.md` — they live in `rules/`, and every runtime-specific file is a
thin pointer or mirror of that one source. Edit `rules/` once, every
agent that reads this repo inherits the change.

| Path | Responsibility | Loaded by |
|---|---|---|
| `rules/00-triage-and-tags.md` | Canonical. Triage tiers, `AX-VERIFIED`/`AX-UNVERIFIED` tag format, file-level evidence header, the two-sided test. | Referenced by every adapter |
| `rules/01-active-mode.md` | Canonical. The on-demand audit workflow and report format. | Referenced by every adapter |
| `SKILL.md` | Claude-native entry point: frontmatter for skill discovery, points at `rules/` rather than duplicating it. | Claude Code, Claude.ai |
| `AGENTS.md` | Condensed always-on rules per the `agents.md` convention. | Codex, Cursor, OpenCode, Antigravity, Kiro (auto-load) |
| `.cursor/rules/ax-dx.mdc` | Cursor-native mirror, `alwaysApply: true`. | Cursor |
| `.agent/rules/ax-dx.md` | Antigravity-native rule pointing at `rules/`. | Antigravity |
| `.kiro/steering/ax-dx.md` | Kiro steering file, live-inlines `rules/` via `#[[file:]]` refs — zero drift. | Kiro |

Two delivery modes, same as within the skill itself: passive rules
(`AGENTS.md`, `.cursor/rules/`, `.agent/rules/`, `.kiro/steering/`) apply
automatically to every edit an agent makes in the project. Active mode
(`SKILL.md`'s `/ax-dx` command, where the runtime supports slash-command
skill discovery) runs the on-demand audit.

## Installing

**Claude Code:** copy the whole `ax-dx/` folder into `~/.claude/skills/ax-dx/`. Passive mode applies automatically based on `SKILL.md`'s trigger conditions; active mode is available via `/ax-dx`.

**Codex, Cursor, OpenCode, Antigravity, Kiro:** drop this repo's files into your project (or install project-wide). `AGENTS.md` and the runtime-native rule files auto-load — nothing further to configure. Where a runtime supports skill-style commands, `/ax-dx` (bare or with a path) triggers the active-mode audit; otherwise, passive rules still apply to every edit even without an explicit command.

**Everywhere:** if you ever need to change a rule, change it in `rules/00-triage-and-tags.md` or `rules/01-active-mode.md` — every adapter points back to those two files, so nothing needs to be updated in more than one place.
