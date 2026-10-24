```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee and claim about existing code in the discussion record.

**Claims attacked**
- D2: "a name match over-selects safely … never under-matching." Broken (F1).
- D2 Bound: the closure misses only extension methods, `var`-typed factory results, reflection and source-generated code. Broken (F2).
- D3 Rationale: "narrowing never hides a failure the whole run would show at that gate." Broken by F1 and F2.
- D4: an over-long filter falls back at "the gate's shell-safe argument length". Broken: no such bound exists (F7).
- D5: a hub with an empty `inert_paths` list behaves exactly as today. Held.
- D5: the inert carry obeys the green record's rules (clean tree, `carry: false`, newest 20). Held: `greenWholeCommitsKept`, `openGreen`.
- D5: a non-empty list joins every tier's fingerprint. Held.
- D6: the build runs "inside the slot hold of the first opted-in tier … whose `dotnet` scope is an empty test scope". Broken (F5).
- D6: a failed build is the call's first failing run, and verify sites read it right. Broken for the landing failure record (F6).
- D6: the build's timeout line is read as timed out. Held, as long as the line keeps `tierTimeoutPrefix`/`tierTimeoutMid`.
- D6: `--no-build` tiers never restore. Held.
- D7 Linux: idle nodes outlive the tied child and serve later gate calls. Not established: the record has no evidence that the tier's `Wait` returns while nodes live (F3).
- D8: a `carry: false` tier "always runs, whole", and nothing can leave it unrun. Broken (F4).
- Constraint: a filtered pass never stands in for a whole run. Held: the `recordPass` cases against D4's no-`{targets}` and over-long-filter paths.
- D10: Finalize carries Merge-In's whole run when no commit landed between. Held: `_lyx` is a link into weft, so run records do not change the code tree.
- Auto-mode assumption: VSTest exits 0 on a gated project. Unverified (F8).
- Open risk, MTP: "a selected class always exists, so this arises only for a project the tier does not run". Broken (F9).

**Findings**

F1 — MEDIUM, design, Decision 2, test-class rule. Classes that only inherit their test methods are never selected, so the scope under-matches.
- Counterexample: NUnit `abstract class SolverTestBase { [Test] public void Converges() … }` and `class Pipe1DTests : SolverTestBase {}`. Neither attribute appears on `Pipe1DTests`.
- Under D2, only `SolverTestBase` is a test class. Its filter term `FullyQualifiedName~…SolverTestBase` matches no test, because NUnit names inherited tests `…Pipe1DTests.Converges`.
- The derived class's tests never run at a narrowed gate. MSTest `[DataTestMethod]` is also missing from the attribute list.
- Evidence: decision-record.md:42-44.

F2 — MEDIUM, design, Decision 2 Bound and Decision 3 Rationale. The list of misses is incomplete: dispatch through a base type or interface is missed by design.
- Counterexample: the changed file declares `class Circle : Shape` with `override Area()`. The test names only `Shape` and `ShapeFactory.Create("circle")`.
- The seed is `Circle`. The closure grows only toward files that name `Circle`, so the test is never selected.
- The same holds for DI-resolved implementations and any test that holds an interface.
- So D3's "every case where the name closure cannot see the change's reach runs everything" is false. Ordinary polymorphism, not just the four named exotic forms, is left to the whole gates.
- Evidence: decision-record.md:46-48, 63.

F3 — MEDIUM, design, Decision 7 Linux bullet. The claim that warm nodes outlive the tied child is unchecked against how the tier's child is waited on.
- `runInvocations` sets `cmd.Stdout = counter`, which is not an `*os.File`. Go therefore pipes the output, and `Wait` blocks until every holder of the write end closes it.
- Neither `runTiedChild` nor `proc.StartTied` sets a `WaitDelay`.
- If an MSBuild node or the compiler server inherits that stdout, the tier hangs until the node's idle timeout and then times out. That turns D7's warmth into a gate hang.
- The record cites no evidence on whether the SDK redirects worker-node stdio. That fact is open research the design rests on.
- Evidence: internal/gatecli/tierrun.go:138-140 (`cmd.Stdout = counter`), internal/gatecli/child.go:22-43, internal/proc/tied.go:37-45, 65-71, decision-record.md:124.

F4 — MEDIUM, design, Decision 8. "Always runs, whole" has two bypasses the Decision does not bound.
- (a) `dropCovered` drops a listed tier that a higher listed tier `covers`. A `carry: false` tier covered by another tier never runs.
- (b) At a verify site, `verifytree.Verify` skips the whole `lyx gate test` call when its per-command record names HEAD's tree. A gate key holding a `carry: false` capture or compare tier is skipped on an unchanged tree, for example on a Finalize resume.
- D8's rationale ("a capture on an unchanged tree must write its baseline") depends on neither happening.
- Evidence: internal/gatecli/tiers.go:84 (`dropCovered`), pattern/PATTERN-gate-slot.md "Verified tree" (skip on same command and tree), decision-record.md:133, 139.

F5 — MEDIUM, consistency, Decision 6, first bullet, and the `PATTERN-gate-slot` Constraint. The build cannot run inside the slot hold of an empty-scope tier, because that tier holds no slot.
- `runListed` sends an empty-invocation tier through `runWithSlots(run, true, …)`, and `runWithSlots` skips the acquire when `empty` is set.
- D6 never says the empty-scope opted-in tier now takes its slots for the build.
- Taken as written, the empty-seed build runs unslotted. That breaks the Constraint "the build step runs inside held slots".
- Evidence: internal/gatecli/run.go:353-358, 400-413, decision-record.md:102, 158.

F6 — MEDIUM, design, Decision 6, failed-build bullet. Landing sites misattribute a build failure.
- `landingshed.verifyGate.recordFailure` takes `failure.Tier` from `gateslot.FailedTier(log)`. That parses only `gate test: tier&lt;N&gt; …` summary lines and otherwise falls back to `slices.Max(list)`.
- A build failure prints no tier line of its own. Either the first skipped opted-in tier's failed line, or the highest tier, is recorded as the failing tier, with no failing tests.
- The Merge-In and Finalize fixers are then pointed at a test tier for a compile break.
- D6 does not say how a build failure reaches the landing failure record.
- Evidence: internal/landingshed/verifygate.go:96-104, internal/gateslot/runlog.go:188-203, decision-record.md:107.

F7 — LOW, decision, Decision 4. "The gate's shell-safe argument length" names no existing bound, so the plan writer must invent the limit.
- The only budget in the code is `greenCommandBudget`, 6000 bytes, which applies to rendered targets of a carried remainder.
- On Windows the real limit is cmd's whole command line, which a per-filter cap does not bound.
- Evidence: internal/gatecli/green.go:37-38, decision-record.md:75.

F8 — MEDIUM, design, Auto-mode assumptions, second bullet. The VSTest behaviour on a gated project is assumed, not researched, and the narrowed tier0 design rests on it.
- Case: a narrowed tier0 selects a `NORCE.Models.SlowTests` class and runs `dotnet test SlowTests.csproj --no-build --filter …` without `-p:RunSlowTests=true`.
- The record cites no source for the exit code, nor for whether "no test" or "not a test project" is reported as an error.
- Evidence: decision-record.md:173.

F9 — MEDIUM, design, Open risks, first bullet. The premise is false, and the MTP case is left open.
- Under a narrowed scope the filter is `(&lt;tier filter&gt;)&amp;(&lt;terms&gt;)`. Take tier filter `Category!=Slow` and a selected class whose tests are all `[Category("Slow")]`. The class exists, yet the expression matches zero tests.
- Microsoft.Testing.Platform then exits non-zero for zero tests run, in a real test project the tier does run.
- The risk also hands an MTP hub a known failure without a decision.
- Evidence: decision-record.md:71, 178.

Tool calls:
1. `internal/proc/tied.go` 30-90: `StartTied`/`Wait` and whether a WaitDelay applies to the tier's child (F3). The manifest named the file without its content.
2. `ls -la` of the worktree root and `git ls-files _lyx`: whether `_lyx` commits change the code tree between Merge-In and Finalize (the D10 attack). My context did not show where `_lyx` lives.
