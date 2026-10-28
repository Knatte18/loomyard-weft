---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/hubpull_integration_test.go:30
    summary: Acceptance criterion 5, the pair pull waiting for a held parent lock, has no test; acquireBranchLock's waiter, holder notice and fabric-pull run id run only uncontended
    fix: In internal/fabriccli/hubpull_integration_test.go add a case that takes the parent lock of the prime's code branch through a parentlock.Store in the board's parent-lock dir as another owner, starts `pull --hub` in a goroutine, waits for the holder-naming notice on stderr (or polls the store for the second waiter) rather than sleeping, then releases the held lock and asserts the pull finishes with the prime moved and the lock record owned by run id fabric-pull while held. Add a row cancelling the command context while the lock is held that asserts the prime side is stopped and the later sides still ran.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/hubpull_integration_test.go:148
    summary: No test shows a side after a stopped side still pulling and moving; the only side after the dirty prime is the no-upstream beam, which returns before any pull
    fix: In TestPullHub_StoppedSideLeavesTheOthersPulled add a second beam that is pushed and moved on origin (as moveBeamOnOrigin does) after the dirty prime, and assert it is reported moved and its file is present, so a loop that ended at the first stop fails. Keep the no-upstream beam as a row after it.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/hubpull.go:137
    summary: With SkipGit set via FABRIC_SKIP_GIT=1 Fabric.Pull returns a zero result, so every pair side is reported stopped with a misleading origin could not be reached detail while Bolt reports up_to_date, and the verb exits non-zero
    fix: In pullPairSide, before Open, return HubPullUpToDate with a detail saying git is skipped when opts.SkipGit is set, matching pullBoardSide; add a PullHub row with a SyncOptions with SkipGit true asserting every side is up_to_date.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/hubpull.go:162
    summary: One unreadable origin record or failed NamedWorktree in localBeamLocations aborts the whole beam list, so later beams are never attempted, against the rule that a stop on one side never skips the next
    fix: Make localBeamLocations return per-slug results (a location or an error) and have PullHub report a failed slug as a stopped beam side with the error as its detail, then continue with the remaining beams; keep a returned error only for the failed worktree list. Add a row with a corrupt origin record for one beam beside a healthy moved beam.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/fabriccli/weft_verbs.go:324
    summary: The diff rewrites the existing non-hub help bullets on fast-forward and warp history rewrite, a change the record puts out of scope (changing what plain `lym fabric pull` does)
    fix: Restore the original wording of the two bullets in the `pull` Long text in internal/fabriccli/weft_verbs.go and keep only the appended --hub paragraph.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/hubpull_integration_test.go:195
    summary: Beam selection, the WeftPulled-false stop, pullBoardSide's skip and error stops and PullHub's list-error return have no test row; several beams, a non-beam task pair beside a beam and a beam owned by another machine are unpinned
    fix: Extend the moved-sides test with a second beam and a non-beam task pair and assert the beam order and that the task pair is absent; add rows for a Bolt skip (a dirty board) asserting stopped with the skip's way-forward detail, and for a pair whose weft origin is unreachable asserting the stopped detail.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/fabricengine/hubpull_integration_test.go:195
    summary: TestBoltPull_WaitsForTheBoardWriteLock waits a fixed real 200ms with time.After, which the testing rules and PATTERN-test-speed forbid, and it passes vacuously if Bolt.Pull ignored the lock but ran slowly
    fix: Replace the fixed wait with a signal or a bounded poll, so hold the lock, start the pull, poll with a short tick until a second flock attempt on the board write lock is observed blocked or the pull has not completed by a deadline, then assert HEAD is unchanged; or make the wait interval injectable and short.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/fabricengine/bolt.go:243
    summary: The Bolt.Pull doc comment is stated in terms of another named symbol as PullThenCommitWritten reports it, and the PullHub comment leans on Pull and the pull-first path
    fix: Rewrite the Bolt.Pull and PullHub doc comments to state the skip and detail contract inline (a non-empty skip carries a detail naming the way forward) without naming PullThenCommitWritten, Pull or the pull-first path.
---

### [MEDIUM] Parent-lock wait has no test

**Location:** internal/fabriccli/hubpull_integration_test.go:30
**Issue:** The decision record's fifth acceptance criterion says the pair pull waits while another owner holds the branch's parent lock and runs once it is released. The CLI test only takes and releases the lock uncontended, and the engine test uses a fake `PairLock` that never contends. The waiter, the holder notice, the `fabric-pull` run id and the Ctrl-C end of the wait are untested.

### [MEDIUM] No test of a moving side after a stopped side

**Location:** internal/fabricengine/hubpull_integration_test.go:148
**Issue:** The record's rule that a stop never stops the next side is pinned only against a no-upstream beam, which returns before any pull. A loop that ended at the first stop would pass for every side that actually pulls.

### [LOW] SkipGit reports every pair stopped

**Location:** internal/fabricengine/hubpull.go:137
**Issue:** `EnvSyncOptions` sets `SkipGit` from `FABRIC_SKIP_GIT=1`. `Bolt.Pull` then returns `up_to_date`, but `Fabric.Pull` returns an empty result with `WeftPulled` false, so each pair is reported `stopped` with an "origin could not be reached" detail and the verb exits non-zero.

### [LOW] One bad beam slug skips every later beam

**Location:** internal/fabricengine/hubpull.go:162
**Issue:** `localBeamLocations` returns an error for the whole list when one origin record or worktree lookup fails, and `PullHub` returns early, so the remaining beams are never attempted. The record says a stop on one side never stops the next.

### [LOW] Unrequested edit to the plain pull help

**Location:** internal/fabriccli/weft_verbs.go:324
**Issue:** The two existing bullets of the `pull` help text are shortened. The record lists changing what `lym fabric pull` does without `--hub` as out of scope, and `fabricengine/doc.go` keeps the longer wording.

### [LOW] Uncovered added behavior

**Location:** internal/fabricengine/hubpull_integration_test.go:195
**Issue:** No row covers several beams in list order, a non-beam pair beside a beam, a beam owned elsewhere, the `WeftPulled`-false stop, Bolt's skip and error stops, or `PullHub`'s list-error return.

### [LOW] Fixed real-time wait in a test

**Location:** internal/fabricengine/hubpull_integration_test.go:195
**Issue:** The test sleeps 200ms on a real clock. It cannot fail falsely, but it passes vacuously for a slow ignoring pull and breaks the repo's determinism rule.

### [LOW] Doc comments lean on other symbols

**Location:** internal/fabricengine/bolt.go:243
**Issue:** The `Bolt.Pull` comment names `PullThenCommitWritten`, and the `PullHub` comment names `Pull` and the pull-first path, which the self-contained comment rule forbids.

## Rejected

- NIT from code-tests that the CLI row `EveryMovedSidePulled` repeats the engine moved test: it adds the exit code and lock-released checks at the envelope level, so it is not a duplicate.
- Detached beam HEAD reported `up_to_date` (code-correctness): `upstreamSHAAt` documents a detached HEAD as no upstream, so the report follows the decided no-upstream rule.
- A Bolt lock error carrying no way forward (code-correctness): the lock acquisition error is an operational failure whose text names the cause, and the record asks for a way forward only on a stop its own path already names.
