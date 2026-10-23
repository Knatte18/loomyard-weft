---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/websterengine/verifyfixer.go:99
    origin: lens:code-pattern-fit
    summary: "Two FixVerifySpawnOrAttach refusals have no test that reaches them, which PATTERN-refusal-way-forward requires: the recovery engine that cannot audit forks, and the gate config with no round list"
    fix: "In internal/websterengine/verifyfixer_test.go, add two rows to TestFixVerifySpawnOrAttach. One uses an Engines fixture whose recovery engine cannot audit forks, the other a Gate config with no round list. Each row asserts that nothing was spawned, that st.VerifyFixers is unchanged, and that the error text names the way forward."
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/webstercli/fixverify.go:79
    origin: lens:code-tests
    summary: "The fix-verify CLI verb has no behavioural test beyond the pause refusal: done envelope with reply, dead envelope with non-zero exit, no-run refusal and persistFixVerifyTerminal are unpinned"
    fix: "In internal/webstercli, add a table-driven test of the fix-verify verb over the existing shuttle fakes, extending the recover-batch verb tests if they share a fixture. Rows: no state.json refuses with the run-first way forward; done returns status done with the reply field; dead returns dead_reason with a non-zero exit; and a terminal result saves state with the fixer's Status set."
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/websterengine/verifyfixer_test.go:1
    origin: lens:code-tests
    summary: "TestFixVerifyAwait has no row for dead with reason asking, the turn-ended-without-reply branch that the focus directive and the summary both name"
    fix: "Add a row to TestFixVerifyAwait whose events file shows a turn end after EventsOffset while no reply file exists. Want Status dead, DeadReason DeadReasonAsking, and the strand stopped with the run dir kept."
  - id: F4
    severity: LOW
    class: design
    location: internal/websterengine/verifyfixer.go:150
    origin: lens:code-correctness
    summary: "A strand spawned by Start is live but unrecorded when FindRun fails, or when SaveState fails in fix-verify; the next call spawns a second strand and neither the run-entry reclaim nor reset can stop the first"
    fix: "In FixVerifySpawnOrAttach, when FindRun fails after Start succeeded, call deps.Stopper.StopStrand(run.StrandGUID) and return the error joined with any stop error. In internal/webstercli/fixverify.go, when SaveState fails after a spawn, stop rec.StrandGUID through the same seam before reporting the error. Apply the same change to recover-batch only if it has the identical gap."
  - id: F5
    severity: LOW
    class: consistency
    location: internal/websterengine/verifyfixer.go:95
    origin: lens:code-pattern-fit
    summary: "The missing-model-spec refusal names no way forward, though PATTERN-refusal-way-forward requires one on a webster refusal"
    fix: "Append a way forward to the error text, in the shape of the sibling refusals in this function. If the missing key cannot occur after wiring, delete the check instead."
  - id: F6
    severity: LOW
    class: consistency
    location: contracts/specs/webster-spec.md:49
    origin: lens:code-affected-set
    summary: "webster-spec.md line 49 still says Merriam spawns one fixer fork on a gate failure, while line 62 of the same diff now says the gate routes each failure to a fork or the cold fixer"
    fix: "Reword the sentence at webster-spec.md:49 to say Merriam answers a gate failure with one fixer, a fork or the cold fixer, and make the Verify gate fixes summary-section text cover the cold fixer's commits too."
  - id: F7
    severity: LOW
    class: consistency
    location: contracts/specs/loom-plan-spec.md:309
    origin: lens:code-affected-set
    summary: "loom-plan-spec.md lines 309 and 311 name the verify-fix fork alone as the author of Pattern-Departure trailers; cold fixer commits carry them too"
    fix: "In loom-plan-spec.md lines 309 and 311, replace the verify-fix fork with the verify-gate fixers."
  - id: F8
    severity: LOW
    class: consistency
    location: internal/websterengine/template_test.go:1
    origin: lens:code-tests
    summary: "New assertions pin stencil static prose: You are the FIXER fork, Merriam reads only that reply, starting COLD and Edit or Write, plus matching not-contains checks; verifyfixer_test.go also asserts starting COLD"
    fix: "In template_test.go and verifyfixer_test.go, remove the assertions on those prose strings. Keep the checks on the report path, the reply path and the directive markers, which already tell the fork prompt from the cold prompt."
  - id: F9
    severity: NIT
    class: consistency
    location: internal/websterengine/recordbatch.go:643
    origin: lens:code-affected-set
    summary: "A comment still says the run-exit audit covers the verify-gate fixer fork, though the audit now also reads cold fixer sessions"
    fix: "Change the comment at recordbatch.go:643 to say the verify-gate fixers."
---

### [BLOCKING] Refusals with no reaching test

**Location:** internal/websterengine/verifyfixer.go:99
**Issue:** `PATTERN-refusal-way-forward` requires that a test reach every webster refusal, and the decision record repeats it as a constraint.
Only the no-report refusal has a test.
The recovery-engine-cannot-audit-forks refusal and the no-`round`-list refusal are untested.
Both are named in the summary and in the verb's help text.

### [MEDIUM] fix-verify verb has no behavioural test

**Location:** internal/webstercli/fixverify.go:79
**Issue:** The only `webstercli` test that touches `fixVerifyCmd` is the pause-refusal row.
The terminal envelopes, the exit code on dead, the reply field and the state save under a fresh lease are unpinned.

### [MEDIUM] No test row for dead with reason asking

**Location:** internal/websterengine/verifyfixer_test.go:1
**Issue:** `TestFixVerifyAwait` covers done, timeout, died and running.
The turn-ended-without-reply branch at `verifyfixer.go:196` is not covered, though it is one of the terminal results the verb reports.

### [LOW] Spawned strand left unrecorded on a failure after Start

**Location:** internal/websterengine/verifyfixer.go:150
**Issue:** If `FindRun` fails after `Start`, or if the CLI's `SaveState` fails after a spawn, the strand is live and absent from `st.VerifyFixers`.
The call returns an error, so the next call spawns a second strand.
The run-entry reclaim and `reset --to start` both read only the recorded list, so neither stops the first.

### [LOW] Missing-model-spec refusal names no way forward

**Location:** internal/websterengine/verifyfixer.go:95
**Issue:** The refusal text ends without a way forward, unlike its sibling refusals in the same function.

### [LOW] webster-spec.md still describes a single fixer fork

**Location:** contracts/specs/webster-spec.md:49
**Issue:** Line 49 contradicts line 62, which this diff rewrote.

### [LOW] loom-plan-spec.md names only the fork for departure trailers

**Location:** contracts/specs/loom-plan-spec.md:309
**Issue:** The cold fixer's commits are read for `Pattern-Departure` trailers too, and `AppendVerifyGateFixCommits` now says so.

### [LOW] Assertions on stencil static prose

**Location:** internal/websterengine/template_test.go:1
**Issue:** The rubric's test-economy rule makes each new assertion on a stencil's static prose a finding.
The named strings are listed in the finding's summary.

### [NIT] Stale comment naming the fork alone

**Location:** internal/websterengine/recordbatch.go:643
**Issue:** The comment predates the cold fixer, whose sessions the run-exit audit now reads.

## Rejected

- code-correctness F2, reset stopping already-stopped fixers: `StopStrand` falls back to `removeStrandIfPresent` for a gone strand (`internal/shuttleengine/stop.go:54-63`), and the batch-recovery loop in the same function already stops terminal strands the same way.
- code-pattern-fit F3 and code-tests F-T3 are folded into F3 above; code-pattern-fit F1 and code-tests F-T2 are folded into F1 above.

## Holistic checks, no finding

- Gate routing: the shuttle re-prompts while `gateFails < Attempts` (`internal/shuttleengine/wait.go:535`), so failure number `attempts` is the last one re-prompted, and `failures >= attempts` routes exactly that failure cold.
  The case where a fork added no commit (`failures > 1 && len(fixCommits) == lastFixCommits`) and the sticky cold flag are correct.
- Scope against the board entry: the template drops to 2, the `recovery` model key is reused, and the diff adds no `webster.yaml` key.
  Nothing in the diff changes the task's own board entry.
- The summary's claims about routing, the verb's results and refusals, state, stencils and docs hold against the diff, except the pause refusal that F1 does not touch.
