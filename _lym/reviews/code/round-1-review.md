---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun.go:274
    summary: runInvocations' {test_targets} wiring, including the empty-case skip, has no test that would fail if it broke
    fix: Add a test through runInvocations (or the public gate test path) with a fake shell command holding {test_targets} over a temp module where one package has no test of the tier; assert that the rendered command receives only the kept package, that {targets} stays whole, and that when every package is dropped no command runs, the stderr line is printed and the exit is 0. Add it as rows in an existing runInvocations test if one exists.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/impactset/tiertests.go:481
    summary: testFunctionPrefixes omits Example, so a package whose only tests are ExampleXxx functions with an Output comment is dropped from {test_targets} and its examples stop running
    fix: Add "Example" to testFunctionPrefixes in internal/impactset/tiertests.go (an example takes no testing.TB, so RequiredTier reports none and it counts for every tier), and add a row to TestHoldsTierTest with a package holding only an ExampleA function.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/doc.go:138
    summary: Docs and refusal messages still name {targets} alone where HoldsTargets now also accepts {test_targets}
    fix: In internal/gateslot/doc.go:138 say all_targets is required when the command holds {targets} or {test_targets}. In internal/gatecli/tiers.go:165 reword the guard-step refusal to "must hold {targets} or {test_targets} and {args}", and update the comments at tiers.go:53, :80 and :192, gatecli/doc.go:132, :176 and :309, and the run.go:486 message to name both placeholders; keep any test that asserts the old message in step.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testtargets_test.go:141
    summary: tierTestTargets' go-list failure fallback and a "..." pattern target have no test row
    fix: Add rows to TestTierTestTargets for a "./..." target over the temp module (kept packages listed through go list) and for a failing go list (a nonexistent goBinary), asserting the targets come back unnarrowed.
  - id: F5
    severity: LOW
    class: decision
    origin: handler
    location: internal/gatecli/doc.go:574
    summary: The diff only adds an opt-in placeholder; no tier command in the repo adopts it, and the doc example is rewritten to a gate.yaml the diff cannot show
    fix: Defer to the run's parent. The entry asks that the gate apply to every tier, but the hub's tier commands live in the hub's gate.yaml, outside this diff; the parent should confirm the hub's tier0 to tier2 commands use {test_targets}. The rewritten gatecli/doc.go example (build tags and covers replaced by LYM_TEST_TIER) is a claim about that file; leave the code and check it against the hub's gate.yaml.
---

### [MEDIUM] runInvocations wiring is untested

**Location:** internal/gatecli/tierrun.go:274
**Issue:** The trigger on `{test_targets}`, `{targets}` staying whole, and the empty-case `continue` with its stderr line are pinned by no test. `TestTierTestTargets` calls the helper directly. A `break` in place of `continue`, or a command run with no packages, would let `go test` run the whole scope and fail no test.

### [LOW] Example functions are dropped

**Location:** internal/impactset/tiertests.go:481
**Issue:** `go test` runs `ExampleXxx` functions with an `// Output:` comment. A package holding only those is dropped, which breaks the entry's "no test lost". It is latent: no `func Example` exists in `internal` or `cmd` today.

### [LOW] Stale `{targets}`-only wording

**Location:** internal/gateslot/doc.go:138
**Issue:** The `all_targets` rule, the guard-step refusal and several comments and the run.go:486 message still describe `{targets}` alone, though `HoldsTargets` accepts `{test_targets}` too.

### [LOW] Missing rows for the fallback and a pattern target

**Location:** internal/gatecli/testtargets_test.go:141
**Issue:** No row makes `go list` fail, and none passes a `...` target through `tierTestTargets`, which is the form real tier commands use.

### [LOW] No tier adopts the placeholder in this diff

**Location:** internal/gatecli/doc.go:574
**Issue:** The change is opt-in. Whether the hub's tiers use `{test_targets}` cannot be read from the diff, and the doc example now shows a different tier model than before. This is for the parent to confirm.

## Rejected

- code-tests F4 (HoldsTargets swaps in green.go unpinned for a `{test_targets}`-only command): the swaps are a mechanical substitution of one shared predicate, and the planning row in `TestPlanTierRuns` already pins the predicate; a green-record row would be a per-site test of a one-line change.

## Focus departures

none
