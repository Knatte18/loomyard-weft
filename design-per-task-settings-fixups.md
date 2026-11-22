# Per-task beam settings: drop the member-line exception and fix the open findings

`per-task-settings-fixups` · Task · bug, dobby, config, reconcile, board

Depends on: [`orch-autonomy-mode`](design-orch-autonomy-mode.md)

## Goal

Finish what `per-task-settings-one-lookup` (`118bee8ad`) left: remove the beam-member exception from the landing notice, and fix the review findings it landed open.
The full findings, with locations and proposed fixes, are in round 2's review in the run's archived records:
`git -C loomyard-weft show archive/per-task-settings-one-lookup/d3f72e425e2a:_lym/reviews/code/round-2-review.md`.

## 1. A beam member gets its landed line only when its land_notice is on

`rememberRuns` in `internal/dobbyengine/landnotice.go` sends a member its landed line while the beam's pull request is recorded, whatever its `land_notice` reads.
This was Code-Burler finding F12, `member-line-overrides-explicit-off`.

Decision, operator, 2026-11-22: no exception.
Per-task beam settings follow one rule, and `land_notice` off means no line.

- `rememberRuns` sends a member's line only when its reading is on; the follow-up stays on that line.
- The package doc of `landnotice.go`, the doc comments of `landNotices` and `rememberRuns`, and `LandNoticeState.Member` drop the exception; `Member` goes if nothing else reads it.
- `pattern/PATTERN-landing.md` and `pattern/PATTERN-config-layers.md` lose any mention of it.
- Keeping the pull request's description current without the orch is out of scope: note `beam-pr-describes-itself`.

## 2. The findings landed open

- **F26 `reinterpret-discovers-worktrees-and-anchors`** (BLOCKING): `settingPlaces` in `internal/dobbyengine/reinterpret.go` lists the hub dir for `.git` entries and derives each anchor from the prime's.
  The weft siblings count as worktrees, and a pair with its own anchor is read and written at the wrong dir, against PATTERN-cwd-resolution.
  The reinterpretation gets the code worktrees as told geometry from `hubreconcile`.
- **F2 `per-task-keys-declared-at-several-sites`** (BLOCKING): the literal `"beam"` remains in `dobbyengine.ConfigSpec`, the configreg registration and `beamcli` `mergeParent`, beside `beamsetting.Module`.
- **F27 `registration-test-lacks-keep-marker`** (BLOCKING): `TestLookup_BeamCarriesThePerTaskSettingsReinterpretation` needs `//testtiming:keep` with its reason.
- **F28 `duplicate-row-in-prepare-run-settings`** (BLOCKING): a duplicate row in `TestPrepareRunSettings` in `internal/configreg/runsettings_test.go`.
- **F5 `carry-misses-runs-in-flight`** (MEDIUM): the one-time carry of the retired board fields into seeded runs misses a run on another machine and one behind an early board save, since board verbs run no reconcile and the marker is hub-wide.
- **F13 `beam-project-notify-on-land-dropped`** (MEDIUM): a beam project's `notify_on_land` is folded away on load, so a board save before the carry loses it.

F5 and F13 only matter while a hub still holds the retired board fields; check first whether any hub does.
