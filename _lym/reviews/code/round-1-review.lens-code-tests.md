```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. **`exempt` lifts the gate on an added entry whose title opens `No bug fixed`.** Pinned by the new row "a record whose added entry opens No bug fixed passes and runs nothing" in `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` (`internal/loomshed/redfirstgate_test.go:67-73`).
   - The row's command is `exit 3`, and the loop asserts both `Passed` and that nothing ran (`:263-268`).
   - If the exemption broke, the row would fail with passed false and ran true.
2. **The title comparison ignores case, for both titles.** Pinned by the existing lower-case `no test seam` row (`:22-23`, `:60-62`).
   - Both titles go through one `ToLower` loop (`internal/loomshed/redfirstgate.go:188-189`), so a lower-case row per title would duplicate coverage. No gap.
3. **`Final` uses the same exemption.** Pinned by the existing row "a final evaluation of a record whose added entry opens No test seam passes and runs nothing" (`:189-195`), which reaches the shared `exempt`. No separate row for the new title is needed.
4. **The reworded green finding in `redFirstFailure`.** Pinned by the green row (`:88-102`). It asserts the new opening sentence, the bug-fix way forward, the no-bug way forward, `lym creel post --parent --expect-reply` and `under a title opening "No bug fixed"`.
5. **`parentRecordAsk(gateslot.NoTestSeamTitle)` in the no-test and unjudged hold lines and findings.** Pinned by the rows at `:104-158`, which assert `under a title opening "No test seam"` in both `Send` and `Findings`. Behaviour is unchanged.
6. **The green refusal of `lym gate red-first` (`redFirstCall.run`, `internal/gatecli/redfirst.go:245-252`).** Pinned by `TestGateRedFirst/ATestThatPassesOnTheBaseExitsWithTheGreenCode` (`internal/gatecli/redfirst_integration_test.go:100-106`).
   - It asserts `under a title opening "No bug fixed"`, which the `%q` format produces.
   - The reach report's "no mapped test" note comes from the test being out of process, not from a gap.
7. **The `exempt` log line's wording.** No test, and none needed: it is a log field.

**Loosened checks:** none. Every changed assertion either replaces old strings with the new wording at the same strength or adds a condition (`redfirst_integration_test.go:103`).

**Test economy:** the new loomshed row covers a second title, which no existing row covers. The rows at `:88-102` and `:100-106` change existing assertions rather than add tests. Nothing is redundant or tautological.

**Findings:** none.

**Tool calls**
1. `sed -n 20,62p; 180,272p internal/loomshed/redfirstgate_test.go`: the test table's fields, its first rows and the loop's assertions. The manifest held only the diff hunks of this file.
