---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/prebuild.go#needsPrebuiltLym
    summary: A prebuilding tier whose command holds neither {targets} nor {test_targets} runs the whole set under explicit targets, yet the check reads the caller's ignored targets and can skip the build.
    fix: "In needsPrebuiltLym, return true (after the PrebuiltLym check) when !gateslot.HoldsTargets(run.tier.Command), since such a command runs whole whatever the invocations' targets say. Update the invocationsUseLymBinary or needsPrebuiltLym doc comment to state it. Add a row to TestInvocationsUseLymBinary (or a new table over needsPrebuiltLym in internal/gatecli/prebuild_test.go) with a tier whose Command is e.g. \"go test ./...\" and PrebuiltLym true, targets [\"./plain\"], want true."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/tiers.go:48
    summary: The PrebuiltLym field doc still says the setting makes every run of the tier prebuild lym, which the diff made conditional.
    fix: "Change the comment above PrebuiltLym in internal/gateslot/tiers.go to say the tier prebuilds lym from the worktree first when a test of its targets may use the binary."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/pool.go:30
    summary: The PrebuiltLymEnv doc says a tagged gate run builds the binary once for all packages; a run whose targets' tests do not use it now builds and exports none.
    fix: "In internal/gateslot/pool.go, change the second line of the PrebuiltLymEnv doc to: A tagged gate run whose targets' tests may use the binary builds it once for all packages, and each test binary runs that build instead."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/benchmarks/running-tests.md:90
    summary: The doc says the harness builds lym once for a tagged tier as lym gate test --tags does, but lym gate test now builds only when a targeted package's tests use the binary.
    fix: "In docs/benchmarks/running-tests.md line 90, drop the clause \"as `lym gate test --tags` does\", or replace it with a sentence saying that lym gate test builds it only when a targeted package's tests may use the binary."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-test-speed.md:75
    summary: The new sentence says the run builds only for a kit-importing test file or a pattern target, but the code also builds for an import-path target, an invocation with no target and the kit's own directory.
    fix: "Rewrite pattern/PATTERN-test-speed.md line 75 to: It builds only when a test it runs may use the binary: a targeted package's test file imports `internal/testkit/lymbin`, the kit's own directory is targeted, or a target is a pattern such as `./...`, an import path or absent."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/copies_test.go#TestRunTierCopies_BuildsThePrebuiltLymOnce
    summary: No test pins the skip on the --copies path; reverting copies.go:65 to build whenever the tier asks leaves every test green.
    fix: "Add a case to internal/gatecli/copies_test.go that runs runTierCopies with PrebuiltLym true and an invocation whose targets name a directory under the temp root holding only a test file that imports \"testing\", using the same stand-in go script; assert the builds file is absent (no build) and both copies passed."
  - id: F7
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/gatecli/prebuild_test.go#TestInvocationsUseLymBinary
    summary: No row pins the `.`, `..` and `../x` branches of isDirectoryTarget, so dropping one goes unnoticed.
    fix: "Add rows to TestInvocationsUseLymBinary: {dir: \"plain\", targets: [\".\"]} want false, and {dir: \"mod\", targets: [\"../plain\"]} want false, so each branch is exercised against a directory whose tests do not import the kit."
---

### [MEDIUM] The prebuild check reads targets the tier's command ignores

**Location:** `internal/gatecli/prebuild.go#needsPrebuiltLym`

**Issue:** Under explicit targets, `scopeInvocations` (`internal/gatecli/tiers.go:293-298`) returns `targets: request.targets` even when `!gateslot.HoldsTargets(tier.Command)`.
In that case the command runs once, whole (`tiers.go:285-286`, and `tiers.go:154-156` counts the tier among `wholeTiers`).
`invocationsUseLymBinary` never looks at the command, so `lym gate test --tier N ./plain` on a prebuilding tier whose command is `go test ./...` skips the build while every package's tests run.
No test loses the binary: with `LYM_PREBUILT_LYM` unset, `lymbin.buildOnce` compiles `lym` in each test binary that imports the kit.
That replaces one build with one build per such package, and it goes against the board entry's "A call over the whole set still builds it".
The scenario fixture's `tier2: standIn+" whole"` (`internal/gatecli/test_integration_test.go:642`) shows the gate supports this command shape.
The hub's current tiers all hold a placeholder, which keeps the severity at MEDIUM.
Raised by the code-correctness and code-scope lenses and by the handler.

### [LOW] The gateslot `PrebuiltLym` field doc still promises an unconditional prebuild

**Location:** `internal/gateslot/tiers.go:48`

**Issue:** "PrebuiltLym makes a run of the tier prebuild lym from the worktree first." is no longer true for a run whose targets' tests do not use the binary.

### [LOW] The `PrebuiltLymEnv` doc says every tagged run builds the binary

**Location:** `internal/gateslot/pool.go:30`

**Issue:** "A tagged gate run builds it once for all packages" now holds only when a test of the run's targets may use the binary.

### [LOW] The running-tests doc compares the harness to a gate behaviour that changed

**Location:** `docs/benchmarks/running-tests.md:90`

**Issue:** The harness in `cmd/testtiming` still always builds `lym` for a tagged run, but `lym gate test` now builds only conditionally, so "as `lym gate test --tags` does" no longer holds.

### [LOW] The rule doc states a narrower build condition than the code runs

**Location:** `pattern/PATTERN-test-speed.md:75`

**Issue:** The "builds only when" sentence names a kit-importing test file and a pattern target.
`invocationsUseLymBinary` and `TestsUseLymBinary` also build for an import-path target, an invocation with no target, an unreadable directory and the kit's own directory, as the `gatecli` package doc states.

### [LOW] The `--copies` skip has no test

**Location:** `internal/gatecli/copies_test.go#TestRunTierCopies_BuildsThePrebuiltLymOnce`

**Issue:** The only copies test uses an invocation with no target, which always builds, and the new no-build integration subtest runs without `--copies`.
Nothing fails if `runTierCopies` goes back to building whenever the tier asks.

### [NIT] `isDirectoryTarget`'s `.` and `..` branches are unexercised

**Location:** `internal/gatecli/prebuild_test.go#TestInvocationsUseLymBinary`

**Issue:** The rows use only `./x` and absolute targets.
Dropping a branch fails safe, since the target then builds, but no row would notice.

## Rejected

- The existing CLI-level build subtests (`test_integration_test.go:1968`, `2074`) target an absent `./pkg` and so build through the unreadable-directory fallback: they still check what their names claim, and the import rule's positive case is pinned at unit level by `TestInvocationsUseLymBinary`'s `PackageWhoseTestsImportTheKit` row, so this is no coverage gap.

Handler checks with no finding:
- The change description's route count, 26 test files in 11 package directories importing `lymbin`, matches a grep of real import lines.
- `kitDeniedImports` in `internal/testkit/enforcement_test.go` keeps every other kit from importing `lymbin`.
- `gateEnv` strips an inherited `LYM_PREBUILT_LYM` (`internal/gatecli/run.go:912`), so a skipped run exports none.
- The `two-way` merge-danger call holds: the diff changes no record format, config meaning or outward push.
- The hub tier1 command change stays out of the diff, as the board entry asks.
