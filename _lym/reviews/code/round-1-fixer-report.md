## Changes

- **F1 (MEDIUM), fixed as proposed:** `internal/adoptcli/doc.go` now says a process the worker starts is held by a pause only when it is a run's own process, and that an adopt check passes.
- **F2 (LOW), fixed with my own change:** the proposed test through the root's pre-run cannot reach the gate, because `PersistentPreRunE` installs it only when `!testing.Testing()`.
  I moved the call into `installPauseGate(cmd)` in `cmd/lym/main.go`, which the pre-run calls, and added `TestInstallPauseGate_HoldsOnlyARunsOwnVerbs` to `cmd/lym/main_integration_test.go` (tier 1).
  It runs `board get`, `machine name`, `beam start`, `shed run` and `beam dobby` from the real command tree against a hub with a live hub entry, and asserts `pauserecord.GateCovered()`.
  Dropping or mis-passing `runCommand(cmd)` fails it.
- **F3 (LOW), fixed as proposed:** `TestEnvScrub_MatchesTmuxkitServerEnviron` builds its environ entry and want-list case from `pauserecord.RunProcessEnv`.
- **Fork finding, fixed:** `internal/hibernateengine/doc.go` said a pause holds the network calls of "other processes"; it now says a run's own processes.

- **Gate findings, fixed:** the doc comments of `installPauseGate` and `TestInstallPauseGate_HoldsOnlyARunsOwnVerbs` each had a line of 120 characters or more.
  Each is now split into shorter sentences, one per line.

- **Second gate findings, no change:** the verify run failed in `tools/tokencount` (a tmux server left running by the package) and in `internal/slackengine` (`TestRunRelay/WatchThatFailsToOpenFallsBackToTheOnDiskCheckWithItsOwnBackoff`).
  The branch diff touches neither package.
  A rerun of tier 0 for both packages passed, so both read as flakes, not defects of this round.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, `internal/hibernateengine/doc.go` stale "other processes" claim, fixed as proposed.
  Its other searches found no further site.

## Disputed

None.

## Deferred

None.

## Rulings

None.
