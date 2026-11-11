```yaml
lens: code-correctness
outcome: reported
```

**Task:** check the correctness of the code the diff changes, case by case. This covers the `code-correctness` lens only.

**Case list** (function or branch, the case I ran, verdict)
- `coverageTier`: no `coverage` flag, a `-run` word after `--`, an undeclared tier, `NoCarry`, and a command lacking `{args}` or a target placeholder each returned false. A declared plain tier returned true. Correct.
- `withGateScope`: a coverage key with explicit targets is left alone. A coverage key drops the cards and sets `coverage`. Correct.
- `computeScope`: explicit targets and `--all` return before any narrowing. The `none` and `dotnet` scopers return before the Go branch, so they equal `diff`. A `targets.Whole` fallback skips `narrowCoverage`. An inert-only group skips it too. An empty `Modules` gives a share that runs nothing. Correct.
- `coverageSelections` / `openSelector`:
  - A failure at any setup step becomes a miss, so the package runs whole.
  - `Selector.Select` returning `(Selection{Miss: nil}, err)` becomes `MissMapUnreadable`. Correct.
  - The setup runs lazily and the selections are cached. `DeriveTargets` only builds module `.`, so `ModuleRel: "."` and `selectionOf(pkg)` with a module-relative `pkg` agree. Correct.
  - `request.goBinary` defaults to `defaultGoBinary` (`rungate.go:29`), so it is never empty.
  - Outside a hub the empty `coverageMaps` gives `MissNoMap` and every importer runs whole. Correct.
- `narrowCoverage`:
  - Changed packages and binary builders come out whole.
  - A miss comes out whole.
  - An empty union has no tests and is not whole, so it lands in neither `modules` nor `selected` and runs nothing.
  - Over budget turns only the selected packages whole. It leaves the `none selected` ones alone and sets `names` to nil.
  - A single module is the only case that exists today. Correct.
- `importerPackage`: the miss branch returns before any file read. A read, match or parse failure becomes a whole run. With `Recorded` empty and no miss, every declared test is unrecorded and runs. The `"."` package resolves to dir `"."`. Correct apart from F1 below.
- `coverageUnion`: a guard, an unrecorded test, a test marked out-of-process or unresolved, and a selected test each go in. A recorded unmarked unselected test stays out. A watch-skipped test never reaches `declared`, so the skip wins. Correct.
- `tierTestNames`:
  - A file constrained to another GOOS is dropped by `MatchFile` with the tier tag.
  - A test with `tiergate.Require` for another tier is dropped.
  - A test with no `Require` always counts.
  - The covered tiers are added through `tiers`.
  - `holdsTierTest` admits the same files. Correct.
- `planTierRuns`: the selected run is added after the tier's own run. The own run is dropped only when it has no invocation and is not `buildOnly`. A `SharedBuild` tier whose own run is dropped still builds, because `runGreenAware` runs the build in the selected run (`!buildDone`). A whole share, `NoCarry` and an empty `selected` add no selected run. Correct.
- `selectedTestsRun`: the `-run` word sits in `run.args`. `emptyTierFailure` reads only `request.args` (`tierrun.go:119`, `:388`), so the empty-tier check still covers the run. `--stop-on-fail`, the retry and the closing line key on `run.number`, so they treat both runs as one tier. Correct.
- Green record: the selected entry keeps `Args=["-run", P]`.
  - `newestWholeAncestor` rejects it because its args are non-empty.
  - `covered` matches it only on equal args.
  - `recordPass` records it as `Packages`, not `Whole`, because its targets never equal `all_targets`.
  - With no call args, `argsMatch` lets a prior whole or package entry for the same tree cover the selected run (`isGuardRunFilter` shape). Those entries did run every test, so this is correct.
- `tierNumbers`: a tier with two runs is listed once. Correct.
- `impactset.impactedFor` / `splitUnchanged` / `packageArgsOf`:
  - A changed package stays in `Packages` only.
  - A binary builder counts only when the `cmd/lym` package changed.
  - A package that is both changed and a builder stays changed.
  - A package that is both an importer and a builder counts as a builder.
  - Empty sets give nil. `Packages` is unchanged. Correct.
- `fabricengine.CodeWorktreeChanges`: it wraps the error and does not list the records checkout. Correct.
- `gateslot` loader and template: `ScopeCoverage` is accepted and named in the scope error. Correct.

**Findings**

- **F1, LOW, design**, `internal/gatecli/coveragescope.go:141-147`. A `FindGuardTests` failure in an importer (for example a misplaced `//lym:guard` marker, a `GuardScanError`) is reported as `miss: Go file does not parse: …`. The package still runs whole, which is safe, but the label gives the wrong reason, and the guard-scan refusal that `computeScope` gives for other packages is lost. Evidence: `importerPackage` builds `testcover.Miss{Reason: testcover.MissParse, ...}` for any error from `tierTestNames` or `FindGuardTests`. Fix direction: label a guard-scan error with its own text, or surface it through `impactset.IsGuardScanError` as the refusal.

No BLOCKING or MEDIUM defects found in this lens.

**Tool calls**
1. A grep for `ScopeForm` consumers, the `goBinary` defaults, `Dir:` in `targets.go` and the `computeScope` callers. The manifest lacked who reads the new `"selected"` scope form, whether `goBinary` can be empty, and whether `DeriveTargets` ever yields a non-root module.
2. A read of `runlog.go:415-439` (`TierSummaryLine`). The grep showed it prints the scope form, and I needed to confirm it does no validation of the value.
