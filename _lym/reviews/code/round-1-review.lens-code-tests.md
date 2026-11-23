```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **Production behaviour added or changed:** none. The diff adds one test file, `internal/loomcli/amendduringwebster_test.go`, so no production behaviour lacks a test.
- **The test's claim that Webster's record keeps its bytes through `decisionVerb` with `--amends-plan` and the goto:** no row would fail if the real path broke. This claim is pinned only against fakes (finding 1).
- **The test's claim that the Plan-Burler-Fix plan gate passes over the done cards' landed Creates:** `TestAmendsPlanDuringWebster_KeepsDoneBatches` pins it.
  - The pin is real. Without `state.json`, the targets `sub#Foo1` to `sub#Foo4` exist in `sub/a.go`, so they would report `create-already-exists`. `internal/loomshed/gates_test.go:597-610` (`NoRunRecordFailsCreateAlreadyExists`) shows that is what happens.
  - `TestPlanGate`'s `DoneCardCreateThatExistsPasses` (`gates_test.go:581-595`) and `PendingCardIsStillChecked` (`gates_test.go:612-635`) already pin the same behaviour (finding 2).

**Findings**

1. **MEDIUM**, design. Location: `internal/loomcli/amendduringwebster_test.go:72-120`.
   - **Summary:** the byte-equality check on Webster's record cannot fail through any code the test drives, so it guards nothing on the reported path.
   - **Evidence:**
     - `newDecisionFake` points the decision record and `implementerSeam` at other temp dirs (`decision_test.go:243-255`).
     - The fake's `deps()` writes only `f.amendmentPath` (`decision_test.go:369-384`), which is `anchorPath/amendment.yaml`, a different file from Webster's record.
     - The real wiring passes `anchorImplementerSeam(target)` (`decision.go:1647`), which the test never uses.
     - The goto calls bare `shedengine.Goto` with a hand-built producer list and no `Guard` or `Admitted`. That path writes only under `runDir` (`shedengine/goto.go:127-158`).
     - Loom's real goto attaches `prReviewGotoAdmitted` (`arm.go:479`, `prreviewgoto.go:64`) and `gotoGuardEndingFixers` (`fixerendguard.go:54`), and the test drives neither.
   - **Effect:** if a change to loom's goto hooks, its recipe wiring or the verb's real deps came to remove `state.json`, this test would still pass.
   - **Fix:** drive the goto through the loom CLI's goto wiring, with the recipe's producers, its `Guard` and its `Admitted`, anchored at `anchorPath`. Set the fake's `implementerSeam` to `anchorPath`. Otherwise, narrow the doc comment at lines 32-36 to the fake-backed `decisionVerb` and the bare `shedengine.Goto` it drives.

2. **LOW**, consistency. Location: `internal/loomcli/amendduringwebster_test.go:122-128`.
   - **Summary:** the plan-gate assertion repeats coverage `TestPlanGate` already has.
   - **Evidence:** `DoneCardCreateThatExistsPasses` (`internal/loomshed/gates_test.go:581-595`) pins that a done card's landed Create passes the gate. `PendingCardIsStillChecked` (`gates_test.go:612-635`) pins that a done card raises no `create-already-exists` while a pending card is still checked. Neither depends on the goto, and this test's fixture adds nothing they lack.
   - **Fix:** keep the assertion only as the end of the end-to-end path, with a `//testtiming:keep` reason naming the record surviving the amend and goto steps. Otherwise, drop it and lean on `TestPlanGate`'s subtests.

**Tool calls**

1. Grep of `internal/loomcli/decision.go` for `implementerSeam` and Webster state reads, and of `internal/loomcli` for the goto wiring. My context held neither the verb's real seam wiring nor loom goto's `Guard` and `Admitted` sites.
