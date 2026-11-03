---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: contracts/stencils/drift/drift-template-baseline.md:52
    summary: Step 4 asks why the task makes each skipped test meaningless or nondeterministic, which presumes the agent chose the skips although a scope the parent already recorded can carry a skip list chosen for another reason.
    fix: In contracts/stencils/drift/drift-template-baseline.md line 52, replace "Name each skipped test and why the task makes it meaningless or nondeterministic." with "Name each skipped test and why it is skipped."
    origin: lens:code-correctness
---

### [NIT] Skipped-test reason in Step 4 presumes the agent chose the skips

**Location:** contracts/stencils/drift/drift-template-baseline.md:52
**Issue:** Step 4 ends with "Name each skipped test and why the task makes it meaningless or nondeterministic."
Step 2 limits `partial` to such tests, so for a scope the agent records itself the line is accurate.
When the scope was already recorded (the `scope_note` at line 44 and the closing sentence of Step 4), the skip list can come from the parent or the hub config, and a reason such as a slow test is then outside what the line tells the agent to write.
The line also goes beyond what the entry's Expected section asks for, though the verify stencil's own line 23 already reads a report that names the scope's skipped tests, so the addition does not contradict it.

## Judged against the fasit

- Expected bullet 1 (keep `full` for a task whose intended change the suite reaches, have `baseline.md` name the tests and lines expected to drift): met by the `full` bullet at line 36 and Step 4 at line 51.
  Verify's own rules (`drift-template-verify.md:29-31`, `autonomousVerifyJudging` and `operatorVerifyJudging` in `internal/driftengine/seats.go`) still accept intended drift with a reason and treat other drift as a defect, so the new Step 4 wording leaves no contradiction there.
- Expected bullet 2 (`--skip` for a test the task makes meaningless or nondeterministic, `off` for a task the suite cannot reach): met by the `partial` bullet at line 38 and the `off` bullet at line 35.
- Rules the entry did not ask to change are intact: the reachability check before the list, Step 3's single-call rule and the `scope_note` marker.
  The marker set is unchanged against `contracts/stencils/markers_test.go:62`.
- Commit `e12efa51` and `_lym/landing/summary.md` both match the diff: scope routing, `--skip` reserved for meaningless or nondeterministic tests, `baseline.md` naming the expected lines, no Go change, `drift.scope` path untouched.
- No `PATTERN` entry is touched or violated, no declared PATTERN change exists to land, and the diff adds no test, so test economy raises nothing.
- Forks `code-correctness` and `code-affected-set` raised the same single NIT, kept once above; `code-tests` raised nothing.
