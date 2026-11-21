```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code for correctness (lens fix-code-correctness).

**Findings**

1. **BLOCKING** — `internal/shuttleengine/gate.go`, new field `GateResult.Sent func()`.
   - **Problem:** the field makes `GateResult` non-comparable, and two test files compare it with `!=`. They no longer compile, so the packages' tests can't run.
   - **Evidence:** `go vet` over the other packages that use `GateResult` fails in two places:
     - `internal/verifytree/reach_test.go:130:23`: `got != step.inner` (struct containing func() cannot be compared)
     - `internal/gatefixer/rowgate_test.go:335:22`: `got != writerOnly` (same error)
   - **Why it was missed:** the gate and vet runs covered only `loomshed`, `loomcli`, `shuttleengine`, `shedrun` and `shedadapters`. The remaining packages that use `GateResult` reported nothing else.
   - **Fix:** change those two comparisons to field-wise checks, or drop the func field and carry delivery another way.

**Other lens checks, no findings**
- **`holdResult`, `arrival` and `final` flow:** with the hold-send sequence run step by step, `told` resets on a fresh hold, and a refusal after a passing run re-offers the line. A new refusing commit stays silent once told, and `final` on the held tree answers from `heldResult`.
- **`Sent` wiring in `wait.go`:** `Sent` is carried through `evaluateGate` and reset on every real evaluation. It is called only after a successful `sendWithin`, so a failed send never marks the line as told.
- **`latestDarnOutcome`:**
  - An empty or unparseable `StartedAt` still names the file, so the old rows hold.
  - The stat error on an invalid path is still returned.
  - The new "no findings of its own" row fails without the mtime check.
- **`PreflightStencils` via `c.env.DarnTable`:**
  - Nothing else assigns `DarnTable` outside tests, so production behaviour is unchanged.
  - The new wiring rows would fail under the old inline wrap.
  - The `way forward:` count is 1 in both rows.
- **Added `redfirstgate_test.go` rows:** each fails with its defect in place. Without the `Sent` hook, `told` reset or held-tree shortcut, the hold steps and the `Final` test fail. Without the `TimedOut` guard, the exit 4/5 timeout rows fail.

**Tool calls**
1. `grep -rl GateResult` plus a comparison grep: list the packages using `GateResult`. My context only showed the five packages the fixer compiled.
2. `go vet` on those packages: confirm the build break above.
