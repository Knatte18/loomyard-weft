# Fixer report — discussion round 1

All 13 findings were validated against the code, and each premise holds.
All are fixed, in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.

## Changes

- **F1** (D10), fixed as proposed: cleanup keeps the handoff `State.LastHandoff` names and removes nothing when `LastHandoff` is empty or names a missing file.
  The Bound, the prescribed `doc.go` sentence and the Acceptance row now say the same.
  The premise was confirmed in `internal/orchengine/watcher.go` (`toIdle` clears `PendingHandoff` and leaves the file on disk) and in `launch.go`'s `ChooseStartPrompt`, which reads `LastHandoff` only.
- **F2** (D9), fixed as proposed: the slug's status lock is held, along with the run lock, from before the records run directory is removed until the commit is made.
  Every `*.lock` file is kept in the scratch bullet and in the orphan bullet, and the Acceptance row on the re-seed race is added.
  The Bound's wording and the batten Acceptance row now say "lock files" instead of the singular.
- **F3** (D9), fixed as proposed: a tracked-but-deleted candidate needs a `done` status in the records HEAD tree and a free run lock, and is otherwise skipped and reported.
  The Acceptance row is added.
- **F4** (D1), fixed as proposed: the mode and owner checks apply off Windows only, and on Windows only the real-directory check applies.
  The Acceptance root row is reworded.
- **F5** (D4), fixed as proposed: on Windows the redirect sets `TMPDIR`, `TMP` and `TEMP`.
  A failed removal is reported on stderr and leaves the exit code unchanged, and the next start's sweep reclaims the directory.
  The Acceptance row is added.
- **F6** (D4, Goal), fixed with the first of the two options the fix offers: a test package with no `TestMain` gets one that calls `tmuxkit.Main`, and the tmux-isolation scan fails a package without one, except its allowlist entries.
  The Acceptance row is added, and a Scope In bullet for this work is added so Scope agrees with D4.
  The premise was confirmed: `internal/lock`, `internal/orchengine` and `internal/configengine` use `t.TempDir()` and have no `TestMain`, and `cmd/lyx/tmuxisolation_test.go` today requires a `TestMain` only for packages with tagged test files.
- **F7** (D6, support log Q3), fixed as proposed: each allowlist entry is bounded to one file and one creation call, carries an obstacle reason, still cleans up on normal exit, and lives in the scan test.
  Q3's auto-pick in the support log is updated to match.
- **F8** (D7), fixed as proposed: the Bound is now stated against each segment's verb, and every refusal lives in the shared module function.
  The Acceptance row on the orch segment outside the idle phase is added.
- **F9** (D11), fixed as proposed: a board read failure fails the segment and removes nothing.
  The Bound and the `.scratch` Acceptance row are extended to match.
- **F10** (D11), fixed as proposed: the sole-declarer clause is deleted.
  Confirmed: `tools/sandbox/report.go:129` and `internal/testkit/scankit/scankit.go:38` both use `.scratch`.
- **F11** (D2, D1), fixed as proposed: entries within the grace are skipped untouched, and past it a lock-less entry is removed and a locked one only when its lock can be taken.
  The sweep never creates a lock file, and D1's owner takes its lock with a blocking acquire.
  Confirmed: `lock.TryAcquireWriteLock` calls `flock.New(path).TryLock()`, which creates the file.
- **F12** (D3), fixed differently: the proposed text names "the same real-directory, mode and owner checks D1 applies".
  After F4, D1 applies mode and owner checks off Windows only, so that wording would wrongly imply them on Windows.
  D3 instead says "after the same root checks D1 applies on its platform, failing the same way when they fail".
- **F13** (D9), fixed as proposed: a records run directory counts as absent only on a not-exist error, and any other stat error skips the slug and is reported.

## Disputed

None.

## Deferred

None.
