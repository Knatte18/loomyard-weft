```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff adds or changes, I ran one concrete input through the code and recorded whether the result is correct.

**List**
- `gitrepo.WriteTreeAtRevision` / `writeBlobFile`, a tree holding a `100755` file: **wrong.** `file.Mode.IsRegular()` is false for `filemode.Executable`, so the file is skipped and the 0o755 branch never runs.
- `WriteTreeAtRevision`, rev `"not-a-sha"`: correct, it returns `ErrInvalidSHA`.
- `WriteTreeAtRevision`, a write failing midway: correct. It leaves a partial tree, and the caller's `defer os.RemoveAll(dir)` cleans it up.
- `fabricengine.ExportCodeTreeAt`: correct, it wraps the error.
- `dotnetimpact.ChangedTests`:
  - empty base with one new test: correct.
  - overload `M(int)` added beside an unchanged `M()`: correct, one entry is returned.
  - two new overloads of `M`: two entries with the same `Method`, which gives a duplicate label. Cosmetic only.
- `qualifiedName` with nested `Outer+Inner` and an empty namespace: correct.
- Parser `testMethod`:
  - generic `Check&lt;T&gt;()`: correct.
  - `(` with no identifier before it, where `matchingBackward` returns -1 and nameIndex is -2: guarded, correct.
  - method that never ends (EOF): `methodEnd` returns the last index, correct.
- `attributeList` with stacked lists (`memberStart` kept at the first list): correct.
- `endMember` at `{`, `}`, `;` and `typeDecl`: `[Test] int P { get; }` is not recorded, correct.
- `dotnetRedFirstCall.run`:
  - all tests abstract: unjudged exit, correct.
  - none changed: no-test exit, correct.
  - no `{targets}` tier: unjudged, correct.
  - `exportBase` error: exit 1 with a way forward, correct.
  - `keepHeadProjectFiles` error: exit 1, correct.
  - signal: 128+n, correct.
- `run` → `runTest` with `checkoutRoot`: `request.root` swaps after `openGreen`; the build and tier dirs resolve in the checkout, correct. Side effects: run records carry the temp checkout as `Worktree`, and `touchedFiles`/`untrackedWorktreeFiles` fail softly (logged) because the checkout is not a git repo. Acceptable.
- `judgeDotnetRedFirst`:
  - exit 0: green. **Wrong when the filter selected nothing** (finding 2).
  - named failure: correct.
  - compile error in the project: **over-broad** (finding 3).
- `failedNameMatches` with `"Scales(2)"`, `"Ns.C.AddsTwo"` and `"AddsTwoX"` against `AddsTwo`: correct.
- `projectCompileError`: absolute MSBuild path under projectDir, with and without the trailing `[proj]`: correct.
- `filterTerms`: a duplicate term is removed by Sort and Compact. Not empty, since `selectable` is non-empty.
- `nearestProject`:
  - `"T.cs"` at root with no project: reason returned, the loop ends at `"."`. Correct.
  - unreadable directory: the error is swallowed and the search walks up to an ancestor project. Minor.
- `keepHeadProjectFiles`, a deleted file: removed from the checkout and counted. Correct; the count also includes no-op removals (cosmetic).
- `byProject`: correct.
- `gateslot.TierFailureNames`:
  - a build header resets `underTier`: correct.
  - `ReportOutsideAnyTestSuffix` lines are dropped: correct.
  - cut line: correct.
  - no report: nil, correct.
- `redFirstVerdictExit`: a nil `extra` is harmless in `maps.Copy`. The Go path's behaviour is unchanged.
- `runRedFirst`: a go hub follows the same branch as before; a dotnet hub skips `redundancyModule`. Correct.
- `scopeInvocations`: terms present mark the invocation partial, and with no terms it is unchanged for Go. Correct.

**Findings**

1. **BLOCKING · design · `internal/gitrepo/blobread.go` (`WriteTreeAtRevision`, the `if !file.Mode.IsRegular()` line)**
   - Executable blobs are silently left out of the base checkout, which contradicts the doc ("An executable file is written with mode 0o755") and fabric's `ExportCodeTreeAt` doc ("keeps its executable bit").
   - Evidence: go-git `plumbing/filemode/filemode.go:149-152` defines `IsRegular()` as `Regular || Deprecated` only. That makes the `filemode.Executable` branch in `writeBlobFile` dead code.
   - Impact: any `.cs`, `.csproj` or script committed as `100755` is common in repositories edited from WSL or Git Bash. Such a file is missing from the base, so the base build fails, or a missing test file reads as a compile error and gives a false red.
   - Fix: filter on `file.Mode == filemode.Regular || file.Mode == filemode.Deprecated || file.Mode == filemode.Executable`, or `IsFile()` minus `Symlink`.

2. **MEDIUM · design · `internal/gatecli/redfirstdotnet.go` (`judgeDotnetRedFirst`, `if exitCode == 0 { return green }`)**
   - Exit 0 counts as green even when the filter matched no test. With VSTest, `dotnet test --filter` that matches nothing exits 0.
   - Example: a generic fixture `class T&lt;X&gt;` produces `FullyQualifiedName~N.T.M`, but the real name is `N.T\`1.M` or `N.T&lt;X&gt;.M`. Name forms in the summary's "Bounds" fail the same way.
   - The gate then holds the writer with "passed on the base, does not show the bug". That halt is one the writer cannot act on, which breaks the board entry's "Halt only when … a reason the writer can act on".
   - Fix: treat a passing run that discovered zero tests (the tier summary or counter's discovered count) as unjudged.

3. **MEDIUM · design · `internal/gatecli/redfirstdotnet.go` (`judgeDotnetRedFirst`, the `projectCompileError` branch returning `red: tests`)**
   - A compile error in any file of the test project makes every changed test of that project red, not only the test that fails to build.
   - Example: a feature test in `NewFeatureTests.cs` reads a member the change adds, while the bug test in `BugTests.cs` builds and passes on the base. The verdict is red, exit 0, and the gate passes without the bug test ever failing.
   - This goes beyond the entry's "a test that does not compile … because it reads a member the change adds, as red".
   - Fix: make red only the tests whose `File` matches the erroring `.cs` path in the checkout, and judge the rest as unjudged or green.

4. **LOW · design · `internal/gatecli/redfirstdotnet.go` (`nearestProject`, `entries, _ := os.ReadDir(...)`)**
   - A read error other than "not found" is swallowed and treated as "no project here", so the search climbs to an ancestor `.csproj` and groups the test under the wrong project.
   - Fix: return a reason for any error that is not `fs.ErrNotExist`.

Tool calls:
1. go-git `filemode.IsRegular` and `IsFile`, plus `verbDiffBase`, `validSHA` and `HoldsTargets`: the manifest did not hold go-git's mode predicates or how `--base` is resolved.
2. `shaPattern`, the body of `verbDiffBase` and `treeForRev`: needed to check whether a short `--base` SHA reaches `WriteTreeAtRevision` as given. It does, through the same path the existing `CodeFileAt` uses, so I raised nothing.
