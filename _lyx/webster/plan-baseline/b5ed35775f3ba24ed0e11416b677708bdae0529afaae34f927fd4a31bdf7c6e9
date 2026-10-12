# Card 11 — landing-parent-guard

**Create:**
- `internal/landingshed#ParentLockDeps`
- `internal/landingshed#parentGuard`
- `internal/landingshed#newParentGuard`
- `internal/landingshed#parentGuard.ensure`
- `internal/landingshed#parentGuard.release`
- `internal/landingshed#parentGuard.moved`
- `internal/landingshed/parentguard_test.go`
- `internal/loomcli#parentLockDeps`
- `internal/loomcli#loopLockOwner`
- `internal/loomcli#TestLoopLockOwner`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#landingshedAllowedImports`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, geom websterengine.Geometry, taskBranch, originURL, parentBranch string, pushSkipped bool, pushBranch func() error, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error), verify verifySource, lock landingshed.ParentLockDeps) landingshed.Deps`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#markDoneFixtureDir`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `pattern/PATTERN-wait-mark-display-only.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/parentlock#Store.Acquire`
- `internal/parentlock#Store.Release`
- `internal/parentlock#NewWait`
- `internal/parentlock#WriteWait`
- `internal/parentlock#RemoveWait`
- `internal/fabricengine#Fabric.ReadParentTips`
- `internal/fabricengine#Fabric.HoldsWork`
- `internal/fabricengine#Fabric.ChangedPaths`
- `internal/landingshed#ReadMergeRecord`
- `internal/landingshed#MergeRecordPath`
- `internal/landingshed#MergeReviewedPath`

**Intent:**
Wire the parent lock and the code-side parent reads into landing's told surface, and add the one guard every lock-taking row will share (D5, D9, D12), before any row calls it.

`Deps` gains:
- `Lock ParentLockDeps`: `Store *parentlock.Store`, `Owner func() (parentlock.Owner, error)` (the identity the lock is taken in), `WaitPath string` (the run's wait record) and `PauseRequested func() (bool, error)`.
- `ReadParentTips func(fetch bool) (fabricengine.ParentTips, error)`, `HoldsWork func(base, source string) (bool, error)`, `ChangedPaths func(from, to string) ([]string, error)` and `MergeBase func(ref string) (string, error)`, each over the task pair.
- `MergeRecordPath` and `MergeReviewedPath`, the told files of card 10.

`landingshedAllowedImports` admits `internal/parentlock`.

`parentGuard`, built by `newParentGuard` from `Deps`, holds the lock seams, the parent branch, the push skip, the tip reads, the merge-record path and the wait-mark callback.
- `ensure(ctx, producer)` takes the lock through `Store.Acquire` with `Owner()`'s identity.
  Its `Waiter.OnWait` writes `NewWait(branch, holder slug)` to `WaitPath` and sets the driver strand's wait mark through `Deps.VerifyWaitMark` with the label `parent lock <branch>`.
  On every way out it removes the wait record and clears the mark; the mark stays display only, and a mark error is logged.
  `Waiter.Paused` reads `PauseRequested`, and a read error counts as not paused with one Warn.
  It returns "" once held.
  A pause seen during the wait returns `paused while waiting for the parent lock on "<branch>" held by <slug>; way forward: run "lyx loom goto --to <producer>", then "lyx loom resume"`.
  An `ErrUnreadable` returns `the parent lock record of "<branch>" is unreadable: <cause>; way forward: run "lyx batten lock release <branch>" from the prime, then "lyx loom resume" in the task worktree`.
  A cancellation and every other error are returned as an error.
  A caller returns either reason as a Stuck with `Halt` set, so a row with an `on_stuck` route never bounces on a lock wait.
- `release(producer)` compare-and-releases the run's own record with `Owner()`'s identity, and logs a failure without changing any verdict.
- `moved(producer)` is the moved-parent check (D9).
  It reads the merge record; a missing or unreadable one counts as moved, so the row reroutes to Merge-In, which writes one.
  It reads `ReadParentTips(fetch)` with `fetch` false under the told push skip.
  The parent has moved when the local tip differs from `merged_tip` and does not hold the task's work, or when an upstream tip exists, is read, differs from `merged_tip` and does not hold the task's work.
  A parent with no upstream, or under the push skip, is judged on the local tip alone.
  "Holds the task's work" is `HoldsWork(tip, task head)`.
  A local parent ahead of or diverged from its upstream therefore reads as moved, and Merge-In's Stuck applies after the reroute.
  It returns the reroute reason `parent branch "<branch>" moved since Merge-In merged <merged_tip> (local <local>, upstream <upstream>); rerouting to Merge-In`, or "" when it has not moved.

In loomcli, `parentLockDeps` builds the lock seams:
- the store at `parentlock.Dir(fabricengine.BoardDir(l.HubPath))`, the hub-level ephemeral tree reached by path and never junctioned into a pair (`PATTERN-hub-containment`);
- `Owner` from `loopLockOwner`;
- `WaitPath` from `parentlock.WaitRecordPath(loomengine.LoomScratchDir(l))`;
- `PauseRequested` reading the run status file's `pause_requested` under its status lock.
`loopLockOwner` names the run (`Slug` from the worktree name, `Pair` the worktree path, `RunID` the resolved run id).
It takes the process identity from the loop's pid record (`shedverbs.ReadLoopPIDRecord` at `shedrun.LoopPIDFile`) when that record names a live loop, so the lock follows the detached `--until-stop` loop (D5).
Otherwise it takes the calling step process's own identity (`proc.SelfRecord`), so a step run outside a loop holds the lock only while it lives; the teardown mark counts as no loop.
`landingDeps` takes the built `ParentLockDeps` and fills the tip, holds-work, changed-paths and merge-base reads over `fabricengine.Open(l)` and the two record paths under `loomengine.LoomScratchDir(l)`.
`loomCLI.loomPreRun` and `loomCLI.buildLoomShed` pass `parentLockDeps(c.location, c.runID, c.shedPaths.StatusPath, c.shedPaths.StatusLockPath)`.
Every test call site passes a lock value.
`envkit.LandingDeps` fills the new seams with fakes.
The loomcli and battencli `intentionallyNil` lists name the new landing seams `wire` leaves nil, for the reason their existing landing entries give.

`parentguard_test.go` drives `ensure`, `release` and `moved` against a real `parentlock.Store` in a temporary directory and fake reads:
- `ensure` takes a free lock; it waits behind another owner with the wait record and the mark set, and is granted after the release with both cleared;
- a pause during the wait returns the paused reason, and an undecodable record returns the release-verb reason;
- `release` leaves another process's record alone;
- `moved` is one table: equal tips, a local commit, an upstream push, no upstream, the push skip ignoring a moved upstream, a moved tip that holds the task's work, and a missing merge record.
`TestLoopLockOwner` asserts the loop's identity for a live loop record, and the step's own for a dead loop and the teardown mark.
`PATTERN-wait-mark-display-only.md` names the landing wait-mark callback's lock-wait label beside the verify.

**ImpactSummary:** `landingshed.Deps` gains told seams and a guard no producer calls yet; `landingDeps` is re-signed with its two production callers and its five test callers; the seam-gap lists and the envkit builder follow.

**Commit:** 11: landing: parent lock and parent tip seams, and the shared guard
