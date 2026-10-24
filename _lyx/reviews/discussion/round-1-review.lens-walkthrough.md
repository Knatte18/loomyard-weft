```yaml
lens: walkthrough
outcome: reported
```

walkthrough lens: each Decision run end to end on one concrete case.

**Walkthrough list**

- **D1, met.** Case: `scoper: dotnet` with `--card 3` on a Models plan. `computeScope` calls `cardPackages` under every scoper before the scoper branch. `CardPackages` only drops non-Go dirs and refuses nothing, so the `dotnet` branch is reached.
- **D2, not met (finding 2).** Case: diff changes `Foo.cs`; `FooTests : SolverTestBase` names `Foo`, and its `[Test]` methods sit in the base class.
- **D3, not met (finding 1).** Case: under `--card`, with no `build` declared, a card renames `Foo` to `Bar` in its Target file.
- **D4, met for `dotnet`, gap for `go` (finding 5).** Case: tier0 is `dotnet test {targets} --no-build {filter}` with `filter: Category!=Slow`, narrowed to one project with one term. It renders `--filter '(Category!=Slow)&amp;(FullyQualifiedName~FooTests)'`, and the same tier at whole scope renders `--filter 'Category!=Slow'`.
- **D5, met.** Case: `inert_paths: ["*.md"]`, Merge-In whole-green at commit C1, then a Code-Review fix touching only `docs/x.md`. At Finalize the green record has no entry for the new tree. `newestWholeAncestor` returns C1, the diff is inert-only, so the tier is carried and an entry is written. A mixed `.md` + `.cs` diff narrows.
- **D6, partly met (findings 3, 4).** Case 1: card gate, tier0 with `shared_build`; the slot is held, the build runs, then `runTier` starts its own clock. That holds. Case 2: empty test scope from a non-empty seed. Case 3: a failed build at Merge-In.
- **D7, unverified (finding 6).** Case: Linux; the build step spawns reused MSBuild nodes while its stdout is the gate's pipe.
- **D8, met.** Case: `lyx gate test --tier 3 --wait -- &lt;bench args&gt;` on a timing tier with `slots: all`, `manual: true`, `carry: false`. The computed form runs it whole, the tier is never carried, and the drain holds the pool.
- **D9, met.** `lenses.yaml` is hub-tunable lens prose that burler fans read (`internal/lenses/doc.go`).
- **D10, met.** Case: `merge-in` and `finalize` are `[0,1] all`. Merge-In passes whole on merge commit M and writes whole entries for M's tree. Finalize's call differs from Merge-In's, so `verifytree` does not skip it. With no new commit, `covered` carries both tiers.

**Findings**

1. **MEDIUM · design · D3 rationale with D2 card seed** (decision-record.md:40, 61-63)
   - Under `--card` with no `build` declared, a card that renames a type leaves the card gate green while a whole run would fail to compile. This breaks D3's claim that "narrowing never hides a failure the whole run would show at that gate".
   - Steps:
     - The card seed reads only the working tree, so it seeds `Bar` (the new name).
     - No test file names `Bar` yet, so the closure selects no test class and the scope is the empty test scope.
     - `scopeInvocations` returns no invocation (internal/gatecli/tiers.go, `case len(sc.modules) == 0`).
     - `runListed` prints "the scope is empty" and runs nothing (internal/gatecli/run.go `runListed`).
     - `FooTests` still names `Foo`, so it no longer compiles, and nothing at the card gate builds it.
   - The diff form avoids this because it seeds from the base version as well. The card form has no base, and the record only covers the compile break when a build step is declared.
2. **MEDIUM · design · D2 test-class rule** (decision-record.md:42-44)
   - A test class whose test methods are all inherited from a base fixture is never selected, which breaks D2's "never under-matching".
   - Steps:
     - `FooTests : SolverTestBase` carries no test attribute and holds no attributed method, so it is not a test class by D2's rule.
     - `SolverTestBase` is a test class, but its file may not name `Foo`.
     - Even if it were selected, its term `~SolverTestBase` does not match the inherited tests, whose names are `…FooTests.Method`.
   - Result: those tests do not run at the narrowed gate, and Decision 3 lists no fallback for this case.
3. **LOW · consistency · D3/D6 empty test scope with build** (decision-record.md:62, 102; internal/gatecli/run.go `runListed`, `runWithSlots`)
   - D6 runs the build "inside the slot hold" of a tier whose scope is the empty test scope from a non-empty seed. Today an empty tier calls `runWithSlots(run, true, …)`, and that skips the acquire when `empty` is set. The record never says that such a tier now takes its slots.
   - The record also never says how the build step tells a seed-empty scope from an inert-only one. Both arrive as an empty `scope.modules`, and D2's stated result (projects and their terms) carries no flag for the difference.
4. **MEDIUM · consistency · D6 build failure readers** (decision-record.md:107-109; internal/gateslot/runlog.go:28, 179-201; internal/landingshed/verifygate.go `recordFailure`)
   - Case: a failed build at Merge-In. The record does not say what summary line the build or the blocked tiers print, or how `FailedTier` and the landing failure record attribute a build failure.
   - Steps:
     - `RunRecord.Tier` is an int, and `TierSummaryLine` prints `tier%d`, so a build record written through the same path reads as `tier0 … exit N`.
     - The opted-in tiers that "fail without running" also print summary lines with non-zero exits.
     - `FailedTier` takes the first such line, so `recordFailure` records a test tier (tier0, or the first blocked tier) with no failing tests for a compile failure.
     - When no line matches, `recordFailure` falls back to `slices.Max(list)`.
5. **LOW · consistency · D4 `{filter}` rendering cases** (decision-record.md:67-71)
   - D4 allows `{filter}` "under any scoper", but its rendering list covers only:
     - whole scope;
     - explicit targets;
     - darn tree-scope targets;
     - narrowed `dotnet` scope.
   - Three cases are left undefined: a narrowed or `--card` scope under the `go` scoper, and a guard-step run of a tier holding `{filter}`.
6. **MEDIUM · design · D7 Linux warm nodes** (decision-record.md:124; internal/gatecli/tierrun.go `runInvocations`, `cmd.Stdout = counter`; internal/proc/tied.go has no `WaitDelay`)
   - The tier's child gets a non-file writer, so Go reads its output through a pipe, and `Wait` returns only when every holder of that pipe closes it.
   - If a reused MSBuild node or the compiler server keeps the inherited stdout open, the build or tier call blocks until the node's idle timeout and then fails on `timeout_min`. It would not end promptly and leave the nodes warm.
   - The record cites no evidence on whether the SDK redirects its worker nodes' streams, yet D7's Linux behaviour rests on that fact.

Tool calls:
- `gatecli.cardPackages` and `planparser.CardPackages`: to check whether the card form under `dotnet` reaches the new branch or is refused first; my context did not hold them.
- `internal/lenses/doc.go` and `tierSummaryPattern` in `runlog.go`: to check D9's lens reach and how `FailedTier` reads a build line; my context had only the `FailedTier` body, not its pattern.
