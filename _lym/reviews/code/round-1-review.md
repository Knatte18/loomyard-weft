---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/adoptrecord/watch.go:124-137
    summary: A beam named as an adopt subject with no owner recorded resolves to its parent subject, so the adopt does not cover its own worktree
    fix: In ownerSubject, or in covers before it, treat a non-empty slug that the record itself lists as its own subject when the entry has no owner of its own, instead of walking to the parent. Walk up the chain only for a slug the record does not list, stopping at the first beam with an owner. Add a TestWatchCovers row for a beam listed in the record with no owner recorded under a foreign-owned root, expecting it covered.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/ownerarchive.go:756-774
    summary: The only-tips-origin-lacks decision has no test row for origin ahead of the tip, a diverged origin, or a branch absent on origin
    fix: Add rows to TestArchiveOwnerTips in ownerarchive_integration_test.go. One has origin ahead of the local tip and expects no tag. One has a diverged origin branch and expects a tag. One has the branch absent on origin and expects a tag.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/ownerarchive.go:249-306
    summary: Nothing tests the pairs the archive covers (own pair with root for the prime, one pair per run) or the skip while an adopt covers the worktree
    fix: Add a test in beamcli over a hub fixture that asserts the slugs handed to the archive for the prime and for a beam with a run under it, and that ownerArchiveBeforeRefusal archives nothing while an adopt record covers the worktree.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/loop.go:518-552
    summary: The adopt-hold test row asserts only that no pass is spawned, though D4 requires no task start and none of the tidy, test-map and landing steps
    fix: Extend the AnAdoptInFlightKeepsTheDobbyUpAndIdle row of TestRun_ForeignOwner in handover_test.go with spies on the task start, tidy, test-map and landing seams, expecting none to run. Add a row where the adopt lifts and the next probe hands over.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/handover.go:403-410
    summary: archiveFailures.Recover is never called and archiveFailing is never cleared, so a later archive failure after a recovery logs nothing to the dobby log
    fix: In probeOwner, when the beam is no longer owned elsewhere or the hold applies, call d.archiveFailures.Recover with a recovery message and reset d.archiveFailing to false. Also reset them after a successful archive, before the exit.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptrecord/watch.go:105-137
    summary: Nested parent chains, a parentOf error, the once-per-episode failure log, the orch hold transitions and the orchcli accessor wiring have no test
    fix: Add rows to TestWatchCovers for a beam with no owner under a beam with one and for a parentOf error. Add a second-tick row to the watcher_owner_test.go table that lifts the adopt and expects the strand stopped. Add one test of UnderAdoptWatch over a hub fixture.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/handover.go:362-372
    summary: The failed Bolt pull episode, the pauseRunsForOwner skip of done and already-owned-elsewhere runs, and its error paths have no test row
    fix: Add rows to TestRun_ForeignOwner for a done run and a run already recorded OwnedElsewhere, expecting no pause. Add a failing PullBolt row to TestRun_BoltPullBringsInANewOwner, expecting the owner read to go on and one episode line.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/ownerarchive.go:695-781
    summary: The held teardown lock, the joined failures over several pairs, and the no-branch-push guarantee have no test
    fix: Add rows to TestArchiveOwnerTips for a held teardown lock expecting ErrPairPushRunning, for two pairs where one fails expecting the other still tagged, and an assertion that the origin branch tip is unchanged after the archive.
---

### [BLOCKING] A beam adopted by name with no owner of its own is not covered

**Location:** internal/adoptrecord/watch.go:124-137
**Issue:** `ownerSubject` returns the slug itself only when the entry is a beam project with an owner recorded.
For `lym adopt b` where `b` is a beam whose entry has no owner yet, it walks to the parent and returns `root`.
`record.Covers("root")` is then false, because the record lists `b` and not `root`.
Until the owner write of `b` stands, `b`'s orch watcher stops its strand and its dobby archives and exits.
D4 says a subject under adopt covers its own worktree, so the window the decision exists to cover is open in this case.
The tests only use beams that already have an owner, so none catches it.

### [MEDIUM] Archive tip decision lacks origin-ahead, diverged and absent-branch rows

**Location:** internal/fabricengine/ownerarchive.go:756-774
**Issue:** `archiveOwnerTip` has three branches that decide whether a tip is archived: origin holds the tip as an ancestor, origin diverged, and origin lacks the branch.
`TestArchiveOwnerTips` covers only origin equal to the tip and a commit origin lacks, so a broken `isAncestorOrEqual` branch would pass.

### [MEDIUM] The pairs the owner archive covers are untested

**Location:** internal/beamcli/ownerarchive.go:249-306
**Issue:** `archiveOwnerTips` builds the worktree's own pair, named `root` for the prime, and one pair per run `DobbyRuns` lists.
`ownerArchiveBeforeRefusal` skips the archive under an adopt.
D11 step 2 depends on the right pairs and slugs, and no test pins them.

### [MEDIUM] The adopt-hold row does not pin the steps D4 says are skipped

**Location:** internal/dobbyengine/loop.go:518-552
**Issue:** The hold gates `drivePasses`, the task start, tidy, test-map and `landBeam` through `scheduling` and `adoptHolds`.
The only hold row asserts that no pass was spawned.
Removing the gate from any other step would leave it passing.

### [LOW] The archive failure episode never resets

**Location:** internal/dobbyengine/handover.go:403-410
**Issue:** `archiveFailing` is set on the first failure and never cleared, and `archiveFailures.Recover` is never called.
After the owner returns, or a hold applies, a later failure in a new episode writes no dobby log line.

### [LOW] Watch, orch hold and wiring edge cases are untested

**Location:** internal/adoptrecord/watch.go:105-137
**Issue:** No test covers a nested parent chain, a `parentOf` error, the once-per-episode log, the orch hold ending, or `UnderAdoptWatch`.

### [LOW] Bolt pull failure and run-pause skips are untested

**Location:** internal/dobbyengine/handover.go:362-372
**Issue:** No test covers a failed Bolt pull, `pauseRunsForOwner` skipping a done run or one already owned elsewhere, or its two read-error paths.

### [LOW] Lock, multi-pair and no-branch-push behaviour of the archive are untested

**Location:** internal/fabricengine/ownerarchive.go:695-781
**Issue:** No test covers the held teardown lock, the joined failure over several pairs, or that no branch moves on origin.

## Rejected

- Fork finding on `archiveBeforeOwnerRefusal` running the archive for a non-owner refusal: `beamUpRefusal` (`internal/orchcli/refusal.go:57`) checks the foreign owner first, so whenever the owner is foreign the refusal is the owner refusal.
- Fork finding on the run report listing no run for a D11 tag having no report-level test: `TestOwnerArchiveTagName` pins the name and that `ArchiveTagSlug` and `ArchiveTagPrefix` miss it, and every report reader goes through those; the fork itself called it acceptable.
- Test-economy candidates in the redundancy report: every listed test is "not judged", and I found no duplicated coverage among the new tests.
