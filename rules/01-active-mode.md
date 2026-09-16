# ax-dx — Active Mode (canonical)

This file is the single source of truth for ax-dx's on-demand audit
workflow. See `00-triage-and-tags.md` for the triage rubric and tag
formats this mode applies — this file only covers how the audit runs and
what it produces.

## When this mode applies

Active mode triggers only on explicit invocation — a slash command
(`/ax-dx`), a direct request to run ax-dx, or an explicit ask to "audit"
a codebase, folder, or file with this skill. It never triggers
implicitly; implicit triggers (writing, editing, reviewing code as it
happens) are passive mode, covered in `00-triage-and-tags.md`.

## Scope

- Bare invocation (`/ax-dx`, or "audit the project" with no path) →
  audit the entire current repository/codebase.
- Invocation with a path (`/ax-dx src/`, `/ax-dx src/api/users.ts`) →
  audit only that folder or file.

## This mode is read-only

Active mode never edits, inserts tags into, or otherwise modifies any
file. It reports what tier each unit falls into and what claims would
be tagged `AX-VERIFIED` or `AX-UNVERIFIED` if passive mode were applied
to that code — nothing more. Writing those tags into the actual files is
a separate, explicit follow-up request from the user, never something
this mode does on its own.

## Procedure

1. Enumerate the files in scope.
2. For each function/module, apply the Mechanism 1 triage from
   `00-triage-and-tags.md` exactly as in passive mode — do not relax
   rigor just because this is a bulk pass.
3. For Tier 1 and Tier 2 units, check existing docstring/comment claims
   against the implementation, tests, and call sites, using the same
   verification standard as passive mode. Never fabricate a source; a
   claim that can't be checked is `AX-UNVERIFIED`, not silently skipped.
4. Compile findings into the report below and present it directly in
   the response. Do not write the report to a file unless asked.

## Report format

```
# AX-DX Audit Report

## Summary
Files scanned: <n>
Tier 0 (skipped): <n> units
Tier 1 (light):   <n> units
Tier 2 (full):    <n> units
Claims checked:   <n>
  Verified:       <n>
  Unverified:     <n>

## Findings by file

### <file path>
Tier: <0 | 1 | 2>
- VERIFIED: <claim> — source: <test/call site/assertion>
- UNVERIFIED: <claim> — <why it couldn't be confirmed>
Concealment risks found: <list, or "none found">

[repeat per file with Tier 1 or 2 units]

## Notes
This is a read-only report — no files were modified. Ask explicitly if
you'd like these tags applied to the code.
```

Leave Tier 0 files out of the per-file findings entirely — list only the
count in the summary. The report should surface signal, not pad itself
with files that had nothing worth flagging.
