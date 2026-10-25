---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 continue-pass paragraph, D3
    summary: Continue passes skip the origin reconcile on the false premise that InnerRun's owner check covers the gap; that check fires only on a sleep seen inside one process, so a run acted on after a machine wake gets no reconcile, against PATTERN-owner-machine
    fix: In D4, state that the owner check today is due only after a sleep the same process watched, a pause lift or a retry while owned elsewhere (internal/battenshed/innerrun_owner.go, the slept/released/retry trigger), so it never fires in a short pass; then decide how a wake reaches the runs, for example dobby watches its own awake versus wall clock as InnerRun's SleepWatch does and makes the next pass of every run it drives a full pass with the origin reconcile after any sleep it sees, or persists the owner trigger's state in the memory record. Drop the clause "between those InnerRun's own owner check covers it" and D3's "the owner check inside each pass covers a run another machine takes over" unless the chosen mechanism makes them true, and add an acceptance criterion that the first pass after a machine wake runs the origin reconcile.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5
    summary: Pair teardown's claim settlement clears the status of a removed pair whose batten run never seeded or halted, so after lyx fabric remove the task reads empty and its parent's dobby starts it afresh, the undo D5's cancel-to-draft rule exists to prevent
    fix: In D5, beside the cancel rule, decide that pairteardown's claim settlement (internal/pairteardown/claim.go settleClaim, the claimClear arm for unseeded, paused, blocked, failed and awaiting runs, written through writeBoardClaim) leaves the task draft instead of clearing its status, and restate D5's bound as no code path other than an explicit release leaving a task statusless; add the case to the acceptance criterion on cancel, as a task whose pair lyx fabric remove removed is left draft.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 memory-record paragraph, Open risks first bullet, Constraints notice-marker bullet
    summary: The InnerRun memory record is left to cover every field, a deferred research question; part of the state lives per Call today, notably the parent lock's idle watch, which restarts on every one-check pass so an idle owner's parent lock is never released
    fix: Do the inventory now and record it in D4 with its source, internal/battenshed/innerrun.go, notice.go, innerrun_binaryrestep.go and innerrun_parentlock.go. Name the producer fields that persist (noticeEpisode deliveries and arbiterOffAt, driverBusyNoticed, ownerNoticed, ownedBy, the failures episode set, restepWatch) and the per-Call childWait fields that must now persist across passes (lockWatch idle watch and owner, resumeRetryAt, notParkedLogged, busyLogged and busyKey, lastProbe, the idle sleep watch), and say for each which rule it keeps. Remove the Open risks bullet once the list stands, and correct the Constraint that notice markers already survive a restart to name the in-memory latches the record now persists.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 first paragraph and per-run bound, D12
    summary: A pass is not one check, because holdSpawnWhileCovered, the driver's PauseHold, the Land polls, the prime-lock wait and the row-notice retry spacing all sleep in-process, so a pass stays in flight through a whole pause and D12's pass at the lift is skipped as a pass in flight
    fix: In D4 decide which in-pass waits become a yield and which stay, and bound the rest; for example, InnerRun's spawn hold under a covering pause or a standing owner (internal/battenshed/innerrun_pause.go holdSpawnWhileCovered) and the driver's pause hold return Yield instead of sleeping, while the Land polls, the prime-lock wait (10 minutes) and the row-notice spacing stay in-pass with their existing bounds stated. Add to D4's bound the longest a pass may stay in flight, and make D12's lift pass consistent with it.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 bound, D2
    summary: lyx reed down of a beam's session runs EndBeforeSessionDown, which ends every process under the beam's slug and so the beam's dobby while member runs still need it; D1's bound names only a cancel or owner pause of the beam's landing run
    fix: In D1, name lyx reed down (internal/pairteardown/skip.go EndBeforeSessionDown, reached from internal/reedcli/up.go) as a path that ends the beam's dobby, and decide either that EndBeforeSessionDown spares a RoleDobby record, or that the ending is accepted with lyx beam start, through lyx orch start, as the way back; state the choice in D1's bound and make D2's always-a-dobby claim match it.
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D4 continue-pass paragraph
    summary: The hidden --continue-pass flag skips the origin reconcile and the Open-Window re-run, but D4 states no bound on who may pass it or on which run state, so a hand call on a halted run or a run off Run-Shed skips both
    fix: In D4, state what --continue-pass can skip and what bounds it, for example that the pass refuses the flag, naming a plain lyx batten run <slug>, unless batten's status reads running at Run-Shed and the caller's dobby state record names the run as one it drives, and that the flag never skips the owner judgement F1 settles.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Goal, D4 halted-run paragraph and continue-pass paragraph, D6
    summary: The Goal and D6 say the orch never starts, resumes or cleans up a run, yet D4 has a halted or paused batten run wait for a resume by the orch, and the continue-pass paragraph's every pass that resumes a halted or paused run sits under dobby's passes without saying it is a hand pass
    fix: Decide who resumes a halted or paused batten run and state it once in D4, for example the operator, or the orch on the operator's word or after judging the stop, by a hand lyx batten run <slug>; reword the Goal's and D6's never-resumes claims to match, and rewrite the continue-pass sentence as both run on the first pass after a dobby starts and on every hand pass, never a dobby pass, that resumes a halted or paused batten run.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 respawn cap
    summary: The cap of three respawns per child state episode can be spent in harmless gaps of a long running episode, such as driver hand-overs, leaving a driver that dies later in the same episode unrespawned
    fix: In D4, key the respawn cap on the run of passes that found no driver at work, resetting the count whenever a pass sees a live driver strand or a held run or driver lock, instead of the child's state episode alone.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 pairteardown paragraph
    summary: D4 names two readers of a live batten, but driverChecksSkipped in EndSession is a third; between passes it reads Finalize Done with no live batten, so lyx fabric remove skips the quiet wait while Land is pending
    fix: In D4, state the rule on pairteardown's liveness read itself, a run reads as having a live lifecycle owner when a registered batten lives or a live dobby drives it, so every reader of it, driverChecksSkipped included, takes it.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8 config migration
    summary: The copy of loom's arbiter keys into darn.yaml covers only a hub layer, so a value set in a worktree layer, the local layer or a task's run_settings.loom stops reaching darn without notice
    fix: In D8, either extend the copy to every layer darn reads loom's arbiter keys from today, naming them, or state that only hub-layer values carry over and that the other layers' values now apply to loom alone.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 landing count
    summary: Counting a landing at the end of a dobby pass misses a landing made by a hand pass, or by a pass a dead dobby left running, so every board_tidy_every landings undercounts
    fix: In D10, count from the batten history of every run the dobby drives or has driven, read on each tick and keyed by run in the state record, rather than only at the end of a pass the dobby ran.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 migration
    summary: An older binary on another machine still writes statusless tasks after the draft_status key exists, and the owner machine's dobby starts them unreleased; the support log's rejection reasons about the starter's binary, not the writer's
    fix: In D5, state as an accepted bound that a task an older binary creates after the migration starts without a release until every machine runs the new binary, or decide a guard against it.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D1 re-exec, D4 continue-pass paragraph
    summary: D4 runs the reconcile and the Open-Window re-run on the first pass after a dobby starts, and D1 re-execs dobby on every binary change, but the record never says whether a re-exec counts as a start
    fix: In D4, state whether the first pass after a re-exec is a full pass or a continue pass.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7, D4 failing-pass notice, D5 refusal notice
    summary: D7 posts dobby's own failures one line per episode, while D4 and D5 post one notice per error or refusal text, two dedup keys for the same notices with no episode defined
    fix: In D7, replace one line per episode with the keys D4 and D5 use, one notice per run and error text for a failing pass and one per task and refusal text for a refused start, and say when each key is cleared.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 max_runs, D13
    summary: D5 counts halted runs as in flight, and D13 backs the timer off only with no run in flight, so a dobby holding only halted runs, which get no passes, never backs off
    fix: In D13, define in flight for the backoff as a run dobby passes, running and not under a cancel marker, apart from D5's max_runs count.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 self-trigger guard
    summary: An fswatch event for a batten status write a pass made can arrive after that pass has exited, so it reads as no pass in flight and triggers one more pass beyond D4's per-run bound
    fix: In D4, ignore a batten status write whose modification time is no later than the exit of the slug's last pass, so only a write from outside dobby's passes triggers a pass.
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D11 last sentence
    summary: D11's closing sentence restates the operator's wording and the rejected place, dobby, as rationale-by-rejection, which support-log.md's Rejected alternatives already holds
    fix: Delete D11's last sentence; its constraint, that the check runs in Finalize, is already D11's first sentence, and the rejected dobby placement stays in support-log.md.
---

### [BLOCKING] Continue passes skip the reconcile on a false premise about the owner check

**Location:** `_lyx/discussion/decision-record.md` D4 continue-pass paragraph, D3.
**Issue:** D4 says a continue pass skips the origin reconcile because "between those InnerRun's own owner check covers it".
In `internal/battenshed/innerrun_owner.go:67-75`, the check becomes due only when this process's `ownerSleep` watch sees a sleep, a covering pause is lifted, or a retry is due while the run is owned elsewhere.
`ownerSleep`, `ownerDue` and `ownerCovered` are in-memory producer fields (`innerrun.go:165-170`).
A short pass process never spans a machine sleep.
Dobby lives through the suspend, so every pass after the wake is a continue pass, and no pass judges ownership against origin.
A run another machine adopted meanwhile is respawned or landed on the old machine.
That breaks `PATTERN-owner-machine` ("acting on an entry after a wake: origin reconcile first").
All four lenses found it independently.

### [BLOCKING] Pair teardown's claim clear restarts a removed task

**Location:** `_lyx/discussion/decision-record.md` D5.
**Issue:** `internal/pairteardown/claim.go:45-50` returns `claimClear` for an unseeded run and for a paused, blocked, failed or awaiting one, and `writeBoardClaim` (`claim.go:166-172`) writes a nil status.
After `lyx fabric remove` of a halted run's pair, the task is statusless.
With its dependencies done, its parent's dobby starts a fresh run of a task the operator just removed.
D5 moves only `lyx batten cancel` to `draft`, so the bound "a task is started only from an empty status" no longer guards anything.

### [BLOCKING] InnerRun memory inventory deferred; per-Call state resets every pass

**Location:** `_lyx/discussion/decision-record.md` D4 memory-record paragraph, Open risks, Constraints.
**Issue:** The record leaves the field list to Plan ("must cover every field"), which the rubric counts as a deferred research question.
The design's once-per-episode and idle-bound claims rest on that list.
Part of the state is not producer memory at all: `childWait` is "the state of one Call's wait" (`innerrun.go:554-600`).
That includes `lock lockWatch`, whose `idle` watch counts the owner's idle time "across the checks of one Call" (`innerrun_parentlock.go:14-19`).
With one check per Call, that watch restarts on every pass, so an idle owner never reaches the idle bound and its parent lock is never released.
The producer fields are unnamed as well: `driverBusyNoticed`, `ownerNoticed`, the `noticeDelivery` attempts and `dropped`, `restepWatch` and others.
The Constraint that notice markers "already survive a restart" holds only for the file markers.

### [MEDIUM] A pass can stay in flight through in-process waits

**Location:** `_lyx/discussion/decision-record.md` D4, D12.
**Issue:** `holdSpawnWhileCovered` (`innerrun_pause.go:51-70`) sleeps for as long as a pause covers the slug or an owner stands.
The driver's `PauseHold` waits too.
Elsewhere in a pass, the Land row polls (`land.go:377`, `land.go:414`), the prime-lock wait runs up to ten minutes (`primelockwait.go`), and row notices pause 30 seconds between attempts (`rownotice.go:215`).
So a pass can block for a whole pause, which contradicts "one check of its child per Call".
The lift pass D12 promises is then skipped, because D4 never runs two passes of one slug at once.

### [MEDIUM] `lyx reed down` ends a beam's dobby

**Location:** `_lyx/discussion/decision-record.md` D1 bound, D2.
**Issue:** `EndBeforeSessionDown` (`internal/pairteardown/skip.go`), reached from `lyx reed down`, ends every process registered under the pair's slug.
That includes a beam's dobby, registered under the beam's slug.
D1's bound names only a cancel or owner pause of the beam's landing run.
D2's "a run's parent always has a running dobby" fails until someone runs `lyx beam start`.

### [MEDIUM] `--continue-pass` has no stated bound

**Location:** `_lyx/discussion/decision-record.md` D4 continue-pass paragraph.
**Issue:** The flag is a new escape hatch that skips the origin reconcile and the Open-Window re-run.
The rubric's attack-surface rule asks the Decision to state what the flag lets through and what bounds it.
D4 says when dobby passes the flag, but not what a hand call with it on a halted run, or on a run off Run-Shed, would skip.

### [MEDIUM] Who resumes a halted run disagrees across Goal, D4 and D6

**Location:** `_lyx/discussion/decision-record.md` Goal, D4, D6.
**Issue:** The Goal says the orch "stops starting, resuming or cleaning up runs", and D6 that "the orch never starts, stops or cleans up a batten run".
D4 says "a halted or paused batten run waits for a resume by the orch or the operator".
D4's continue-pass sentence, "every pass that resumes a halted or paused batten run", sits among dobby's passes.
It agrees with the acceptance criterion that a halted run gets no pass from dobby only if that pass is a hand pass, which the record does not say.

### [LOW] Respawn cap can be spent in harmless gaps

**Location:** `_lyx/discussion/decision-record.md` D4 respawn cap.
**Issue:** Every pass reads the per-pid spawn confirmation as unconfirmed (`innerrun.go`, `SpawnConfirmedFile`).
Passes that fall into short driverless gaps of one long running episode can use up all three respawns.
A real driver death later in that episode then gets only a notice.

### [LOW] Third live-batten reader in pairteardown

**Location:** `_lyx/discussion/decision-record.md` D4 pairteardown paragraph.
**Issue:** `driverChecksSkipped` (`internal/pairteardown/skip.go:25-50`) also reads `battenAlive`.
Between passes, a Finalize Done with no live batten makes it skip the quiet wait while Land is pending.

### [LOW] Darn's arbiter copy covers only the hub layer

**Location:** `_lyx/discussion/decision-record.md` D8.
**Issue:** The arbiter keys are `CategoryNone` (`internal/loomengine/config.go:592-593`), so a local layer may hold them, and a task's `run_settings.loom` may carry them too.
D8's copy into `darn.yaml` reaches only a hub layer, so darn silently loses values set in those other places.

### [LOW] Landing count misses landings outside dobby passes

**Location:** `_lyx/discussion/decision-record.md` D10.
**Issue:** A landing made by a hand pass, or by a pass a dead dobby left running, never ends a dobby pass, so it is never counted.

### [LOW] Older binaries still create statusless tasks

**Location:** `_lyx/discussion/decision-record.md` D5 migration.
**Issue:** `boardengine` keeps the `draft_status` key (`store.go:62-70`), but an older binary's upsert on another machine still writes a task with no status.
The support log rejects this on the starter's binary, while the defect lies in the writer's binary.

### [LOW] Re-exec versus first pass after a start

**Location:** `_lyx/discussion/decision-record.md` D1, D4.
**Issue:** The record does not say whether a re-exec on a binary change counts as a dobby start.
If it does, the next pass of every run is a full pass, Open-Window re-run included.

### [LOW] Two dedup keys for dobby's own notices

**Location:** `_lyx/discussion/decision-record.md` D7, D4, D5.
**Issue:** D7 says "one line per episode", while D4 and D5 say one notice per error or refusal text.
The record defines no episode for dobby's own failures.

### [LOW] Backoff with only halted runs

**Location:** `_lyx/discussion/decision-record.md` D5, D13.
**Issue:** D5 counts halted runs as in flight, and D13 backs the timer off only when no run is in flight.
So a dobby holding only halted runs, which get no passes, never backs off.

### [LOW] Late fswatch event after a pass exits

**Location:** `_lyx/discussion/decision-record.md` D4.
**Issue:** D13 coalesces events into ticks of at most one a second.
A pass's own final status write can therefore be delivered after the pass exits, and it then triggers one more pass.

### [LOW] D11 rationale-by-rejection

**Location:** `_lyx/discussion/decision-record.md` D11.
**Issue:** "The operator called this a job for the Go run driver; it stays in Finalize … rather than in dobby" is deliberation narrative.
The rejected placement is already in support-log.md's Rejected alternatives, and the requirement itself is already D11's first sentence.

## Rejected

- walkthrough F5, the arbiter consume is not named in D8's move: D8 moves "the arbiter seam" into shed, which covers its consume, and listing the seam's members is call-site enumeration the rubric's Do-not-flag list leaves to Plan.
- interactions F8, D6 and D9 act on one notice redundantly: D6 also removes the ready snapshot, `LandDeps.ReadyRoutes` and the orch stencil's start steps, which D9 does not, so neither decision is idle and they do not contradict each other.
- bound-breaker F10, second half, `arbiter_timeout_min: 0` changes meaning: a 0 means "the default" today and under D8 alike, and D8 sets `shed.yaml`'s default to today's loom value, so a hub that tunes nothing sees no change.
- Board-entry bullets (D4 note removal, D5 draft and cancel, D10 report handling): no Decision edits this task's own entry or files or promotes a task. D4 removes another entry, a note. D10's tidy agent amends other notes, and the board's task guard still refuses it a task creation.
