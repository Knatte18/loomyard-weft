---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The fix key's second clause, making the Wait died path remove a held strand, is already true: strandRemovable (internal/shuttleengine/stop.go:218) includes Held, so removeStrandIfPresent in endFailedRelaunch and settleStaleRelaunch removes a held strand whatever the mark says. Not made. The declined path keeps the run going by design, so a strand whose release failed stays until a stop or the died path removes it. Made: stopForRelaunch returns the release failure wrapped beside the end failure, and reedengine/doc.go states who owns clearing a hold."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: deferred
    reason: "Commit c3f46285's commit message records that the embedded-line test's NoMutant row failed on the shared env slice, so dropping the hunk can fail this branch's verify gate. The fix key's alternative applies: leave it, and have the parent record the reason in the landing summary, which is the parent's to write."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Fixer report, code round 1

Every change is committed once the tier-0 gate passed, in the commits listed under each finding.
The gate ran over `internal/shuttleengine`, `internal/shuttleengine/claudeengine`, `internal/orchengine`, `internal/reedcli` and `internal/reedengine`, and again over the first two shuttle packages after the self-check changes.

## Changes

- **F1** (`a81556bb`): `readRegistry` in `claudeengine/resume.go` reads a registry directory that does not exist as an empty registry, which covers `checkResumable`, `sessionHolder`, `processLiveness` and `registryHoldsLiveEntry`.
  Any other read failure stays unreadable.
  The `TestProcessLiveness`, `TestSessionHolder` and `TestCheckResumable` rows that pinned the opposite now expect dead, no error and no warning for a missing directory.
  New rows pin a live entry beside a missing second directory reading alive, and a registry path that is a regular file staying unreadable.
- **F2** (`4107f4be`): `relaunchReed` has a `beforeRelaunch` hook that snapshots the run record and the events file size when reed is asked to relaunch.
  `checkRecordedBeforeRelaunch` fails unless the record already holds the relaunched session id, the prompt line, `Started` false, the relaunching mark and both offsets at the events size.
  It runs in the switch-relaunch row, the fresh-relaunch row and the start-path repeat row.
- **F3** (`4107f4be`, `b12040a9`): see the disposition above.
  `TestRun_StopForRelaunch_ReturnsAFailedHoldReleaseBesideTheEndFailure` pins that both failures are in the returned error.
- **F4** (`4107f4be`): `apiErrorWatch.turnEnds` is seeded once per watch from the turn ends between `LaunchOffset` and the cursor, through `Run.turnEndsBetween`.
  A new row of `TestWait_RefusedResumeStartsFresh` covers a copied resume whose load turn ended and whose next turn end is an API error.
- **F5** (`16505557`): `validateAccounts` refuses a `~` or `~/` config directory when the home directory is empty, naming the key, the directory and the unresolved home.
  `TestLoadConfig` has the row.
  The comment in `config.go` follows.
- **F6** (`c5fada97`): the `lym reed resume` help names both causes of a held strand.
- **F8** (`eb514d4a`, `b5f2cb3b`): the three `//testtiming:keep` markers sit above the three named tests.
- **F9** (`3ecb167d`): the orphan-stop test has a relaunching run within its bound and a dead pid, which stays running and unlisted.
  The step-stop test has a relaunching run past its bound, which is stopped and whose held strand is removed.
- **F10** (`4107f4be`, `b5f2cb3b`, `b12040a9`): rows and tests for the declined relaunch with its restored `handledOffset`, a failing `SwitchAccount` on the Wait path and the start path, a failing `Fresh` restart that ends the watcher, and a failing `RefusedResume` ask that restarts nothing.
  The declined-relaunch test skips where chmod cannot make the record write fail.
- **F11** (`4107f4be`): start-path rows for a repeat that resumes the same session with its transcript copied, and for a repeat whose check reports no transcript and starts fresh.
- **F12** (`bd5d7628`): a `TestClaude_PickAccount` row probes the account file's lock from inside the names call and fails when the lock is free.
- **F13** (`c67a1971`): a `TestLocateSessions` row with the env module's `CLAUDE_CONFIG_DIR`, and a `TestClaude_LiveReadersReadTheLaunchedDirectory` row where the resolved directory lacks the transcript and another known directory holds it.

## Gate repair

The line-length gate flagged four comment lines this round added, in `accountrelaunch_test.go`, `claudeengine/resume_test.go`, `stop.go` and `stop_test.go`.
Each is now split at a sentence or clause boundary, in a final commit that touches comments only.
The tier-0 gate over the two shuttle packages passes after it.

## Self-check

- fix-code-correctness, MEDIUM: the `reedengine/doc.go` sentence promised a removal no code does on the declined path. Fixed with my own change: the sentence now says a hold whose release failed stays until the caller removes the strand.
- fix-code-correctness, LOW: the declined-relaunch test depends on chmod blocking writes. Fixed: it skips when a probe write succeeds.
- fix-code-correctness, LOW: `errors.Join` made the declined warning multi-line. Fixed: one `%w; end its hold: %w` wrap, which keeps the audited return-site count of `stopForRelaunch`.
- fix-consistency: no findings.
- fix-proposal, MEDIUM: the same `doc.go` promise. Merged with the fix-code-correctness finding and fixed with it.
- fix-proposal, LOW: `readRegistry` treats a missing config directory like a missing `sessions` directory. Disputed, `every fix worse`, below.
- fix-proposal, LOW: the same chmod dependency. Merged with the fix-code-correctness finding and fixed with it.
- fix-affected-set: no findings.

## Disputed

- fix-proposal LOW on `readRegistry`, ground `every fix worse`.
  A listed account that never logged in has no config directory at all, which is the case F1 names ("listed accounts that never ran"), and a directory on an unavailable volume fails with the same `ErrNotExist`, so no check can tell them apart.
  A narrower rule that counts only a missing `sessions` child of an existing config directory would bring back the F1 defect for that account: its first switch ends the run died.
  Evidence: `internal/shuttleengine/claudeengine/resume.go:303`, and the F1 finding text in the review.

## Deferred

- F7, see the disposition above.
  The parent records the reason for keeping commit `c3f46285` in the landing summary.

## Rulings

None reached me.
