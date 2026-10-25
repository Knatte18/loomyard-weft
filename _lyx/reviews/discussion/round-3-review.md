---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:181-185 and 197-204 (D6, D7)
    summary: "Under scope full, a drift acceptance the agent session made before an error, transient or interrupted stop is never put to the parent, because the resumed row compares first and returns Done"
    fix: "In D6 and D7, make the scope-full bound hold at every entry of Drift-Verify and not only at the end of an attempt. Mark each acceptance made by an agent session `unruled` in the scope record. Under scope full, on entry and before any compare, a record holding an `unruled` acceptance ends the row Stuck in a decision halt (reason opening with `shedadapters.DecisionHaltPrefix`) naming `lyx loom resume` to stand by the acceptances, after any `lyx drift accept` of the parent's own, and `lyx drift withdraw` to reject one. The resume from that decision halt marks the surviving acceptances `ruled`; the stop chain hands a decision halt only to the parent, so that resume is the parent's. Add an acceptance criterion: an agent session that accepts under scope full and then ends in an error, transient or interrupted stop does not reach Done on resume until the parent has ruled."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:62, 145-147 and 247 (D2, D5, acceptance criteria)
    summary: "The record claims a mid-run flag-off records Drift-Await and Drift-Verify Skipped, but an entry gate is evaluated only when a row is entered, so an entered, waiting or halted row runs regardless"
    fix: "In D5's flag-off paragraph (line 145), D2's gate paragraph (line 62) and the second acceptance criterion (line 247), state that the gate is evaluated when the run enters the row (`shedengine.Shed.enter` skips the evaluation when `EnteredRow` names the row; `lyx loom goto` sets `EnteredRow` to its target). A flag turned off while `Drift-Await` waits or is halted does not skip that row, and one turned off while `Drift-Verify` is halted does not skip it on resume; it takes effect at the next entry. Drop the claim that a capture left running is ignored in those cases, and either name the way forward for an operator who gives up regression at an entered row (`lyx drift defer` at `Drift-Await`, `lyx drift waive` at `Drift-Verify`) or state that there is none. Keep the acceptance criterion for the not-yet-entered case."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:146, 159-162 and 247 (D5, D6, acceptance criteria)
    summary: "Loom Drift-Verify halts on no passed capture after a flag off-and-on, naming waive, but waive refuses unless no passed capture record exists and that record exists and says passed"
    fix: "Define once, in D5, the capture `Drift-Verify` counts: a capture record in state `passed` that, in loom, also has `awaited_at` set. Use that single predicate in D6's halt on no passed capture, in `lyx drift waive`'s precondition (line 161), in the waive acceptance criterion (line 257) and in the flag off-and-on criterion (line 247), so the halt that names `lyx drift waive` is the halt waive admits. Alternatively name another way forward for that case and say waive refuses it."
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:111-113 and 121 (D4)
    summary: "The compare marker has no stated liveness test or write point, and the capture-bounded watchdog claim ignores a capture waiting for a gate slot, whose timeout has not started"
    fix: "In D4's activity paragraph, state that the compare marker is backed by an OS lock taken before the compare's slot wait, so a marker left by a dead compare reads as none and a compare queued for a slot reads as activity, as the capture's lock does. Reword the sentence that bounds a capture-held watchdog hold `at most until the capture's own timeout` to cover the slot wait: the timeout counts from the command's start (line 121), so a capture queued for a slot keeps every watchdog off until it gets one."
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:105-109 and 149 (D4, D5)
    summary: "The record says an adopt ends the old machine's processes, a running capture included, but adopt tears down only this machine's pair and a named job exists on one machine"
    fix: "In D5's adopt paragraph (line 149) and D4's Windows paragraph (line 108), drop the claim that the adopt ends the old machine's capture. State that the adopting machine has no capture record or baseline, and that a capture still running on the other machine ends at its own timeout or with that machine's teardown. Keep `pairteardown` ending a capture of this machine's own pair, and the adopt ending a local pair through `pairteardown`."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:14-19 and 105-109 (Scope In, D4)
    summary: "Scope In omits the pairteardown change that terminates the capture's named job and the new runproc capture role, both of which D4 and the Constraints require"
    fix: "Extend the Scope In bullet on the capture's process plumbing (line 16) to name the `runproc` capture role, `pairteardown` terminating the capture's named job on Windows through `proc.TerminateNamedJob`, and the capture's `proc.HoldNamedJob`, matching D4 lines 98 and 108 and the Constraints bullet on `pairteardown`."
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:107 (D4)
    summary: "The record leaves to the plan whether a step reaches spawnLoop, the one other proc.DetachBreakaway caller, which breakaway-OK jobs would now let leave the step"
    fix: "Settle it in D4 and drop the plan check. `spawnLoop` in `internal/shedverbs/loopwait.go` is the only non-test `proc.DetachBreakaway` caller, reached from the default branch of the loop waiter at `loopwait.go:111`. State whether a step child can run that waiter, and name what the plan does if one can; or state that a loop started from a step is meant to leave the step's job."
---

### [BLOCKING] A scope-full acceptance survives an interrupted attempt and skips the parent

**Location:** `_lyx/discussion/decision-record.md` D6 (lines 178-186) and D7 (lines 197-204)
**Issue:** D2 promises that under scope `full` no drift is accepted before the parent rules.
D6 keeps that promise only at the end of an attempt: when the agent ends, an attempt whose session accepted drift is Stuck in a decision halt, even on a passing compare.
D6 also says an error or transient stop of the row is the one exception and is resumed, never settled with an acceptance, and that each resume spawns a fresh agent.
The acceptance, though, is already in the scope record's accepted list, and D6's first step on every entry is the mechanical compare with `{skip_file}` listing the accepted tests.
So a session that accepts drift and then ends in an error, transient or interrupted stop (a watchdog kill, an API failure) leaves the acceptance behind.
The resumed row compares, passes with the acceptance skipped, spawns no agent and returns Done.
The "no acceptance by its own session" test D6 uses to return Done after the parent's resume cannot tell this resume from the parent's.
The tackler may resume an error or transient stop, so no decision halt is reached.
This is a guard that is bypassable with no stated bound, which the attack-surface question requires the Decision to bound.
The fix puts the check on every entry and makes the parent's resume from the decision halt the only way an unruled acceptance becomes ruled.

### [MEDIUM] The flag-off claim holds only for a row not yet entered

**Location:** `_lyx/discussion/decision-record.md` D2 (line 62), D5 (lines 145-147), acceptance criteria (line 247)
**Issue:** Focus 5 asks for consistency between D2, the acceptance criteria and the parent's ruling that `Drift-Await` carries `entry_gate: drift.enabled`.
The ruling is honoured by the wiring.
But `Shed.enter` in `internal/shedengine/entry.go` evaluates a gate only when `EnteredRow` does not name the row, and a Stuck halt leaves `EnteredRow` set.
`Goto` sets `EnteredRow` to its target.
So when the flag is turned off while `Drift-Await` waits on a running capture, or while `Drift-Await` or `Drift-Verify` is halted, a resume runs the row anyway.
D5 says "With the flag turned off mid-run, `Drift-Await` and `Drift-Verify` both record Skipped, so a capture left running is ignored", which is false for those cases.
The acceptance criterion is true only when the flag goes off before the run enters the row.
The case matters because turning the flag off is the operator's stated way to give up regression for a run.

### [MEDIUM] The waive precondition contradicts the halt that names it

**Location:** `_lyx/discussion/decision-record.md` D5 (line 146), D6 (lines 159-162), acceptance criteria (lines 247, 257)
**Issue:** In loom, D5 has `Drift-Verify` count a capture as passed only once `awaited_at` is set, so after a flag off-and-on a capture that passed with no `Drift-Await` is not counted and the row halts on no passed capture.
D6 says that halt names `lyx drift waive`, and that waive refuses unless the run is halted at `Drift-Verify` with no passed capture record.
The capture record exists and says `passed`, so waive refuses and names accept or fix, neither of which can act with no compare run.
The one route left is an unnamed `lyx loom goto Drift-Await`, which `shedengine.CheckGotoTarget` admits as a row before the current one, followed by `lyx drift defer`.
Focus 1 asked whether the named verb sequence lifts each halt.
Every sequence the record names for the `Drift-Baseline`, `Drift-Await` and `Drift-Verify` halts does lift its halt, and none names a goto past the current row, except this one case, whose named waive does not lift it.

### [LOW] The compare marker's liveness and the slot-wait bound are unstated

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 111-113, 121) and the acceptance criteria (line 263)
**Issue:** Focus 3 asks whether the waits and compares count as activity.
They do by design: the capture lock is an OS lock read by `agentActivityFor`, which already reads a verify marker and a parent-lock wait record in `internal/loomcli/looparm.go`, so `Drift-Await`'s wait and darn's foreground capture read as activity while the lock is held.
The compare is different: D4 names a "compare marker" and does not say whether it is written before the compare waits for its gate slot, or how a marker left by a killed compare is told from a live one.
A marker with no liveness test blinds the step watchdog.
The sentence "at most until the capture's own timeout" also fails for a capture queued for a slot, since the timeout counts from the command's start.

### [LOW] The adopt cannot end the previous machine's capture

**Location:** `_lyx/discussion/decision-record.md` D4 (line 108), D5 (line 149)
**Issue:** D5 says the run's processes on the old machine are ended by the adopt, a running capture included.
`internal/adoptcli/doc.go` shows the adopt acting on this machine's pair through `pairteardown` and recording the owner over a live pair with `--keep-local`.
It reaches no process of another machine, and a Windows job name lives in one machine's object namespace.
The design does not depend on the claim, since the adopting machine has no capture record, so the fix drops the claim.

### [LOW] Scope In leaves out the pairteardown and runproc changes

**Location:** `_lyx/discussion/decision-record.md` Scope In (line 16), D4 (lines 98, 108), Constraints (line 232)
**Issue:** D4 registers the capture under a new `runproc` role and has `pairteardown` terminate its named job through `proc.TerminateNamedJob`, which today runs only for the loop's job (`internal/pairteardown/loop_windows.go`).
The Scope In bullet on process plumbing names `agentActivityFor` and the two Windows jobs only, so a plan writer reading Scope alone misses both.

### [LOW] Whether a step reaches spawnLoop is left to the plan

**Location:** `_lyx/discussion/decision-record.md` D4 (line 107)
**Issue:** Giving `proc.StartTied`'s job and the loop's job `JOB_OBJECT_LIMIT_BREAKAWAY_OK` lets any `CREATE_BREAKAWAY_FROM_JOB` start leave them.
Focus 2 asks whether the capture survives its step.
It does by the design: `proc.StartTied` in `internal/proc/tied_windows.go` makes an unnamed kill-on-close job with no breakaway flag, as the focus says, so the record's change to that job and to the loop's named job, with `proc.DetachBreakaway` for the capture, is what lets it leave, and `proc.Detach` sets no breakaway.
The only other `proc.DetachBreakaway` caller is `spawnLoop` (`internal/shedverbs/loopwait.go:158`).
The record hands "whether any step reaches it" to the plan, a fact the design rests on and which the code can answer now.
