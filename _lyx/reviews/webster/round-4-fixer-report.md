# Round 4 fixer report

All three findings fixed, one commit each, none pushed.

- F1 (`fc71fd002`): `internal/fabricengine/checkout.go` — broke the `switchOrForkWeft` doc before `so rollback can delete it;`.
- F2 (`841df36cd`): `internal/fabricengine/remoteleftover.go` — restructured the header so `Add adopts a live pair's branch;` and the leftover clause each have their own line, and broke the `leftoverKind` doc before `and Add adopts it.`
- F3 (`c63604a6d`): `internal/landingshed/deps.go` — put the `ConfigChanges` compound predicate on one line and broke the `Notify` doc before `so the queue wiring lives in the closure.`

Comment-only changes; `go build ./...` and the untagged tests of `fabricengine`, `landingshed` and `pattern` pass.
No test was added: no finding was a coverage gap.

## Deferred

Nothing deferred.
