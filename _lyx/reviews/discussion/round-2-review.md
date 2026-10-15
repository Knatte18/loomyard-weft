---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: The attach Requires check derives forks from the attaching spec's ForkSubagents, while the run record's ForkSubagents decides Wait's fork audit, and D7 leaves Wait's behaviour open for a fork-authorized record whose engine lacks the fork-audit capability, so D8's never-skip bound does not hold on the attach path.
    fix: In D4's Attach bullet, state that the attach check's forks requirement comes from the run record's ForkSubagents, the field that decides the end-of-run fork audit, joined with the spec's own Requires; in D7's first bullet, add that Wait over a ForkSubagents record whose engine lacks the fork-audit capability fails the run with an error naming the engine, never finishing without the audit; extend the matching acceptance bullet (line 227) with a record carrying ForkSubagents attached by a spec without it.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:130
    summary: An attach refused under D8 carries D8's way forward, to give the role a model alias of an engine that meets the requirements, which cannot clear it, because the live run keeps its recorded engine and the next attach refuses again; D4's respawn path names no step that gets there.
    fix: In D8's refusal bullet, state that an attach refusal names its own way forward instead of the alias one, namely ending the run's live strand so the next probe answers not found and the caller's Start uses the spec's engine, naming the existing verb that ends a strand, which must be one no agent setting denies (PATTERN-no-denied-recovery); replace D4 line 81's the respawn path is then the operator's way forward with a pointer to that D8 sentence; and state whether a finished run, every output file on disk, is still refused or is harvested, since its harvest drives no further turn.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:225
    summary: D7's bound and R4 name only engine-model-choice filling Spec.Engine as the way Webster's default-engine audit can diverge from the batch run's recorded engine; a changed shuttle.yaml engine default across a Merriam restart or recover diverges the same way once a second engine is registered, and no risk names who acts then.
    fix: In R4, add the second divergence condition, a changed engine default between a batch's spawn and its later audit, recover parse or archive, which becomes possible only when a second engine is registered (codex-engine), and name that task as the one that resolves Webster's audit and parse engine per batch; reword D7's bound (line 118) to say the default is the engine that ran the sessions only while the default is the one registered engine and no Webster spawn spec names an engine.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:83
    summary: D4 makes InterruptSettle resolve the engine from a guid's record and refuses an unregistered engine, but its callers, tender's settle and the orch watcher's settled-key check, have no failure path today, and the record leaves open what a settle wait does when that resolution fails.
    fix: Add one sentence to D4's guid-less-methods bullet saying what an InterruptSettle whose run record cannot be resolved does for its callers, either an error that tender's settle and the orch watcher log and then wait the longest settle they resolved, or a zero settle logged at Warn, and pick one.
---

### [MEDIUM] The attach Requires check reads forks from the wrong source

**Location:** `_lyx/discussion/decision-record.md:81` (D4 Attach bullet), with D7 line 111 and D8 line 137.

**Issue:** D4 says an attach checks "the spec's `Requires`, including the `forks` that `ForkSubagents` implies", which is the attaching caller's spec.
`internal/shuttleengine/attach.go` reconstructs the run with the caller's normalized spec but the candidate's persisted state, and `internal/shuttleengine/wait.go:1670` runs the end-of-run fork audit on `run.state.ForkSubagents`, the record.
`internal/shuttleengine/doc.go:326` states the rule: "the record, not a later Call's Spec, also decides the fork audit."
So a record carrying `ForkSubagents` that a spec without it attaches is checked for no `forks`, yet `Wait` still audits it.
D7 moves the audit into an optional capability that `Wait` "takes", but never says what `Wait` does for a fork-authorized record whose engine lacks it.
If an implementation skips the audit there, which is the natural reading of an optional capability, `Requires` has let a run skip a check, against D8's bound.
Within one binary, `Start` and `shuttleengines.New` keep that state out, but D6 already accepts that a binary can disagree with the one that started a run, and an attach after a process restart is the path where it does.
Verdict on the focus question: the bound does not hold on the attach path as written.

### [MEDIUM] An attach refusal names a way forward that cannot clear it

**Location:** `_lyx/discussion/decision-record.md:130` (D8), with D4 line 81.

**Issue:** D8's refusal names one way forward, "give the role a model alias of an engine that meets them", and D4 says an attach refuses "with D8's error".
On attach, the live session belongs to the recorded engine (D4 line 80), so a changed alias leaves the record unchanged and the next attach is refused again.
D4's "the respawn path is then the operator's way forward" names no step: `attach.go` removes superseded strands and answers not found only when no candidate is attachable, so a refused attach removes nothing and the caller starts nothing.
`lyx shuttle` has no stop verb (`internal/shuttlecli`), so the step that ends the strand must be named.
The refusal also reaches a run `attach.go` would otherwise harvest as finished, every output file on disk, and the record does not say whether that harvest is refused too.

### [LOW] Webster's default-engine bound omits a changed default

**Location:** `_lyx/discussion/decision-record.md:225` (R4), with D7's bound at line 118.

**Issue:** `websterengine` audits forks and parses `TurnEndedAfter` through one engine handed in by its wiring (`internal/websterengine/recordbatch.go`, `recoverbatch.go`, `runwrites.go`, `archive.go`), keyed by session id, not run guid, while `Start` and `Wait` resolve the spec's and the recorded engine.
D7's bound gives one reason the two agree, that no Webster spawn spec names an engine, and R4 hands its break to `engine-model-choice`.
They also diverge when `shuttle.yaml`'s `engine:` changes between a batch's spawn and a later audit, recover or archive, which Webster reaches across restarts.
In this task, `New` admits only registered names and `claude` is the only one, so this cannot happen yet, and the finding is LOW.
Neither the bound nor R4 names that condition or the task that makes it reachable.
Verdict on the focus question: the record states one of the two divergence conditions and who acts on it, but not the other.

### [LOW] InterruptSettle's failure behaviour is open

**Location:** `_lyx/discussion/decision-record.md:83` (D4 guid-less-methods bullet).

**Issue:** The callers D4 names are reached correctly: `tenderengine`'s `settle` (`internal/tenderengine/pause.go:383`) loops over the keyed sessions and takes the longest settle, so with a guid each session's own engine answers, and a pause that spans two engines waits the longer of the two;
the orch watcher (`internal/orchengine/pause.go:74`) holds the orch strand as `st.Strand`;
shuttle's own interrupt guard reads the run's engine directly.
Verdict on the focus question: every engine-consulting `Runner` method has a way to pick its engine.
What is left open is the failure case: D4 line 79 refuses an engine-consulting operation on an unresolvable record, but `settle` and the watcher's settle comparison have no error path, and the record does not say what wait they use then.
