# Webster-Burler round 1 — fixer report

All seven findings were valid against the code and fixed, each in its own commit on top of `aa76dfb0d`.

## Fixed

- **F1** (`6be88de1e`): deleted `TestGoTestArgs_AssemblesDirCapTagsPackagesAndFlagsInOrder` from `internal/gatecli/cli_test.go`.
  The packages-only shape is now an exact `slices.Equal` assertion in the scenario's "-C names a worktree" subtest, which previously checked only `Args[2]`.
- **F2** (`850551a4a`): `TestRenderBatchGate` and `TestRerunCardVerifies_Slotted`, and their subtests, call `t.Parallel`.
  `TestPublishVerify_FailureRecord`'s doc comment now names the swapped package-level `NewGitHubClient` as the process-global state that keeps it serial.
  The touched tests pass under `-race`.
- **F3** (`04af9ef7c`): `TestGateTest_Scenario` gains a table of three refusal rows over the shared hub, each breaking it in setup and repairing it after:
  - a `gate.yaml` holding `slots: 0` exits 1 naming `lyx config gate`;
  - a slot directory replaced by a file exits 1 with "cannot acquire a gate slot";
  - a go binary that does not exist exits 1 naming "put go on PATH".

  Each row also asserts that the stand-in never ran and that no slot or wait record is left behind.
  The rows pass run alone as well as in sequence.
- **F4** (`063228df9`): `gateslot.LoadConfig` now stats `configengine.ConfigFile(baseDir, "gate")` and returns `gate config absent; run "lyx fabric reconcile"` on `os.ErrNotExist`, without matching error text or wrapping `configengine`'s second reconcile command.
  `TestLoadConfig`'s error rows now assert exact messages, so the stacked text that passed before would now fail.
- **F5** (`ad2579bc0`): `checkedPublishFailure` keeps `LogPath` only when its cleaned path equals `paths.PublishFailureLog`.
  `failedTestPackage` now requires the first character to be a letter or digit.
  The `TestPublishFailure` table gains a row for an in-directory log name carrying a newline and prose, and a `-c` package row.
- **F6** (`97f4047b2`): `pattern/PATTERN-config-strictness.md` lists `gateslot` among the strict callers.
- **F7** (`06ed24de5`): `TestConfigOverRealHub` gains a subtest that drives `lyx config menu` through `Command()` with stdin set, from the task pair.
  It asserts exit 1, the "prime worktree" refusal and an unchanged hub `board.yaml`.
  With the guard line removed from `runMenu`, the subtest fails: the menu edited and committed the hub file.
  The guard was then restored.

## Verification

Package-scoped untagged and `integration` runs pass for every touched package: `gatecli`, `gateslot`, `hubgeom`, `websterengine`, `landingshed`, `loomshed`, `configcli` and `cmd/lyx` (test budget and pattern scans).
`lyx loom lint-comments --base aa76dfb0d` is clean.
`lyx gate test` could not be used for these runs: the hub's `_board/_lyx/config/gate.yaml` does not exist yet because production `lyx` predates this branch, so the verb refuses.
Those runs were package-scoped `go test -p 2` instead.

## Disputed

None.

## Deferred

None.
