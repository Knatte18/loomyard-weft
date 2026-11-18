```yaml
lens: row-validity
outcome: reported
```

Row-validity lens: whether each planned test row can reach its branch, fails when the code is wrong, and is needed.

1. **MEDIUM — Card 6 (D3 budget): the zero-budget row of `TestGateTestReachStep` cannot be reached through a full gate call.**
   - Card 5's `decodeConfig` refuses `reach_budget_min` below 1, so gate.yaml cannot set it to zero.
   - The call's minute unit is `testRequest.timeoutUnit`, which also scales the tier timeouts. Shrinking it turns a tier red, and a red tier makes the step judge nothing, so the budget branch is never met.
   - The redundancy step reaches its own budget branch through a hand-built step with a `minuteUnit` field: `stepAt` at `internal/gatecli/redundancy_integration_test.go:236-247`, asserted at `:392-401`.
   - Card 6 gives `reachStep` no such field and no direct-drive helper. Fix: name a `minuteUnit` field on `reachStep` and drive the budget row through a hand-built step.

2. **MEDIUM — Card 8: `scriptedGate` cannot record the re-run mark, and the card does not list the harness change.**
   - The planned subtest asserts that the inner gate's first call saw `On()` false and its second saw it true.
   - `internal/gatefixer#scriptedGate` (`internal/gatefixer/rowgate_test.go:83-93`) holds only `results`, `err` and `calls`. It holds no reference to the entry's `Rerun` and records nothing per call.
   - The card lists `scriptedGate` under Uses only. It needs to move to Edit (gaining a `*shuttleengine.RerunMark` and a per-call record), or the card needs a new fake.

3. **MEDIUM — Card 13 (D9): no fixture can feed the "logger call gets no mutant" row.**
   - `planFixture` (`internal/testcover/mutant_test.go:164`) writes `mutantSource` (`:19`, `package fix`), which holds no call into `internal/logger`.
   - The card lists `planFixture` under Uses only.
   - Adding a logger import and call to `mutantSource` would shift the block index constants and line expectations `TestPlanMutants` pins.
   - Fix: list a new fixture source or helper (or `planFixture` under Edit) carrying an `internal/logger` import and call. `fileSites` parses without go/types (`internal/testcover/mutant.go` imports none), so recognising the import path works in a temp directory.

4. **LOW — Card 2 (D1 index criterion): the stale-index row of `TestReaderIndex` passes even if the reader trusts a stale index.**
   - The row asserts only that each row's answers equal the no-index row's.
   - That fails when the code is wrong only if the rewritten map changes a reach pair.
   - It also needs the rewrite to change the file's size or modification time (for example with `os.Chtimes`). By the card's own bound, a same-size rewrite within one time tick keeps its stamp.
   - Fix: state both conditions on the row.

5. **LOW — Card 9: `TestBatchGateSpec_ReachTurnOnTheBatchEntry` needs its verify directory outside the git worktree or ignored.**
   - `verifytree.Verify` returns `StatusDirty` and runs nothing on any untracked path (`internal/verifytree/verify.go:334-339`).
   - A `Geometry.VerifyDir` (`internal/weaverengine/geometry.go:91-92`) inside the `gitkit` worktree would therefore fail the `card` entry as dirty, not pass it.
   - The card does not say where the directory goes.

6. **LOW — Card 12: the planned `TestVerifyGate_FailedEvaluationWritesReport` row needs a new log fixture the card does not name.**
   - The test's `gateFake.logs` holds `failingPackageLog` (`internal/websterengine/verifygate_test.go:348-358`), which is raw go test output.
   - A row whose output carries a triage line needs a log holding a failure report, and the card should name it.

7. **LOW — Card 11 (test economy): one `TestFailedTests` row repeats existing rows.**
   - The row is "the same report at the line cap with triage dropped". A handwritten report from which the triage lines were dropped is the plain report the existing rows already parse (`internal/gateslot/failedtests_test.go:68-111`).
   - The at-cap drop is pinned by the new `TestRenderFailureReport` cap row next to the existing cap case (`internal/gatecli/failurereport_test.go:144-160`).
   - Fix: keep the "holding triage lines" row and cut the at-cap one.

8. **LOW — Card 14: `TestGateMutate` leaves its toolchain open.**
   - It says "a stand-in or real `go` as those tests use". Its kill, survivor and timeout rows depend on building and running real mutant binaries, so the choice decides whether they can reach their branches.
   - Fix: name the toolchain the test runs on.

Tool calls made:
- Read `scriptedGate`, the `mutantSource`/`planFixture` lines, whether `mutant.go` uses go/types, the prime use in the map tests, and the redundancy budget seam. My context lacked the harness and fixture definitions.
- Read the map stand-in's build branch, the `TestGateRedundant` toolchain, whether the webster and weaver tests exist, the shedrecipe verify test, and the existing cap rows. These were needed to judge whether the rows reach their branches and whether any repeat.
- Read whether the shedrecipe test covers verify, the webster test's fake and log, the weaver Geometry fields, and the webster gate's seams. My context lacked the fixture shapes.
- Listed the test names in `entries_gate_test.go`. I needed to confirm that the named test holds the verify block (it does, from line 110).
