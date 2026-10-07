---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:49 (D3), :115 (Constraints, Windows), :139 (Acceptance)
    summary: D3 and the acceptance criterion promise a whole-tree kill on Windows, but a new process group on Windows gives no tree kill; the Constraint then offers the fallback D3 contradicts
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:60-66 (D5), :131 (Open risks)
    summary: D5 drops a non-live gated strand whose outputs exist but whose gate is unpassed, losing the gate re-prompt; its bound does not cover this and the Open risk wrongly calls it unchanged
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:78-93 (D7)
    summary: D7's outcome list leaves states unhandled or contradictory (awaiting with a parked driver, a Go-driven running run, precedence, the unfinished-merge guard start applies)
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:98 (D8), :105 (Constraints)
    summary: D8 says no PATTERN entry is added, yet D5 changes who answers "did this run finish" under PATTERN-completion-signal and D2 sets a display-only wait-mark rule that spans modules
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:38-45 (D1, D2), :129 (Open risks)
    summary: D2's bound says display only but not what a stale or wrongly targeted mark shows; the driver strand's mark has no clearing path and D1's marker precedence is unstated
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:70-76 (D6), :88 (D7), :24 (Scope Out), :107 (Constraints)
    summary: D6's state list is ambiguous about awaiting, its Bound covers only the revive and not the re-spawn, the way forward differs from D7's for the same state, and the Constraints call batten's notice a refusal with a spec row though batten has no refusal-spec section
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:6 (Goal), :128 (Open risks), :138 (Acceptance)
    summary: Goal says reed revives only strands with work left and every Go-side wait shows, but Scope and D5 cover shuttle strands and three named waits only; the elapsed-time criterion has no counterpart for the start-time fallback
  - id: F8
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:13,36,53,93,119 and support-log.md:36
    summary: Incident history and provenance tags in decision-record.md carry no plan requirement and repeat the support log; they belong in support-log.md
---

### [BLOCKING] D3 promises a whole-tree kill that Windows cannot deliver

**Location:** `decision-record.md` D3 (line 49), Constraints last bullet (line 115), Acceptance criterion at line 139.
**Issue:**
D3 says `verifyrun` starts the shell "in its own process group (Unix) or a new process group (Windows)" and kills "the whole group".
The acceptance criterion says "no process of its group survives".
On Windows, `CREATE_NEW_PROCESS_GROUP` (the flag `internal/proc/proc_windows.go` already carries as `createNewProcessGroup`) only isolates console control events.
Go's `os.Process.Kill` calls `TerminateProcess` on one process, and nothing kills a Windows "group".
A tree kill there needs a Job Object or a process-table walk (`reedengine/proctree_windows.go` walks one through `Get-CimInstance`), and `verifyrun` may import only the standard library and `internal/logger`.
So the Decision rests on a false premise on Windows.
The Constraints' last bullet then says the kill "must hold on Windows or be stated as Unix-only with the Windows fallback (kill the shell only, as today) logged", which leaves the choice open and contradicts D3's text.
The same bullet lists "pane options" as a Windows risk, but reed already drives `set-option -p` in `spawn.go` on both platforms, so no carve-out is needed there.
**Fix:**
Make the Windows call in D3: Unix kills the process group; Windows kills the shell only, as today, and logs the degraded kill.
Scope the acceptance criterion to Unix, with the Windows case a failed gate naming the timeout whose pipe read `WaitDelay` still bounds.
Drop the open "must hold or be stated" bullet and its pane-options clause.
Record the Windows job-object alternative in the support log's rejected alternatives.

### [BLOCKING] D5's bound does not cover a dropped gated strand

**Location:** `decision-record.md` D5 (lines 60-66), Open risks last item (line 131).
**Issue:**
D5 drops a non-live strand whose done-when paths all exist, and sets the list to the run's output files.
For a gated run (`handleGatedBoundary` in `shuttleengine/wait.go`), "outputs present" is not "finished": the gate runs after the outputs exist, and a failing gate re-prompts the live agent while attempts remain.
After a crash in that window, `reed resume` today relaunches the agent, so the gate can still re-prompt it.
Under D5 the strand is dropped, `Attach` harvests the run through the file contract, and a failing gate finds no agent to re-prompt, so `handleGatedBoundary` finalizes after a failed `Send` with the attempts unspent.
That weakens a guard (the gate's fix attempts) without a bound, which the rubric's attack-surface question requires.
The Open risk admits the effect but calls it "today's behaviour for a dead strand, unchanged by this task".
That is wrong: today `reed resume` revives the agent, and this task is what stops it.
D5's Rationale also says output files "are what shuttle already treats as finished", which holds for the negative classifications `PATTERN-completion-signal` lists and not for a gated run's own finish.
**Fix:**
Shuttle sets the done-when list only on a strand of an ungated run.
A gated run's strand carries none and is relaunched as today, so a strand with its gate unpassed keeps its re-prompt.
State that bound in D5, correct the Rationale to say outputs mean finished for an ungated run, and delete the Open risk.
Add an acceptance row: a gated strand with all outputs present is relaunched.

### [MEDIUM] D7 leaves states unhandled or contradictory

**Location:** `decision-record.md` D7 (lines 78-93).
**Issue:**
D7 lists five outcomes without saying how they are ordered or which run states each covers.
- A parked live driver over an awaiting run matches the first outcome (the shared branch with `start`, whose hand-back set `runHaltedAtHandBack` includes awaiting) and the last (awaiting: refused).
- A running run whose driver is a Go process holds the run lock and has no driver strand, so it falls under "no live driver strand: refused", with a way forward naming `lyx batten run`.
  `start` tells a live Go driver from an absent one by the run lock, not by the seed, so D7 can too without breaking `PATTERN-driver-choice-single-site`.
- A done run with no strand matches two refusals, and a missing status file matches none.
- `start` refuses over an unfinished merge (`refuseOverUnfinishedMerge`) before it resumes a driver.
  D7 says it takes "the same branch" and lists no merge refusal, so the guard is either dropped silently or unstated.
  `start` also holds a bootstrap lock, and D7 does not say how the new verb serialises against it.
**Fix:**
Order the outcomes by run state first (done or awaiting: refused; running: no-op when a driver strand is live or the run lock is held, else refused; halted: the strand and marker rules) and add the merge-in-progress refusal and the bootstrap lock.
Each new refusal gets a row in the spec and a test, as the Decision already requires.
Add the unlisted refusals to the acceptance criterion.

### [MEDIUM] D8 contradicts the same-commit PATTERN rule

**Location:** `decision-record.md` D8 (line 98); Constraints (line 105).
**Issue:**
D8 states "no PATTERN entry is added, since no new cross-cutting rule arises".
But D5 moves a "did this run finish" answer into reed through a list of paths filled from `OutputFiles`.
`PATTERN-completion-signal` says only `shuttleengine`'s own paths answer that, and its Constraints line here already invents the pairing ("no other finished test is invented").
D2 also sets a rule that spans shuttle, landing, loom and reed: the wait mark is display only and no Go decision reads it.
`CLAUDE.md` puts a new cross-cutting invariant into PATTERN in the same commit.
**Fix:**
D8 states that `PATTERN-completion-signal` and its background file gain the done-when list as the one place reed learns a strand finished (filled only by `shuttleengine` from `OutputFiles`, read only by `reedengine` `Resume`).
D8 also adds one entry for the wait mark: display only, set only through reed's one op by the shuttle and the landing callback, never read by a Go decision.
Remove the "no PATTERN entry is added" clause.

### [MEDIUM] D2's bound does not say what a stale or misdirected mark shows

**Location:** `decision-record.md` D1 (lines 34-35), D2 (lines 38-45), Open risks (line 129).
**Issue:**
D2's bound is "display only, never read by any Go decision", which says what the mark cannot do and not what it can show.
Two cases are left open.
- A crashed step leaves a mark on a surviving pane.
  The Open risk says it clears "until shuttle next touches that strand", but the landing callback marks the driver strand, which shuttle never touches, so a stale mark there has no clearing path.
- The callback targets "the driver strand it already resolves"; if that strand is gone or replaced when the callback fires, the Decision does not say whether the verify fails, the mark lands elsewhere or the call is a no-op.
D1 lists three readers "in order" but does not say that the first present one wins, nor that a dead-pid marker is skipped rather than ending the search.
**Fix:**
D2 states: a mark can show a wait that has ended, never hide a real one (status is unaffected); a stale mark ages visibly through its elapsed time; the next set or clear on the strand and pane death remove it, and the landing callback clears at the start of every verify on the driver strand.
A callback whose strand is gone is a logged no-op and never fails the verify.
D1 states the first present, live marker wins.
Update the Open risk to name the driver strand.

### [LOW] D6 and D7 disagree in detail and D6's bound is partial

**Location:** `decision-record.md` D6 (lines 70-76), D7 (line 88), Scope Out (line 24), Constraints (line 107).
**Issue:**
- "A halted child (blocked, paused, failed) or an awaiting one whose driver strand is not live" can be read as revived whatever the driver's state for the halted case; the revive condition is the not-live driver for both.
- D6's Bound covers the revive ("at most once per halt episode") and not the re-spawn of a running child, whose only stated limit is a parenthesis ("idempotent against a live driver").
- A failed revive names `lyx loom start --no-attach`, while D7's no-live-driver refusal names `lyx batten run <slug>` first for the same state.
- Scope Out puts "notice wording" in another task, yet D6 specifies what the halt notice names.
- Constraints call D6's notice a refusal needing a spec row and a test; `contracts/specs/refusal-spec.md` binds webster, shed and loom only, and says batten refusals are not in its table, so only D7's refusals carry that duty.
**Fix:**
Reword D6's state list as "a halted or awaiting child whose driver strand is not live".
Extend its Bound to the re-spawn: it only runs for a child with no live driver and no held run lock.
Make the two way-forward lines agree (D7 names `lyx batten run <slug>` for a run batten holds, and `lyx loom start --no-attach` once D6's revive has failed or when batten holds no run).
Change the notice clause to a notice naming the way forward, without a spec row, and scope it to the one clause D6 adds.

### [LOW] Goal and acceptance claims overreach Scope

**Location:** `decision-record.md` Goal (line 6), Open risks (line 128), Acceptance (line 138).
**Issue:**
The Goal says "every Go-side wait after an agent's turn shows" while Scope names the verify gates, the gate entries and the background-shell wait.
It says "reed revives only strands with work left", but D5 relaunches every strand without a done-when list (driver, orch, operator, a pre-change strand) as before.
The elapsed-time acceptance criterion at line 138 has no row for the fallback to a start time that the Open risk at line 128 allows, so the plan cannot satisfy both.
**Fix:**
Goal: "the Go-side waits Scope names show"; "reed never revives a shuttle strand whose run is finished".
Add to the line-138 criterion that on a tmux whose format does not expand the arithmetic, the segment shows the start time instead.

### [LOW] Incident history and provenance belong in the support log

**Location:** `decision-record.md` Scope (line 13), D1 Rationale (line 36), D2 Rationale (line 45), D3 Rationale (line 53), D6 Rationale (line 76), D7 Rationale (line 93), Auto-mode assumptions (line 119).
**Issue:**
Completeness-before-leanness check: the requirement each carries is already in a Decision.
- D1's Decision says the background-shell wait gets a marker, so the `bugfix-webster` story (stated again in Scope and in the first Auto-mode assumption) is history; `support-log.md` Interview item 2 and Q1 already hold it.
- "the 8-15 minutes observed under contention" in D3 and the "2026-10-07" incident in D7's Rationale are measurements and incident narrative, and the dated operator attributions in D2 and D6 are provenance.
  D2's Decision already says shuttle sets the mark; D6's already says the halt stays the child's.
Nothing needs extracting first.
**Fix:**
Cut those clauses from `decision-record.md`.
Add the observed-duration range to `support-log.md`'s Q3 and the dated operator attributions to the relevant ledger answers.
The D8 package list is a perishable enumeration, so reduce it to "each touched package's `doc.go`".
