---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4)
    summary: Runner.InterruptSettle consults the engine with neither a guid nor an event, so D4 leaves it on the default engine, and tender's settle reads one engine's settle for a pause over runs of two engines
    fix: In D4's bullet on methods that gain the guid of the run they read, add InterruptSettle beside the context-token, compaction and skill-load reads, resolving the engine from that run's record; state that tenderengine's settle passes each session's guid and the orch watcher's session seam passes the orch run's guid. Add an Acceptance criteria scenario that InterruptSettle on a run of the non-default fake answers that fake's settle.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7)
    summary: D7 hands Webster the default engine for fork audits and for event parsing (TurnEndedAfter in the recover path), bypassing D4's per-record rule, and states no bound saying this holds only while Webster's spawn specs name no engine
    fix: Add a Bound line to D7 stating that websterengine audits and parses its sessions through the default engine, which is the engine that ran them only because no Webster spawn spec names an engine in this task. Add an Open risk stating that engine-model-choice, once it fills Spec.Engine for Merriam or batch roles, must either keep those specs on the default engine or resolve Webster's audit and parse engine from the batch run's record.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4, D8)
    summary: The attach-mismatch routing edge (the record's engine wins over the spec's) states no bound on Requires, so an attach drives a run whose spec requires forks or another requirement on a recorded engine D8 would refuse at Start
    fix: In D4's Attach bullet, add that an attach checks the spec's Requires, including the forks implied by ForkSubagents, against the recorded engine and refuses with D8's error when the recorded engine does not meet them, so the respawn path is the operator's way forward. Add an Acceptance criteria scenario for an attach whose recorded engine does not meet the spec's Requires.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (R2)
    summary: R2 says a run in flight across a change of the engine default is read with the new default, but D4 writes the resolved engine into every new record, so only records written before this task lands lack the key
    fix: Reword R2's first clause to say a run started before this task lands, whose record has no engine key, is read with whatever default is configured at read time; keep the clause about long-lived set holders as written.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: D10's InstructionFiles capability has no must-cover scenario
    fix: Add an Acceptance criteria bullet under the go test list that the Claude engine's InstructionFiles answers CLAUDE.md and CLAUDE.local.md.
---

### [MEDIUM] InterruptSettle is an engine-consulting operation with no run to resolve from

**Location:** `_lyx/discussion/decision-record.md` D4.
**Issue:** `Runner.InterruptSettle()` (`internal/shuttleengine/run.go:1009`) reads `r.engine.(InterruptGuard)` and takes no argument.
D4 lists only the event-only methods (context tokens, compaction, skill-load classification) as gaining a guid, so this method has no record to resolve from and falls to the default engine.
Its callers include `tenderengine`'s `settle` (`internal/tenderengine/pause.go:389`), which loops over a pause's sessions and calls `s.pair.InterruptSettle()` per session, wired to `shuttlePair.InterruptSettle` (`internal/tendercli/deps.go:227`).
With runs of two engines in one pause, every session reads the default engine's settle.
D4's claim that every per-run engine-consulting operation picks the recorded engine does not hold for this method.

### [MEDIUM] Webster reads its sessions through the default engine, unbounded

**Location:** `_lyx/discussion/decision-record.md` D7.
**Issue:** D7 has `webstercli` and `loomcli` resolve the default engine and assert fork audit.
`websterengine` uses that engine not only for `AuditForks*` (`runwrites.go`, `recordbatch.go`) but also for `TurnEndedAfter` (`strand.go:53`, called from `recoverbatch.go:729`), which parses a batch session's events file directly.
Meanwhile `Start` and `Wait` resolve the spec's engine and its recorded engine.
In this task no Webster spec names an engine, so the two always agree and no run is audited by an engine other than the one that ran it; that settles the focus question for this task.
The agreement is an unstated premise, though: the Scope's Out list hands filling `Spec.Engine` from role aliases to `engine-model-choice`, and nothing in the record tells that task that Webster's audit and parse engine must then follow.
The two refusals themselves are consistent: D7's guards the Webster wiring's engine and points at `shuttle.yaml`'s `engine:`, D8's guards a spawn's engine and points at the role's alias.

### [MEDIUM] The attach-mismatch edge skips the Requires check

**Location:** `_lyx/discussion/decision-record.md` D4 and D8.
**Issue:** D4 introduces a routing edge: on attach, the record's engine drives the run even when the caller's spec resolves to another engine.
D8 checks `Requires`, and the `forks` that `ForkSubagents` implies, only at `Start`.
An attach therefore lets a spec whose requirements the recorded engine does not meet drive a live run, including the end-of-run fork audit D7 makes depend on the recorded engine's capability.
D8's own bound ("`Requires` only refuses runs and never lets a run skip a check") is not true of this path, and D4 states no bound for it.

### [LOW] R2 misstates which records read with a changed default

**Location:** `_lyx/discussion/decision-record.md` Open risks, R2.
**Issue:** D4 has `Start` and `Open` write the resolved engine into the record, and D5 makes only an absent key read as the default.
A run started after this task lands carries its engine, so a later change of `engine:` does not change how it is read.
R2's first clause states the risk for every in-flight run, which contradicts D4 and D5; it holds only for records written before landing.

### [LOW] No scenario covers InstructionFiles

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.
**Issue:** D10 decides Claude's `InstructionFiles()` answer, and no acceptance bullet covers it.
The other focus-named Decisions are covered: D9's capability-missing path, D11's `effort`/`version` mapping and D13's scan each have a scenario, and every scenario names a behavior rather than a test to write.

## Rejected

- D4's claim that Stop, the step-stop sweep and strand removal consult no engine: verified, not a defect. `stop.go` and `stepstop.go` read no engine, and `StopStrand` removes a strand with no run record by itself.
- D4's default-engine rule for strands with no run record: verified for `SessionIdle`, the one engine-consulting method that proceeds without a record (`session.go:153`); every other one fails on a missing record through `FindRun`.
- D13's scan wording: not a defect. The only package under `internal/shuttleengine/` is `claudeengine`; `reedengine` is not a concrete engine and is dropped from `cliwire`'s allowlist only because `sessionnamer.go` is its sole importer there; `shuttlefake` lives under `internal/testkit/` and is imported from tests, which the scan exempts.
- D8's closed `Requirement` set: bounded. An unknown string is refused at `Start`, the only entry point that reads `Requires` (`OpenSpec` has no such field), and `Supports` absent meeting none only refuses runs.
- D6's skipped-run state toward the lock decisions: its bound is stated.
- Relocation check: no deliberation narrative in Decisions or Constraints needs moving; D7's sentence on the board's wording is a one-clause departure note that carries the decision.
