```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guarantee and existing-code claim in the decision record, using counterexamples.

**Claims attacked**
- D1, the derivation is told absolute paths and called once per base and tier `dir`: held.
- D2 Bound, "misses only references that never spell the type's name": **broken** by interpolation holes (F1).
- D2, filter terms are "never under-matching": **broken** by nested test classes (F2) and by test-project attribution (F3).
- D2, the card seed comes from owner chains: **broken** for type glyphs (F4).
- D2, inherited fixtures get their own filter term: held.
- D3 fallback list, the old and new names of a renamed file: held.
- D3 and D5, the changed set from the diff: **broken** for untracked files under `none` (F9).
- D4 `{filter}` quoting on Windows: held. `QuoteArg` double-quotes the value, so `&amp;`, `|` and `!=` are literal under cmd (verifyrun/run.go:65-70).
- D4, a command without `{targets}` runs once, whole, as it does today: held (tiers.go `scopeInvocations`).
- D4, `greenCommandBudget` stays under cmd's limit for one project plus its filter: held.
- D4, an `IsTestProject=false` project tests nothing and exits 0 under VSTest: held as cited, not verifiable offline.
- D5, a dirty tree reads no record (`openGreen`): held. The newest-20 bound and the carry chain: held.
- D5, the `go` guard step on an inert-only scope behaves as for a Markdown-only scope today: held.
- D6, "a failed build is the call's first failing run, so the call exits with the build's code": **broken** (F5).
- D6, `--no-build` implies no restore: held. The `build: true` landing record keyed on the first failing summary line: held.
- D6, a build timeout writes no landing record: consistent with `recordFailure` skipping `TimedOut` (landingshed/verifygate.go:63).
- D7 Linux, the group is killed only on cancel: held. tied_linux.go:18 sets `Setpgid` and kills `-PGID` only in `killPlatform`; `killTreePlatform` kills only through a remaining record.
- D7, warm nodes do not hold the output pipe: held as cited.
- D8, `slots: all` blocks new gate runs: held. The drain lock is released once every slot is held (gateslot/doc.go:15), and every slot stays held, so single-slot acquires wait.
- D8, its three listed skip paths: held.
- D8, existing fingerprints are unchanged: held.
- D10, "Merge-In is the one whole-solution run per task": **broken** (F6).
- Constraint, a narrowed dotnet pass writes no green entry: held as a stated change to `recordPass`; green.go shows today's targets-only whole test.
- Constraint, `build: {}` in the template is shaped like `tiers: {}`: held.
- Open risk, "never more than today": **broken** (F7).
- Acceptance tier1, the node-pipe hazard check: **weak** (F8).

**Findings**

F1 — MEDIUM, design. Location: D2 lexer bullet and Bound, decision-record.md:37 and :48.
The lexer "drops comments and string literal content", but a type spelled inside an interpolated-string hole is real code: `$"{Formatter.Format(v)}"`.
If a test names `Formatter` only there, a change to `Formatter` selects nothing.
That breaks the Bound's own claim that only references which never spell the name are missed.
The rule should lex interpolation holes as code. Evidence: decision-record.md:37, :48.

F2 — MEDIUM, design. Location: D2 result bullet, decision-record.md:45.
The record never says how a filter term is spelled, only that it "matches every test of that class… never under-matching".
VSTest/NUnit spell a nested test class's FullyQualifiedName with `+` (`Ns.Outer+Inner.Test`).
A `FullyQualifiedName~Ns.Outer.Inner` term therefore matches none of its tests.
The term spelling, nested classes included, is a fact the design rests on and is left to the plan. Evidence: decision-record.md:45; the `+` spelling is from general NUnit/VSTest knowledge, not a file read here.

F3 — MEDIUM, design. Location: D2 test-project rule and the auto-mode assumption, decision-record.md:42, :200; D3, :58.
A selected test class can have no project or the wrong one under the "nearest ancestor `.csproj`" rule:
- its file has no `.csproj` ancestor, e.g. under Models' `.scratch/`, which the board puts under the project root and D2's skip list (:37) does not exclude;
- its directory holds two `.csproj` files;
- it is linked into a project by `&lt;Compile Include&gt;` from another directory.

D3 falls back to whole only when such a file is *changed*.
A *selected* class there is silently not run or run in the wrong project, which breaks "never under-matching". Evidence: decision-record.md:37, :42, :58, :200; board entry, "Project root…holds `.scratch/`".

F4 — MEDIUM, design. Location: D2 card-seed bullet, decision-record.md:40; D3, :60.
A C# glyph naming a type, `NORCE.Models#Circle`, parses with `Owner` nil and `Name` `Circle` (quarry v0.4.0 glyph.go:23-44).
D2 seeds only "the type names…in its owner chain", so this Target yields no owner type.
D3 then sends the card gate whole, for an ordinary Target shape.
The rule should also seed `Name` when the owner chain is empty. Evidence: decision-record.md:40, :60; glyph.go Owner/Name fields.

F5 — LOW, consistency. Location: D6 failure bullet, decision-record.md:119; acceptance criterion, :224.
"The call exits with the build's code" does not hold when a tier that did not opt in comes earlier in plan order and has already failed.
`runTest` keeps the first failure's code (run.go, the `code == 0` guard in the plan loop).
Example: tier0 has no `shared_build` and fails, then tier1 opts in and its build fails; the call exits with tier0's code.
Evidence: internal/gatecli/run.go `runTest` loop.

F6 — LOW, consistency. Location: D10, decision-record.md:174; Goal, :5.
"Merge-In is then the one whole-solution run per task" is false whenever a diff touches a non-`.cs`, non-inert path:
- a `.csproj` package reference, `.resx` or test data;
- or a card Target that is a deleted file.

Each of these makes card, batch and webster gates whole by D3 (:57, :60). The claim should carry that condition. Evidence: decision-record.md:57, :60, :174.

F7 — LOW, design. Location: Open risks, decision-record.md:205.
"Never more than today" breaks when the closure selects classes in every test project.
D4 (:77) then runs one `dotnet test` per project, in sequence, inside one tier, each paying startup and discovery.
Today's single solution-level `dotnet test` runs those projects in parallel. Evidence: decision-record.md:77, :205.

F8 — LOW, design. Location: tier1 acceptance criterion, decision-record.md:229.
A "small fixture solution" whose projects form one reference chain may build entirely in-proc and never launch an out-of-proc MSBuild node.
The check would then exercise only VBCSCompiler, not the node-pipe hazard D7 bounds.
The fixture needs projects that MSBuild builds in parallel. Evidence: decision-record.md:141-145, :229.

F9 — LOW, design. Location: D5 first bullet, decision-record.md:97; D2 diff seed, :39.
The changed set comes from `git diff --name-status` (impactset/derive.go:33), which lists no untracked files.
Under `none`, every computed scope is whole today. With a non-empty `inert_paths`, a dirty tree with untracked `.cs` files and only inert tracked changes now runs no tier.
The record does not say whether untracked paths join the changed set.
Verify sites refuse a dirty tree, so only an agent's own computed call reaches this. Evidence: internal/impactset/derive.go:33; decision-record.md:97.

**Tool calls**
1. `verifyrun.QuoteArg`: whether `&amp;`, `|` and `!` in a filter are safe under cmd; the manifest named it but its body was not in context.
2. `gatecli/child.go` `runTiedChild`, `proc/tied_linux.go` and a grep for impactset's diff source: the Linux group-kill claim in D7 and whether untracked files reach the changed set.
3. `gateslot` pool/doc drain lines: the `slots: all` drain claim in D8.
4. `proc/tied_linux.go` `killTreePlatform` and impactset `derive.go` diff lines: the `runproc.End` reach claim in D7 and the source of the changed set.
