```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (lens fix-code-correctness).

**Findings**

- **LOW** — `internal/gatecli/redundantmutants.go`, the cover pass in `runMutants`. The F2 change passes with the defect in place.
  - A cover that is also a candidate now has its kill cleared and rerun alone.
  - In the fixture, TestDoubleChecked is a candidate and a cover of TestAll. It kills the Double mutant, so it is queued for a rerun.
  - With the old `ran[id][name]` skip, the kills and verdicts come out the same, because a rerun alone of an undisturbed test gives the same result.
  - Nothing in `TestGateRedundant` pins the rerun. The only related assertion is `Runs == 0`, and the run count depends on parallelism.
  - So F2, and the "covering kill rerun alone" part of F12, have no test that fails when the change is reverted. No deterministic test is easy here. A unit-level fake of `runJobs` would be needed, which the project's no-mocks-of-own-code rule discourages. The fixer should say so in the report.

- **LOW** — `redundant_integration_test.go`, `redundancy_test_timeout_sec: 4`.
  - The 4s value now bounds the baseline run and every mutant run of every test, because `mutantTimeout` is capped at `testTimeout`.
  - On a heavily loaded machine, a slow but healthy mutant run could time out. That counts as a kill, and it would change the pinned `Unique` and `MutantsKilled` values.
  - This is a flake risk, not a defect seen in the run. It passed.

**Traced and correct**
- `ScanSourceReaders`, walking the review's F7 case.
  - Starting from inner, outer is reached at depth 1 and finds the kit, so outer is memoed true and inner is true.
  - Starting from outer, inner returns `(false, 0)`, is left out of memo, and a later root calling inner reads outer's memo and gets true.
  - The ping/pong cycle: pong is not memoed, and ping is memoed false because its lowest depth is 0, which is not below its own depth.
  - `TestKitBehindCycleFromInner` fails with the old code on about half of the 16 draws, so the loop catches the defect almost surely.
  - `TestImportedFunction` fails with the old code: `other.Run` resolved to `scanner.Run`, which reaches the kit.
- `run` and `runRedundant` agree on indexing: `coverings[j]` is aligned with `contained[j]` and `mutantVerdicts[j]`.
- Every early return in `run` carries four values, and `coverings` is also set on the `len(mutants) == 0` path.
- Clearing kills only after the whole cover loop has chosen its jobs is correct. Clearing inside the loop would have stopped a candidate that is also a cover from expanding its own covers.
- `mutantTimeout` and the timeout-as-kill count are pinned by `MutantsKilled: 7`. Without timeout-as-kill the dropped `n -= 1` mutant would not count, giving 6.
- The new `buildMutants` drop loop is pinned by `Unbuildable == 1` and `Mutants == 12`.

No tool calls were made, since the context already held every changed file.
