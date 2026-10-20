---
verdict: APPROVED
---

# Alignment at parent-review — verify-diff-scope

The design covers the board entry's two operator decisions and its open items, with no work that belongs to another entry.

## Goal coverage

- Tree scope for a diff that touches no production code (decision of 2026-10-18): D1 and D2 add it as hub-wide `darn.yaml` config, classify the diff by tree path, never by import graph, and run the touched trees' tests plus the scans.
- Scoped gate for darn's writer and fix rounds (decision of 2026-10-20): D3 runs `impactset`'s touched packages, reverse dependencies and `//lyx:guard` tests at Darn-Write and Darn-Burler-Fix.
- Whole suite once before landing for a production diff (decision of 2026-10-20): D4 runs the hub's whole `verify` at Merge-In, the gate right before Publish, for every diff that touches production code.
  The decision record states this reading of "at Publish" as an assumption, matching the code: darn's Publish runs only `publish_verify`.
- No hard-coded Go or tree layout (open item): the trees and the scoped command are config, off by default, and a non-Go hub falls back to the whole command at the writer gates.
- Scan packages as config (open item): D5 settles it through the `//lyx:guard` marker and `lyx gate test --guards` and names the departure; the board leaves names and shape to the design.
- Final gate for a non-production diff (open item): the board settles it only for production diffs; the design makes a tree-scoped diff's last gate tree-scoped and states the bound this sets on `PATTERN-test-speed`.

Loom's recipe is left out.
The board's Seen sentence names loom's Merge-In and Publish, but loom's Merge-In runs the plan's own `## verify:`, not `darn.yaml`'s, and the 2026-10-20 decision and its Background speak of darn's gates.
The decision record names the exclusion and its reading of that sentence in Scope Out; I read it as within the entry, not a goal point left out.

## Related board entries

- `gate-language-neutral`: proposes hub-declared tiers with `{targets}` in `language.yaml`.
  The design names the expected fold of its two `darn.yaml` keys into that mechanism and keeps `--guards` on the Go verb; no contradiction, and that entry is not running.
- `dotnet-scoped-verify`, `dotnet-verify-protocol`, `regression-baseline-verify`: .NET test selection and tiers stay out; the design's whole-run-at-the-last-gate for production diffs and docs-only skipping agree with their wants.
- `darn-verify-parent-fix`, `darn-verify-tier-unrunnable`: named as unsettled in Open risks; the design does not take their work.
- `row-entry-gate`, `scratch-defeats-test-cache`, `gate-machine-local-config`: no shared behaviour.

## Overlap with runs in flight and parent commits

`lyx loom peers` lists no peer run and no parent commit (`own_row` 3), so there is no overlap and no landing order to give.
