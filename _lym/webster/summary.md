# Decision record on the board entry

Two batches ran, both `done`, with no pattern departures and no renames off the plan.

## Batch 01 — board-decision-record

Head `689221ed`.
The batch carried cards 1–6: the board engine's `Task.Discussed`, record file constants, `StoreDecisionRecord` and `DecisionRecord`; the write scaffold that moves the record on rename and removes it with its entry; the `design_prefix` refusal; the replay that claims, names and applies record files; `lym board get` showing `discussed` and printing the record; and loomrecipe's discussion approval row.
The scope guard listed these files as changed outside the batch's own targets:
`internal/boardcli/verbs_test.go`, `internal/boardengine/decision_test.go`, `internal/boardengine/replay.go`, `internal/boardengine/replay_test.go` and `internal/loomrecipe/discussionapproval.go`.

## Batch 07 — discussion-approve-seam

Head `24eb0f68`.
The batch carried cards 7–9: the approve seam storing the approved record, `decision add` copying the committed record onto the entry, and the new `lym loom decision store` verb.
The scope guard listed `internal/loomcli/decision.go` and `internal/loomcli/decision_test.go` as changed outside the batch's own targets.

## Verify gate findings

The plan-level verify failed on the package `github.com/Knatte18/loomyard/cmd/lym` (attempt 1 of 2); the findings file noted that no recorded card touched a failing package.
The full log is `.lym/verify/verify-2.log`.
One fixer fork was spawned for it.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `d4633f36f9fdbd1f1c5a3b83834616851025c0c7` fix: keep the board upsert and decision add help pages under the Long cap
