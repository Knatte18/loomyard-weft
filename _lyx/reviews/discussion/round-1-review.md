---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 (lines 142-143), D7 (line 151), D2 (line 56), Acceptance criteria (line 194)
    summary: Under scope full the halt's own way forward, accept and resume, either loops forever or lets the arbiter bless drift before the parent rules, because D6, D7 and the acceptance criterion disagree on which acceptances trigger the Stuck.
    fix: In D7, make each accepted entry in the scope record carry who accepted it, the live Drift-Verify agent session or a caller at a Drift-Verify halt. In D6, fire the full-scope Stuck only on acceptances the agent session made in the current attempt, and name the parent's lyx drift accept plus lyx loom resume as the way forward. In D2, state whether the arbiter may run lyx drift accept under scope full; if not, have lyx drift accept refuse under full while the stop is with the arbiter and say so; if so, state that the arbiter's acceptance stands in for the parent's ruling. Rewrite the acceptance criterion to "Under scope full, an acceptance the agent session made ends Drift-Verify Stuck, and the parent's accept and resume then reaches Done on a passing compare".
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 (line 102), Scope In (line 18), Constraints (line 160), PATTERN changes (line 201)
    summary: D4 rests on a false premise. proc.StartTied already ends a timed-out child's whole tree on Linux and Windows, as the gate does, so the new runproc entry and the PATTERN-run-process-owner change are unneeded, and the Windows rule leaves a hung command holding a gate slot and the capture lock with no bound.
    fix: "In D4, end the timed-out command the way the gate's runTiedChild does: start it with internal/proc's StartTied and kill its tree through Tied.Kill on both platforms, then record failed. Cite internal/proc/tied.go StartTied and internal/gatecli/child.go runTiedChild as the evidence. Delete the Windows hold sentence. Delete Scope In's runproc-entry bullet, the ## PATTERN changes section, and the words \"but PATTERN-run-process-owner, changed below\" in the first Constraint. Mark Q12 in the support log's question ledger as superseded."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 (lines 151-153), D2 (line 56)
    summary: Under scope partial, lyx drift accept has no stated bound on what it lets through. The agent or the arbiter can accept every failing test, the compare then runs on the rest or on none, and the row returns Done with no review. The only bound is the agent's judgement, and that is stated only in the support log.
    fix: "Add to D7 a paragraph stating what an acceptance lets through under partial: up to every test the compare runs, including a compare left with no test. Then add a bound the Decision enforces, for example: the row is Stuck for the parent when an accepted test is one baseline.md did not name as expected drift, or when the accepted list plus the skip list covers every test the capture wrote. If the agent's judgement is to stay the only bound, say so in D7 in those words, and also state whether the arbiter may accept under partial (D2's sentence)."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 (line 118), D3 (lines 82-84), D2 (line 53)
    summary: In loom, D5's way forward lyx loom goto Drift-Baseline goes on to Drift-Baseline's on_done, Plan-Write. Plan-Write's fresh spawn archives the approved plan through NewPlanDirRotator and the whole Plan perch reruns, and the record does not state this cost.
    fix: "Choose one and write it into D5 and D3. (a) State that the goto re-runs Plan-Write and the Plan perch from scratch, and name that cost in the halt text. (b) Give a recapture route that returns to Drift-Await: let a main lyx drift capture also run at a Drift-Await halt while HEAD is the branch point (widen D4's row guard to Drift-Baseline or a halted Drift-Await), and name that capture followed by lyx loom resume as the halt's way forward. Update the acceptance criterion on line 188 to match."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 (line 120), D6 (line 127), Constraints (line 161)
    summary: D5 presents an adopt on another machine as passing via awaited_at. But the capture record and baseline live under never-tracked .lyx/drift/, and EndOwnedElsewhere ends a running capture, so after an adopt Drift-Verify always halts and regression is lost. The record says neither.
    fix: "In D5 and D6, state what an adopt does to regression: a capture running at the adopt is ended with the run's processes; the adopting machine has no capture record or baseline; Drift-Await passes on awaited_at; Drift-Verify then halts naming lyx loom goto Describe, since no capture may run on changed code. Add that case to the Acceptance criteria. Drop \"or an adopt on another machine\" from line 120 unless the record also adds a way to carry the baseline."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 (line 101), D5 (line 114)
    summary: The capture times itself on its own monotonic clock, but Drift-Await runs in another process and can only bound timeout_min plus one minute from a wall-clock start time in capture.json. After a machine sleep it halts Stuck on a healthy capture still inside its own timeout.
    fix: "In D5, drop Drift-Await's own time bound on a capture whose lock is held. The capture enforces its own timeout (F2's tree kill) and releases the lock on exit, so Drift-Await waits on the lock alone and reads the record once it is free. Delete the timeout-plus-one-minute clause and keep D4's monotonic-clock sentence as the capture's own rule. Adjust the timed-out-capture case in the acceptance criterion on line 187 to read the failed record."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D6 (line 135), D1 (line 42), Constraints (line 162)
    summary: lyx drift compare and lyx drift capture --side are bounded only by a counter whose reset is unstated, and no Decision restricts them to the Drift-Verify row. Webster's implementer can run them unbudgeted or drain the later budget, and a resume after a past-budget halt gives the new session either no runs or an unbounded series.
    fix: "In D6, state that lyx drift compare and lyx drift capture --side refuse unless the run's current row is Drift-Verify, naming the way forward. State when the Go-written counter resets: for example, at each fresh Drift-Verify agent spawn, so each attempt has verify_max_runs and resumes are bounded by the parent who resumes. Add both to the acceptance criterion on line 195."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 (lines 118-119), D6 (line 127)
    summary: D5 offers lyx loom goto Webster to run without regression. After a failed or missing capture, D6 then halts again at Drift-Verify because the scope has no passed capture record. After a HEAD-moved halt the passed capture still exists, so regression does run.
    fix: "Make D5 and D6 agree. Either reword D5's way forward to say the goto defers the regression question to a second halt at Drift-Verify, which then names lyx loom goto Describe; or have D6 return Done, with the reason no baseline, when the capture record is failed or missing and Drift-Await's halt was passed by that goto, recorded in the scope record. In the HEAD-moved case, replace without regression with the actual outcome: the passed baseline is compared."
  - id: F9
    severity: MEDIUM
    class: decision
    origin: lens:board-fit
    location: _lyx/discussion/decision-record.md Open risks (lines 175-176), D2 (lines 53, 63)
    summary: Question for the run's parent. Commit fc75aef79 makes the entry's placement of Drift-Verify, after the Code perch and before Describe and Parent-Align, impossible, because Parent-Align now runs before the Code perch. Should Drift-Verify sit after Code-Bouncer, comparing merged parent commits, or between Webster and Parent-Align, leaving the Code perch's fixes uncompared?
    fix: The parent rules on the placement. The record then keeps or moves Drift-Verify in D2, drops the first Open risks bullet, and adjusts D7's parent-commit acceptance sentence to match.
  - id: F10
    severity: LOW
    class: decision
    origin: lens:goal-scope
    location: _lyx/discussion/decision-record.md D2 (lines 58-60)
    summary: Question for the run's parent. The operator decision of 2026-10-25 says that with the flag off the Drift rows are skipped like the PR rows. D2 keeps Drift-Await ungated, showing Done, so Webster stays held behind a capture already running. Does the operator accept that departure?
    fix: "The parent rules. If accepted, D2 cites the ruling; if not, Drift-Await gets entry_gate: drift.enabled, and D2 states what a flag turned off mid-capture then lets through: Webster building beside a running capture."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 (line 95), Constraints (line 165)
    summary: D4 claims the capture holds no lock an agent's own work waits on, but the plan gate's type load is a non-tier gate-slot holder. On a hub with few slots, Plan-Write's gate queues behind the capture.
    fix: "Replace \"and holds no lock an agent's own work waits on\" in D4 with \"so the plan gate's type load, also a slot holder, may queue behind it\"."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D6 (line 140), D2 (line 54)
    summary: D6 says no code change goes uncompared. But loom Finalize's merge and merge fixer run after Drift-Verify, and darn Finalize's reroute to Code-Bouncer goes back past darn's Drift-Verify. D2 names only darn's perch fixes as uncompared.
    fix: In D6, scope the claim to the row's own attempts (no fix the agent or a gate re-prompt makes goes uncompared). In D2, add that Finalize's merge and merge-fixer changes, and in darn a Finalize reroute's changes, go uncompared.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:board-fit
    location: _lyx/discussion/decision-record.md Scope Out (line 25), Notes for the plan writer (line 207)
    summary: Scope Out puts the end-to-end run on OSL-1046. The operator's 2026-10-25 line on board note csharp-sandbox-hub says the Drift rows are proved on the sandbox hub with its own harness, with Models only the final confirmation.
    fix: In Scope Out, say the sandbox hub's capture and compare harness belongs to note csharp-sandbox-hub, and the Models run on OSL-1046 is the final confirmation. Reword the sandbox plan-writer note to match.
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria (line 182)
    summary: The first acceptance criterion has a darn run record Drift-Await as Done, but darn has no Drift-Await row.
    fix: Rewrite as "a loom and a darn run record Drift-Baseline and Drift-Verify as Skipped, a loom run records Drift-Await as Done, and neither spawns an agent or runs a command".
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 (line 93)
    summary: D4 presents the .git/info/exclude entry for .lyx/ as new work, but WireJunctionsWith already writes it at clone.
    fix: "Reword line 93 as an existing fact: \"The task worktree's .lyx/ is already excluded through .git/info/exclude at clone, so the baseline directory is never dirt.\""
---

### [BLOCKING] Scope full: accept-and-resume loops or bypasses the parent

**Location:** `decision-record.md` D6 lines 142-143, D7 line 151, D2 line 56, Acceptance criteria line 194.
**Issue:** D6 makes the halt Stuck when "a session that accepted any drift" and names "`lyx drift accept` and resume" as the way forward. The acceptance criterion says "any acceptance ends `Drift-Verify` Stuck", and D7's accepted list records no accepter.
If the rule reads the list, every resume halts again.
If it counts only the session's acceptances, the arbiter, which D2 lets accept, settles a `full` drift before the parent sees it.
Four lenses found this independently.

### [BLOCKING] D4's runproc entry and Windows hold rest on a false premise

**Location:** `decision-record.md` D4 line 102, Scope In line 18, Constraints line 160, PATTERN changes line 201.
**Issue:** `internal/proc/tied.go` `StartTied` installs a cancel that kills the child's whole tree, through a process group on Linux and a kill-on-close job object on Windows.
`internal/gatecli/child.go` `runTiedChild` uses it on a gate timeout.
The capture's command is an unregistered child of the capture process, so ending it this way touches no runproc record, and the PATTERN change is unneeded.
On Windows, the stated rule keeps the lock and a hub gate slot until a hung command exits, which nothing bounds.

### [MEDIUM] Acceptance under scope partial is unbounded

**Location:** `decision-record.md` D7 lines 151-153, D2 line 56.
**Issue:** This is the rubric's attack-surface test.
D7 bounds who may accept, through the row guard, but not what an acceptance may let through.
The bound the writer chose, the agent's judgement, sits only in `support-log.md` (bound-breaker 2), which Plan-Write never reads.

### [MEDIUM] goto Drift-Baseline re-plans in loom

**Location:** `decision-record.md` D5 line 118, D3 lines 82-84, D2 line 53.
**Issue:** `Drift-Baseline` → `Plan-Write` in the loom wiring.
`internal/loomshed/planwrite.go` `NewPlanDirRotator` archives the plan directory on Plan-Write's fresh spawn, so the recapture route discards the approved plan and reruns the Plan perch.

### [MEDIUM] Adopt loses the baseline

**Location:** `decision-record.md` D5 line 120, D6 line 127, Constraints line 161.
**Issue:** `.lyx` resolves into the weft's never-tracked `.lyx`, and `internal/tendercli/deps.go` `EndOwnedElsewhere` ends the run's registered processes when another machine owns the run.
After an adopt, `Drift-Verify` finds no passed capture record.

### [MEDIUM] Drift-Await's bound runs on a different clock from the capture's

**Location:** `decision-record.md` D4 line 101, D5 line 114.
**Issue:** A start time read from `capture.json` in another process is wall time.
After a sleep, wall time has run past the bound while the capture's monotonic timer has not.

### [MEDIUM] compare and side-capture verbs have no row guard and no stated counter reset

**Location:** `decision-record.md` D6 line 135, D1 line 42, Constraints line 162.
**Issue:** `scope` and `accept` carry row guards, but `compare` and `capture --side` carry none, and they stay allowed to every agent.
"One agent session" names no reset point across halts and resumes.

### [MEDIUM] goto Webster does not run without regression

**Location:** `decision-record.md` D5 lines 118-119, D6 line 127.
**Issue:** The scope record still says `partial` or `full` with no passed capture, so D6 halts a second time.
In the HEAD-moved case the passed capture is compared.

### [MEDIUM] Drift-Verify placement against Parent-Align (parent question)

**Location:** `decision-record.md` Open risks lines 175-176, D2 lines 53 and 63.
**Issue:** The entry is plainly outdated by landed commit `fc75aef79`, a case the rubric's `decision` class names.
The support log (line 47) kept the question in Open risks, which routes it nowhere.

### [LOW] Drift-Await Done-not-Skipped departs from an operator decision (parent question)

**Location:** `decision-record.md` D2 lines 58-60.
**Issue:** The departure overrides a dated operator decision, so only the parent can settle it, even though the record states and justifies it.

### [LOW] The capture can hold back the plan gate

**Location:** `decision-record.md` D4 line 95, Constraints line 165.
**Issue:** `internal/gateslot/doc.go` lists the plan gate's type load as a non-tier slot holder.

### [LOW] "No code change goes uncompared" is too broad

**Location:** `decision-record.md` D6 line 140, D2 line 54.
**Issue:** Finalize's merge fixer in loom, and darn Finalize's `on_reroute: Code-Bouncer`, change code after `Drift-Verify`.

### [LOW] Where the end-to-end proof happens

**Location:** `decision-record.md` Scope Out line 25, Notes for the plan writer line 207.
**Issue:** Board note `csharp-sandbox-hub` carries an operator line placing the Drift rows' proof on the sandbox hub, which the record's Out bullet does not reflect.

### [NIT] AC1 names a darn Drift-Await

**Location:** `decision-record.md` Acceptance criteria line 182.
**Issue:** Darn has no `Drift-Await` (D2 line 54).

### [NIT] .lyx exclude is existing behaviour

**Location:** `decision-record.md` D4 line 93.
**Issue:** `internal/fabriccli/clone.go` lines 113-116 record that `WireJunctionsWith` already excludes `.lyx`.

## Rejected

- goal-scope finding 2 (one gate slot breaks "never beside other heavy test runs"): the operator's own mechanism is "its commands still take a gate slot", and one slot within the hub's slot count is what a slot means for every tier. Holding every slot through the drain lock would stall all tier runs for a capture's whole life, which the entry never asks for.
- goal-scope finding 3 (the arbiter may accept): this is not a separate defect. It is merged into F1 for scope `full` and into F3 for scope `partial`.
- interactions LOW (Windows lock held after a timeout blocks goto Drift-Baseline): this is subsumed by F2, whose fix ends the tree on Windows and removes the hold.
- interactions LOW and walkthrough finding 5 (counter reset): these are merged into F7.
- walkthrough finding 2's extra claim that Drift-Await re-halts on every Webster reroute: this is merged into F8, and F8's fix covers it.

## Focus verdicts

- Focus 1 (D7 and D6 acceptance bounds): F1 and F3.
- Focus 2 (capture, compare, `--skip`, `drift.enabled`). The executable keys are bounded by D1's layer rule, which holds. The `--skip` list is bounded by the row guard and the scope rules, and since `off` is a legal scope, skipping every test lets nothing through that `off` would not. The flag turned off mid-run is stated in D2 and holds. The compare and side-capture verbs are F7.
- Focus 3 (Models-side facts in Open risks): not a deferred research question. The operator decision of 2026-10-25 places Models on OSL-1046 and off this machine, and Scope Out puts the harness side there, so the writer was told not to gather those facts. The pass rule for a vacuous compare is bounded through F3's fix.
- Focus 4 (cross-check of D2, D3, D5, D6 and the acceptance criteria): F1, F4, F8 and F14.
- Focus 5 (relocation): no relocation finding. The spec allows `Rationale` paragraphs in Decisions. The D4 clock sentence carries a requirement (F6 rewrites it, and F2 removes the Windows detail). The first Constraint's restatement of PATTERN entries is short and names the drift paths the plan needs.
- Focus 6 (board and filing): no Decision, Open risk or Note edits this task's board entry, or files or promotes a task. OSL-1046 is named as where the Models side runs, not filed. The placement question is routed as F9.
