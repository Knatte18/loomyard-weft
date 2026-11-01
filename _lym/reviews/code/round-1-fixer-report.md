# Fixer-report, code review round 1

## Changes

All eight findings were fixed as proposed, in five commits.

- F1 (`ideengine: fix F1, F4`): `seed` records a `windowDir` beside `launchPath`; for the prime it is `filepath.Dir` of the workspace file, the hub directory. `Spawn` passes it to `placeWindow`. `checkPrimeWritesHubWorkspace` now sets a window command and asserts one marker run after a prime `Spawn`.
- F2, F3, F6, F8 (`desktophook: fix F2, F3, F6, F8`): the wait and the two command bounds became an unexported `timing` value, with `productionTiming` holding `time.Sleep`, 30s and 5s. `export_test.go` exposes `AfterWindowWithTiming`. `TestAfterWindow` now has rows for the wait (the sleep sees the configured duration and the marker is absent when it begins), a hanging command cut off at a 50ms timeout, a blank command, and an unset command with a nonzero wait. The unset row's vacuous marker check is gone; the no-command rows assert that no wait ran. The hang row uses `exec sleep 60`, so the kill reaches the sleep and leaves no orphan.
- F4 (`ideengine: fix F1, F4`): new step `checkWindowCommandFollowsALaunch` calls `OpenTask` and asserts one marker run, then makes `CodeLauncher` fail and asserts `OpenTask` and `Spawn` return its error and leave the run count at one.
- F5 (`reedcli: fix F5`): new tier 1 test `TestPlaceTerminalWindow` runs `placeTerminalWindow` on a directory in no worktree (nil, no marker) and on the prime worktree of a hub copy (the configured command runs there).
- F7: disputed after the gate; see `Disputed`. The first fix reverted the help line to "Keys the current template does not recognize are", and the verify run then failed `TestCLIForm_EveryCommand`; a later commit restored "Keys the template lacks are".
- After the self-check, `placeWindow`'s parameter, doc and warn key were renamed to `windowDir`, and the header of `spawn_hub_integration_test.go` now names `OpenTask` and the window command (commit `ideengine: name placeWindow's directory windowDir after the self-check`).
- After the gate, the comments of `TestAfterWindow` in `internal/desktophook/window_test.go` and of `seedResult` in `internal/ideengine/spawn.go` were rebroken at sentence boundaries, one sentence per line, with no change of meaning.

Gate: `lym gate test --tier 1` passed for `./internal/desktophook`, `./internal/ideengine` and the `reedcli` `TestPlaceTerminalWindow`.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW: `placeWindow` doc, parameter and log key still named a launch path. Fixed as proposed.
- fix-consistency, LOW: header of `spawn_hub_integration_test.go` did not cover `OpenTask`. Fixed as proposed.
- fix-affected-set, LOW: the same stale `placeWindow` name. Duplicate of the fix-consistency finding; fixed with it.

## Disputed

- F7, ground `premise false`: the rewording is not unrelated to the window command.
  The branch adds the `desktop` module, whose name `buildConfigLong` lists twice in the `lym config` help page.
  With the original wording the page is 2012 bytes, and `TestCLIForm_EveryCommand` in `cmd/lym/clitree_test.go` fails with "lym config: Long is 2012 bytes; the cap is 2000" (verify log line 15396).
  The shorter wording is 21 bytes shorter, which brings it under the cap, and `lym gate test ./cmd/lym ./internal/configcli` passes with it.

## Deferred

None.
