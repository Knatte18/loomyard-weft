# Decision record — a pause holds every prompt, verify command and new batch in its scope

## Goal

A pause holds everything lyx would start in its scope, so the operator never kills processes or re-pauses by hand.
It closes the four gaps of issue #762 and the 2026-10-21 webster note: a gate re-prompt reaching a covered session, a verify command running on under a pause, `lyx creel interrupt` refused under a pause, and webster Master opening a batch under a pause.

## Scope

In: D1–D8 below.

Out:

- Processes other than verify commands: driver chains, parked pushes, background shells, Monitors and webster forks under a pause belong to `run-process-ownership`, which depends on this task.
- A pause hold for the card `**Verify:**` rerun (`internal/cardverify`) and batten's `deploy` and `Command` rows, which run through `verifyrun` directly and are no verify site; they still get D4's SIGTERM-first cancel on their timeouts.
- Any change to what `lyx tender pause` itself interrupts, or to its grace.
- Any change to `proc.ConfigureGroupKill`'s cancel: `gitexec`'s cancelled git commands and the `lyxbin` test kit keep their immediate group SIGKILL.

## Decisions

### D1 — Shuttle's typing chokepoint refuses a covered session

`sendVerified`, through which every shuttle line but `Runner.SendConditioned`'s reaches a pane, reads the pause coverage of the target session's own slug after `awaitIdleSession` returns and immediately before `typeVerified`, and refuses a covered session with a new `ErrPaused` that wraps `pauserecord.CoveredError`'s text and carries the slug the read judged.
A read before the idle wait is allowed as an early exit, never as the only check, since a tender pause's grace is the window in which a send waits for idle.
The read repeats before each replay of `typeVerified`'s choreography.
`SendConditioned`'s only caller, tender revive, reads coverage itself under the record lock and sends nothing to a covered session.
The slug comes from the target strand's name as reed reports it, parsed through `agentname`; a prime strand has the empty slug, which only a hub entry covers.
A name that does not parse, and a reed lookup of the target's name that fails, fall back to the runner's own geometry slug, the lookup failure with one Warn per change of error.
The record path comes from the runner's pause geometry, and every production runner that types a line, but the orch runner, is told one: the tender and creel runners, which are not told one today, get the hub's through `SetPause`.
The orch runner stays told no geometry, so its run-start lines, the only lines it types, read uncovered, and `lyx orch start` keeps starting under a hub pause as today; the orch watcher already settles an orch launched after the entry without an Esc (`settledWithoutInterrupt`).
A runner told no geometry reads uncovered, which is what tests and the orch runner use.
An unreadable record reads uncovered with one Warn per change of error, as `observePause` does, so a corrupt file never freezes sends.
Rationale: one check at the one typing site, keyed on the target rather than the caller, holds for every caller but the two named above, including a hub-level runner sending into another pair.
Bound: it only refuses; it types nothing, records nothing and never waits.
Both fallbacks can let a line into a session that only a slug entry covers when the runner's slug is empty, as it is for every runner told the hub geometry.
`lyx orch start`'s own lines reach the orch it launched under a hub entry, which that entry's watcher leaves running.
Keys stay outside it: `Runner.Interrupt`, `InterruptRestoring`, `Inject` and the pause's own Esc are key deliveries under `PATTERN-pane-text-injection`.

### D2 — Each caller of the chokepoint holds or reports `ErrPaused`

- Gate re-prompt and pending send (`handleGatedBoundary`): held like `ErrSessionBusy`: no attempt spent, the memo cleared, the writer kept at the boundary, and the boundary re-evaluated once the run reads uncovered.
- Run start's prompt line and its skill-load line: on `ErrPaused`, wait through `pauserecord.WaitUncovered` on the slug D1's refusal read, which `ErrPaused` carries, then send again, so a pause landing after the start's first wait holds the start instead of failing it, and the wait and the refusal judge the same slug; the orch runner, told no geometry, never gets `ErrPaused`.
- The deferred task prompt (`Run.DeliverPendingPrompt`), which burler's `ResumeFix` reaches after the run start's own wait: on `ErrPaused`, `DeliverPendingPrompt` itself waits through `pauserecord.WaitUncovered` on the slug `ErrPaused` carries, then sends again, so `ResumeFix` receives no refusal.
- Seat advisor notices (`seatengine` notice sender): retried instead of dropped, with the retry interval doubling up to five seconds while the refusal is `ErrPaused`, per `PATTERN-daemon-wakeups`.
- Creel delivery (`creelcli` `mapDeliveryError`): `ErrPaused` maps to a held outcome, so the inject stays queued and is delivered after the resume, as creel's own `Paused` check already holds it.
- `lyx tender resume`'s resume line: a send refused because another live entry covers the session claims the session into that entry through D6, so the later resume gives it the line.
- `lyx loom resume`'s driver resume line and `lyx shuttle send`: returned as a refusal with `CoveredError`'s way forward, which names `lyx resume`; both are operator verbs.
Rationale: a refused line either reaches the session after the resume or is refused visibly; a dropped line strands the session idle.

### D3 — Shuttle's gate loop holds its boundary while covered

`run.covered` judges the slug D1 judges for the run's own session, falling back as D1 does, never the runner's slug alone.
While it reads true, Wait's tick neither calls `handleGatedBoundary` nor sends a pending send.
A gated Done arrival under cover marks the boundary as reached and waiting on the pause; the first tick that reads uncovered evaluates it as if the arrival had come then.
The off-boundary `Expired` probes keep running as today, since they send nothing, so an entry's own wait bound can still end a run under cover.
An `Expired` probe that fires under cover finalizes the run, and that finalize's gate evaluation then waits in D4's held Verify until the resume.
Rationale: evaluating a gate under a pause runs verify commands and webster's gate side effects for a verdict whose re-prompt cannot go out; D1 alone would still run them.
Bound: the hold only delays an evaluation; no verdict or attempt count changes because of it.

### D4 — `verifytree.Verify` holds its command under a pause

`verifytree.Site` gains an optional pause hold, told by the caller as two functions: one reading whether a pause covering the scope has passed its grace end (`Entry.GraceEnd`), and one waiting until no pause covers the scope.
Before starting its command, after the dirty and skip checks, Verify waits through the second function while a pause covers the scope, writing the `paused` marker during that wait, so a Verify entered under a covering pause starts no command until the resume.
While the command runs, Verify reads the first once a second; once it answers true, Verify cancels the command.
`verifyrun.Run`'s cancel sends SIGTERM to the command's process group, then, after the wait delay, SIGKILL to the same group whether or not the shell has already exited, through a timer of the new cancel's own and not through `exec.Cmd.WaitDelay`, which kills the shell alone and only while `Wait` has not returned.
This SIGTERM-first cancel, with its own SIGKILL timer and group-empty poll, is a new cancel used only by `verifyrun`, living in `internal/proc`'s platform files beside the group kill; `proc.ConfigureGroupKill` keeps its immediate group SIGKILL for `gitexec` and the `lyxbin` test kit.
After a cancel, `Run` returns only once the group is empty, polled through `kill(-pgid, 0)`, or once that SIGKILL is sent, so no member outlives the return and no late SIGKILL can reach a group that has reused the pid.
The change applies to every `verifyrun.Run` caller: batten's `deploy` and `Command` rows and `cardverify`'s card `**Verify:**` rerun also get SIGTERM first on their timeouts, and act on the timeout only once that return says the group is gone.
The SIGTERM reaches `lyx gate test`, whose terminating-signal handler (`gatecli` `terminatingSignalSet`) kills its tied `go test` tree and releases its slot; a bare SIGKILL skips that handler and leaves the test binaries running, since `proc.StartTied` puts them in their own group.
Verify then sets its running marker to a new `paused` state, which loom status shows as paused, waits through the second function, and starts over from its dirty-tree check, so a tree changed during the pause is verified afresh.
The activity readers, loomcli's `agentActivityFor` and battencli's `readAgentActivity`, read a `paused` marker as no activity, since a held Verify does no work: the loop watchdog already skips its idle timeout while the step is covered, and batten's quiet notice treats it as it treats a held run.
The stopped run's log stays and is pruned like any other.
A nil hold runs as today, which is what tests use.
A caller's own context cancel during the hold returns the cancel error, as today.
Rationale: the grace matches the pause's own: a short pause lets a nearly done verify finish, and a long one stops it.
Bound: Verify stops only the process group of the command it started, and only after the covering entry's grace end; a run is never recorded as passed or failed because of a stop.

### D5 — Every production verify site tells the hold

The loomshed verify gate (Darn-Write, Darn-Burler-Fix, Webster-Burler-Fix), the webster verify gate, landingshed's Merge-In, Finalize and Publish, and `lyx webster verify` pass their scope's hold.
Each builds it over `pauserecord.ReadCoverage` and `pauserecord.WaitUncovered` for its worktree's slug, from the pause geometry its wiring already resolves.
A site whose run holds a parent lock reads hub entries only, the exemption `WaitUncovered` already gives such a caller.
Rationale: `PATTERN-verified-tree` makes `Verify` the one verify path, so wiring every site makes every verify command holdable.

### D6 — `pauserecord` gains one claim helper

A new function records one session in the live entry covering a slug, under the record lock: it records nothing for an uncovered slug, and nothing when a live entry already owns the session.
It returns whether a pause covers the slug and the covering entry.
An orch session is recorded in the shape the orch watcher writes (`GUID` the orch strand's reed GUID, the value `orchengine` `State.Strand` holds, `Orch` true) and is covered only by a hub entry; the watcher then finds the orch already owned and sends no Esc of its own, and still posts its resume line.
Rationale: D2, D7 and D8 each need "record this session so the resume gives it its line" against whichever entry covers it; tender's `claimSession` and the orch watcher claim a known entry and stay as they are.

### D7 — `lyx creel interrupt` interrupts a covered strand

Under a covering pause, creel claims the strand into the covering entry through D6, then sends the guarded interrupt as for an uncovered strand.
The claim is a new `Delivery` seam method used only by the interrupt; `Paused` stays the read-only check the delivery worker holds mail on.
Right before the key, under the receiver's pane lock, creel re-reads that the claimed entry is still live, and sends no key when it is gone.
`lyx tender resume` never takes creel's pane lock, so a resume can still land between the re-read and the key; if its line then fails busy, the strand is interrupted with no resume line.
That window is an accepted residual of the check-then-key order, the same one tender's pause has.
The claim comes before the key, so an interrupt that dies after the claim leaves the strand recorded, as tender's pause orders it.
A strand that reads no running turn stays recorded, as tender's `no_running_turn` session does.
The privilege rule and another sender's hold still refuse as today, and an unreadable record still refuses the interrupt with its error.
Rationale: the pause wants work stopped, and recording the strand is what lets `lyx tender resume` give it its line, which the refusal existed to protect.
Bound: the change lets an interrupt through only for a strand a live entry covers at the key, and records it in that entry; it adds no sender to the privilege rule.

### D8 — webster's batch verbs refuse under a pause

`lyx webster begin-batch` and `lyx webster recover-batch` read the pair's pause coverage before any state mutation and refuse while covered.
Before refusing, the verb claims Master's session into the covering entry through D6, with the GUID reed's `ResolveStrandGUID` returns for the verb's own `LYX_STRAND_NAME`, so `lyx tender resume` gives Master its resume line.
The refusal names the pause and its way forward: end your turn now, a running batch finishes, and after the resume line arrives run the same verb again.
A failed GUID lookup still refuses, records nothing, and logs a Warn naming the strand, with its own refusal text: end your turn now, no resume line will come, so run the same verb again once `lyx tender status` shows no pause.
It never names `lyx resume`, which would lift the operator's pause.
An unreadable record refuses nothing and logs one Warn, as loomcli's `refuseCovered` does.
Rationale: both verbs start a new fork; Master is woken by a returning fork outside creel and outside the pause's Esc, so the verb is the only gate on its next step.
Bound: the refusal only defers the verb; record-batch, await-batch and status stay open, so Master can record a finished batch under a pause.
After a failed lookup Master is woken by nothing the pause sends, so it moves on only when it is next prompted and finds `lyx tender status` showing no pause.

## Constraints

- `PATTERN-verified-tree`: every verify still goes through `verifytree.Verify`; D4 changes what Verify does, not who calls it.
- `PATTERN-pane-text-injection`: D1 sits ahead of the pre-write guard.
- `PATTERN-guarded-interrupt`: D7's interrupt is shuttle's guarded interrupt, unchanged.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: each of D8's two refusal texts is a new webster refusal with a trailing `way forward:` clause, reached by a test, naming no denied form and not `lyx resume`.
- `PATTERN-daemon-wakeups`: D4's watch polls once a second only while its command runs, its hold is `pauserecord.WaitUncovered`, which backs off, and D2's notice retry backs off.
- `PATTERN-told-geometry`: `verifytree` stays resolver-free; the hold is told.
- `PATTERN-creel-store`: D7's record write goes through the creel `Delivery` seam, implemented in `internal/creelcli`.
- `PATTERN-leaf-packages`: `internal/pauserecord` keeps importing only the standard library and `internal/state`.

## Auto-mode assumptions

- D4's stop timing (after the grace end) and rerun from the start are auto-picks (ledger Q2).
- D8's hold of `recover-batch` is an auto-pick (ledger Q5).
- D2's claim of a refused tender resume line is an auto-pick (ledger Q6).

## Open risks

- An `Expired` or liveness finalize of a gated run under cover, whose final evaluation reaches a held Verify, waits inside that Verify until the resume.
- A tender resume landing between D7's re-read and its key, whose line then fails busy, leaves the interrupted strand with no resume line.
- A pause landing after D1's last read, during an input-box engine's deferred submit, lets that already-typed line be submitted into a covered session.
- After D8's failed GUID lookup no line wakes Master, so a Master that ended its turn stays idle until the operator prompts it.
- A session claimed by D7 or D8 that is busy again by the resume gets one extra carry-on line, the same cost tender's resume already accepts.
- On Windows `verifyrun`'s cancel ends the shell alone, so D4 leaves the test tree running there; the `verify-watchdog` board note owns that gap.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d4-sigterm-cancel-home-unstated (class decision, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - undeclared-pattern-cuts-unrecorded (class decision, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - boundary-held-survives-turn-change (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- A gated shuttle run whose gate fails at a boundary while a pause covers it sends no re-prompt and spends no attempt; after the resume it evaluates and re-prompts.
- A gated Done arrival under cover runs no gate closure until the run reads uncovered, including a run whose target slug a slug entry covers while its runner's slug reads uncovered.
- `sendVerified` into a covered session, including from a runner whose own slug differs from the target's, types nothing and returns `ErrPaused`; the start path, the deferred task prompt, the seat notice sender, creel delivery and tender resume handle it as D2 says.
- A pause added while a send waits for an idle session types nothing and returns `ErrPaused`.
- A run start whose target slug differs from its runner's slug, refused with `ErrPaused`, waits on the target's slug and sends once that slug reads uncovered.
- `lyx orch start` under a live hub entry starts the orch and types its start lines without waiting for the resume.
- A verify command running when a covering entry passes its grace end gets SIGTERM on its group, its marker reads `paused`, and after the resume the command runs again from the dirty check; a group member that ignores SIGTERM is killed after the wait delay, and `verifyrun.Run` returns only after that; a nil hold runs as today.
- A Verify entered under a covering pause starts no command until the resume, and its marker reads `paused` meanwhile.
- A covering entry resumed before its grace end lets a running verify finish and record its result.
- A caller cancel during D4's hold returns the cancel error.
- A `paused` marker reads as no activity to `agentActivityFor` and `readAgentActivity`.
- Each verify site of D5 passes a hold; a parent-lock holder's hold ignores a slug entry.
- `lyx creel interrupt` of a covered busy strand interrupts it and leaves it recorded in the covering entry, and `lyx tender resume` then gives it the resume line; an entry resumed between the claim and the key gets no key.
- Under a pause, `lyx creel interrupt` still refuses on the privilege rule, on another sender's hold and on an unreadable record.
- A covered strand that reads no running turn stays recorded after `lyx creel interrupt`.
- An orch claimed by D7 under a hub entry gets no second Esc from the watcher and gets its resume line after the resume.
- Creel's delivery worker under a pause records no strand in the entry.
- `lyx webster begin-batch` and `recover-batch` under a covering pause change no state, record Master in the entry, and refuse with the D8 message; a test reaches the refusal.
- `lyx webster begin-batch` under a covering pause whose GUID lookup fails records nothing and refuses with the failed-lookup text, which opens with end your turn now.
- `record-batch`, `await-batch` and `status` run under a covering pause.
- An unreadable record lets `begin-batch` run with one Warn.
- D6's helper records nothing for an uncovered slug and nothing for a session another live entry owns.

## PATTERN changes

- `PATTERN-pause-holds-prompts` — adds: Typing a line into a session a pause covers: shuttle's send refuses with `ErrPaused` by the target's own slug, read after the idle wait, and a conditioned send's caller reads coverage itself; of the production runners that type a line, only the orch runner is told no pause geometry and reads uncovered, so `lyx orch start`'s start lines are never held; the caller holds it until the resume or claims the session.
- `PATTERN-verified-tree` — changes: Running a verify command: only via `verifytree.Verify`, told its scope's pause hold. (test)

## Notes for the plan writer

- `run-process-ownership` and `batten-cancel-verb` end a paused run's processes; a Verify held by D4 lives inside the driver or shuttle process and returns on its context cancel, so ending that process or cancelling ends the hold.
  At the loomshed and webster verify gates, which pass `context.Background()`, a held Verify ends only on the resume or on the end of its process.
- D8 settles only that no new batch starts under a pause; what a pause does to a running fork stays with `run-process-ownership`'s `pause-background-work` note.
