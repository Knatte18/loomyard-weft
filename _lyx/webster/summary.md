# Per-role models and convergence guard

Batch 01 (`01-loom-config-keys`, cards 1–4) finished `done` at head `067ba1fe9`.
It added the eight new `loom.yaml` keys with load-time grammar checks, template comments and docs.
PR-Rework now spawns on `rework`, falling back to `plan`, resolved at run start.
`BouncerConfig` carries separate seed and judge model pairs, and each segment's judge and seed are resolved at run start and threaded per Bouncer row through `Env.RowBouncerModels`; the row's old model keys are retired.
Reported deviations outside the declared file-ops were compile-driven edits: `internal/loomengine/rework.go`, `internal/loomengine/review.go`, `internal/shedrecipe/recipe.go`, `internal/shedrecipe/fixture_test.go`, `internal/shedrecipe/entries_bouncer.go`, `internal/shedadapters/bouncer.go`, several `internal/shedadapters` test files and `internal/loomcli/wiring_test.go`.

Batch 05 (`05-convergence-guard`, cards 5–7) finished `done` at head `85aee7d86` with no reported deviations.
It added the convergence guard, which refuses a CONVERGED verdict over an open BLOCKING finding and a ledger that drops a key, archiving the judgment so the round is re-judged.
The usage record gained seed and judge lists with merge-on-write, and a record without a review half reads as missing.
The Bouncer records every seed and judge pass whose result reached it.
The fork reported editing `cmd/lyx/testdata/test-budget.yaml` with `sed` (the `internal/shedadapters` row, 159 to 160), against the no-script edit rule.
The fork also noted that `docs/research/haiku-judge.md` still names the removed `Env.JudgeModel`, `JudgeEffort` and `JudgeVersion`.

## Verify gate fixes

The gate's first attempt failed on `internal/battencli.TestWire_EverySeamFilled`: `RowBouncerModels` was nil after `wire()`.
The fixer fork added `RowBouncerModels` to `nilLegal` in `internal/testkit/envkit/envkit.go`, beside `RowReviewModels` and `RowClusterFans`, because batten wires no Bouncer rows.
It touched neither the test nor its assertions.
The fix is commit `4c20a3750` (`fix: list RowBouncerModels as a legal nil seam in envkit`).
The fork's gate run over `internal/battencli`, `internal/testkit/envkit` and `internal/loomcli` passed, and the worktree was clean.
