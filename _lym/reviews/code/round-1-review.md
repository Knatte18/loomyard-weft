---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/amendduringwebster_test.go:66
    summary: The test spells Webster's record file name as the literal state.json, repeating the name websterengine owns as its stateFileName constant, which PATTERN-code-conventions forbids.
    fix: "In internal/loomcli/amendduringwebster_test.go, drop statePath and both os.ReadFile calls (lines 66-70 and 114-120). Read the record before decisionVerb and again after shedengine.Goto with websterengine.LoadState(websterengine.Dir(anchorPath), websterengine.ScratchDir(anchorPath)), failing with t.Fatalf on an error, and assert the second value is non-nil and reflect.DeepEqual to the first, with an Errorf message in the form \"LoadState() after the goto = %+v; want %+v\". Change the doc comment line \"Webster's run record is then byte for byte what the batches wrote,\" to say the run record then holds the same batches the record held before. Add no production accessor."
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/amendduringwebster_test.go:72
    summary: The run-record check holds by construction, because neither decisionVerb's fake seams nor the bare shedengine.Goto with no Guard or Admitted hook is handed any path under the anchor's Webster directory.
    fix: "In internal/loomcli/amendduringwebster_test.go, after f := newDecisionFake(t), set f.implementerSeam = weaverengine.ActiveDeps{AnchorRoot: anchorPath} so the verb's implementer read runs over the planted record, as production's anchorImplementerSeam does (internal/loomcli/decision.go:1647). In the shedengine.GotoRequest, also pass the Guard and Admitted that loom's goto composes at internal/loomcli/arm.go:478-479, built over a loomCLI whose location is anchored at anchorPath, where a tier0 fixture can build one. Where it cannot, leave the goto bare and narrow the doc comment (lines 32-36) to say the record survives decisionVerb over its test seams and the engine's goto, not loom's goto hooks."
    origin: lens:code-correctness
---

### [BLOCKING] Webster's record file name repeated as a literal

**Location:** internal/loomcli/amendduringwebster_test.go:66

**Issue:** `statePath := filepath.Join(websterengine.Dir(anchorPath), "state.json")` repeats the name `internal/websterengine/state.go:66` declares as `stateFileName`, which `SaveState`, `LoadState`, `archive.go` and `inflight.go` all reference.
`PATTERN-code-conventions` requires a name that can go stale to be one named constant, never a repeated literal, and a violation of a listed constraint is BLOCKING.
The repository already avoids this literal for the same file: `internal/loomcli/decision_test.go:233` derives it with `filepath.Base(weaverengine.StateRel())`.
`websterengine.LoadState` reads the record through the owned name, and a value comparison is enough to show that no batch is lost.

### [MEDIUM] The run-record assertion cannot fail through the path it drives

**Location:** internal/loomcli/amendduringwebster_test.go:72

**Issue:** `newDecisionFake` points `implementerSeam` at its own `t.TempDir()` (`internal/loomcli/decision_test.go:253`).
Every other seam the verb calls is a closure over the fake (`decision_test.go:355-394`).
So the only anchor paths `decisionVerb` gets are `amendment.yaml` and the plan approval read.
The goto is a bare `shedengine.Goto` over a status file in a separate `runDir`, with no `Guard` or `Admitted` (`internal/shedengine/goto.go:123,154`).
Loom's goto composes `gotoGuardEndingFixers(settledGuard())` and `prReviewGotoAdmitted()` at `internal/loomcli/arm.go:478-479`.
Nothing between the two reads of the record can reach it, so the byte check passes whatever the production wiring does.
The decision record's first added entry asks for a test that shows the record is not lost on the reported path.
The doc comment, the commit message and `_lym/landing/summary.md` all claim the path is driven end to end, which overstates what the test checks.
The plan-gate half does depend on the record: without it, `sub#Foo1` to `sub#Foo4` resolve against `sub/a.go` and fail with `create-already-exists`, as `internal/loomshed/gates_test.go:597-610` shows for the same shape.

## Rejected

- code-tests LOW, that the plan-gate assertion duplicates `TestPlanGate`'s done-card subtests: the gate step ends the composed scenario, and the subtests do not cover a record that has gone through the amendment and the goto. The redundancy report rates the test unique, so it is neither a redundancy finding nor a candidate for `//testtiming:keep`.
- The missing `Root-Cause:` trailer: the board entry reads as a bug, but the added entry "No test seam: the reported path loses no run record" lands the test as a guard, not as a fix, and the added entries win.
- Commit 7da9d99's `fix:` subject: it splits a comment and claims no bug fix.
- The assistant's alternative of adding an exported path accessor to `websterengine`: that would put a production change outside the guard-only scope of the second added entry, so F1's fix uses `LoadState` instead.

## Focus departures

- "The same test's state-record assertion: settle whether ... the test's Goto uses a stub producer list that never reaches Webster's code."
  Settled as F2. The cause is the missing `Guard` and `Admitted` hooks and the fake's separate implementer anchor, not the stub producer list: `Goto` runs no producer body, whatever list it is given.
