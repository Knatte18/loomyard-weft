```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, guards and ordering promises with concrete counterexamples.

**Findings**

1. `BLOCKING`, Decision 7: on Windows, MSBuild nodes die with the gate, so "SDK defaults keep build processes warm" fails on OSL-1046, the platform the task exists for.
   - The tier's tied child on Windows is placed in a job object created with `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE` (`internal/proc/namedjob_windows.go`, `tied_windows.go`).
   - Child processes inherit job membership, so the MSBuild worker nodes and the compiler server that `dotnet build` / `dotnet test` start are killed when the job closes or is terminated at the end of each tier.
   - "Idle nodes reparent away … neither the tier's normal end nor `runproc.End` reaches them" holds only on Linux (Setpgid plus Pdeathsig to the shell alone).
   - On Windows every gate call starts nodes cold.
   - The record must either design breakaway (a job that permits `CREATE_BREAKAWAY_FROM_JOB`, or the build server started outside the job) or drop the claim. In either case it should say which.
   - Decision 7 also lists the Windows teardown lock as a risk, but that risk cannot arise while job close kills the nodes.

2. `BLOCKING`, Decisions 4 and 9 (green record): a filtered run can be recorded as whole green.
   - `green.go` records an invocation whole "when it ran every target of its tier". Outside the go scoper (`scoperGo` false, so `dotnet` too) a non-whole run records nothing, and a whole one records whole.
   - Counterexample: a hub whose `all_targets` is its single test project, e.g. `Tests/Tests.csproj`, a common layout. A `card` gate narrows to `{targets}=Tests/Tests.csproj` plus `{filter}` with class A.
   - The targets equal `all_targets`, so the pass is written whole. `webster`, `merge-in` and `finalize` on the same tree then carry the tier, and the full suite never runs.
   - The record never says that a run carrying scope filter terms is narrowed, never whole. It must state that such a run writes no green entry.
   - It must also state that a narrowed `dotnet` pass never uses the go scoper's package-entry form, which holds today only because `scoperGo` is false.

3. `BLOCKING`, Decisions 2 and 3, card form: a card whose Targets map to no `.cs` file seeds nothing, and gets an empty scope that runs nothing.
   - A C# Create handle is `plan:<namespace>#<owner.member>`. A slash-free self glyph reads as a namespace, and `diskPathForRef` gives it no disk path (`internal/planparser/doc.go`, `internal/planglyph/doc.go`).
   - A card that creates members through namespace handles, or targets a namespace, yields no file.
   - Decision 3 sends only a deleted Target to whole. A Target with no resolvable `.cs` file is not covered, so the closure is empty and the card gate runs no tier.
   - Fix: a Target with no file falls back to whole, or the handle resolves to the file the card wrote.

4. `MEDIUM`, Decisions 2 and 3: a changed `.cs` file that declares no type seeds nothing, so a compile-breaking change can run no tier.
   - Examples: `GlobalUsings.cs` (`global using …`), `AssemblyInfo.cs` (`[assembly: InternalsVisibleTo(...)]`, whose removal breaks the test project's compile), top-level-statement files.
   - Such a change either selects no class (empty scope, nothing runs) or misses every affected class.
   - It belongs in Decision 3's fallback list.

5. `MEDIUM`, Decisions 3 and 6: the empty scope also skips the build, so a card that breaks the compile passes its card and batch gates.
   - Decision 6 runs the build only when an opted-in tier's command will run. A closure that selects no class (an extension-method-only change, or findings 3 and 4) runs nothing.
   - The go scoper's card scope always compiles the card's packages. Here compile errors surface only at the `webster` end gate, after every batch has stacked on the break.
   - The stated bound ("misses caught by whole gates") covers missed tests, not a missed compile.

6. `MEDIUM`, Decision 6: the record never says how the build's timeout and the tier's timeout overlap.
   - gatecli's existing rule is that the tier's `timeout_min`, "counted once its slots are held, bounds the prebuild and every invocation".
   - The build is also "bounded by its own `timeout_min` counted from the hold". So whether the build eats the first opted-in tier's budget is unstated. Example: build 8 min, tier `timeout_min: 10`, leaving the tests 2 min.
   - The record must say whether the tier's count starts after the build.

7. `MEDIUM`, Decision 5: `inert_paths` is outside the tier fingerprint, so widening it temporarily leaves lasting false greens.
   - Counterexample: a hub adds `*.json` to `inert_paths`. A JSON test-data change is carried and a whole entry is written for the new tree, then the hub removes the pattern.
   - That entry stays whole green for a tree whose tests never ran on the change. It is found as a base for computed scopes and carries at whole gates.
   - Either hash `inert_paths` into the fingerprint of the entries it carried, or mark carried entries as not serving as a base.

8. `LOW`, Decision 2: test detection by the literal attribute names misses `[TestAttribute]`, `[NUnit.Framework.Test]` and `[Test, Category("Slow")]` unless the lexer handles the suffix, qualifier and list forms.
   - A missed class is never selected, which is under-selection, against the record's "never under-matching" claim.

9. `LOW`, Decision 5 under the go scoper: the record does not say whether an inert-only empty scope still plans the guard step.
   - Today a Markdown-only empty scope runs only the guard step. It does not apply to loomyard (empty list) but is unstated for a Go hub that sets the key.

**Claims attacked**
- Decision 2, name closure over-selects safely: held, apart from findings 4 and 8.
- Decision 2, partial classes, inheritance, interface implementations: held.
- Decision 3, fallback list complete: broken (findings 3 and 4).
- Decision 3, `_lyx` files in the warp diff forcing whole every time: held, since `_lyx` sits behind the weft junction and is excluded from warp git (loomyard's narrowed Go scopes work the same way).
- Decision 4, `{filter}` composition and the over-length fallback: held.
- Decision 4, narrowed `dotnet` runs not counted whole: broken (finding 2).
- Decision 5, inert carry over an ancestor: held for tree-to-tree diffs; broken across a change to `inert_paths` (finding 7).
- Decision 5, `none` scoper base resolution: held.
- Decision 6, build once, failure fails the remaining opted-in tiers, fingerprint includes `build`: held.
- Decision 6, timeout bound: unstated (finding 6).
- Decision 6, empty scope runs no build: holds as written, but it is a gap (finding 5).
- Decision 6, `--no-build` never restores: held.
- Decision 7, warm nodes: broken on Windows (finding 1).
- Decision 7, `runproc.End` does not reach nodes: held on Linux; groups are per tier through Setpgid.
- Decision 8, `slots: all` blocks new gate runs, including non-tier holders, while agents keep working: held.
- Decision 8, `--wait` keeps the drain: held.
- Decision 8, `carry: false`: held.
- Decision 9, the slow tier carries on the same tree: held, given finding 2 is fixed.

**Further tool calls**
- Grepped `internal/gatecli/green.go`, `run.go` and `tierrun.go` for scoper branches, to see how the green record treats a non-go scoper; my context had only the doc's summary.
- Grepped `internal/proc` and `internal/gatecli` for job-object, Pdeathsig and Setpgid use, to check the tied child's lifetime on Windows; the proc doc does not mention job objects.
- Read `internal/planparser/doc.go` on the csharp glyph alphabet and the `CardTargetDirs` header, to check how C# card Targets map to files; my context lacked the target-to-path mapping.
