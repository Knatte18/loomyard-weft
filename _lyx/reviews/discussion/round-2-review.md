---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D10 premises the strict configengine.Load, but orch config loads through LoadOrTemplate, so the stated loader, and the Constraints line "keeps its current loader", disagree with the code
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:61
    summary: D7 removes the DEFER guard with no stated bound; Open risks calls the edge unbounded
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:98
    summary: D13 leaves the queue's storage form, concurrent-writer handling and delivery semantics on a crash open
---

### [BLOCKING] D10 names the wrong loader

**Location:** `decision-record.md` D10 (and D5, Constraints PATTERN-config-strictness line).
**Issue:** D10 says "the load is the strict `configengine.Load`".
`internal/orchengine/config.go:46` loads through `configengine.LoadOrTemplate`, and Constraints says orch config "keeps its current loader", so the record contradicts itself and the code.
PATTERN-config-strictness requires a caller to adopt exactly one loader, so a plan writer following D10 would switch loaders.
The value check also has no stated home: `LoadConfig` only unmarshals, so the two checks (D5, D10) must be added after the unmarshal.
**Fix:** D10 and the Constraints line name `LoadOrTemplate` as the kept loader, say an absent file resolves the template (value `bypass`) and a present invalid file errors, and place both value checks in `orchengine.LoadConfig` after the unmarshal.

### [MEDIUM] D7 removes a guard with no bound

**Location:** `decision-record.md` D7.
**Issue:** D7 drops the DEFER handshake, a guard against losing in-flight work.
It defers to Open risks, which calls the missing DEFER "the unbounded edge".
The attack-surface rule requires the Decision itself to say what the removal lets through and what bounds it.
**Fix:** add a `Bound:` line to D7: a compact cycle can lose a background task's in-flight completion if the survival assumption is false; `cycle_mode: clear` restores the DEFER handshake, and the compact smoke test asserts the completion.

### [LOW] D13's queue form is left open

**Location:** `decision-record.md` D13.
**Issue:** several batten processes append while the watcher removes, yet the record does not say how the queue is stored, who enforces the cap, or what a crash between typing and removal does.
**Fix:** state one file per notice, written atomically, cap checked by the appender (so concurrent writers may overshoot transiently), delivery at least once.

## Focus departures

None.
Verdicts on the directive's four questions: (1) D7/D11 record the premise in Auto-mode assumptions and the consequence in Open risks, not a defect, but D7's own bound is missing (F2); (2) D12 defines the prefix, single line, 200-character cut, control characters and leading slash, not a defect; (3) D12/D13 define the cap, drop rule, absent strand and persisted episode state, with only the storage form open (F3); (4) D10 names the fix and surfacing point and the running watcher, but the loader is wrong (F1).
