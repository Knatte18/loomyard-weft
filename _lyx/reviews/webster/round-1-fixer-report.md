# Round 1 fixer report

All five findings are fixed, one commit each, none pushed.

## Fixed

- **F1 (BLOCKING)** — `9d04c26e4`: removed the committed `tools/tokencount/tokencount` binary and added `/tools/tokencount/tokencount` and `.exe` to `.gitignore`.
- **F2 (BLOCKING)** — `306d42f78`: `TestReferenceRule_Match` and its subtests now call `t.Parallel()`.
- **F5 (LOW)** — `3fbcaf0b0`: deleted the row "quoted search pattern returns neither" from `TestReferenceRule_Match`, since `TestRefScanner_Matches` covers it.
- **F4 (LOW)** — `424c776a2`: renamed the loop variable in `costBatcher.Batch`'s dynamic program from `before` to `prefix`, so it no longer shadows the `before` parameter.
- **F3 (MEDIUM)** — `3692fc1f4`: added the subtest "peak rows are priced at each fork's own position" to `TestCalibrate`.
  It runs the `grow` profile over alpha and asserts each row's `Estimate` and `EstimatedGrowth` at positions 1 to 4.
  With `addRun` temporarily pricing every fork at position 1, the subtest failed on the rows at positions 2 to 4; the mutation was reverted before the commit.

## Verification after the last fix

`go build ./...`, `go vet ./...` and `go vet -tags integration ./...` pass.
`go test ./...` passes with no failing package.
`go test -tags integration` passes for `tools/tokencount`, `internal/batcher`, `internal/fabricengine` and `cmd/lyx`.
`lyx loom lint-comments` passes.
The tmux and llm tiers were vetted before the fixes and no fix touches a tagged file.

## Deferred

- F1's history: commits `340ed2e13` and `a39151dac` still carry the 11 MB binary in the branch history.
  A fixer must not rewrite history, so removing it from the history (a squash or rebase before landing) is for `ly:orch` to decide.
- The live `_lyx/config/batcher.yaml` sits in the fabric repo, outside the diff and this round's write surface.
  It still carries `startup_context: 60000` in the active `cautious` profile, which `batcher.Active` now refuses; changing it to `master_base: 52000` and `batch_growth: 7000` is for the operator or `ly:orch` once this branch is deployed.
