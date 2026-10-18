# Card 11 — orch-start-withdraws

**Create:**
- `internal/orchengine#Watcher.retryFreshLaunch`
- `internal/orchcli#orchCLI.withdrawFirst`
- `internal/orchcli#orchCLI.withdrawSecond`
- `internal/orchcli#orchStrandFullName`

**Edit:**
- `internal/orchengine#State`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#ResetForFreshLaunch`
- `internal/orchengine#TestResetForFreshLaunch`
- `internal/orchengine#TestWatcher_CreelFailuresHoldTheCycle`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#orchCLI.adoptStrand` -> `func (c *orchCLI) adoptStrand(st orchengine.State, guid string, creel orchengine.Creel, name string) error`
- `internal/orchcli#startHarness`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#TestStart_NoStrandLaunchesAndSpawnsWatcher`
- `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`
- `internal/orchcli#TestStart_Refusals`
- `internal/orchcli#TestStart_LiveStrand`
- `internal/orchengine/doc.go`
- `internal/orchcli/doc.go`

**Intent:**
`lyx orch start` withdraws the old cycle's queued steps before and after a fresh launch (decision record D5, "Fresh launch").

Wherever `start` calls `ResetForFreshLaunch` (a relaunch, `--continue`, `--adopt`, and an adopted strand in `adoptStrand`), it signs as `ORCHWATCH` through `c.newCreel` and runs two rounds on the orch strand's name, the listed strand's name or else the fixed `<shortname>:orch` (`orchStrandFullName`, with the hub's shortname from `fabricengine.ReadShortname`).
A hub with no shortname record fails `start` before any round as a fail-loud wiring guard naming the record, since `lyx creel` cannot serve such a hub either.

- First, through `withdrawFirst`, before it starts or binds the session and, on a relaunch, before it removes the old strand: a withdraw, and a release only when an orch strand is listed, since creel refuses a release to an unlisted name the caller holds no hold on.
  When the withdraw fails, a refusal or an unreachable daemon alike, `start` starts and binds no session, removes nothing and fails with creel's error and the way forward: repair what the error names, such as a worktree whose reed state the sender-resolution refusal names as unreadable, or a daemon that would not start, whose log lines the error quotes, then run `lyx orch start` again.
  Since those refusals carry no code (card 4), the way forward `start` appends is one fixed text that also names two causes rerunning alone does not repair: when creel refused the `ORCHWATCH` program claim from a non-orch strand's pane, run `lyx orch start` from the orch's pane or from a terminal outside every strand pane; when the error names a strand whose pane process is unknown, restart or remove that strand, then run `lyx orch start` again.
  Input reaching it: `start` with no live strand and a creel seam whose withdraw fails.
  Cost: while that cause stands no orch can be started, continued or adopted.
  A failed first release does not fail `start`: `withdrawFirst` answers its error text, which `start` records in `State.FreshLaunchFailure` when it saves the reset state, as for the second round (decision record D5: "When the second withdraw or either release fails, `lyx orch start` records that in the orch state").
- Second, through `withdrawSecond`, after it saves the reset state: a withdraw and a release, catching a step an old watcher still running queued in between.
  A failure of either is recorded in `State.FreshLaunchFailure`, a new field holding the error text, and does not fail `start`; the state file gains the `fresh_launch_failure` key.

`withdraw` answers a removed resume line, which `start` hands to `orchengine.ClearResumeNoticed` with the pause record path and the orch GUID the state recorded before the reset (`st.Strand`), the GUID a pause entry names, so a continued or adopted session, whose GUID the entry still names, gets the line posted again by the watcher, and a relaunched one, whose GUID differs, gets none.
A continue or adopt after `lyx orch stop`, which leaves no orch strand listed, passes the first withdraw with an empty result, sends no first release, starts the session and records no failure.

Watcher (`Watcher.tick` through `retryFreshLaunch`): while `State.FreshLaunchFailure` is set, every tick first retries the withdraw and the release on the orch strand, signing as `ORCHWATCH`, before it posts or queues anything, the bind color included; until both succeed it posts and queues nothing, starts no cycle, renews no hold while idle and shows the failure in `State.Stuck`, which the retry rewrites on each failing tick without setting `State.StuckByCreel`, and which no other successful creel call clears while the field is set (card 10's clearing rules cover only `creelFailed`'s reasons, `stepOutcome`'s stall reasons and a restarted watcher's unowned reason, and this card adds `State.FreshLaunchFailure` as an owner to that restart rule, so a restarted watcher keeps the failure text while the field is set); the tick in which both succeed clears `State.FreshLaunchFailure` and `State.Stuck` in the same state save (and hands a withdrawn resume line to `ClearResumeNoticed`).
Bound: while the second withdraw has failed and no retry has succeeded, an old `ORCHWATCH` inject queued between the two withdraws, a `/clear` or `/compact` included, is typed on the relaunched session's first idle, unpaused probe within its ten-minute TTL, and old `ORCHWATCH` mail posted in that window is reminded until a retry succeeds; while a release has failed, `ORCHWATCH`'s hold stays on the relaunched orch until a retry succeeds or the hold lapses.
This exposure is accepted, and once both withdraws succeed at most the one step of an old watcher's tick that read the state before the reset reaches the new session.

Tests:
`startHarness` holds a `fakeCreel` (card 9) that `newStartHarness` wires through `newCreel`, recording calls in the harness order beside adds, removes and starts; `newStartHarness` writes the hub's shortname record (`fabricengine.ShortnameFileName` in the location's board directory), so `orchStrandFullName` has a shortname.
`TestStart_NoStrandLaunchesAndSpawnsWatcher` gains rows: a relaunch withdraws and releases before removing the old strand and starting, then withdraws and releases again after saving, in that order; a continue or adopt after stop, with no strand listed and nothing queued, sends no first release, addresses `<shortname>:orch`, starts the session and records no failure; a first release, a second withdraw or a second release that fails, refused or unreachable, records `fresh_launch_failure` and `start` succeeds; a withdrawn resume line clears the noticed mark of the entry owning the old recorded GUID and of no entry owning another GUID.
Every row of `TestStart_EnsuresTheRunsStrandBeforeTheOrch` ends in a fresh launch with no orch strand listed, so each row's `wantCalls` gains the fake creel's calls at the positions `start` makes them: the first withdraw, with no first release, and the second withdraw and release after the start; its runs-strand ordering assertions stay.
`TestStart_Refusals` gains rows: a first withdraw refused, and one that cannot reach creel, start and bind no session, remove nothing and report creel's error with the way forward; a first withdraw refused for a program claim from a non-orch pane reports creel's refusal with the way forward, which names the orch's pane or a terminal outside every strand pane.
`TestStart_LiveStrand`'s adopted-strand row asserts the two rounds run around `adoptStrand`'s reset, and a row whose first withdraw fails there binds nothing and reports creel's error.
`TestWatcher_CreelFailuresHoldTheCycle` (card 10) gains rows: with the field set the watcher retries both on every tick, posts and queues nothing and renews no hold until a retry succeeds, shows the failure in `State.Stuck` with `State.StuckByCreel` unset while it fails, then clears the field and leaves `State.Stuck` empty, and no old `/clear` the fake held queued reaches the session once the retry's withdraw removed it; a retry whose withdraw removes a resume line clears that entry's noticed mark.
`ResetForFreshLaunch` clears `FreshLaunchFailure`, so a relaunch whose second round succeeds leaves no earlier failure behind; `TestResetForFreshLaunch` asserts it.
`internal/orchengine/doc.go` and `internal/orchcli/doc.go` describe the two rounds, the recorded failure and its retry.

**ImpactSummary:** Adds creel withdraw rounds to orch start's relaunch and adopt paths and one state field the watcher retries; the start verb's envelope and every non-fresh path are unchanged.
