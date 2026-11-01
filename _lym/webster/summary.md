# Parts-Write and Parts-Plan-Write producers

All five batches recorded `done`, with no pattern departures and no renames reported.
Each batch's `deviations` list names the files its glyph scope guard found changed outside the batch's own targets; all are informational.

## Batch 01 (cards 1–6), head 03eed6de

Built the `plan_split` and parts config keys, the decision-id export, `internal/partsparser`, the part scope in planparser and planglyph, and `JoinParts`.
Deviations: `internal/discussionparser/decisionids.go`, the `internal/partsparser` files, `internal/planparser/join.go`, `join_test.go`, `validate.go` and `validate_test.go`.

## Batch 07 (cards 7–11), head b7c717ae

Built the loomshed part checks, the writer seat directives, the writer seat kind, the writer resume and join, and the multillm writer mapping.
Deviations: `internal/loomshed/partcheck.go`, `partcheck_test.go` and `seam_enforcement_test.go`, `internal/seatengine/engine.go`, `table.go` and `table_test.go`, `internal/shedadapters/multillm.go`.

## Batch 12 (cards 12–15), head d50cea7c

Moved the plan format and glyph text into four included blocks, then added the Parts-Write and Parts-Plan-Write tables, their stencils and the two rotators.
Deviations: `contracts/stencils/citation_enforcement_test.go`, the `internal/loomengine` parts files and tests, `internal/loomshed/partsrotate.go` and `partsrotate_test.go`.

## Batch 16 (cards 16–17), head 4ebcffeb

Registered the two producers and added their two recipe rows.
Deviations: `internal/battencli/overview_test.go`, `internal/loomcli/wiring.go`, `internal/loomrecipe/fixture_test.go`, `plansplit_test.go` and `sequence_test.go`, `internal/reportengine/timing_test.go`, `internal/shedrecipe/entries_gate.go` and `entries_parts_test.go`.

## Batch 18 (cards 18–20), head 88ed0dfc

Added `lym loom validate-parts`, `validate-plan --part`, the chair-only `lym loom join-parts`, and the writer-aware settled-verdict refusal.
Deviations: `internal/loomcli/arm.go`, `joinparts.go`, `settledguard.go`, `validate.go` and `validateparts_test.go`.

## Verify gate findings

The plan-level verify failed on `cmd/lym.TestCLIForm_EveryCommand`.
A fixer then ran.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `377fd58d42b79638acc9c882df9a0cdb887979c7` fix: shorten validate-plan's Long under the CLI form cap
