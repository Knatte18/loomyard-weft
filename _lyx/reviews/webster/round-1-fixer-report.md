# Webster round 1 — fixer report

## Fixed

- **F3** (`0b09f9e46`): the two refusals card 6 added now name a way forward.
  `refuseOverUnfinishedMerge` refuses an unreadable recorded parent branch through the new `parentBranchUnreadableMessage`, which tells the caller to run `lyx loom start --parent <branch>` in the task worktree when there is no origin record, and otherwise to fix the named file and re-run the verb.
  `runDriverSpawnAndWait` refuses a status read that fails for a reason other than a decode failure by telling the caller to re-run `lyx loom start`, and to fix the named file or directory if the failure persists.
  `contracts/specs/refusal-spec.md` has two new loom rows, `start: status read failed` and `start/resume: parent branch unreadable`.
  `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` now takes a list of `wantIn` strings.
  Its parent-branch row asserts the way forward.
  A new row puts a directory where the status file belongs, so the read fails with `state.ErrRead`, and asserts that refusal's retry.
- **F4** (`3b5fd1e4d`): `marker = shedrun.ParkMarker` now has its space in `internal/loomcli/start_midmerge_test.go`, and `internal/testkit/envkit/envkit.go`'s `nilLegal` map is realigned to gofmt's column.
  The merge-in had left that map half-aligned: on `main` and at the merge-base it is gofmt-clean.
  Both edits were made with Edit.
  `gofmt -l` reports neither file now.
- **F5** (`ef72e1ab6`): removed the `clean pair with a truncated status file` row and the `truncatedStatusBody` constant from `TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds`.
  The unknown-field row covers the same `state.ErrDecode` branch.

- **Gate finding** (`25e7df7cd`): the round's verify failed in `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why`, at `smoke_test.go:601` with `probe run lock: ... run.lock: no such file or directory`.
  The gate re-ran only the two Publish-failure subtests with `-run`.
  Both depended on the seeded status file and the `.lyx/shed/<slug>/` run directory that the earlier steps leave behind.
  Run alone, `killDriversAndClearLog`'s run-lock probe found no directory, and the poison helpers found no status file to rig.
  Running `-run` over exactly those two subtests reproduced the failure; the full test passed every time.
  `internal/loomcli/smoke_test.go` gained a `bootstrapUnlessSeeded` closure.
  When no status file exists, it runs one genuine `loom start`.
  Each of the two subtests calls it before `killDriversAndClearLog`.
  With `-count=2`, each subtest alone, the pair together and the full `TestSmokeBootstrapLifecycle` all pass under `--tags tmux`.
  `go vet -tags tmux ./internal/loomcli` and `lyx loom lint-comments` are clean.

Checks run after the fixes: `lyx gate test ./internal/loomcli ./internal/testkit/envkit ./contracts/...` and the same packages with `--tags integration` both pass.
`lyx loom lint-comments --base d2ad9fc55` is ok.

## Disputed

- **F1**: the review says the fix at HEAD has no tmux-tier run behind it, and that is no longer true.
  The gate's failure after this round came from running the subtest without the steps it depends on, not from the cause this finding names; the gate-finding fix above covers it.
  `lyx gate test --tags tmux ./internal/loomcli -- -run TestSmokeBootstrapLifecycle -count=1 -v` at `d2ad9fc55` passes every subtest, including `--- PASS: TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why (0.38s)`.
  The cause, `currentProducer`'s strict status read, was already removed in `d2ad9fc55`, the commit after the logged run, so this round made no code change for it.
  This round's later commits change only refusal text, gofmt whitespace and one test row, all off the smoke path.
- **F2**: same evidence, from the same run: `--- PASS: TestSmokeBootstrapLifecycle/malformed_status_proceeds_to_the_success_envelope_and_logs_why (0.33s)`.

## Deferred

Nothing deferred.
