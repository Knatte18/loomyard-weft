---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedcli/watchdog_integration_test.go:372
    summary: Publish verify failed on TestWatchdogDaemon/DownThenUpDoesNotKillDaemon, whose idle window of three 200ms cycles is shorter than a down-then-up under a loaded package run
    fix: In the DiscoversBothSessions subtest of TestWatchdogDaemon, pass runWatchdogLoop a timing copy whose IdleCycles is widened (for example 15, a three-second idle window at watchdogTestCycle) instead of compressedWatchdogTiming() as it stands, and say in the comment above that call why this one loop widens the window; leave compressedWatchdogTiming and the other watchdog tests untouched.
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/fabricengine/replay.go:436
    summary: replayWeft and replayBolt tag set-aside commits only after the branch moved, so a failed tagging whose restore also fails, or any error in replayBolt's by-entry loop, leaves the set-aside commits reachable only through the reflog
    fix: In replayWeft, tag every classified set-aside commit (tagSetAside) before cherryPickOnto, so a later stop cannot orphan one. In replayBolt do the same for the commits the classification loop sets aside before cherryPickOnto, tag each lost-value or refused original in replayBoardCommit as it is appended, and make every error return after cherryPickOnto succeeds in replayBolt (the Apply, blob-read, WorktreeChangedFiles, StageAndCommit and CurrentSHA errors) restore the starting tip through ResetKeep(start), joining a restore failure to the cause. Drop tagSetAsideOrRestore once nothing is tagged after the move. Add rows to TestRecordsReplay in replay_integration_test.go for a tagging failure (a pre-existing set-aside tag at a different sha) and a by-entry Apply error in replayBolt, each asserting the branch tip is the starting tip and every set-aside commit is tagged.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/destroy.go:1828
    summary: cherryPickOnto wraps every ResetKeep error as ErrReplayDirty, so a git failure that is no local-change refusal is reported as an uncommitted-change stop
    fix: >-
      In gitrepo/reset.go add an exported predicate (for example IsLocalChangeRefusal(err error) bool) that is true only for the checked *gitexec.GitError git raises when reset --keep would overwrite an uncommitted change (the not-uptodate and would-be-overwritten stderr).
      In cherryPickOnto wrap ErrReplayDirty around the ResetKeep error only when that predicate holds, and otherwise return the error wrapped with the onto sha and without ErrReplayDirty.
      Add a row to TestRecordsReplay with a ResetKeep failure that is no dirty refusal (for example a held index.lock) asserting errors.Is(err, ErrReplayDirty) is false and the error names the git failure.
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pull.go:328
    summary: settleUnmovedWeft turns a failed local read of the weft's upstream, head or ahead and behind sets into a warn and returns success with weft_pulled false, which card 6 forbids
    fix: >-
      In settleUnmovedWeft split the condition `err != nil || !hasUpstream || len(behind) == 0`.
      When err is non-nil return it wrapped with the weft path and a message that the weft's state against its upstream could not be read, and keep the warn-and-continue return only for !hasUpstream and len(behind) == 0, which are the failed-fetch case.
      Add a row to the Pull tests in pull_integration_test.go for a weft whose upstream ref cannot be read, asserting Pull returns that error and not a nil error with WeftPulled false.
  - id: F5
    severity: MEDIUM
    class: decision
    location: PATTERN.md:11
    summary: Thirteen overview entries outside the two declared ones differ from main, and pattern/PATTERN-recipe-setting-lookup.md is new, none of it among the declared PATTERN changes
    fix: Settle with ly:orch which holds, then apply it. Either the cuts that keep the overview under the headroom line are declared as PATTERN changes of this task (the entries durable-vs-ephemeral-state, hub-suffix, push-both-sides, recipe-setting-lookup, transient-stop, driver-choice-single-site, batcher-registry, creel-store, stencil-ownership, model-choice-mapping, pane-binary-resolution, edit-directive and testkit, plus the new background file), or restore main's overview text on every one of them but the fewest whose cut keeps the Headroom finding of internal/pattern quiet with the two declared entries added.
---

### [BLOCKING] Publish verify failure: TestWatchdogDaemon/DownThenUpDoesNotKillDaemon

**Location:** internal/reedcli/watchdog_integration_test.go:372
**Issue:** The log `.lyx/verify/verify-7.log` shows `runWatchdogLoop exited after a down immediately followed by an up (err=<nil>)`, in a `reedcli` package run that took 110s against under 5s in the runs of verify-4 and verify-6.
Neither the branch's own range nor the merge-in diff `ac1014b3^1..ac1014b3` touches `internal/reedcli` or `internal/reedengine`, and the loop's exit is the idle-cycle rule: `compressedWatchdogTiming` sets a 200ms cycle with `IdleCycles` of 3, so the daemon returns nil once discovery sees no session for 600ms.
The test runs `eng2.Down()` then `eng2.Up()`, and on a loaded machine that gap passes 600ms, so the daemon exits as designed.
The cause is a wall-clock window too narrow for the test's own gap, not a defect the branch introduced; it still blocks Publish, and the failure is raised as the rubric requires.

### [BLOCKING] Set-aside commits are tagged after the branch moved

**Location:** internal/fabricengine/replay.go:436, internal/fabricengine/replay.go:787, internal/fabricengine/destroy.go:1794
**Issue:** The focus question is whether the branch and the set-aside commits end in a state recoverable without the reflog when tagging fails.
They do not in two cases.
First, `tagSetAsideOrRestore` restores the starting tip with `ResetKeep(startTip)` after a failed tagging; if that restore fails too, the branch sits at upstream plus the replayed commits and the set-aside commits are dangling, named only in the error text.
Second, `replayBolt` returns `outcome, err` from every failure in its by-entry loop and after `cherryPickOnto` (`replayBoardCommit`, `WorktreeChangedFiles`, `CurrentSHA`) with the branch already reset onto upstream, the commits set aside by classification untagged, and no restore: only the reflog holds them.
Card 4 orders it the other way, tagging each set-aside commit and then replaying the rest, which would leave nothing to restore for the classified commits.
No test pins the tagging-failure path: `tagSetAsideOrRestore` and its restore are reached by no test in `internal/fabricengine`, a behavior the diff adds with no coverage.

### [MEDIUM] Every ResetKeep error becomes ErrReplayDirty

**Location:** internal/fabricengine/destroy.go:1828
**Issue:** The doc comment says a reset git refuses over a local change returns `ErrReplayDirty`, and `ResetKeep` documents that refusal as one particular `*gitexec.GitError`; the code wraps any `ResetKeep` error, an index lock or a damaged repository included.
Callers read `ErrReplayDirty` as a stop with uncommitted paths to commit (`pull.go:389`, `originreconcilemoves.go:234`, `bolt.go:256`, `pushclassified.go:511`), so `Pull` builds a weft dirty stop that names no path and a way forward that cannot help, while the real failure is buried in the cause.
`TestRecordsReplay` has no row for a reset failure that is not a dirty refusal.

### [MEDIUM] settleUnmovedWeft swallows a failed local read

**Location:** internal/fabricengine/pull.go:328
**Issue:** `err` of `upstreamSHAAt`, `CurrentSHA` and both `CommitsNotIn` calls joins the `!hasUpstream` and `len(behind) == 0` cases in one warn-and-continue branch that returns `nil, nil`, leaving `WeftPulled` false.
The first two are the failed-fetch case the comment names; a failed local git read is an infrastructure error, and card 6 says `Pull` never returns success with `weft_pulled: false`.

### [MEDIUM] Undeclared PATTERN overview changes

**Location:** PATTERN.md:11
**Issue:** The focus question is whether every entry outside the two declared reads as on main.
It does not: against main, the overview lines of thirteen other entries are shortened and `pattern/PATTERN-recipe-setting-lookup.md` is added, while the declared list names only `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible`.
The two declared entries carry their declared text exactly.
The overview ends at 15741 bytes against main's 15743, so the cuts pay for the two additions under the `Headroom` check in `internal/pattern`, and the cut detail survives in the background files I read for `durable-vs-ephemeral-state`, `hub-suffix`, `transient-stop`, `driver-choice-single-site`, `stencil-ownership`, `edit-directive`, `testkit`, `push-both-sides` and `pane-binary-resolution`.
Whether such cuts need declaring, or fewer entries should be cut, is the author's choice, which the declared-changes rule leaves open.

## Rejected

- `pushclassified.go` and `coalesce.go`: every push failure that reaches `pushClassified`'s push or its retry ends in `RecordPushFailure`, which posts one notice per episode through the installed notifier, and `pushSideDetached` returns nil only after that record; the `other` class, the detached child, the auth, transport, gone and rewound paths each have a `TestPushFailureClasses` row, so no defect.
  A `checkOrigin` error from a failed local read of the upstream or the entry judgment returns unrecorded, but no push was refused there, so it stays outside the declared text.
- `reconcileVerdict` and `runReconciler`: the closure's `reportOfRun` narrows the report to the run's own entry before the verdict reads it, and `TestReportOfRunKeepsOnlyTheRunsOwnEntry` covers a stopped parent entry beside an active own entry, so not a defect.
