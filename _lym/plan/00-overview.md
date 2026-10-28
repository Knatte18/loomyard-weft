---
format: 5
approved: true
language: go
---

# Plan: validator-checks

`lym loom validate-discussion`, `lym loom validate-plan` and the gates that call the same functions learn to refuse the recurring review findings a machine can see.
The discussion side gains decision-id, acceptance and citation checks; the plan side gains seam-test, tier-statement and `doc.go` checks, with the `//lym:seam` marker and its scan the seam check reads.
Each finding names the item and its way forward, the specs carry the rules and the writer stencils point at them.

## Card Index

1 — decision-id-check — D1 and D2: decision headings carry a unique `D<n>` id, each id named in an acceptance bullet, in both discussion checks
2 — citation-extract — the record's parser returns every backticked citation with its line and mark, skipping the parts no writer authors
3 — index-resolve-glyphs — `planindex.Index` gains `ResolveGlyphs`, sharing the resolve status policy and reporting declaring files
4 — citation-check — D3 and D4: a loom record's cited paths and symbols must exist; the discussion gate gains the worktree root and index
5 — seam-marker-parser — `//lym:seam` parsed beside `//lym:guard`, stacking in any order, with the PATTERN line and background
6 — seam-marker-scan — a tier0 scan refuses an unmarked reflective test, and today's reflective tests get their markers
7 — test-tier-check — D7 `test-tier-unstated`, with the tier tags both plan-context builders fill and every mode's rows
8 — seam-test-check — D6 `seam-test-unlisted`, with the seam-test reader both builders fill
9 — doc-go-check — D8 `doc-go-unlisted`, and the Plan writer stencil's pointer to the three card checks

## Shared Decisions

- The discussion checks enter through `ValidateLoomDiscussion` and `ValidateDarnDiscussion`, and the plan checks through `planContextFindings`, so every gate and its verb call one function (`PATTERN-gate-slot`).
- `internal/discussionparser` only extracts; judging a citation against the tree and quarry is `internal/loomshed`'s, through the `planindex.Index` seam, so the parser stays stdlib-only and `internal/planglyph` stays the sole quarry caller.
- The tests for D6 to D8 share one new table, `TestValidatePlan_CardChecks`, which drives `ValidatePlan` over a go plan; card 7 creates it and cards 8 and 9 add rows.

## PATTERN changes

- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by gate key; one `tierN` tag, the lowest that fits; it pins behavior no test covers; injected clocks, `t.Parallel` by default; a test reflecting over a struct's fields carries `//lym:seam`. (test)
