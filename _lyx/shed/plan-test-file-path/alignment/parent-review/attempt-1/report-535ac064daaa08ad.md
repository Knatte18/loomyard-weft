---
verdict: APPROVED
---

The design covers each point of the board entry's Idea and adds no work that belongs to another entry.

- Drop the plain test-file path from `**Edit:**`: the plan stencil rewrite in Scope, with new tests as `**Create:**` `plan:` handles and changed tests as member glyphs.
  The entry's premise that "the declaration carries the file" is false, since `handle-malformed` refuses a file unit under `language: go`.
  D1 records this, and the result matches the operator's view that test files are no exception to glyph references.
- The plan gate refuses a plain `_test.go` path under `**Edit:**` and names the way forward, with `loom-plan-spec.md` and the stencil updated together: D3, D4 and D5, plus the spec row in Scope.
- Webster's pack and fork prompt still tell the fork where new tests go: D6 checks them, and D2's `**Uses:**` rule replaces the whole-file read the plain path gave.

Adjacent entries stay separate.
Removing stencil wording tests is left to `lighter-test-rules`, which depends on this task, while tests this task breaks are in Scope.
The fork deviation rule is left to `webster-deviations-list` (D6).
The 2026-10-15 pack audit commit is Out, since the entry names it only as context.
`stencil-intent-audit`, `enclose-kickstart` and `labels-from-diff` are not touched.

## Overlap

No overlap with any run `lyx loom peers` lists or with the parent commit it lists, so no landing order applies.

- `report-timing` (row 12) and `creel-core` (row 12) change no path in `internal/planparser`, `internal/planglyph`, `contracts/stencils/loom/loom-template-plan.md` or `contracts/specs/loom-plan-spec.md`.
  Both list `cmd/lyx/testdata/test-budget.yaml`, a shared bookkeeping file that this run may also touch under its test-budget constraint.
- `engine-registry` (row 9) plans shuttle engine registry work and does not touch the plan parser, the plan gate or the plan stencil.
- `sibling-overlap-check` (row 7) changes `internal/boardengine` and the `ly:board` skill.
- Parent commit `474fb66` (reed: the viewer follows the run until the operator moves it) changes reed, loomcli and a stencil seed-gate test, none of which this run's decisions concern; its `test-budget.yaml` line is bookkeeping only.
