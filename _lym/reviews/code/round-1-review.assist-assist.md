---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/testcover/mutant.go:182
    summary: Inactive literal rewrites can change constant-expression arithmetic without a compile error
    fix: In fileSites, skip operands of constant expressions whose conversion to runtime arithmetic changes their semantics, or mutate the evaluated expression as a whole while preserving its contextual type. Extend TestMutantBinary_KeepsTheTypeOfAByteContextLiteral with a uint64 return of (1 << 64) / 2 and assert the original result with no mutant active.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/gatecli/redundantmutants.go:410
    summary: Recovery beyond the former build-round limit has no regression coverage
    fix: Extend TestGateRedundant with a controlled compiler stand-in that refuses successive mutant builds through mutantBuildRounds before succeeding. Assert successful completion and the exact dropped-mutant count, exercising the caller's switch to whole-file dropping and its continuation beyond the former limit.
    origin: lens:code-tests
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/testcover/mutant.go:116
    summary: Embedded-line tests do not cover separator precedence or a matched separator with an empty value
    fix: Extend TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine with lines containing both separator orderings and whitespace-only values after matched colon-space and equals separators. Assert exact mutated text for the first separator and indentation fallback.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/testcover/testfuncs.go:50
    summary: Discovery uses different build constraints from the coverage binary
    fix: Select test files using the coverage toolchain's effective build environment and tags, and supply that selection to ScanTestFuncs and both callers. Extend existing map or discovery coverage with complementary feature and !feature files under an enabled GOFLAGS build tag, checking mapped names and failed_tests.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: consistency
    location: internal/gatecli/redundantmutants.go:35
    summary: The constant comment counts one more line-only round than the loop performs
    fix: Describe mutantBuildRounds as the first build round that drops all mutants of blamed files, matching round >= mutantBuildRounds.
    origin: lens:code-affected-set
---

### [BLOCKING] Inactive arithmetic changes

**Location:** internal/testcover/mutant.go:182

**Issue:** `func Value() uint64 { return (1 << 64) / 2 }` originally returns `1 << 63`.
Replacing its literal operands makes the shift runtime uint64 arithmetic, which yields zero even with every mutant off.
The binary compiles, so compile-error recovery cannot discard these sites.
`runMutants` then classifies a correct test as failing unmutated and excludes its evidence.
The same issue occurs with an int32 return of `(1 << 31) / 2`, whose inactive runtime result becomes negative.
This violates the requested literal-type preservation or skip behavior for untyped constant expressions.

### [BLOCKING] Retry recovery lacks coverage

**Location:** internal/gatecli/redundantmutants.go:410

**Issue:** The fallback unit test directly supplies `wholeFiles` but does not exercise the caller's threshold.
`TestGateRedundant` checks one unbuildable mutant at redundant_integration_test.go:341.
Restoring the old terminal round-limit error would leave those assertions satisfied.
The changed continuation behavior has no regression case, contrary to PATTERN-test-speed's behavior coverage rule.

### [BLOCKING] Separator branches lack coverage

**Location:** internal/testcover/mutant.go:116

**Issue:** The embedded fixture has one separator per value line.
Its `list:` line has no matching colon-space separator, so it does not exercise the matched-but-empty branch.
Neither earliest-separator selection nor whitespace-only value fallback has an asserting row.
These are changed behaviors without coverage under PATTERN-test-speed.

### [MEDIUM] Discovery and build disagree

**Location:** internal/testcover/testfuncs.go:50

**Issue:** With `GOFLAGS=-tags=feature`, discovery omits a `feature` test and retains a `!feature` test.
`gateEnv` and `HolderEnv` preserve those flags for the coverage builder.
The missing test produces no result line, which `RunAlone` treats as failure, recreating a false `failed_tests` entry.
The other scan caller, `redundancyRun.judgeRuns`, also loses enabled tagged covering tests.
The affected-set lens independently reported the same defect; this finding consolidates both reports.

### [LOW] Threshold comment disagrees

**Location:** internal/gatecli/redundantmutants.go:35

**Issue:** The comment defines the constant as the number of line-only builds.
The loop starts at one and uses whole-file dropping on the threshold build itself.
The function and package documentation already describe that implementation correctly.

## Review evidence

Reviewed the committed range from `122a41f1ae4e0f613d26111c19b9b566b379c488` to HEAD.
The board entry and decision record license the embedded-value change through the false-candidate completion criterion.
The fixture tag removal supports the discovery change and the repository's runtime tier selection rule.
No loosened check or duplicated test was identified.
`go build ./internal/testcover ./internal/gatecli` passed.
No test gate was rerun: the available shell tool exposes no command-timeout parameter for the required 3600000 ms setting.
The review used source traces and the supplied current-tree redundancy report; it did not reproduce the reported boardengine run.
