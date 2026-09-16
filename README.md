# ax-dx

**Make code legible to both the human maintaining it and the AI agent modifying it next.**

[License: MIT](https://github.com/Tusm11/skill-axdx/blob/main/LICENSE) · [AGENTS.md](https://agents.md/)

AI coding agents increasingly work on code they didn't write and don't have the original context for. A human can ask a teammate why an invariant exists. An agent usually can't.

That creates a subtle problem:

**documentation can look authoritative without actually being true.**

A docstring might say that a collection is always sorted. A comment might claim that an operation is idempotent. A function might appear thread-safe while quietly mutating shared state.

ax-dx treats these statements as **claims that need evidence**, rather than facts simply because they appear in documentation.

The goal isn't to add more comments.

The goal is to leave behind a **small layer of verified knowledge** that helps the next agent understand what the code actually guarantees.

---

## The core idea

ax-dx uses three ideas together:

**Triage → Verify → Record**

Before documenting or reviewing anything, ax-dx determines how much attention the code deserves.

| Tier       | Typical code                                                  | Treatment                    |
| ---------- | ------------------------------------------------------------- | ---------------------------- |
| **Tier 0** | Trivial/local behavior already obvious from code or tests     | Leave untouched              |
| **Tier 1** | Non-obvious behavior with limited impact                      | Add targeted verified claims |
| **Tier 2** | Public APIs, shared state, side effects, important invariants | Full evidence treatment      |

The classification considers:

* **Blast radius** — what happens if the assumption is wrong?
* **Non-obviousness** — can the behavior be established just by reading the implementation?
* **Existing evidence** — is there a test, invariant, caller contract, or implementation detail supporting the claim?

When uncertain, ax-dx prefers the lighter tier.

The point is to avoid turning every function into documentation ceremony.

---

## Evidence-backed claims

For Tier 1 and Tier 2 code, ax-dx identifies claims that matter to future maintenance.

For example:

```python
def merge_sorted(a: list[int], b: list[int]) -> list[int]:
    """Merge two sorted lists into one sorted list."""

    # AX-VERIFIED:
    # Inputs must be sorted ascending.
    # Evidence: test_merge_sorted_rejects_unsorted_input.
    # The function does not validate this at runtime.
    ...
```

The important part isn't the comment itself.

It's the relationship between the **claim and its evidence**.

### AX-VERIFIED

Use when the claim can be established from something concrete:

* implementation behavior
* tests
* caller contracts
* type constraints
* configuration
* established invariants

### AX-UNVERIFIED

Use when a claim exists but its truth cannot currently be established.

```python
# AX-UNVERIFIED:
# "Safe to call concurrently."
# No synchronization or concurrency test was found.
```

ax-dx does **not silently rewrite the claim**.

An unverified statement may reflect an intentional design decision that only the developer understands. It is therefore surfaced as a discrepancy for human review.

---

## Concealed behavior

ax-dx also looks for behavior that an agent could easily miss while reading a function or module.

Examples include:

* mutation hidden behind an apparently pure API
* shared mutable state
* implicit global configuration
* externally visible side effects
* retry-sensitive operations
* assumptions enforced by callers rather than the function itself
* behavior that is important but absent from the public documentation

The question is simple:

> **What would a future agent need to know before safely changing this code?**

---

## File-level evidence

Some files carry considerably more risk than others.

For higher-impact Tier 2 files, ax-dx can maintain a short evidence header containing:

* important invariants
* external dependencies
* concealed side effects
* shared state
* relevant caller assumptions
* areas where documentation is still unverified

The header is intentionally compact.

It exists so an agent doesn't have to reconstruct the entire file's behavioral contract before making a small change.

---

# Two modes

## Passive mode

Passive mode runs while an agent is already writing, editing, or reviewing code.

The agent quietly applies the ax-dx discipline:

```text
             Code change
                  │
                  ▼
               Triage
             ┌────┴────┐
             │         │
          Tier 0    Tier 1/2
             │         │
           Leave     Inspect
          alone        │
                       ▼
                  Find claims
                       │
                 ┌─────┴─────┐
                 │           │
             Evidence      No evidence
                 │           │
            VERIFIED     UNVERIFIED
```

Most code should pass through with little or no modification.

That is intentional.

---

## Active mode

Active mode performs an explicit repository or path-level audit.

```text
/ax-dx
/ax-dx src/
/ax-dx src/api/users.ts
```

The audit is **read-only**.

It reports:

* what was classified
* which claims were verified
* which claims could not be verified
* concealed behavior discovered
* files requiring deeper attention
* evidence used for each finding

It does not modify files automatically.

If you want the findings written back into the code, that is a separate explicit operation.

---

# Example audit

```text
# AX-DX Audit Report

## Summary

Files scanned: 14

Tier 0: 41 units
Tier 1:  9 units
Tier 2:  3 units

Claims checked: 22
Verified:       17
Unverified:      5

## src/api/users.ts

Tier: 2

VERIFIED
  create_user is idempotent on retry
  Evidence: test_user_create_retry_safe

UNVERIFIED
  "safe to call concurrently"
  Evidence not found

CONCEALED
  Shared mutable counter is modified by create_user
  but this side effect is not documented.

## Action

Human review recommended for 1 unverified concurrency claim.
```

The report is deliberately about **evidence**, not confidence scores.

---

# What ax-dx is not

ax-dx is not another generic "clean code" checklist.

It does not try to make every function:

* heavily documented
* maximally abstract
* production-ready
* covered by unnecessary tests
* surrounded by defensive boilerplate

It also does not replace:

* security auditing
* performance testing
* production observability
* architecture review
* conventional testing
* code review

Those are separate concerns.

ax-dx asks a narrower question:

> **Can a future AI agent distinguish what the code actually guarantees from what someone merely claimed about it?**

---

# Portability

The canonical rules live in `rules/`.

Agent-specific files are thin adapters around that source.

| Path                          | Purpose                                |
| ----------------------------- | -------------------------------------- |
| `rules/00-triage-and-tags.md` | Triage model and verified-claim format |
| `rules/01-active-mode.md`     | Audit workflow and report format       |
| `SKILL.md`                    | Claude-compatible skill entry point    |
| `AGENTS.md`                   | Agent-wide passive rules               |
| `.cursor/rules/ax-dx.mdc`     | Cursor integration                     |
| `.agent/rules/ax-dx.md`       | Antigravity integration                |
| `.kiro/steering/ax-dx.md`     | Kiro integration                       |

This keeps the behavioral rules in one place.

If a rule changes, update the canonical rule rather than maintaining several independent versions.

---

# Agent compatibility

ax-dx is designed around runtime-native instruction mechanisms rather than depending on one particular coding agent.

| Agent            | Integration                                                          |
| ---------------- | -------------------------------------------------------------------- |
| **Claude Code**  | `SKILL.md`                                                           |
| **Codex**        | `AGENTS.md` / skills                                                 |
| **Cursor**       | `.cursor/rules/` + `AGENTS.md`                                       |
| **OpenCode**     | `AGENTS.md` / Claude-compatible skills                               |
| **Antigravity**  | `.agent/rules/` + `AGENTS.md`                                        |
| **Kiro**         | `.kiro/steering/`                                                    |
| **Other agents** | Canonical `rules/` can be adapted to their native instruction format |

The principle stays the same regardless of runtime:

**one rule set, multiple adapters.**

---

# Installation

### Claude Code

Copy the skill into the Claude skills directory:

```bash
cp -r ax-dx ~/.claude/skills/ax-dx/
```

Then use:

```text
/ax-dx
```

### Other coding agents

Copy the repository's agent-specific files into the project.

For agents supporting `AGENTS.md`, the passive rules can be loaded automatically.

For runtimes with native rule systems, use the corresponding adapter under:

```text
.cursor/
.agent/
.kiro/
```

---

# Design principles

### 1. Evidence over authority

A comment isn't true because it is written confidently.

Claims should point toward something that can establish them.

### 2. Less documentation, more signal

Most code doesn't need additional annotations.

Only behavior that matters and isn't already obvious should receive attention.

### 3. Preserve uncertainty

When a claim cannot be verified, don't quietly "correct" it.

Surface the uncertainty so a human can resolve the intent.

### 4. Optimize for the next maintainer

The next maintainer may be:

* a developer who joins the project later
* another AI agent
* the same agent several months later

The code should leave enough evidence for any of them to understand its important contracts.

### 5. Keep code and evidence separate

The implementation remains idiomatic code.

The evidence layer exists beside it to expose assumptions that aren't obvious from the implementation alone.

---

# Contributing

Contributions are welcome.

When changing ax-dx:

* Put shared behavioral rules in `rules/`.
* Keep runtime-specific adapters thin.
* Avoid adding documentation requirements without a concrete maintenance benefit.
* Prefer evidence that can actually be checked.
* Add examples when introducing a new type of claim or failure mode.
* Test changes against a real codebase and at least one supported agent.

The guiding test for new rules is:

> **Does this give a future agent useful information that it could not reliably obtain from the code itself?**

If not, it probably doesn't belong in ax-dx.

---

# License

MIT © 2026 Abhiram
