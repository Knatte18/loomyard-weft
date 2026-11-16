```yaml
lens: code-correctness
outcome: reported
```

Code-correctness lens: I traced each changed function and branch with a concrete case, including empty, nil, zero, boundary and failure inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `runRedundant`, no contained test: returns before the stale check and the slot, with no `mutation` key. Correct.
- `runRedundant`, map `Files` differ from the worktree: refuses with the sorted changed names. Correct.
- `runRedundant`, `check.run` fails after a signal: the cancelled message, then 128 plus the signal number. Correct.
- `runRedundant`, `check.run` fails with no cancel: prints the failure message. Correct.
- `runRedundant`, the contained loop: `mutated.Reason` goes to `not_judged`, `Redundant` to a candidate, and the default branch always has `Unique` non-empty. Correct.
- `runRedundant`, the `covering` field it prints is the coverage pass's list, not the list the mutation verdict used. Wrong (F1).
- `sourceCheckingTests`, package `.` with module rel `.`: `packageDirOf` gives `.`, which `FindGuardTests` handles; the guard names merge into the reader set. Correct.
- `changedGoFiles`: nil map against non-empty current lists every file; a file on one side alone is listed; equal maps give nil. Correct.
- `mutationCheck.run`, plan with no mutants: skips the build, and every candidate gets `ReasonNoMutant` from a nil `Kills`. Correct.
- `mutationCheck.run`, every mutant of a candidate's blocks dropped: `Mutants` is empty, so `ReasonNoMutant`. Correct.
- `mutationCheck.run`, a cover pick when a block is covered only by a source-check test: the pick is partial, so a kill there refutes. Conservative, correct.
- `runMutants`, baseline: a skipped or failing test gets no timeout. A candidate then gets `FailsUnmutated`; a cover never runs, so the candidate can only be refuted. Conservative, correct.
- `runMutants`, a cover that is also a candidate and already ran the mutant in the candidate pass: `ran[id][name]` skips it, so its unconfirmed kill stands. Wrong (F2).
- `runMutants`, a cover whose entry misses the mutant's block: skipped, which is right because it cannot execute the site. Correct.
- `mutantTimeout`, `testTimeout` 0: no cap. With elapsed 0.1 the bound is the 10 s floor. Correct in production; mixes units (F5).
- `planMutants` / `ProfileBlocks`, timeout and non-zero exit: each returns an error. `c.tiers` is never empty after `redundancyModule`. Correct.
- `buildMutants`, round 1 refused with no line a mutant spans: returns the error. Round 5 still refused: returns the error. A non-compile failure: returns the error. Correct.
- `runJobs` / `runSideBySide`:
  - empty jobs: zero workers, no error;
  - cancel mid-run: `RunAlone` returns a fail result with no error, then `c.ctx.Err()` is returned;
  - a binary that does not start: `errors.Join`.
  
  Correct.
- `runJob`: a tier with `prebuilt_lym` and no `cmd/lym` gives an empty `lymBin`, so no env entry is added. Correct.
- `buildLymFor`: the build-timeout and cancel branches are told apart through `c.ctx.Err()`. Correct.
- `holdSlot` / `close`:
  - inherited slot: nil lease, and `HolderEnv` keeps `InheritEnv`;
  - lease held but `MkdirTemp` fails: `close` releases the lease;
  - `Acquire` cancelled: no lease, and the error is reported as cancelled.
  
  Correct.
- `PlanMutants` / `fileSites`:
  - nested sites (literal in condition, literal in dropped statement) and no target in a file;
  - const decl, array length, composite key and tag literals stay fixed: pre-order Inspect marks them before visiting the literal;
  - for init and post statements are not dropped.
  
  Correct.
- `PlanMutants`, a parenthesised `if (a < b)`: yields both a condition mutant and a comparison mutant on the same expression. A harmless duplicate, correct.
- `applyMutants`: same-offset prefixes go outer first and suffixes inner first, and no newline is inserted. Correct.
- `droppable`: `panic`, `:=` and blank-only assignments are excluded. Wrapping keeps every reference, so dropping never makes a variable or import unused. Correct.
- `UnbuildableMutants`: a dropped mutant stays dropped, another file's line matches nothing, and a non-`MutantBuildError` returns nil. Correct.
- `JudgeMutants`, nil kills:
  - `FailsUnmutated` before `NoMutant`;
  - `Killed` empty gives `KillsNone`;
  - the removable greedy pass orders by seconds then name, the same order as the coverage pass.
  
  Correct.
- `JudgeMutants` with `runRedundant`: `Removable` is the coverage-removable set intersected with the mutation-removable set. Both properties hold for any subset of their set, so the intersection is sound. Correct.
- `RunAlone` with profile `""`:
  - pass;
  - timeout gives `ActionFail` with `TimedOut`;
  - `os.Exit(0)` with no `--- PASS` line reads as a fail, so a kill.
  
  Correct.
- `ProfileBlocks`: `ErrWaitDelay` is cleared before the timeout and failure checks. Correct.
- `BuildMutantBinary`: an error without file:line gives a plain error; lines are taken as base names. Correct.
- `ScanSourceReaders`:
  - direct reference, through a helper, through a method;
  - a two-function cycle where the kit reference comes after the call into the cycle gives a false negative (F3);
  - an imported `pkg.F()` selector resolves to same-named methods of the package (F4).

**Findings**

**F1. MEDIUM, design.** `internal/gatecli/redundant.go:257` and `:260`: the printed `covering` of a candidate or refuted test is not the set the mutation check judged it against.
- Both lines print `verdict.Covering`, the coverage pass's list. For a removable test that list comes from the tests the greedy pass keeps (`internal/testcover/verdict.go:44`, `:111`), and it can name a source-check test.
- `Unique` and the redundancy verdict are computed against `testcover.CoveringTests` over `covers` (`internal/gatecli/redundantmutants.go:138`), and `run` never returns that list.
- A refuted line can therefore name a "covering" test that was never run against its `unique_mutants`, and leave out the cover that was. A confirmed candidate can list covers other than the ones that killed its mutants.

**F2. MEDIUM, design.** `internal/gatecli/redundantmutants.go:219`, `:222`, `:235`: a covering test that is also a candidate keeps a kill it made beside other runs, and that kill can confirm another candidate falsely.
- The candidate pass runs with `rerunFailed` false (`:222`) and sets `ran[id][candidate.Name]` (`:219`).
- The cover pass then skips any cover with `ran[id][name]` set (`:235`). So a candidate K that is also another candidate's cover keeps a kill made beside other runs, without the solo rerun.
- That kill counts as the cover's kill in `JudgeMutants`, so a disturbed run can make the other candidate look redundant.
- This contradicts the documented contract "A covering test that kills beside other runs runs once more alone, and that run stands" (`redundantmutants.go:165`, `internal/gatecli/doc.go:1151`). The documented bound covers only false refutation, not false confirmation.

**F3. LOW, design.** `internal/testcover/sourcescan.go:31`: a function in a call cycle can be cached as not reaching the scan kit when it does.
- `memo[fn] = false` is set while a function is in progress.
- Take A whose body calls B and then references the kit, and B that calls A. B reads A's in-progress `false`, finds nothing itself, and its final `memo[B] = false` sticks. A later turns true.
- A test that calls B is then not marked, and gets judged by mutation. That is the false confirmation the research report found for source scans.
- The test's cycle row (`ping`/`pong`) has no kit reference, so it never reaches this order.

**F4. LOW, design.** `internal/testcover/sourcescan.go:47-48`: a call to an imported package's function is matched against the package's own methods.
- Any selector call, `strings.Join(...)` included, is looked up in `src.methods` by selector name. So a test calling `otherpkg.Run()` is marked a source reader whenever a same-named method of the package reaches the kit.
- The test is then wrongly listed as not judged. `spawnscan.analyzeCall` skips import selectors (`internal/testcover/spawnscan.go:258-261`).

**F5. NIT, consistency.** `internal/gatecli/redundantmutants.go:252`: the floor and the factor of the mutant timeout use different units.
- The floor scales by `c.request.secondUnit` but the factor term uses `time.Second`, so the bound mixes two units.
- No effect in production, where `secondUnit` is `time.Second`.

Tool calls:
1. grep of `testmappackage.go` `BuildBinary`/tags, test-file build tags, and `impactset.FileTier`: to check whether tiered tests need build tags the mutant build omits (they do not). The manifest lacked the testmap build call and `FileTier`.
2. grep of evidence line numbers in `redundant.go`, `redundantmutants.go`, `doc.go` and `sourcescan.go`: to cite exact lines. My context held these files only as diff hunks, without line numbers.
