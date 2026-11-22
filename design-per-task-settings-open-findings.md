# Fix the review findings per-task-settings-one-lookup landed open

`per-task-settings-open-findings` · Note · bug, dobby, config, reconcile, board

## Seen

`per-task-settings-one-lookup` landed as `118bee8ad` after round 2 of its Code-Burler accepted on budget, with these findings open.
The full findings, with locations and proposed fixes, are in round 2's review in the run's archived records:
`git -C loomyard-weft show archive/per-task-settings-one-lookup/d3f72e425e2a:_lym/reviews/code/round-2-review.md`.

## Findings

- **F26 `reinterpret-discovers-worktrees-and-anchors`** (BLOCKING): `settingPlaces` in `internal/dobbyengine/reinterpret.go` lists the hub dir for `.git` entries and derives each anchor from the prime's.
  The weft siblings count as worktrees, and a pair with its own anchor is read and written at the wrong dir, against PATTERN-cwd-resolution.
  The reinterpretation should get the code worktrees as told geometry from `hubreconcile`.
- **F2 `per-task-keys-declared-at-several-sites`** (BLOCKING): the literal `"beam"` remains in `dobbyengine.ConfigSpec`, the configreg registration and `beamcli` `mergeParent`, beside `beamsetting.Module`.
- **F27 `registration-test-lacks-keep-marker`** (BLOCKING): `TestLookup_BeamCarriesThePerTaskSettingsReinterpretation` needs `//testtiming:keep` with its reason.
- **F28 `duplicate-row-in-prepare-run-settings`** (BLOCKING): a duplicate row in `TestPrepareRunSettings` in `internal/configreg/runsettings_test.go`.
- **F5 `carry-misses-runs-in-flight`** (MEDIUM): the one-time carry of the retired board fields into seeded runs misses a run on another machine and one behind an early board save, since board verbs run no reconcile and the marker is hub-wide.
- **F13 `beam-project-notify-on-land-dropped`** (MEDIUM): a beam project's `notify_on_land` is folded away on load, so a board save before the carry loses it.

F5 and F13 only matter while a hub still holds the retired board fields; check first whether any hub does.
