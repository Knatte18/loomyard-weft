```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism run end to end on a concrete case, with the end state checked against the decision's purpose.

**Findings**

1. **BLOCKING — Decision 2 (card seed), Decision 3, AC "card scope seeds from the `.cs` files the cards' Targets name".** A C# card's member-glyph Targets name no file, so the card gate can silently run no test.
   - Case: a card with `**Targets:**` `NORCE.Models#Foo.Bar`.
   - Under csharp the glyph's unit is a namespace. quarry v0.4.0's `glyph/unitpath.go` says "a C# namespace is not a path".
   - `planparser.diskPathForRef` maps a glyph to a path only for a self glyph (`IsSelf()`). It skips a member glyph, and `CardTargetDirs` builds on that same mapping.
   - So the new planparser sibling gets no `.cs` file for this card, and the seed is empty.
   - The closure then selects no class. Decision 3 makes that the empty scope, so the `card` and `batch` gates pass with zero tests. That defeats Wanted 4's scoped in-batch check.
   - Decision 3's whole fallbacks don't catch it: no deleted Target, no non-`.cs` path.
   - Fix: seed from the owner type names the C# glyphs spell, which needs no file. Fall back to whole for any Target that yields neither a type name nor a `.cs` path, so the scope is never empty for that reason.

2. **MEDIUM — Decisions 4 and 9 (green record under a narrowed `dotnet` pass).** The record never says what a filtered pass writes, and the existing writer can record it as whole.
   - `green.go` records an invocation whole "when it ran every target of its tier". It records packages only under the `go` scoper.
   - Case: a hub whose `all_targets` is its single test project, such as `Tests/Tests.csproj`.
   - A narrowed run has `{targets}` = that project plus a class `{filter}`, so it "ran every target" and writes a whole entry.
   - The next `webster`/`finalize` gate on the same tree carries the full suite. A class-filtered run then stands in for the whole run, against Decision 2's "misses are caught by the whole gates".
   - Fix: a run with a scope class filter never writes a green entry (or never a whole one), and under `dotnet` only whole entries carry.

3. **MEDIUM — Decision 8 (timing tier) against Decisions 3 and 5.** The timing call's default scope can run nothing.
   - Case: `lyx gate test --tier 9 --wait -- <bench args>` with no scope flag, which is the computed form.
   - Under `scoper: dotnet` the derivation runs for tier 9 too. A diff that touches only the benchmark project selects no test class, and an inert-only diff selects nothing.
   - Both give the empty scope, so tier 9's command never runs. The call passes with zero discovered and no measurement is taken.
   - Fix: a `carry: false` (or manual) tier always runs whole, or the decision and `docs/dotnet-hub.md` make the timing call pass `--all`.

4. **MEDIUM — Decision 6 (build inside the tier's slot hold).** The build's time eats the tier's own timeout.
   - Today a tier's `timeout_min` is "counted once its slots are held" and bounds "the prebuild and every invocation" (gatecli doc, Timeouts).
   - The build runs inside that hold and has its own `timeout_min` "counted from the hold". The record never says the tier's clock starts after the build.
   - Case: tier0 `timeout_min: 10`, and a 6-minute build on a loaded XDR machine. tier0's test run has 4 minutes left and times out, even though the build stayed within its own limit.
   - Fix: say whether the tier's clock starts after the build, or that the tier's timeout must cover both.

5. **LOW — Decision 6 (`{args}` in the build command).** What fills the build's `{args}` is never said.
   - If it is the call's words after `--`, test-run words such as a timing tier's benchmark args reach `dotnet build` and break it.
   - Fix: say what fills it, or drop `{args}` from `build`.

6. **LOW — Decision 6 (build failure output).** Whether the build's output streams to the call's own stdout and stderr, as a tier's does, isn't stated.
   - Merge-In and Finalize keep the gate's output as the `verify-<n>.log` the fixer reads. The failing tier recorded there "never ran", so the compile error is visible only if the build output is in that stream.
   - Fix: state that the build's output passes through like a tier's.

7. **LOW — Decision 4 wording.** "would pass the gate's shell-safe argument length" means "exceed". As written it reads the opposite way.

**Decisions walked**

- **D1, held.** Case: a diff changing `Foo.cs` under `src/csharp/NORCE.Models`. `computeScope`'s `dotnet` branch makes one derivation per base and tier `dir`.
- **D2, broken for `--card`, held for diff forms.**
  - Diff case: `FooTests.cs` names `Foo`, so `NORCE.Models.Tests.csproj` runs with filter `~NORCE.Models.Tests.FooTests`.
  - Card case: finding 1.
- **D3, held.** Case: a `.csproj` change falls back to whole. A `.cs` change outside the tier `dir` falls back to whole.
- **D4, held, apart from findings 2 and 7.** Case: `(Category!=Slow)&(FullyQualifiedName~A|FullyQualifiedName~B)`, quoted as one `--filter` argument. At whole scope with no tier filter it renders empty.
- **D5, held.** Case: Finalize after a Markdown-only fix commit on top of a whole-green webster tree. `verifytree` sees a new tree and runs the call. `newestWholeAncestor` plus the inert diff lets the tier carry, and the carry writes a current-tree entry.
- **D6, held, apart from findings 4–6.** Case: `--gate batch` with tier0 opted in. The solution build runs once, then `dotnet test <proj> --no-build`. A carried tier0 with tier1 running puts the build in tier1's hold.
- **D7, held** as a no-op. Nothing lyx sets disables node reuse.
- **D8, broken by the default scope (finding 3).** Held with `--all`, where the drain blocks other worktrees' gate runs.
- **D9, held** for whole passes. Case: webster [0,1] whole, then Merge-In with the parent unmoved, so the tree is the same and both tiers carry. Narrowed passes: finding 2.
- **D10 and D11, held.** Config and docs only.

**Tool calls**

1. `CardTargetDirs` in `internal/planparser/targetdirs.go`: my context didn't show how card dirs come from C# targets.
2. `diskPathForRef` in `internal/planparser/shape.go`: to see how a glyph maps to a disk path, which my context lacked.
3. quarry `glyph/doc.go`: to find what a C# glyph's unit names. The first grep was inconclusive.
4. quarry `glyph/doc.go` again: the same question, still inconclusive.
5. quarry `glyph/unitpath.go` and `glyph/csharp.go`: these confirmed that a C# unit is a namespace, not a path.
6. `internal/gatecli/green.go`: to see what a narrowed pass writes under a scoper other than `go`, which the exploration didn't cover.
