---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/00-overview.md:36
    summary: The D11 audit lists only log lines written lexically inside a loop body and misses Warn lines in callees and callbacks a loop reaches on every iteration, including the reed watch tick D3 targets, the reed name-repair loop, loomcli's loop wake hook and landingshed's parent-lock wait
    fix: "Re-run the D11 sweep by following each loop's per-iteration call graph, including hooks and callbacks handed to a polling loop in another package, and add a row with a verdict for every line found. At minimum add these rows to the table in 00-overview.md: (1) reedengine Engine.liveBoxLocked (windowsize.go, failed to query live window size / malformed live window size answer), Engine.readBorderTitleRowLocked (failed to read back pane-border-status) and Engine.keepZoomLocked (failed to unzoom / failed to list the strands' window after a step / failed to re-zoom), each reached on every watch tick through reapplyLayout and applyLayoutLockedOpts: guard; Engine.runResizePinsLocked: keep, since it runs only after a successful select-layout on a changed box. (2) reedengine Engine.watchNames (name repair pass failed) and Engine.repairNames (could not repair pane title): guard; repaired pane title: keep. (3) loomcli idleReleaseOn, the loop's OnWake hook run on every wake of shedverbs' child watch (looplock.go: could not list the parent lock records via ownedLockBranches, could not read the agents' states, could not release the idle parent lock): guard; released the parent lock after every agent waited past the idle bound: demote Warn to Info, as card 8 does for battenshed's released the child's parent lock. (4) landingshed parentGuard.pauseRequested, called by parentlock.Store.Acquire's Waiter.Paused on every poll of the unbounded parent-lock wait (could not read the pause flag during the parent lock wait): guard. Carry the guarded and demoted sites in cards: the reedengine sites in a new card inserted after card 3 with lyx loom insert-card (members internal/reedengine#Engine.liveBoxLocked, #Engine.readBorderTitleRowLocked, #Engine.keepZoomLocked, #Engine.watchNames, #Engine.repairNames, each with a test row reaching its Warn, Debug and Info-recovery branches), and the loomcli and landingshed sites in one new card each, inserted after card 12 with lyx loom insert-card (members internal/loomcli#idleReleaseOn and #ownedLockBranches; internal/landingshed#parentGuard.pauseRequested and the parentGuard field or local that holds its episode), each naming the test that reaches its guard and recovery branches. Reword the table's lead paragraph so it says the sweep followed callees and callbacks reached per iteration."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-shuttle-read-guards.md:51
    summary: Card 10 prescribes Recover branches for the pending-send and startup-dismiss guards but names no test that reaches either recovery line
    fix: "In card 10's Tests paragraph: (a) make TestGatePending_FailedSendWarnsAndStaysPending reach the recovery, using newPendingFixture's per-tick step hooks to clear f.reed.SendTextErr after the second failed pending send, so a later evaluation's pending send succeeds; assert one level=WARN and at least one level=DEBUG pending send failed line and one level=INFO gate: pending send delivered again line. (b) Add a row to TestRun_Wait_StartupWindow, trust dismissal fails on the first probes then sends, whose script shows the trust prompt on every probe and whose fake's SendKey fails on the first two calls and then succeeds (a SendKeyErrQueue on fakeReed consumed FIFO, last entry sticking, added to the card's fakeReed and fakeReed member edits as needed), asserting one level=WARN dismiss trust prompt line and one level=INFO recovery line. Also state in the Intent which outcome counts as a dismissal that sends for Recover: playInputsSent returning no error, or sent greater than 0."
---

### [BLOCKING] The D11 audit misses per-iteration Warn lines reached through callees and callbacks

**Location:** `_lyx/plan/00-overview.md:36`

**Issue:**
D11 requires the plan to list every Info-or-above line, Warn included, that sits in a loop or polled path, each with a verdict.
The table's lead paragraph says the sweep covered "every loop and polled path in the module", but every row names a line written inside a loop body or a function the plan already touched; lines reached on every iteration through a callee or a callback handed to another package's loop are absent.
Each instance below was read in the code:

- **The reed watch tick D3 targets.**
  `Engine.reapplyLayout` (`internal/reedengine/reapply.go`) runs on every watch tick inside `keepZoomLocked`, and calls `applyLayoutLockedOpts` (`apply.go:214`), which calls `liveBoxLocked` before its box-equality skip.
  `liveBoxLocked` (`windowsize.go:58`, `:63`) logs `reed: failed to query live window size, falling back to configured box` at Warn on every tick while `display-message` fails, and a non-live box never takes the skip, so the tick goes on every time.
  In a failure episode like the one D1 describes, the tick also reaches `readBorderTitleRowLocked` (`windowsize.go:262`, `failed to read back pane-border-status`) each time.
  `keepZoomLocked` (`zoom.go:54`, `:61`, `:69`) unzooms and re-zooms a zoomed window on every tick, and its three Warns repeat while those tmux calls fail.
  None of these lines is in the table, which lists only `handleWatchOutcome`'s own lines for the watch loop.
- **The reed name-repair loop.**
  `Engine.watchNames` (`namerepair.go:102`) logs `reed: name repair pass failed` at Warn on every wake while `repairNames` fails (has-session, state load or list-panes), and `repairNames` (`namerepair.go:131`) logs `reed: could not repair pane title` on every wake while the title still drifts and `select-pane` fails.
  The table covers `repairSessionName` in this loop and omits both of these lines.
- **loomcli's loop wake hook.**
  `arm.go:330` sets `spec.Loop.OnWake` to `loopIdleRelease`, which `shedverbs`' child watch runs on every wake (`loopwatchdog.go:86`).
  `idleReleaseOn` (`looplock.go`) logs `could not list the parent lock records` (through `ownedLockBranches`), `could not read the agents' states for the parent lock's idle rule` and `could not release the idle parent lock` at Warn on every wake while they fail.
  It also logs `released the parent lock after every agent waited past the idle bound` at Warn, which is a state change and no failure, the same shape card 8 demotes for battenshed's `released the child's parent lock`.
  `internal/loomcli` appears nowhere in the table.
- **landingshed's parent-lock wait.**
  `parentGuard` passes `pauseRequested` as `parentlock.Waiter.Paused` (`parentguard.go:72`), and `parentlock.Store.Acquire` calls it on every poll of a wait with no time limit (`acquire.go`).
  `pauseRequested` (`parentguard.go:169`) logs `could not read the pause flag during the parent lock wait` at Warn on each poll while the read fails.
  The table's only `landingshed` row is `PRAlign.wait`.

The acceptance criterion "No Info-or-above line the plan listed as in a loop or polled path logs more than once while its state is unchanged" is only as strong as the list, so these lines escape it.
This is filed once, against the method, per the rubric's rule for a recurring enumeration gap.
It recurs: round 1's F1 found the same gap for loops the table had not named, and its fix added rows for those loops without changing the method that misses per-iteration callees.

### [MEDIUM] Card 10's pending-send and startup-dismiss recoveries are reached by no named test

**Location:** `_lyx/plan/10-shuttle-read-guards.md:51`

**Issue:**
Card 10 prescribes a `Recover` branch for each guarded site.
The operator-activity, expiry-probe and startup-capture guards each get a test row that fails and then succeeds, asserting the WARN and the INFO recovery line.
Two do not:

- **Pending send.** `handleGatedBoundary`'s `pending send failed` guard calls `Recover` with `shuttle: gate: pending send delivered again` when a pending send succeeds.
  The only named test, `TestGatePending_FailedSendWarnsAndStaysPending`, sets `f.reed.SendTextErr` once (`gate_pending_test.go:124`), which fails every send, and the card asserts only the WARN and DEBUG lines, so the recovery branch is never reached.
- **Startup dismiss.** The `dismiss trust prompt` guard calls `Recover` on "a dismissal that sends", but the only new row, "trust dismissal fails every probe", sets the sticky `SendKeyErr`, so the recovery is never reached.
  "A dismissal that sends" is also ambiguous against `checkLivenessTick` (`wait.go:1196-1207`), where `playInputsSent` can return `sent > 0` together with an error.

The rubric counts a prescribed guard branch with no named test that reaches it as a finding.
Both branches came in with round 1's F1 fix, after round 1's F2 had settled that every guard's recovery needs a reaching test, so this is a gap left by that fix, not a reopened dispute.
