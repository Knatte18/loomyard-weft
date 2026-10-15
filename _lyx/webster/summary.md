# Run-pane timing: seconds, gate spans and entry numbers

All three batches reached `status: done`, with no warnings on any digest.
The digests list files changed outside each batch's own targets; those are informational, not failures.

## Batch 01 — statuspane-format-seconds (cards 1–5, head `5e63687a5`)

Added `FormatSeconds` for closed spans and had shuttle's gate outcome report each entry's gate, held and rework spans.
Shed history entries now store gate spans beside `gate_attempts`, and folds append them.
Burler's review result and webster's run result carry their gate outcome, and every gated row copies its gate count and spans into its history entry.

Deviations listed by the scope guard:
- `internal/shuttleengine/gate.go` and `waitshadow_test.go`: the latter clears gate spans before comparing two real-clock runs.
- `internal/shedadapters/gateattempts.go`.
- Test files in `shedadapters`, `loomshed`, `landingshed`, `loomcli` and `shedengine`, switched from `==` to `reflect.DeepEqual` once `HistoryEntry` and `OutputPointer` gained a span slice.

## Batch 06 — prrework-carry-gate (cards 6–11, head `eefaaff15`)

PR-Rework's Done and Stuck exits keep their session's gate count and spans.
One rows lookup now covers every recipe, batten's included, with `EntryNumbers`.
The checklist draws closed times to the second and open ones in minutes, with round totals.
Entries are numbered in recipe order, each gate entry gets a gate line under its visit, and a gate row's refusal marks its rework visit and sums its rework.

Deviations listed by the scope guard:
- `cmd/lyx/testdata/test-budget.yaml`: the `internal/loomrecipe` row raised from 54 to 55.
- `internal/loomcli/checklist.go` and `checklist_test.go`.

The fork skipped one card-11 test row, a `Rev-Bouncer` bounce inside a review segment marking nothing, because the review block draws no rework mark and existing rows already assert that shape.

## Batch 12 — overview-entry-numbers (cards 12–14, head `f3473e178`)

Batten's overview stage cell carries the entry number.
The Bouncer, BurlerReview and BurlerFix rows take an optional `role_prefix`, and the shipped recipes set it per review segment, pinned by a test.

Deviations listed by the scope guard:
- `internal/battencli/overview.go`.
- `internal/loomrecipe/fixture_test.go`.
- `internal/shedrecipe/seam_enforcement_test.go`: `internal/agentname` added to the import allowlist, since validating a prefix needs it.

## Scope guard

Deviations were informational only and none blocked a batch.
The plan-level verify gate runs after this turn ends.
