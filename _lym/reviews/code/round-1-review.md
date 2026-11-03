---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/driftwiring.go:194
    origin: lens:code-tests
    summary: The CaptureAccepted and CompareAccepted seams carry the core of the change and no test pins their request fields.
    fix: Add a test in internal/loomcli beside the existing driftDeps tests (export_test.go exposes driftDeps) that builds the Drift Deps over a temp anchor with a declared compare and capture step, a scope record with skip s1 and accepted test t1, then calls Deps.CompareAccepted and Deps.CaptureAccepted. Assert that the compare command renders {baseline_dir} as Paths.AcceptedDir, that the skip file lists s1 and not t1 for both seams, and that Deps.Compare still lists both s1 and t1. Use a command that writes its rendered arguments to a file under the temp dir so no real test suite runs.
  - id: F2
    severity: LOW
    class: design
    location: internal/driftengine/recheck.go:537
    origin: lens:code-correctness
    summary: A recheck that finds drift lapses every acceptance, including unchanged ones and parent-ruled ones under scope full, so the parent must rule each again.
    fix: Prefer lapsing only the accepted tests the accepted-profile compare names as differing, or carrying the Ruling of an acceptance Drift-Verify's agent re-accepts unchanged. When the compare output cannot name the tests, keep lapse-all and say in the Drift-Recheck paragraph of internal/driftengine/doc.go that under scope full every lapsed acceptance needs the parent's ruling again.
  - id: F3
    severity: LOW
    class: design
    location: internal/driftengine/capture_test.go:350
    origin: lens:code-tests
    summary: The RunAcceptedCapture timeout branch has no row in acceptedRows.
    fix: Add a row to acceptedRows in TestRunCapture whose command outlives a short injected Timeout, mirroring the existing RunCapture timeout test in the same file. It wants Passed false and a reason naming "timed out after" and the log.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/reportengine/doc.go:130
    origin: lens:code-affected-set
    summary: The reportengine doc lists the tapestry webster phase rows and the darn review phase rows without Drift-Recheck, which timingspans.go now maps to both.
    fix: Add Drift-Recheck after Drift-Verify in the tapestry webster list at internal/reportengine/doc.go:130 and in the darn review list at line 132, matching the loomPhases and darnPhases maps in timingspans.go.
  - id: F5
    severity: LOW
    class: consistency
    location: internal/driftengine/template.yaml:1
    origin: lens:code-affected-set
    summary: The enabled comment in the drift config template still names three rows, and Drift-Recheck now carries the same entry gate.
    fix: Reword the enabled comment at internal/driftengine/template.yaml:1 to name Drift-Recheck with Drift-Baseline, Drift-Await and Drift-Verify, or to say the switch gates every Drift row.
  - id: F6
    severity: LOW
    class: consistency
    location: contracts/recipes/tapestry-recipe.yaml:74
    origin: lens:code-affected-set
    summary: The header comment says a run inside the perch exits to Describe without Drift-Verify, but the perch now exits to Drift-Recheck.
    fix: Reword the comment at contracts/recipes/tapestry-recipe.yaml:74-76 so a run inside the perch exits to Drift-Recheck, which sends it back to Drift-Verify while no compare is recorded, and keep the lym loom goto --to Drift-Verify advice only if it still reads true.
---

### [MEDIUM] Accepted-profile wiring has no test

**Location:** internal/loomcli/driftwiring.go:194
**Issue:** `CompareAccepted` leaves accepted tests in the compare by passing `scope.Skip`, where `Compare` passes `SkipList`, and points the command at `Paths.AcceptedDir`.
That choice is what makes an accepted test whose output changed show as drift again, which is the second `Expected` bullet.
Every `driftengine` test stubs both seams, so swapping in `SkipList` or dropping `ProfileDir` fails no test.

### [LOW] Lapse drops every acceptance on drift

**Location:** internal/driftengine/recheck.go:537
**Issue:** `lapse` clears `Accepted` whole.
Acceptances whose output did not change go too, and under scope full so do the parent's rulings, which Drift-Verify's agent can only re-accept as unruled.
The answer key asks only that a changed accepted test count as drift again, so this is a cost and a tradeoff the code comment states, not a contradiction of the key.
The bounce budget bounds the loop: Drift-Recheck's Stuck carries an `on_stuck`, so the shed counts it and halts for the parent once the budget is spent.

### [LOW] Accepted-capture timeout is untested

**Location:** internal/driftengine/capture_test.go:350
**Issue:** The `timed out after` branch of `RunAcceptedCapture` is its own line of code, and the only timeout test covers `RunCapture`.

### [LOW] reportengine doc omits Drift-Recheck

**Location:** internal/reportengine/doc.go:130
**Issue:** The doc lists each phase's rows, and the tapestry webster list and darn review list stop at Drift-Verify while `timingspans.go` maps Drift-Recheck into both.

### [LOW] Drift config template names three Drift rows

**Location:** internal/driftengine/template.yaml:1
**Issue:** The `enabled` comment names Drift-Baseline, Drift-Await and Drift-Verify, and Drift-Recheck now carries `entry_gate: drift.enabled` too.

### [LOW] Tapestry header comment names the old perch exit

**Location:** contracts/recipes/tapestry-recipe.yaml:74
**Issue:** The comment sends a run inside the perch to Describe, and `Code-Bouncer`'s `on_done` is now Drift-Recheck.
I read lines 74-76 myself to confirm.

## Rejected

- Lens code-correctness F2, a resume after a failed accepted capture repeats the baseline compare. It costs time and leaves no wrong state, and the fork marked the fix optional.
- Lens code-correctness F3, Drift-Recheck passes with unprofiled acceptances after a manual `lym loom goto` past Drift-Verify. Unreachable in the recipe's order, and a manual goto is the operator's own override.
- Lens code-tests finding 3, no row for `CaptureAccepted` returning an error. The pass returns the error through one line with no logic of its own, so a row would pin Go's error return and not a behavior.

## Judgments on the focus directive

These settle the questions the directive posed, each by reading the code:

- The accepted profile is captured at HEAD after the compare passed, with the acceptances in force, so it holds the output the acceptances saw. `CompareAccepted` passes `Skip: scope.Skip`, so accepted tests stay in the compare, where the baseline compare skips them through `SkipList`. Correct.
- Every Done path of Drift-Verify with tests accepted goes through `keepAcceptedProfile`: the passing compare and the agent's Done, which covers resume onto a live agent. Waiver, scope off and no scope end Done before any acceptance exists. A failed capture is a Stuck after `ComparedHead` is recorded and with `AcceptedProfileHead` unset, so a resume re-runs the compare and the capture. Consistent.
- `ComparedHead`, `ComparePassed` and `AcceptedProfileHead` stay coherent across a bounce: `recordCompare` and `lapse` both go through `UpdateScope`, and `lapse` runs after the record, so the profile head ends empty. No acceptance lapses past the bounce budget, which counts Drift-Recheck's Stuck.
- Recipe wiring matches the other Drift rows: the `entry_gate`, name constant, registry entry, interrupt policy and batten-nil seams are all present, and tapestry's tail has the recheck after Code-Review and before Describe.
- Scope: `reportengine` timing spans, `envkit` and `steprun.go` and `paths.go` are each needed by the new row, its seams or its capture site. No change goes beyond the Goal.
- No PATTERN entry is touched or violated, no `Pattern-Departure:` trailer is present or needed, and the directive declares no PATTERN changes.
