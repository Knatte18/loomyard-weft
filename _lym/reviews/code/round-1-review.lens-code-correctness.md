```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over `internal/loomcli/amendduringwebster_test.go`, the diff's only file.

**Per function and branch: the case I ran and whether it is correct**

- **Fixture loop.** n = 1..5. Cards 1–4 get a `Foo&lt;n&gt;` func in `sub/a.go` and a done batch whose card ID is `"%02d-%s"`. That ID matches plankit's card file name (`plankit.go:72`, `"%02d-%s.md"`), so `runplan.DoneCards` sees cards 1–4 as done. Card 5 is skipped by `continue`, so it has no func and no batch. **Correct.**
- **SaveState and ReadFile error paths.** A failure calls `t.Fatal` before anything depends on it. **Correct.**
- **`decisionVerb` call.**
  - `deps.approvedPlan` is replaced by `planApprovedReader(anchorPath)`. The overview exists and `Approved: true` is set, so the reader answers `(true, nil)` and the amends-plan branch admits (`decision.go:989-999`).
  - A non-zero exit fails with the output printed.
  - **Correct as a call.** The weakness is in what the assertion after it can detect; see finding 1.
- **`shedengine.Goto`.**
  - The status is Webster, blocked. Webster is index 3, so `refIdx` is 3. The state is not awaiting, so `limit` is 4 and Plan-Burler-Review is admitted (`goto.go:339-385`).
  - There is no Guard and no Admitted hook. The write lands in `runDir/status.json`, and an error calls `Fatalf`.
  - **Correct as a call.** See finding 1.
- **State byte-equality.** It reads `state.json` again after both steps and compares bytes, and a read failure calls `Fatalf`. The logic is correct, but nothing between the two reads is handed a path under `websterengine.Dir(anchorPath)`. See finding 1.
- **Plan-gate assertion.**
  - The real `NewPlanGate` → `ValidatePlan` → `weaverengine.Active(AnchorRoot: anchorPath)` loads the saved Webster state. `websterBegun` holds, so cards 1–4 are done and `validateFormatAfter` skips their tree checks.
  - Without `state.json`, the `sub#Foo1..4` Creates would resolve against `sub/a.go` and report `create-already-exists`. `TestPlanGate`'s `NoRunRecordFailsCreateAlreadyExists` subtest at `gates_test.go:597-610` shows the same shape fails.
  - The condition `!result.Passed || strings.Contains(..., "create-already-exists")` fails on either a failed gate or the finding.
  - **Correct: the gate half truly depends on `state.json`.**

**Findings**

1. **MEDIUM, class `design`.**
   - **Location:** `internal/loomcli/amendduringwebster_test.go:72-120`
   - **Summary:** The `state.json` byte-equality check holds by construction. No step between the two reads can reach Webster's record, so the check cannot fail, and the doc comment's claim at line 35 is not pinned.
   - **Evidence for the decision step:**
     - `newDecisionFake` points `implementerSeam` at an unrelated `t.TempDir()` (`internal/loomcli/decision_test.go:253`). The test never sets it to `anchorPath`.
     - Every other decision seam is a closure over the fake (`decision_test.go:355-394`).
     - The production wiring hands the verb the real anchor: `implementerSeam: anchorImplementerSeam(target)` at `internal/loomcli/decision.go:1647`.
   - **Evidence for the goto step:**
     - The bare `shedengine.Goto` writes only under `runDir` and is given no Guard or Admitted hook (`internal/shedengine/goto.go:123,154`).
     - Loom's real goto composes `gotoGuardEndingFixers(settledGuard())` and `prReviewGotoAdmitted()` (`internal/loomcli/arm.go:478-479`). The test runs neither.
   - **Result:** A regression on the real path would not turn this assertion red. Examples are a guard or hook that rotates the Webster record, or the verb's implementer read turning into a write.
   - **Fix:** Set `f.implementerSeam = weaverengine.ActiveDeps{AnchorRoot: anchorPath}`. Drive the goto through loom's goto wiring with its guard and admitted hook, or failing that pass them in the `GotoRequest`. Otherwise narrow the doc comment to what the steps can reach.

Tool calls:
1. I grepped `decision.go` for its Webster and implementer-seam use, the loom goto Guard and Admitted wiring, and plankit's card file naming. My context lacked the production seams one hop past the fake and the card ID format.
