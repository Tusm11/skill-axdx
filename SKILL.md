---
name: ax-dx
description: Write and review code that is equally legible and trustworthy to human developers (DX) and to AI coding agents (AX) — not documentation that merely looks thorough, but documentation whose claims are actually verified against the implementation. Has two modes. Passive mode triggers implicitly whenever writing new functions or modules, refactoring existing code, adding or updating docstrings/comments, reviewing a pull request, or documenting an API or library — even when the user doesn't name the skill, e.g. "make this code more maintainable," "document this for the team," "clean this up," "make this easier for an agent to work with later," or "add comments." Active mode triggers on explicit invocation (`/ax-dx` or `/ax-dx <path>`) and produces a read-only audit report of an existing codebase, folder, or file without editing anything. Also trigger on direct mentions of agent experience, AX, developer experience, DX, agent-friendly code, dual-audience documentation, or verifying that docs match the code.
---

# AX/DX: Verified Dual-Audience Code

## The core idea

Most "write clear code" guidance was written for one reader: a human skimming in an IDE. Code today has a second reader — an AI agent that will later read this file to modify it, without the benefit of the original author's context. A human can infer intent from surrounding code; an agent usually can't, and trusts whatever the documentation says.

This skill treats documentation the way an investigative journalist treats a source: a claim is not a fact until it's checked against the primary record. Keep the human-facing code clean and idiomatic (DX), and attach a separate, checkable trail of verified claims an agent can act on with confidence (AX) — spending that verification effort only where it's actually worth it (experience-calibrated judgment), not uniformly on every line.

## Before doing anything: read the canonical rules

The full triage rubric, tag conventions, file-header format, and the
two-sided test that governs what belongs in this skill all live in
`rules/00-triage-and-tags.md`. Read it before writing, editing, or
reviewing any code under this skill's triggers — it is short and is the
actual content of this skill, not optional background.

If this invocation is active mode (explicit `/ax-dx` or an explicit
audit request), also read `rules/01-active-mode.md` for the audit
workflow and report format before proceeding.

## Two modes, one source of truth

- **Passive mode** (default): applies implicitly per the trigger
  conditions above. Follow the procedure in `rules/00-triage-and-tags.md`
  and write tags directly into the code as it's produced or edited.
- **Active mode** (explicit invocation only): follow
  `rules/01-active-mode.md`. Read-only — produces a report, never edits
  files on its own.

Do not duplicate or paraphrase the rules files' content elsewhere in this
repo. If a rule needs to change, change it in `rules/` — every adapter
(AGENTS.md, .cursor/rules, .agent/rules, .kiro/steering) points back to
these same two files, so editing them once updates every runtime this
skill supports.

## Procedure summary (see rules/ for full detail)

**Writing new code:** write the clean implementation first, classify the
tier, tag only what Tier 1/2 requires, never fabricate a verification
source.

**Editing or reviewing existing code:** treat existing docstrings as
leads, spot-check the highest-blast-radius claims, tag what holds up as
`AX-VERIFIED` and what doesn't as `AX-UNVERIFIED`, surface discrepancies
to the user rather than silently resolving them, and actively look for
concealed side effects even where nothing currently documents them.

**Active mode:** same triage and verification standard, applied in bulk,
reported rather than written into files.

Re-run the triage per file, per function — most code should stay Tier 0
and receive none of this. Calibrated judgment about where to spend
verification effort is the point of this skill, not exhaustive coverage.
