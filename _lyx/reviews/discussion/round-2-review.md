---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D8, second paragraph; Acceptance criteria
    summary: A finding no record-batch saw (the integration fork's transcript, Master's activity after the last batch) still hard-errors at run exit, so a policy finding fails a done run and a correctness finding halts with no way forward
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D5, recorded-warning bullets
    summary: D5's recording path is batch-scoped (the batch's state, the record-batch envelope), so it has no place for a warning raised at run exit, where no batch record applies
  - id: F3
    severity: LOW
    location: _lyx/discussion/support-log.md Question ledger item 18
    summary: The roadmap question is left open for the Plan in a file the Plan never reads, though manifest/roadmap.md answers it now
  - id: F4
    severity: NIT
    location: _lyx/discussion/decision-record.md Constraints, second bullet
    summary: The step refusal-kind and envelope-key rule is stated in Scope Out, Constraints and D11, and Constraints pins a tally of five values
---

### [BLOCKING] Run exit gives an undispositioned finding no disposition

**Location:** `decision-record.md` D8, second paragraph; Acceptance criteria.

**Issue:** D8 has the run-exit cross-check "hard-error only on a finding no record-batch dispositioned".
Such findings arise in an ordinary done run.
`runIntegrationStage` confirms a report from the integration fork Master spawns after every batch is terminal-done, and no record-batch runs after it,
yet `runExitAuditCrossCheck` runs `CheckFork` over every fork in `result.ForkAudit.Forks` and `CheckParent` over Master's whole session.
So the integration fork's transcript, and Master's transcript after the last record-batch, reach run exit undispositioned.

- Policy class: a `fabric-reference` in the integration fork hard-fails a done run.
  That is a policy guard halting, which D2 and D12 forbid, and it is the false stop the Goal names.
  "Applies the same classification" hints at a warning but decides neither the outcome nor where the warning is recorded.
- Correctness class: a `fork-contract-write` by the integration fork, or a Master `parent-write` after the last batch, returns a plain error from `mapMasterDone`.
  Every batch is terminal-done, so D6's `recover-batch` does not apply, and the error names no way forward under D1.
  A bare re-step would spawn a fresh Master whose session carries no finding, so the suspect content would land unaudited.

No must-cover scenario exercises either case.

**Fix:** Decide both cases in D8.
An undispositioned policy finding at run exit becomes a recorded warning (D5's path) and the run keeps its outcome.
An undispositioned correctness finding demotes Master's `done` to `stuck` with a stuck reason naming the finding and each suspect path, following the integration stage's regression demotion;
its way forward is to revert or re-derive the named paths on the warp with `git`, then re-step the Webster row.
Add must-cover scenarios for both: a `fabric-reference` in the integration fork's transcript ending in a passing run with one recorded warning, and a Master `parent-write` into the tracked tree after the last batch ending stuck with the path named.

### [MEDIUM] D5's recording path has no run-level home

**Location:** `decision-record.md` D5, the three recorded-warning bullets.

**Issue:** D5 appends a warning "to the batch's state in `state.json`" and returns it "on the verb's envelope".
A run-exit warning (F1) belongs to no batch, and its verb is `lyx webster run`, not record-batch.
The loom Webster row does not forward `RunResult.Warnings` (`internal/shedadapters/webster.go` reads only outcome and stuck reason), so on the loom path a run-exit warning survives only in whatever `state.json` and `summary.md` hold.
Read literally, D5 records a run-exit warning nowhere durable except `summary.md`, and the "once per finding identity" rule has no record to dedupe against.

**Fix:** Make D5's path cover both sites: a run-level warning list in `state.json` beside the per-batch ones, with the same once-per-identity rule;
the envelope of the verb that raised it (record-batch, or `lyx webster run` through `RunResult.Warnings`);
and the same `## Audit warnings` section, which already appends at run exit.

### [LOW] Roadmap question deferred to a Plan that cannot see it

**Location:** `support-log.md` Question ledger, item 18.

**Issue:** Item 18 is `open` with "Plan checks", but the Plan producer never reads the support log, so the deferral reaches nobody.
The answer is available now: `manifest/roadmap.md` carries no refusal-audit item, and CLAUDE.md moves the roadmap only for a planned item, not for hardening.

**Fix:** Resolve item 18: the roadmap carries no such item, so it does not move.
The rule is CLAUDE.md's, so nothing needs to go into the decision record.

### [NIT] The refusal-kind rule is stated three times

**Location:** `decision-record.md` Constraints, second bullet; also Scope Out and D11.

**Issue:** Scope Out excludes "any change to the closed `step` refusal-kind vocabulary or the step envelope key set", Constraints restates it, and D11 adds "It adds no step refusal kind."
The Constraints copy also pins the vocabulary's size ("five values"), which goes stale if that closed set changes elsewhere.

**Fix:** Keep the rule in Constraints without the tally, and drop the Scope Out bullet, since Constraints is where the Plan reads binding limits.
D11's sentence stays, since it states a property of the new verb.

## Focus departures

None.
Both directives are settled: the first by F1 (D8 decides neither case, and no scenario covers the policy case), the second by F2 (D5's path does not hold for a run-exit warning as written).
