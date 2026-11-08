```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. The diff only adds `//lym:watch` markers and doc lines, so I ran each marker through the real matcher, `impactset.WatchedTest.watches` and `watchedTestsInFile`.

**Per-item list (case run, verdict)**

- **Matcher semantics** (`internal/impactset/watches.go:93-104`):
  - A pattern with `/` is matched against the full relative path.
  - Any other pattern is matched against the base name.
  - Every added pattern contains `/`, so `*` never crosses a directory.
  - Result: correct.
- **Stacked `//lym:watch` lines** (`watches.go:128-146`): each line's patterns are appended, and every added block ends in `func Test…` with only `//lym:` and `//testtiming:keep` lines above it. Result: correct, so no misplaced-marker refusal.
- **Rename or delete of a watched file**: `ChangedPaths` uses `--find-renames` and reports a renamed file by both paths (`internal/impactset/targets.go:101-109`). Moving a subject file out of its directory still runs the guard. Result: correct.
- **Walk depth**:
  - I used the repo file counts for each walked tree. Deepest scanned today, patterns reach:
    - `contracts/stencils`: `stencils/a/b/file`, patterns reach `*/*/*/*.md`.
    - `contracts/specs`: flat, patterns reach `*/*`.
    - `plugins/ly/skills`: `skills/a/file`, patterns reach `*/*/*`.
    - `internal/webster*`, `internal/shed*`, `internal/loom*`: `pkg/file`, patterns reach `*/*/*.go`.
    - `internal/boardengine`: `pkg/sub/file`, patterns reach `*/*/*.go`.
    - `internal/fabricengine`, `internal/websterengine`, `internal/webstercli`, `internal/burlercli`: flat, patterns reach `*/*.go`.
    - `internal/planparser`, `internal/planglyph`: `pkg/file`, patterns reach `*/*.go`. The only deeper files are `internal/planglyph/testdata/glyphchain/...`, which `scankit.skipDir` never enters.
  - Result: correct, each pattern set reaches at least one level below today's deepest scanned file.
- **`TestMutationRecord_FabricengineProductionSource`**: it reads `destroy.go` and the result-type files in `destructiveGuardMutatingResultTypes` and `destructiveGuardReadOnlyResultTypes`. All are flat `internal/fabricengine/*.go` files, which the pattern covers. Result: correct.
- **`rawGitMutation`, `uncontainedWrite`, `destructive`**: each scan-package list holds one entry (`internal/websterengine` or `internal/fabricengine`), and their allowlists live in the test's own file. Result: correct.
- **`TestNoDeniedRecovery_ScannedSurfaceHasNone`**:
  - Text surfaces `contracts/specs`, `contracts/stencils` and `plugins/ly/skills` are covered at every depth.
  - Go modules with prefixes `webster`, `shed` and `loom` are covered.
  - `shellguard.Rows` forms are literals in `internal/shellguard/rows.go`. `gateslot.InheritEnv` and `agentname.StrandNameEnv` only feed `lymEnvNamesPattern` (`rows.go:331`), not `Forms`, so they cannot change the outcome.
  - Result: correct.
- **`TestWebsterRetiredSteps_ScannedSurfaceHasNone`**: the matcher helpers `deniedRecoveryTokenPattern`, `tokensMatch` and `hasAnyPrefix` all live in `cmd/lym/norecoverydeny_test.go`, which the pattern lists. Result: correct.
- **`TestSandboxCoverage_AllModulesCoveredOrExcluded`**:
  - Module names come from `newRoot()` in `cmd/lym/main.go`, the only non-test file that builds the root. `cmd/lym` has only `exitsweep.go` and `stencilseed.go` besides it.
  - Each module's `Command()` lives in its own `internal/*cli/*.go` file, which the pattern covers.
  - The shortcut group is skipped, and `shortcutcli` is watched.
  - Result: correct.
- **`TestStencils_NameGoValues`**: the values come from files that are all in the patterns:
  - `gatecli.SlotBusyExit` in `internal/gatecli/cli.go`.
  - `shuttleengine.MessageTail` in `internal/shuttleengine/messagetail.go`.
  - `shedrun.ParkMarkerFileName` in `internal/shedrun/paths.go`.
  - `websterengine.Report` in `internal/websterengine/report.go`.
  - Result: correct.
- **Other stencil guards**:
  - They read `Registry()` and `shippedStencils`, which come from `contracts/stencils/stencils.go` and `registry_test.go` (`contracts/stencils/*.go`), plus `internal/stencil` and `internal/stencilstore`.
  - The directive tests also use `parentdirective.MarkerName` and `editdirective.MarkerName`, which are watched.
  - Result: correct.
- **`TestCwdMutation_*` and `TestVerifiedTree_PlanVerifySitesCallVerify`**:
  - The exact paths match `cwdMutationSubjectFiles` and `planVerifySites`.
  - The `NotVacuous` test reads only the allowlisted `coalesce_integration_test.go`.
  - Result: correct.
- **`TestEnforcement_ScankitStdlibOnly`**: `assertImports` reads only the scankit directory (`scankit.go:358-377`). Result: correct.
- **`TestBannedDeclarations_CliPackagesCallIntoCliwire` and the two planparser scans**: roots equal `policedCliDirs` and the two `scanTargets` roots. Their allowlists and banned-name lists live in the test's own file or in `internal/planparser/classify.go` (`refKindConstNamesInDeclarationOrder`). Result: correct.
- **`TestBoardGuard_NoRawGitImportOrShellOut` and `TestGitrepoBoundary_PinnedRunCallSites`**: both patterns cover the walked roots. Result: correct.

**Findings:** none from this lens.

**Tool calls**
1. Grepped `lym:watch` in `internal/impactset/*.go` and read `UnwatchedTests` and `watches`, because the manifest lacked the matcher.
2. Read `internal/impactset/watches.go:120-185`, to confirm that multiple marker lines accumulate and to see the placement rules.
3. Read `scankit.assertImports` and the imports of `internal/stencil` and `internal/stencilstore`, to settle what `TestEnforcement_ScankitStdlibOnly` and the stencil registry depend on.
