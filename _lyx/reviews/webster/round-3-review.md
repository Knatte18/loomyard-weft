---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomshed/gates_test.go:462
    summary: TestNewPlanGate's done-card-edited table calls NewPlanGate with three arguments but the merged-in signature takes scratchDir, so the loomshed test binary does not build and Publish's verify fails
---

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which is `57243019b..HEAD`.
The Publish failure section names merge-in commit `52ae208c0`.
That merge brought in nothing beyond `57243019b`, which is already the merge base, so the branch's own range and the merge's diff coincide.

Checked against the plan: each of cards 1–26 has its landing in the diff, and the per-card mechanical checks that apply hold.
Card 22 is the only card with a `Delete` group, and `grep` finds no remaining reference to `ShellWaitExpires`, `noticeMaxTasks`, `ExpiredShells`, `expiredShells` or `expiredLabels`.
No card has a `Rename` group.
Package tests pass for `websterengine`, `webstercli`, `fabriccli`, `fabricengine`, `shuttleengine`, `planparser`, `gitexec`, `shedadapters`, `shedrecipe`, `verifytree`, `gatecli`, `loomcli`, `reedengine`, `lyxcwd`, `configcli`, `hubgeom`, `contracts/stencils`, `cmd/lyx` and the testkits.
The `integration` tier passes for the packages the diff touches, apart from `loomshed`.
`loomshed` fails to build in both tiers.

### [BLOCKING] loomshed's test binary does not compile against NewPlanGate

**Location:** `internal/loomshed/gates_test.go:462`
**Issue:** This is the Publish failure's cause.
`/home/knatte/Code/loomyard-LYXHUB/webster-replan-state/.lyx/verify/publish-failure.log` reports `vet: internal/loomshed/gates_test.go:462:107: not enough arguments in call to NewPlanGate; have (string, string, planindex.Index); want (string, string, string, planindex.Index)`.
I reproduced it at HEAD with `lyx gate test ./internal/loomshed`, which ends `FAIL ... [build failed]`.
`NewPlanGate(anchorPath, worktreeRoot, scratchDir string, index planindex.Index)` in `internal/loomshed/gates.go:159` takes a `scratchDir` that main added.
Card 7's `TestNewPlanGate` done-card-edited table (commit `cad8f04b2`) was written against the three-argument form: `NewPlanGate(anchorPath, t.TempDir(), planglyph.NewIndex(...))`.
Every other call in the file passes `""` as the third argument.
The package's test binary does not build, so every `loomshed` test fails to run, including `TestWebsterRecordNote` (card 8), the `done-card-edited` rows (card 7) and `TestPublishFailure` (card 23).
The branch is therefore not mergeable: the plan's `## verify:` command cannot pass while the package is unbuildable.
**Fix:** In `internal/loomshed/gates_test.go:462`, pass the scratch directory argument the other calls pass: `NewPlanGate(anchorPath, t.TempDir(), "", planglyph.NewIndex(fabricengine.NewReferenceRule()))()`.
Then re-run `lyx gate test ./internal/loomshed` and `lyx gate test --tags integration ./internal/loomshed` to confirm nothing else in the package was hidden behind the first compile error.
