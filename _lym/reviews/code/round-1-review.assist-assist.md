---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/testcover/mutant_test.go:299
    summary: Neither new mutation operator has coverage of its enabled behavior
    fix: Extend the existing public-surface mutation integration fixture to execute clause-deletion and embedded-line mutants with the selector off and on. Assert clause-specific unique kills, skipped side effects of deleted operands, three-operand and parenthesized chains, and the exact selected embedded-line change. Keep the checks in the existing behavior tests rather than adding per-helper tests.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/testcover/mutant_test.go:201
    summary: Cross-file data discovery and multiple-reader attribution lack regression coverage
    fix: Extend TestPlanMutants with a declaration-only Go file whose variable is read repeatedly in several blocks. Select only a later reader block and assert the complete sorted, deduplicated Blocks value. Extend an existing runner scenario with candidate and covering tests reaching different blocks of that mutant, and assert that the covering kill is recorded.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:66
    summary: A deleted boolean operand is still evaluated
    fix: Replace value-taking clause-deletion wrappers with generated short-circuit guards that skip the selected operand when its mutant is active. Preserve expression grouping and inactive evaluation order for both operators. Cover an operand with an observable side effect through the existing public-surface operator test.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:423
    summary: Function-wide shadow suppression discards valid package-name reads
    fix: Resolve package references using lexical scope and declaration position in variableReads and fileSites. Keep reads on the right side of a short declaration and reads outside a nested shadowing scope eligible. Add those cases to TestPlanMutants.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:482
    summary: Package-level map keys are excluded from data mutation
    fix: Distinguish map keys from struct field names and constant-only array or slice indices when marking fixed syntax. Generate supported literal mutants for map keys, including package-level map initializers, and add a map-key row to TestPlanMutants.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/testcover/mutant.go:329
    summary: Declaration discovery uses a different build context from the mutation runner
    fix: Obtain the selected package file list using the same configured Go toolchain and environment as the coverage and mutant builds, and pass that list to planning instead of using build.Default.MatchFile. Cover a declaration-only file selected by a custom build tag and its excluded alternative.
---

### [BLOCKING] Execute the new operators in a behavior test

**Location:** internal/testcover/mutant_test.go:299

**Issue:** The planning and overlay tests assert sites and wrapper text, but never execute either new operator.
The helper-file assertion checks only the package prefix and environment-variable name.
The production fixture in `internal/gatecli/redundant_integration_test.go` has no boolean chains or embedded strings.
An identity implementation of either new operator therefore satisfies these assertions.
This violates `PATTERN-test-speed` coverage of new behavior; it is not a mechanical test-failure finding.

### [BLOCKING] Exercise cross-file and multiple-block attribution

**Location:** internal/testcover/mutant_test.go:201

**Issue:** The fixture writes one Go file, and every expected mutant block list is a singleton.
The runner guard fixtures also contain only singleton block lists.
Ignoring declaration-only files or checking only the first reader would pass these cases.
The newly added discovery and multiple-reader selection behavior therefore lacks coverage required by `PATTERN-test-speed`.

### [MEDIUM] Deletion must skip operand evaluation

**Location:** internal/testcover/mutant.go:66

**Issue:** The wrapper evaluates its first argument before deciding to return the boolean identity.
For `advance() && ready`, the deletion mutant still calls `advance()`.
An actual deletion would skip that side effect, panic or wait.
A test that uniquely checks the side effect cannot kill this alleged deletion mutant.
The three-operand traversal itself reaches the individual operands; the defect is their execution semantics.

### [MEDIUM] Respect lexical shadowing boundaries

**Location:** internal/testcover/mutant.go:423

**Issue:** `localNames` collects declarations from the entire function, including nested scopes.
Both reader attribution and constant-use selection then suppress that name everywhere in the function.
With a package constant `prefix`, the right side of `prefix := prefix` is a package reference but receives no mutant.
A nested local named `names` likewise suppresses a separate read of a package-level list.
The landing summary acknowledges this approximation, but it leaves the requested data-mutation gap for these valid uses.

### [MEDIUM] Mutate map keys that are data

**Location:** internal/testcover/mutant.go:482

**Issue:** The new package-variable traversal inherits the unconditional exclusion of every composite key.
For `var enabled = map[string]bool{"feature": true}`, the string key is the only supported literal kind.
It is excluded, so a test pinning the key set gains no data mutant.
Map keys need not remain compile-time constants; their exclusion is not needed to keep this example buildable.

### [MEDIUM] Discover declarations from the actual build

**Location:** internal/testcover/mutant.go:329

**Issue:** A declaration-only file has no coverage block to place it in `sources`.
The fallback selects it with `build.Default.MatchFile`, independently of the configured toolchain and its environment.
`HolderEnv` preserves `GOFLAGS`, including custom build tags, for the coverage and mutant builds.
A `//go:build custom` file containing a package list can thus be compiled but omitted from mutation planning.
An excluded alternative can instead be scanned.

## Focus departures

- The directive asks whether the done-when command stops listing the five named candidates.
  I ran `lym gate redundant internal/boardengine`; it refused because this worktree has no coverage map.
  No candidate verdict was produced, so the done-when outcome remains unverified.
  I did not generate a map under the report-only write restriction.

The implementation findings above are source-level traces, not results from executing a generated mutant binary.
No scope-only finding was retained for the missing confirming run.
