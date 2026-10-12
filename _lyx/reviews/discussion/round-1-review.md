---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:38
    summary: D1 bounds the hub-wide pause by the audience annotation, which gates nothing, so any agent can freeze every run, every network call and the orch and silence every notice with no stated bound
    fix: In D1, replace the sentence naming the audience annotation as the only gate with a stated bound, recommended - a hub-wide lyx pause refuses when LYX_STRAND_NAME names a shuttle-run strand of a pair (an agent), naming the way forward of asking the orch or the operator; the orch and an EXTERNAL caller may run it, and an agent may run lyx pause only for its own pair's slug. Add the refusal to the Constraints refusal list and an Acceptance criterion under Invalid input (an agent strand's hub-wide pause refuses with its way forward; the orch's succeeds). Then drop or reword the Open risks bullet on a forgotten hub pause so it names the remaining case, an operator-left pause, and its signals.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:95
    summary: D4's parent-lock exception lets a covered run's own landing remote calls through a per-run pause with no stated bound, and with D8 a covered run queued for the lock acquires it and lands during the pause
    fix: "In D4, bound the exception: a covered run never takes the parent lock while covered - the parent lock wait keeps waiting through its existing Paused seam rather than granting, with no ErrPaused halt - so the exception reaches only a run that already held the lock when the entry was written. State in D4 what that lets through (the holding run's remote calls of its current landing row) and that D3 step 4 and lyx pause --status name it as a network operation in flight under a parent lock. Add an Acceptance criterion under Holds and readers: a covered run waiting in the parent lock's queue is not granted the lock until the resume."
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:110
    summary: D5 names the parent lock's idle release with no behavior, and a covered holder of the lock, held from Merge-In to Finalize across agent rounds that D4 holds, keeps the lock for the whole pause, freezing other runs through the shared lock that D4's exception claims never freezes them
    fix: Replace the D5 bullet with the behavior and its bound, recommended - paused time is not idle time for a covered holder, so the lock stays held until the resume; other runs' lock-taking rows wait in the queue as for any live holder; lyx pause --status and D3 step 4 name each parent branch a covered run holds and the runs queued behind it. Then reword D4's rationale clause (so a per-run pause never freezes other runs through a shared lock) to say what it prevents, a held network call of the holder's current row, and that other runs still wait behind a covered holder until the resume. Add the queued-behind-a-covered-holder case to the Acceptance criteria.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: D7's orch interrupt fires on the watcher's next tick after a new hub entry, skipping D3's grace that exists to let a push end, and D3's report never says whether the orch was interrupted or whether a watcher is running to do it
    fix: In D7, state that the watcher interrupts the orch only once the entry's grace has ended, with the grace end recorded in the entry by D3 step 1, and only while no network operation marker of the orch's is open or the grace has passed, the rule D3 step 2 uses. In D3 step 4, add the orch's case to the report - delegated to a live watcher, or no live watcher so the orch is not interrupted. Add both to the orch Acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:136
    summary: D7 queues the resume line on every hub entry's resume, while D6 sends a line only into a session the entry owns, so an orch idle at pause time or one that ran lyx pause itself gets a line D6 forbids
    fix: In D7, change the resume clause to - on that entry's resume, when the entry owns the orch, it queues the resume line as a notice; an orch the entry does not own (idle at pause time, or the caller) gets none. Extend the orch Acceptance criterion with - an orch idle at pause time, or that ran lyx pause, gets no resume line.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:146
    summary: Removing landingshed's PauseRequested seam leaves a loom run waiting in the parent lock's queue with no way to be moved, while the goto and shed-busy ways forward of D8 name pause --after <current_producer>, which never fires for a row that cannot finish until it gets the lock
    fix: In D8, replace the landingshed bullet - landingshed's PauseRequested seam reads the run's pause_after naming the waiting row instead of pause_requested, so lyx loom pause --after <row> ends a lock wait with the existing halt, and the refusal-spec row parent lock wait paused stays with its trigger reworded to that condition. Update D8's Bound and the Boundary pause Acceptance criteria with - a loom row waiting in the parent lock's queue halts on pause --after naming it, and goto then moves it.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:168
    summary: D10's post-kill stop names its targets as every live shuttle run of the killed step with no bound on how membership is decided or what it must never stop, such as the loom driver's strand or a pair-level session outliving the step
    fix: In D10, add a Bound line - the stop takes only shuttle runs whose record names the killed step (its step run directory), never the loom driver's strand, the orch or a run of another step; a run the record does not tie to the step is left and named in the halt note. Extend the watchdog-kill Acceptance criterion under Sleep and 567 with - the loom driver's strand and runs of other steps stay live after a watchdog kill.
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:159
    summary: D9's automatic continue line is typed without naming shuttle's pre-write guard, unlike D6's resume line, and says nothing of a guard refusal
    fix: In D9's Shuttle Wait bullet, say the continue line goes through shuttle's guarded paste-submit (PATTERN-pane-text-injection), and that a refused write sends nothing and the stall is held and notified as today. Add the bound to D9's Bound line.
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:8
    summary: The Goal says the loom and webster step-boundary pause goes, while D8 keeps the --before and --after boundary halts and lyx batten pause
    fix: In the Goal, change The loom and webster step-boundary pause goes to The bare loom and webster boundary pause goes.
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:99
    summary: D4's line that lyx shuttle interrupt is not refused re-argues a rejected alternative already in the support log's Rejected alternatives and carries no requirement the D4 table does not
    fix: Delete the line lyx shuttle interrupt is not refused - an interrupt during a pause is harmless - from D4; the support log's Rejected alternatives entry on refusing lyx shuttle interrupt stays.
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:96
    summary: The Question ledger's open entry on the 2026-10-12 notice points at Open risks, which no longer holds it; D9 carries it as a plan check
    fix: Rewrite the ledger entry as resolved - the plan checks whether the 2026-10-12 eleven-hour notice came from an older binary or a gap in the awake subtraction, and fixes the cause (D9).
---

### [BLOCKING] Hub-wide pause has no gate on its caller

**Location:** `_lyx/discussion/decision-record.md:38`

**Issue:** D1 says the audience annotation is the only gate, the same as `lyx batten pause`.
The annotation gates nothing: `internal/clihelp/annotations.go` defines it as the key the command index filters on, and no code refuses a call by it.
So any agent's Bash call can write a hub entry, which by D3, D4, D5 and D7 interrupts every covered session and the orch, holds every network call and shuttle start, suppresses every quiet, api-error, driver-dead, hold and dialog notice, holds the orch watcher's queue, and survives a reboot (D2).
`lyx batten pause` touches one run; this touches the hub.
The only signals the record names are `lyx pause --status` and `paused_since`, which nobody is prompted to read when every notice is held, so the bound does not stop a forgotten or agent-issued hub pause from silently freezing everything.
The attack-surface rule needs the Decision to state what bounds the new verb.

### [BLOCKING] Parent-lock exception lets a covered run land during its pause

**Location:** `_lyx/discussion/decision-record.md:95`

**Issue:** The exception exempts "one whose caller holds a parent lock" from a per-run entry, but a per-run entry only holds operations run on behalf of its own pair, so the exception exempts exactly the covered pair's own landing: Publish's push and PR-Gate's and Finalize's GitHub calls (`internal/landingshed/doc.go`).
That is the network work D4's rationale (#458) says a suspend cuts.
D8 adds a path into it: a covered run's Go row waiting in the parent lock's queue runs on under the pause (Scope Out), acquires the lock when it frees, and its remote calls then pass the exception.
D4 states what the exception is for but not what it lets through or what bounds it.

### [BLOCKING] A covered lock holder still freezes other runs

**Location:** `_lyx/discussion/decision-record.md:110`

**Issue:** D5 lists "The parent lock's idle release" with no behavior.
Read with D5's opening sentence, a covered holder's paused time is not idle, so no idle release happens.
A run holds the parent lock from Merge-In to Finalize (`internal/landingshed/doc.go`), across merge-review agent rounds, and D4 holds those rounds' shuttle starts, so a covered holder sits on the lock for the whole pause and every other run's lock-taking row queues behind it.
D4's exception exists so that "a per-run pause never freezes other runs through a shared lock"; with the holder's agent work held, the exception does not achieve that, and no Decision says what the queued runs do or how the operator sees them.

### [MEDIUM] Orch interrupt skips the grace and the report

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** D3's grace exists so "a running Bash command or push end cleanly", and D7's rationale names the orch as the session that makes network calls and runs landings.
Yet the watcher acts "on a new live hub entry" on its next tick (2 s by default, `internal/orchengine/config.go`), with no grace, so an orch push in flight gets the Esc the grace protects every other session from.
D3 step 4's report lists shuttle sessions only, and D3 says the verb never touches the orch, so the operator suspending on the report's word learns nothing about the orch, nor that no watcher is running to interrupt it.

### [MEDIUM] Orch resume line contradicts D6's ownership rule

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** D6 sends a line only into a session the entry owns and none into an idle-at-pause one.
D7 queues the resume line "on that entry's resume" with no ownership condition, so an orch that was idle at pause time, or that ran `lyx pause` and was skipped, still gets it.
The orch Acceptance criterion covers only the busy orch's line and is silent on the caller's, so the text and the criteria leave the case unowned.

### [MEDIUM] No way to move a loom run out of the parent lock queue

**Location:** `_lyx/discussion/decision-record.md:146`

**Issue:** The premise holds: landingshed's seam is wired only from `internal/loomcli/landingdeps.go` and reads loom's `pause_requested`, which nothing sets after D8.
D8 states the waiter's bound, that it waits until the lock frees or the holder's idle release frees it.
What it leaves out is the operator's loss: today the bare pause ends a lock wait with the "parent lock wait paused" halt, and `lyx loom goto` can then move the run.
After D8, goto on that running run refuses with `pause --after <current_producer>` (`internal/shedengine/goto.go` today names the bare pause), and an `--after` condition fires only once the waiting row finishes, which it cannot do before it gets the lock.
The way forward names a command that exists but does not do what the refusal asks for that case, and with F3 a covered holder can make that wait last the whole pause.

### [MEDIUM] Post-kill stop has no bound on what it stops

**Location:** `_lyx/discussion/decision-record.md:168`

**Issue:** D10 adds an automatic stop of "every live shuttle run of the killed step" from loomcli's loop arming.
It does not say how a run is tied to the step or what is excluded.
The loom driver is a shuttle-launched strand in the same pair and outlives every step, and burler's live-fix resume (`internal/burlerengine/fix.go`) shows that strands are re-adopted across a step death, so membership is not obvious.
The attack-surface rule asks the Decision to bound what the new action can stop.
The pause-covered case is bounded already: D5 rules out a watchdog kill under cover.

### [LOW] Automatic continue does not name the pre-write guard

**Location:** `_lyx/discussion/decision-record.md:159`

**Issue:** D9's continue line is lyx typing text into an agent pane, which `PATTERN-pane-text-injection` puts behind shuttle's pre-write guard.
D6 names the guarded paste-submit and its refusal case for the resume line, and D9 names neither for the continue line, so the record disagrees with itself on how an automatic line is typed.
Its bound on what it fires into is otherwise sound: API-error cause, detected sleep, once per turn, never under cover.

### [LOW] Goal overstates the boundary-pause removal

**Location:** `_lyx/discussion/decision-record.md:8`

**Issue:** The Goal says the step-boundary pause goes, while Scope and D8 retire only the bare form and keep `--before`, `--after` and `lyx batten pause`.

### [LOW] D4 re-argues a rejected alternative

**Location:** `_lyx/discussion/decision-record.md:99`

**Issue:** The line on `lyx shuttle interrupt` explains why a guard was not added.
The D4 table already says which paths are held, so the line carries no requirement, and the support log's Rejected alternatives already holds the deliberation.
Under the completeness-before-leanness test nothing has to be pulled into Decisions first.

### [LOW] Question ledger points at a section that no longer holds the item

**Location:** `_lyx/discussion/support-log.md:96`

**Issue:** The ledger's only open entry cites Open risks, but the 2026-10-12 notice is in D9 as a plan check to find and fix the cause, and the Open risks section no longer names it.

## Verdicts on focus items with no finding

- D9's note on the 2026-10-12 notice is not a defect: it gives the plan a requirement, to find why the notice counted almost eleven hours and fix the cause, so it stays in `decision-record.md`.
  The other rationale lines (D2, D4's #458, D8's operator complaint) each justify their own decision and do not re-argue a rejected one.
- D2 and D5's 7-day retention is not a defect: any idle clock or shuttle deadline that reaches back more than 7 days past a resume has already passed its window on awake time since the resume, so it fires as it would have without the pause.
- D3 and D6 ownership between hub and slug entries is consistent for shuttle sessions: the first entry owns, ownership moves on resume, and the line goes when the last covering entry resumes, which matches the overlap criterion.
  The orch is the gap (F5).
- D8's ways forward name commands that exist: `lyx loom pause --after <producer>` exists (`internal/shedverbs/pause.go`) and `lyx batten pause` stays.
  The lock-queue case is F6.
