# Fixer-report, round 1

## Changes

- F1 (MEDIUM), fixed as proposed, commit `2cb8e492`: `TestRunInvocationsTestTargets` in `internal/gatecli/testtargets_test.go` runs `runInvocations` over a temp module with `echo narrowed {test_targets} && echo whole {targets}`.
  Its rows pin the kept package in `{test_targets}` with `{targets}` whole, the skip with its stderr line and exit 0 when every package is dropped, and that a dropped invocation does not stop the next one.
- F2 (LOW), fixed as proposed, commit `06cbd4eb`: `"Example"` joins `testFunctionPrefixes` in `internal/impactset/tiertests.go`, with a `TestHoldsTierTest` row for an example-only package.
  The review's `location` line 481 does not exist (the file has 66 lines); the variable is at line 18.
- F3 (LOW), fixed as proposed, commit `836baf9b`: `gateslot/doc.go`, the guard-step refusal in `gatecli/tiers.go`, the comments in `tiers.go` and `gatecli/doc.go`, and the `run.go` notice now name `{test_targets}` beside `{targets}`.
  The tests that assert the old messages (`tiers_test.go`, `test_integration_test.go`) are updated in step.
- F4 (LOW), fixed as proposed, commit `2cb8e492`: `TestTierTestTargets` gains a `./...` row and a failing-`go list` row (a nonexistent `goBinary`); the temp module now has a `go.mod`.
- F5 (decision): left out, as the instruction says; the parent recorded its decision (opt-in kept, no code change).

- Gate repair (comment lint): the comments F3 touched in `gatecli/doc.go` (two), `gatecli/tiers.go` and `gateslot/doc.go` were re-broken at sentence boundaries, one sentence per line, with the over-long `gateslot/doc.go` line split at a semicolon. Wording is unchanged.

Verification: `lym gate test ./internal/gatecli ./internal/impactset ./internal/gateslot` and `lym gate test --upto 2 ./internal/gatecli` both passed.

## Self-check

- fix-consistency, LOW: test names still say "without {targets}" (`test_integration_test.go:1699`, `tiers_test.go:233`, `:299`, `:461`). Disputed, premise false: the commands those tests run hold neither placeholder, so the names stay true.
- fix-consistency, LOW: `docs/dotnet-hub.md:16` names `{targets}` alone. Disputed, premise false: it describes dotnet commands, where `{test_targets}` (a Go package narrowing) does not apply.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

- Self-check finding on test names, ground `premise false`: `test_integration_test.go:1699` runs tier2, whose command holds no `{targets}`, and the sibling names in `tiers_test.go` likewise describe commands that hold neither placeholder.
- Self-check finding on `docs/dotnet-hub.md:16`, ground `premise false`: the file's `{targets}` is a dotnet project path, and `runInvocations` narrows `{test_targets}` by Go package tests only.

## Deferred

None.
