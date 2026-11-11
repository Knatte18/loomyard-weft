```yaml
lens: bound-breaker
outcome: reported
```

I reviewed the record through the bound-breaker lens: I attacked each bound, guard and claim about existing code in `decision-record.md`. Two claims break outright, so D2 to D4 need rework.

**Findings**

1. **BLOCKING, D3 (and D5's bound): testmap does not drop tests that spawn a process.**
   - D3 says every test the map leaves out runs, "such as a test the map dropped for spawning a process". That premise is false.
   - `internal/gatecli/testmap.go` and `testmappackage.go` never call the spawn scan: there is no `Spawn`, `OutOfProcess` or `Unresolved` in either file.
   - The spawn verdict (`internal/testcover/spawnscan.go`) is used only to keep tests out of redundancy judging. So an out-of-process test enters the map with only the blocks it covered in its own process.
   - Counterexample: a tier-1 test of importing package Q runs the prebuilt `lym` binary (`prebuilt_lym: true`), and that binary runs a changed function of P.
     - Q's map records the test without any block of P.
     - D4 does not select it, and D3's "not recorded" rule does not catch it.
     - The test is skipped. That narrows the scope with no map evidence, which breaks the record's own "widen, never narrow" rule.
   - Fix: decide that a test the spawn scan marks out-of-process or unresolved is always selected (or makes Q a miss), whether testmap records it or not. Alternatively, have testmap record the verdict.

2. **BLOCKING, D4 (deferred fact): the gate has no source for the blocks of the working-tree and base files.**
   - `KeyBlocks(dir, blocks []ProfileBlock)` keys blocks only from profile positions. Ordinals are stable only when "every block of the package's profile, covered or not" is given (comment on `KeyBlocks` and `CoveredKeys` in `internal/testcover/blockkey.go`).
   - The profile positions come from a coverage build's instrumentation. D4 has the gate key a changed file's blocks "at the tier's base and in the working tree" with no build (D8 forbids building).
   - The record never says where the block boundaries come from:
     - A re-implementation of cmd/cover's block splitting from the AST.
     - `go tool cover` run on the file.
     - A coverage build at gate time.
   - It also never says how the base file is keyed. `KeyBlocks` reads files from `dir` on disk, and the base content is only in git.
   - The design rests on this fact, and the rubric's deferred-research rule applies.
   - Counterexample: the gate's own split differs from the cover tool's by one block. The gate's base keys then fail to match the map's, and D4 either selects nothing (narrowing) or everything.

3. **MEDIUM, D2: block keys carry no package.**
   - `BlockKey.File` is "the block's file name within its package directory" (`internal/testcover/blockkey.go`).
   - A cross-package block table can hold `config.go|Load|h|0` from two packages, and matching on content would merge them.
   - A test covering `gateslot/config.go`'s `Load` could then be selected by an edit to `configengine/config.go`'s `Load` if the two hashes ever matched. More likely, the plan writer keeps a key without a package and lookups go wrong.
   - D2 should state that a cross-package key carries the package's import path, and that the format marker covers that change.

4. **MEDIUM, D5's file-hash rule against how maps are built.**
   - D2 records hashes "of every Go file of every package holding a covered block". Test files of the importing package are not mentioned, and neither are the test-only `_test.go` helpers of a dependency's `export_test.go`.
   - Q's own test files changing makes Q a changed package, so that case is fine.
   - Not covered: a `testkit` package. It is in-module and linked by Q's tests, but holds no covered block because it is outside coverpkg or has no executed blocks. A change to it is in the diff set, so Q is an importer, and Q's map records nothing of it.
     - D4 selects only tests covering its blocks. If the testkit change alters a fixture's data without changing blocks (a package-level var or const), D5's "outside function bodies" rule widens, which holds.
     - If it changes a block that no test of Q covered at build time but now does, no test is selected. The record's stated bound admits this, and the landing gate catches it, so this holds as bounded.
     - Severity stays MEDIUM because testkit packages change often, and the bound is easy to misread as covering only "uncovered packages".

5. **MEDIUM, D6: tier 1 runs twice.**
   - Tier 1's command is `go build ./... && go vet {targets} && go test ...`. Two invocations per tier run `go build ./...` twice and vet twice.
   - That breaks no correctness bound, but it eats part of the saving D1 is for.
   - D6 should say whether the selected invocation's build and vet steps can be skipped, or whether the run stays one invocation.

6. **LOW, D6: the `-run` union can be very long.**
   - A long union of test names can pass `greenCommandBudget`'s byte limit or the shell's line limit. `green.go` already falls back to a whole run past the budget.
   - D6 should state that an overlong pattern runs the selected packages whole, which widens and so stays safe.

7. **LOW, D8: the torn-read claim is moot.**
   - `WriteMap` writes through `state.WriteJSON` under `lock.TrackedFileLockPath`, which is an atomic replace, so a torn read should not occur.
   - The miss rule still holds as a fallback.

**Claims attacked**
- D1, "never adds a package `diff` would not run": held.
- D1, the bound that only tests of importing packages are skipped and the landing gate runs all: held.
- D2, "`BuildBinary` already takes a comma-separated coverpkg list": held (`internal/testcover/runner.go`).
- D2, "redundancy verdicts do not change": held, if the judgment filters with `PackageBlocks`.
- D2, cross-package keys: broken (finding 3).
- D3, tests the map leaves out always run: broken (finding 1).
- D4, the gate keys base and working-tree blocks: broken, no source for the blocks (finding 2).
- D4, a removed or added block selects the right tests: held, given keys.
- D5, the miss list widens: held, except for spawning tests (finding 1).
- D5, the bound "drift in uncovered packages only": held as stated, but easy to misread (finding 4).
- D6, "only adds tests": held.
- D6, the green-record guard: held.
- D6, cost: weakened (finding 5).
- D6, command length: open (finding 6).
- D8, no torn read: held, since writes are atomic (finding 7).
- Constraint, Go 1.26 multi-package coverpkg: held.
- Constraint, the strict loader refuses `coverage` on an old binary: held.

**Tool calls**
- Read the first 80 lines of `internal/testcover/blockkey.go` and `WriteMap`: my context had only the agent's summary of the key fields and nothing on how writes are made atomic.
- Read the head of `internal/testcover/spawnscan.go` and grepped `profile.go`: I needed to know what the spawn verdict marks and where block positions come from.
- Grepped testmap and coveragemap for spawn-verdict use: I needed to know whether testmap drops out-of-process tests, which the agent's summary left unclear.
