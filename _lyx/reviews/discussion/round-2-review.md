---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: A covered run held in the parent lock queue keeps its ticket, and the queue grants only the oldest live ticket, so under a per-run entry every uncovered run queued behind it also waits until the resume, which no Decision states or bounds
    fix: In D4, after the sentence that a covered run is never granted the parent lock while covered, state what happens to its ticket and to the runs queued behind it. Either the covered waiter yields its turn (the grant passes to the next live ticket while it stays queued) so an uncovered run behind it is not held, or the hold is accepted and stated as part of D4's bound. In D5's parent-lock bullet and D3 step 4, make the pause report and lyx pause --status also name each covered run waiting in a parent lock's queue and any runs queued behind it. Add an Acceptance criterion under Holds and readers pinning the chosen behaviour for an uncovered run queued behind a covered waiter under a per-run entry.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:176
    summary: D10 ties a shuttle run to the killed step by its record naming the step's run directory, but run.json carries no such field and shuttle runs live under the worktree's shared .lyx/shuttle root, so the bound rests on a false premise
    fix: In D10, name the tie the stop uses and, if it is new, the field that carries it. Either state that the shuttle run record gains a field naming the shed run and step that started it (written at Start, read by the arming module), or tie the run by the existing RunState PID, the pid of the process waiting on the run, matched against the killed step's pid. Keep the stated never-stop set (loom driver's strand, the orch, other steps' runs) and say why the chosen tie excludes the driver's strand. Adjust the last Sleep-and-#567 Acceptance criterion to name the tie.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:140
    summary: D7 lets the watcher interrupt a busy orch as soon as no network operation marker of the orch's is open, calling it D3 step 2's rule, but D3 step 2 ends the grace early only when sessions also read idle, so a busy orch running a local command gets no grace at all
    fix: In D7, replace the clause or earlier once no network operation marker of the orch's is open, the rule D3 step 2 uses with the rule D3 step 2 states, that is the grace ends early only once the orch reads idle and no network operation marker of the orch's is open, which leaves nothing to interrupt; otherwise the watcher waits for the recorded grace end. Rewrite the matching clause of the orch Acceptance criterion under Holds and readers the same way.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:39
    summary: The agent-caller refusal names asking the orch or the operator as its way forward, but a spawned agent asks its parent, never the operator, which is what PATTERN-parent-directive and the parent directive every role renders say
    fix: In D1, change the refusal's way forward from asking the orch or the operator to asking its parent session, which runs lyx pause or lyx resume for it. Make the same change in the Caller Acceptance criterion and in the Constraints entry listing the refusal row.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:39
    summary: The agent-caller bound rests on LYX_STRAND_NAME, but D1 never says so or what stops an agent from running the verb without it and being read as EXTERNAL with hub-wide rights
    fix: In D1, after the agent-caller sentence, add a bound sentence stating that caller identity is read from the strand-name environment variable, that the agents' settings deny commands naming it, and that the check guards against a mistaken call rather than a determined agent.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:101
    summary: Under a hub entry every fabricengine network call and githubclient call waits with no deadline, so an operator's own lyx board write or lyx selfreport create hangs silently until lyx resume, which D4 neither states nor makes visible
    fix: In D4, add that a call held at entry prints or logs one line naming the pause it waits on and lyx resume as the way forward, once when its wait starts, and add the case to an Acceptance criterion under Holds and readers.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:233
    summary: The Boundary pause criterion says --after works as before, but D8 makes --after naming a lock-taking row also end that row's parent lock wait mid-row, so a pre-set --after Merge-In no longer means halt after Merge-In finishes
    fix: In the first Boundary pause Acceptance criterion, change --after work as before to --after works as before except that, naming a row waiting for the parent lock, it ends that wait with the halt of D8, and add that sentence to D8's landingshed bullet so the changed meaning is stated where it is decided.
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:54
    summary: The support log's turn-10 line says D1 states any caller may run the verbs and the audience annotation is the only gate, which contradicts D1's current agent-caller refusal
    fix: In support-log.md's Interview turn 10, rewrite the scenarios LOW who may call line to say D1 bounds an agent caller to its own pair's slug and refuses the rest, and drop the audience-annotation clause.
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:74
    summary: The Review rounds section keeps the _No rounds yet._ seed line above the Parent review round 1 entry
    fix: Delete the line _No rounds yet._ from support-log.md's Review rounds section.
---

### [MEDIUM] A covered waiter blocks the parent lock queue for uncovered runs

**Location:** `_lyx/discussion/decision-record.md:97`

**Issue:** `internal/parentlock/doc.go` says a free record goes to the oldest live ticket, and an acquirer not first in the queue keeps its ticket and is not granted.
D4 says a covered run is never granted the lock while covered and its wait keeps waiting until the resume.
Under a per-run entry, a covered run at the head of the queue therefore holds back every uncovered run behind it until `lyx resume`, though only the covered slug was paused.
D5 states what runs behind a covered holder do, but nothing states what runs behind a covered waiter do, and neither the report nor `--status` names a covered waiter.
This is the focus directive's covered-waiter question, and it is unsettled.

### [MEDIUM] D10's tie between a shuttle run and the killed step names a field the record lacks

**Location:** `_lyx/discussion/decision-record.md:176`

**Issue:** `RunState` in `internal/shuttleengine/rundir.go` holds the run and strand ids, session, output files, prompt, settings and events paths, outcome, strand name and `PID`, and no shed run or step directory.
Shuttle run directories sit under the worktree's `.lyx/shuttle/<run-id>`, not under the shed run's steps directory, so a record does not name the killed step's run directory.
The bound "only shuttle runs whose record names the killed step's run directory" therefore selects nothing today, and the Decision does not say a field is added.
The never-stop set itself is stated and sound.

### [MEDIUM] D7 drops the grace for a busy orch

**Location:** `_lyx/discussion/decision-record.md:140`

**Issue:** D3 step 2 waits until every in-flight network operation has finished and every covered session reads idle, or the grace has passed.
D7 lets the watcher act once no network marker of the orch's is open, attributing that to D3 step 2.
For a busy orch running a local command, such as a commit or a gate run, no marker is open, so it gets an Esc at the watcher's next tick, without the grace whose rationale in D3 is to let a running Bash command end cleanly.
The report's orch case and the resume line's ownership condition are stated and covered by the criteria.

### [MEDIUM] The agent-caller refusal sends an agent to the operator

**Location:** `_lyx/discussion/decision-record.md:39`

**Issue:** `PATTERN-parent-directive` says the agent asks its parent, and every role's parent directive says a question to the operator is never the way forward.
D1's refusal names "asking the orch or the operator", so the way forward an agent reads contradicts the directive in its own prompt.
The pattern's letter covers stencils, so this is not filed as a constraint violation, but the refusal text reaches the same agent.
Apart from this, D1 now states a real caller bound, its refusal and a matching criterion, which settles the focus directive's D1 question.

### [LOW] The caller bound does not state what it rests on

**Location:** `_lyx/discussion/decision-record.md:39`

**Issue:** The rubric's attack-surface test asks the Decision to state what bounds a new verb.
D1 tells an agent apart from an `EXTERNAL` caller by its strand-name variable, and an agent invoking the verb without that variable reads as `EXTERNAL` and may pause or resume the whole hub.
D1 does not say that the agents' settings deny naming the variable, nor that the check is a guard against a mistaken call rather than a determined one.

### [LOW] Operator commands held by a hub pause wait silently

**Location:** `_lyx/discussion/decision-record.md:101`

**Issue:** D4 holds every `internal/githubclient` call and every network git call in `internal/fabricengine` under a hub entry, with no deadline.
Board writes and `lyx selfreport create` go through them, so an operator command run during a hub pause hangs with no output until `lyx resume`.
D4's bound covers runs waiting, and `--status` names what waits, but nothing tells the operator at the hung command why it waits.

### [LOW] The boundary-pause criterion says --after is unchanged

**Location:** `_lyx/discussion/decision-record.md:233`

**Issue:** D8 makes landingshed's `PauseRequested` seam read `pause_after` naming the waiting row, so `--after <row>` for a lock-taking row now also ends that row's lock wait mid-row.
The criterion "`--before`, `--after` and `--clear` work as before" contradicts that.
The way forward itself works: the halt is a Halt Stuck that leaves the run blocked at its row, and `Goto` in `internal/shedengine/goto.go` clears `pause_after` with the rest, so `lyx loom goto` moves the run and a goto back to the same row does not halt again.
That settles the focus directive's D8 question as not a defect beyond this wording.

### [LOW] The support log contradicts D1's caller rule

**Location:** `_lyx/discussion/support-log.md:54`

**Issue:** Turn 10 records "fixed: D1 states any caller, and the audience annotation is the only gate", but D1 now refuses an agent's hub-wide call and another pair's slug.

### [NIT] Stale seed line in Review rounds

**Location:** `_lyx/discussion/support-log.md:74`

**Issue:** `_No rounds yet._` is the seed line for an empty section and stays above a recorded round.
