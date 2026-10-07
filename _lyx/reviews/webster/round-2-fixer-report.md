# Round 2 fixer report

All four findings are fixed, one commit each, none pushed.

## Fixed

- **F1 (MEDIUM)** — `7f1ce2ff8`: `IsReadOnlyCommand` now refuses a command whose first word is quoted, through a new `commandWordQuoted` helper that compares the original text with its quote-blanked copy at each command start.
  Three rows in `TestIsReadOnlyCommand` (`'tee' cat`, `ls a && "rm" cat x`, `cat a | 'tee' cat`) failed before the change and pass after it.
- **F2 (MEDIUM)** — `cb58c11cf`: the recover-batch refusal over pathless fabric references now names both evidence routes, HEAD at the start commit, or the batch's commits kept with every recorded command read-only, as the refusal-spec row does.
  The existing assertion in `TestRecoverSpawnOrAttach` pins the new wording and failed before the change.
- **F3 (LOW)** — `405d21302`: the `accept-audit --batch` usage line in `internal/webstercli/cli.go` covers both routes.
- **F4 (LOW)** — `9f6c456b8`: coverage for the three untested behaviors, as rows and a subtest of existing tests.
  `TestCalibrate` gains a subtest for a fork with no estimated growth (no growth ratio, `n/a` cell, out of the growth fit).
  `TestBuildProfileReport` gains a run whose review file does not parse (findings not read, batch still counted) and a no-partition run with no plan commit (marked row).

## Verification after the last fix

`go build ./...`, `go vet ./...` and `go vet -tags integration ./...` pass.
`go test ./...` passes with no failing package.
`go test -tags integration` passes for `tools/tokencount` and `cmd/...` after the last fix, and for the other touched packages (`internal/planglyph`, `internal/planparser`, `internal/fabricengine`, `internal/batcher`, `internal/websterengine`, `internal/webstercli`, `internal/loomcli`, `internal/loomshed`) before it; the fixes touch none of those packages' tagged files.
`lyx loom lint-comments` passes.

## Round 1 regression check

The round 1 fixes hold: the committed `tools/tokencount/tokencount` binary is gone from the tree and ignored, `TestReferenceRule_Match` runs in parallel, the dynamic program's loop variable is `prefix`, the redundant rule-test row is gone, and the per-position peak subtest is in `TestCalibrate`.

## Deferred

- Round 1's deferred items stand unchanged, and neither is in this round's write surface.
  Commits `340ed2e13` and `a39151dac` still carry the 11 MB binary in the branch history, and squashing it away before landing is for `ly:orch` to decide.
  The live `_lyx/config/batcher.yaml` in the fabric repo still carries `startup_context: 60000` in the active `cautious` profile, which `batcher.Active` now refuses; changing it to `master_base: 52000` and `batch_growth: 7000` is for the operator or `ly:orch` once this branch is deployed.
