---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 (lines 137, 142-143), D6 (line 150), Acceptance criteria (lines 236, 238)
    summary: Every way to go on without regression is a forward goto (lyx loom goto Describe, darn goto Code-Bouncer) that shedengine.CheckGotoTarget refuses, so a Drift-Verify halt on no passed capture can never be lifted
    fix: >-
      In D6, replace the way forward of the no-passed-capture halt with a new verb, for example `lyx drift waive --reason <text>`.
      It refuses unless the run is halted at `Drift-Verify`, writes a waived mark with its reason into the scope record, and on the next `lyx loom resume` makes `Drift-Verify` return Done without a compare.
      State its bound in D6: it lets the whole regression question through, only from a halted `Drift-Verify`, never from the row's live agent session, and under scope `full` only at the decision halt the parent lifts.
      Name `lyx drift waive` followed by `lyx loom resume` instead of `lyx loom goto Describe` / `Code-Bouncer` in D5 (deferral paragraph and both adopt sentences), D6 first paragraph, and the defer and adopt acceptance criteria.
      Add an acceptance criterion that the waive refuses outside a halted `Drift-Verify` and that resume after it reaches Done.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4 (lines 104-107), Constraints (line 214)
    summary: On Windows the detached capture dies when the Drift-Baseline step exits, since the step runs in a StartTied kill-on-close job and proc.Detach does not break away, and runproc.End signals nothing there, so teardown never ends a capture a caller started
    fix: >-
      In D4, add a paragraph on the capture's Windows lifetime.
      The row and `lyx drift capture` start the capture child with `proc.DetachBreakaway`, falling back as `spawnLoop` does when breakaway is refused, so it outlives the step's tied job.
      The capture holds a named kill-on-close job of its own through `proc.HoldNamedJob`, named from the worktree as `shedrun.LoopJobName` names a loop's job.
      `pairteardown`, and the adopt's end of the old pair, terminate that job through `proc.TerminateNamedJob`, as they end a loop on Windows.
      Rewrite the Constraint on line 214 to say `runproc.EndSparing` ends the capture on Linux and the named job ends it on Windows.
      Add an acceptance criterion that a capture started by the row outlives the row's step.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 (line 130), D3 (line 92), D6 (lines 151, 165), Acceptance criteria (line 231)
    summary: The loop's step idle watchdog (step_idle_timeout_min, 60 by default) kills a step with no agent, verify or parent-lock activity, so Drift-Await's wait, darn's foreground capture and Drift-Verify's mechanical compares are killed after an hour, contradicting D5's no bound of its own
    fix: >-
      In D4, add that while the capture lock is held, and while a Drift row's own mechanical compare runs, the run's activity reading counts as activity now.
      Do this in `agentActivityFor` in `internal/loomcli/looparm.go`, as a live parent-lock wait already does, with a compare marker under `.lyx/drift/` named through `internal/lyxdirs`.
      In D5's waiting bullet, name the watchdog and say this activity keeps it off.
      Add an acceptance criterion that a capture longer than `step_idle_timeout_min` is waited out without a watchdog kill.
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 (lines 68-69), D6 (lines 166, 170), D7 (lines 179-186)
    summary: Under scope full the arbiter may run lyx drift accept at a Drift-Verify halt that carries no DecisionHaltPrefix, such as the dirty-tree or test-gate Stuck, and resume to Done, so drift is accepted before the parent rules
    fix: >-
      In D6, widen the scope-`full` rule on line 170 so that every `Drift-Verify` halt under scope `full` has a reason opening with `shedadapters.DecisionHaltPrefix`, the dirty-tree and test-gate Stucks included.
      Say that an error or transient stop of the row is the only exception, and that it is resumed, never settled with an acceptance.
      In D7, add that `lyx drift accept` under scope `full` refuses unless the run is halted at a decision halt, naming the parent's ruling as the way forward.
      Replace "the arbiter under scope `partial`" with that stated mechanism.
      Add an acceptance criterion that `lyx drift accept` under scope `full` refuses at a non-decision halt.
  - id: F5
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 (lines 72-73), Acceptance criteria (line 229); _lyx/discussion/support-log.md Rejected alternatives (line 317), Question ledger Q6 (line 341)
    summary: D2 and the first acceptance criterion still keep Drift-Await ungated and recording Done with the flag off, against the parent's Added after Discussion ruling that it carries entry_gate drift.enabled and records Skipped
    fix: >-
      In D2, replace the two sentences on lines 72-73 with: `Drift-Await` carries `entry_gate: drift.enabled` like the other Drift rows, per the parent's ruling, so with the flag off it records Skipped and runs nothing.
      In the first acceptance criterion, say a loom and a darn run record every Drift row they carry as `Skipped`.
      In the support log, drop the Rejected alternatives line "`Drift-Await` behind `entry_gate: drift.enabled`" and mark Q6 as superseded by the parent's ruling.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 (line 125), D2 (line 72)
    summary: With Drift-Await gated by the parent's ruling, turning the flag off while a capture runs lets Webster build beside it in the same worktree, and the record states no bound for that case
    fix: >-
      In D5, add one sentence that states the outcome.
      Recommended: `Drift-Verify` and `Drift-Await` both skip with the flag off, so a capture left running is ignored, never compared against, and ends at its own timeout or with the run's processes.
      Turning the flag off mid-run is the operator's choice to give up regression for that run.
      Add an acceptance criterion that turning the flag off with a capture running skips `Drift-Await` and `Drift-Verify`.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 (lines 181-185)
    summary: Under scope partial the arbiter is a caller at a non-decision halt, so it can re-accept a test the parent withdrew and resume, undoing the parent's rejection
    fix: >-
      In D7, say that `lyx drift accept` of a test on the scope record's withdrawn list refuses unless the run is halted at a decision halt.
      Say that `Drift-Verify`'s Stuck on a failing final compare whose failing tests include a withdrawn one has a reason opening with `shedadapters.DecisionHaltPrefix`, so only the parent re-accepts it.
      Add both to the accept/withdraw acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D6 (line 160)
    summary: The claim that a further attempt takes a resume by the parent is false under scope partial, since the arbiter may resume a Drift-Verify halt and each resume respawns the agent with a reset budget
    fix: >-
      In D6, replace "and a further attempt takes a resume by the parent" with a statement of the real bound.
      Each resume, the arbiter's included, gives a fresh agent `verify_max_runs` runs, and the number of attempts is bounded by the shed stop chain's own cap on arbiter resumes of one halt.
      Name that cap from `internal/sheddriver` or the arbiter config.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 (lines 91-97), D5 (line 136)
    summary: Darn's Drift-Baseline is Stuck on a failed capture with no named way to go on without regression, since lyx drift defer refuses outside Drift-Await, which darn lacks
    fix: >-
      In D3, give darn's failed-capture Stuck its way forward.
      It names `lyx drift capture` followed by `lyx loom resume` to capture again, or `lyx drift defer` followed by `lyx loom resume` to go on with regression deferred.
      Widen `lyx drift defer` in D5 to refuse unless the run is halted at `Drift-Await`, or at darn's `Drift-Baseline` on a failed capture, and say `Drift-Baseline` then returns Done on resume.
      Add both refusals and the darn resume to the acceptance criteria.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D3 (lines 95-98)
    summary: D3's re-entry rule leaves three cases unstated, an absent capture record, a re-entry off the branch point where D4 refuses the capture, and a scope record present with baseline.md missing
    fix: >-
      In D3's re-entry paragraph, state the three cases.
      An absent capture record counts as a capture that did not pass.
      A capture the row would start again but that D4 refuses, because HEAD left the branch point or the tree is dirty, ends the row Done when `awaited_at` is already set and Stuck naming `lyx drift defer` otherwise.
      A present scope record skips the agent only when `baseline.md` exists as well, and otherwise the agent is spawned to write `baseline.md` alone.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Constraints (line 212)
    summary: The claim that no row builds in the task worktree between Drift-Baseline and Drift-Await is false, since the Plan perch's Burler reviewers run with tool-use and are told to build and test what they review
    fix: >-
      Rewrite the Constraint on line 212 to state what holds.
      No row's own Go code builds in the task worktree there, and a Plan reviewer's build or `lyx gate test` runs beside the capture within the hub's slot count.
      Move the shared `bin`/`obj` risk of a .NET hub into Open risks as a fact the sandbox hub proves, naming board note `csharp-sandbox-hub`.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D2 (line 66)
    summary: D2 says Finalize's merge and merge fixer go uncompared, but in loom a Finalize reroute runs Parent-Align, the Code perch and Drift-Verify again, so a rerouted merge is compared
    fix: >-
      In D2 line 66, say a Finalize landing's merge and merge fixer go uncompared, while a loom Finalize reroute re-enters `Drift-Verify` through `Parent-Align` and the Code perch and compares again, keeping the scope record's acceptances.
      Keep the darn reroute to `Code-Bouncer` as uncompared.
  - id: F13
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 (line 119), D5 (lines 132-136)
    summary: D4 says Drift-Await commits the loom capture log, but D5 commits it only on a passed-await Done, leaving the logs a Stuck halt names or a deferral passes uncommitted
    fix: >-
      In D4 line 119, say `Drift-Await` commits the capture log on every way out it takes, Done, Stuck and a deferred Done alike, and `Drift-Verify` commits any capture log still uncommitted.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:goal-scope
    location: _lyx/discussion/decision-record.md Open risks (lines 222-223); _lyx/discussion/support-log.md Q1 (line 336)
    summary: The first Open risks bullet asks the parent where Drift-Verify sits against Parent-Align, which the Added after Discussion ruling on Drift-Verify's position already answers
    fix: >-
      Drop the first Open risks bullet.
      In the support log's Q1, replace "raised to the parent under Open risks" with "settled by the parent's ruling recorded in the decision record".
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md Notes for the plan writer (line 253)
    summary: The note that darn's recipe build must fill Env.SettingKnown rests on a false premise, since darn is built by loomrecipe.NewDarn from the same env loomcli's wiring already fills
    fix: >-
      Replace the note's second clause with: darn's build at `internal/loomcli/arm.go` uses the same env, so it is already filled.
      Or drop the note.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/support-log.md Interview turn 3 (lines 281, 291), Review rounds (line 332), Question ledger Q12 (lines 348-349)
    summary: The support log keeps superseded round-0 fixes as current, says no review rounds ran while the ledger cites round 1, and Q12 says no runproc change is needed although D4 adds a runproc role
    fix: >-
      In Interview turn 3, append to the lines for interactions 4 and for bound-breaker 3 "superseded in review round 1, see Q12" and "superseded in review round 1, see D5".
      Replace "_No rounds yet._" under Review rounds with a one-line entry for round 1 naming Q11 to Q15.
      In Q12, replace "so no `runproc` or PATTERN change is needed" with "so no PATTERN change is needed; `runproc` gains only the `capture` role".
---

The record's goal, scope and module layout fit the board entry and the operator's decisions, and its D4 timeout rule matches the gate's own `runTiedChild` on Linux.
Its process, halt-route and drift-bound mechanics break on existing code in four places.
Each of those is a BLOCKING design finding, and the parent's later ruling on `Drift-Await` is not yet carried into the body.

### [BLOCKING] Going on without regression is a refused forward goto

**Location:** `_lyx/discussion/decision-record.md` D5, D6 and the defer and adopt acceptance criteria.

**Issue:** The no-passed-capture halt of `Drift-Verify` names `lyx loom goto Describe`, and in darn `Code-Bouncer`.
Both targets lie after `Drift-Verify` in their recipe.
`shedengine.CheckGotoTarget` (`internal/shedengine/goto.go:149-164`) refuses a target "past the run's current row", and `lyx loom goto --help` says goto "moves a halted run only back".
`lyx loom resume` re-enters the same halt, since there is still no passed capture.
So every deferral past a failed capture, every adopt once `awaited_at` is set, and every scope-`full` missing-baseline halt strands the run for the arbiter and the parent alike.
This also breaks the Constraint that every halt names its way forward.

### [BLOCKING] The capture does not survive its step on Windows, and teardown does not reach it there

**Location:** `_lyx/discussion/decision-record.md` D4 and Constraints.

**Issue:** The loop starts each step through `proc.StartTied` (`internal/shedverbs/loopchild.go:16,34`).
On Windows that puts the step in an unnamed kill-on-close job, whose handle `releasePlatform` closes once the step exits (`internal/proc/tied_windows.go`).
`proc.Detach` sets no `CREATE_BREAKAWAY_FROM_JOB` (`internal/proc/proc_windows.go:31-35`), so the capture `Drift-Baseline` detaches stays in that job and dies as the row returns Done.
`Drift-Await` then always reads a `running` record with a free lock and halts.
In the other direction, a capture `lyx drift capture` starts from a caller's shell is in no lyx job, and `runproc.End` signals nothing on Windows (`internal/runproc/doc.go`).
So `pairteardown` and adopt never end it, against D4's "ends with the run's processes" and the Constraint naming `runproc.EndSparing`.
The design's first real user, Models on OSL-1046, is a .NET project.
The loop already solves both halves for itself, with a breakaway spawn (`internal/shedverbs/loopwait.go:158-161`) and a named job that `pairteardown` terminates (`internal/pairteardown/loop_windows.go`).

### [BLOCKING] The step idle watchdog kills Drift-Await's wait and the long LLM-free runs

**Location:** `_lyx/discussion/decision-record.md` D5, D3, D6 and the loom acceptance criterion on waiting.

**Issue:** The loop kills a `--until-stop` step that shows no activity for `step_idle_timeout_min`, 60 by default (`internal/loomengine/template.yaml:44`, `internal/shedverbs/loopwatchdog.go:170-260`).
`agentActivityFor` (`internal/loomcli/looparm.go:129-170`) counts only a live verify marker, a parent-lock wait, an unheld shuttle wait marker and live agent runs.
None of these is present while `Drift-Await` waits on a capture, while darn's `Drift-Baseline` runs its foreground capture after the scope agent ended, or while `Drift-Verify` runs its mechanical compares.
A 90-minute capture gets `Drift-Await` killed at about an hour as an interrupted stop.
In darn the kill takes the tied capture command down with the step.
D5's "with no bound of its own" and the matching acceptance criterion do not hold.

### [BLOCKING] The arbiter can accept drift under scope full

**Location:** `_lyx/discussion/decision-record.md` D2, D6 and D7.

**Issue:** D2 promises that no drift in a scope-`full` task is accepted before the parent rules.
Under `full`, D6 gives `DecisionHaltPrefix` only to halts "over drift or a missing baseline".
The dirty-tree and test-gate Stucks after three re-prompts carry no prefix, so `arbiterstop.Eligibility` admits them (`internal/arbiterstop/arbiterstop.go:66-70,164-165`).
D7's verbs check only that the current row is `Drift-Verify`, and the arbiter is a caller at that halt.
So the arbiter can run `lyx drift accept <test>` and resume, and the mechanical compare passes with the test skipped.
The row returns Done without the parent.
D7's "the arbiter under scope `partial`" names a bound that no stated mechanism enforces.

### [BLOCKING] D2 and the acceptance criteria contradict the parent's Drift-Await gating ruling

**Location:** `_lyx/discussion/decision-record.md` D2 and the first acceptance criterion; `_lyx/discussion/support-log.md` Rejected alternatives and Q6.

**Issue:** The parent's `Added after Discussion` entry gives `Drift-Await` `entry_gate: drift.enabled`, so it records Skipped with the flag off.
D2 still says it "carries no entry gate" and shows Done with the flag off.
The first acceptance criterion pins "a loom run records `Drift-Await` as Done".
The plan writer would get two opposite instructions, and a test written from the criterion would assert the overruled behaviour.

### [MEDIUM] Turning the flag off mid-capture is unbounded under the gating ruling

**Location:** `_lyx/discussion/decision-record.md` D5 and D2.

**Issue:** D2's own reason for leaving `Drift-Await` ungated was that a capture running when the flag is turned off must hold Webster back.
Under the parent's ruling `Drift-Await` skips, so Webster builds beside the running capture in the same worktree.
The record says nothing about what happens then, or about whether `Drift-Verify` later compares against that baseline.

### [MEDIUM] The parent's withdrawal does not hold against the arbiter

**Location:** `_lyx/discussion/decision-record.md` D7.

**Issue:** Under scope `partial` the parent withdraws test T and resumes, and the final compare fails on T.
The row is then Stuck naming `lyx drift accept`, which is not a decision halt.
The arbiter, as a caller at that halt, accepts T again and resumes.
D7's "only a caller at a halt accepts it again" does not keep the rejection with the parent.

### [MEDIUM] The verify budget is not bounded by the parent

**Location:** `_lyx/discussion/decision-record.md` D6.

**Issue:** D6 says a further attempt "takes a resume by the parent".
Under scope `partial` the arbiter may resume a `Drift-Verify` halt, and each resume respawns the agent and resets its counter.
So the stated bound on the agent's runs is wrong as written.

### [MEDIUM] Darn's failed-capture halt cannot go on without regression

**Location:** `_lyx/discussion/decision-record.md` D3 and D5.

**Issue:** In darn, `Drift-Baseline` is Stuck on a failed capture, and D3 names no way forward.
`lyx drift defer` refuses unless the run is halted at `Drift-Await`, which darn does not have.
A darn task whose harness keeps failing can only recapture.

### [MEDIUM] Drift-Baseline re-entry leaves three cases unstated

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** The first case is an absent capture record, which an adopting machine has because `.lyx/` is never carried; D5's darn adopt path relies on D3 capturing again there.
The second is a re-entry after code changed, such as an operator `goto Discussion-Write`; the row then tries a main capture that D4 refuses, and its outcome is unstated.
The third is an agent that died after `lyx drift scope` and before writing `baseline.md`; it is skipped on re-entry, so `Drift-Verify` is handed no expectation.

### [MEDIUM] Plan reviewers build in the worktree during the capture

**Location:** `_lyx/discussion/decision-record.md` Constraints.

**Issue:** The Constraint says Discussion and Plan rows write only records.
But `Plan-Burler-Review` runs with `tool-use: true` (`contracts/recipes/loom-recipe.yaml`), and its reviewers are told to build, run and test what they review.
In a .NET hub such a build shares `bin` and `obj` with the capture's `dotnet test` in the same worktree.

### [LOW] D2 misstates Finalize's reroute in loom

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** Loom's Finalize reroutes to `Parent-Align`, whose `on_done` is `Code-Bouncer`, which now leads to `Drift-Verify`.
So a rerouted merge is compared, and only a landing's merge goes uncompared.

### [LOW] The capture-log commit rule misses most Drift-Await outcomes

**Location:** `_lyx/discussion/decision-record.md` D4 and D5.

**Issue:** D4 has `Drift-Await` commit the loom capture log, but D5 commits it only on a passed-await Done.
The logs a Stuck halt names, and those a deferral passes, stay uncommitted.

### [LOW] Open risks raises a question the parent already answered

**Location:** `_lyx/discussion/decision-record.md` Open risks, first bullet.

**Issue:** The parent's ruling on `Drift-Verify`'s position settles the bullet's question, so per the rubric the line is dropped.

### [LOW] A plan-writer note rests on a false premise

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer.

**Issue:** `internal/loomcli/arm.go:248` builds darn with `loomrecipe.NewDarn(c.env, ...)`.
The same env is filled with `SettingKnown` at `internal/loomcli/wiring.go:765`.
So the note's premise, that darn's build must fill it too, is false.

### [LOW] The support log is stale against round 1

**Location:** `_lyx/discussion/support-log.md` Interview turn 3, Review rounds and Q12.

**Issue:** Turn 3 records the fixed await bound and the runproc child entry as current, though round 1 replaced both.
Review rounds reads "_No rounds yet._" while the ledger cites round 1.
Q12 says no `runproc` change is needed, though D4 adds a `capture` role to `runproc`'s closed role list.

## Rejected

- Board-fit's MEDIUM `design` copy of the gating contradiction is merged into F5 at BLOCKING `consistency`, because the body contradicts a parent ruling rather than resting on a false premise.
- Goal-scope's MEDIUM severity for the Open risks line is kept at LOW (F14), since dropping a settled line changes nothing the plan writer acts on.
- Interactions' LOW on an absent capture record and walkthrough's LOW on a lost `baseline.md` are merged into F10 with existing-paths' re-entry finding.
- Bound-breaker's claim 15, that `{baseline_dir}` is honoured only in text, is no finding, since D4's written-files pass rule already covers it.
