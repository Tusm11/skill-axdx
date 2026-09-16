# ax-dx

Canonical source: `rules/00-triage-and-tags.md` (triage + tag
conventions) and `rules/01-active-mode.md` (on-demand audit). Read both
before editing code in this project or running an audit.

Summary: classify every function/module as Tier 0 (skip), Tier 1
(targeted inline `AX-VERIFIED`/`AX-UNVERIFIED` tags), or Tier 2 (full
tagging plus a file-level `AX-EVIDENCE` header), based on blast radius,
non-obviousness, and distance from a tested boundary. Never fabricate a
verification source. Default to the lower tier when uncertain — most
code should stay untouched by this.

Active mode (`/ax-dx` or an explicit audit request): read-only, same
standard applied in bulk across the requested scope, output a report,
never edit files unless separately asked to apply the tags afterward.
