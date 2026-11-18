---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:25
    summary: The measurement instruction spells tiers and targets contrary to PATTERN-test-speed.
    fix: Replace the tier-and-target command in the gated darn fragment with a measurement route expressed through a configured gate key, preserving single-class filtering and the narrow measurement exception. Wire the route if needed rather than instructing the writer to bypass the gate-key convention.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/run.go:83
    summary: The tierless success return bypasses the gate key's configured lints and reports.
    fix: In runTest, treat the tierless case as an empty tier plan and retain the configured lint and applicable advisory-report steps before choosing the exit code. Preserve the missing-tier notice and no tier invocation; extend the existing lint integration case to cover a tierless key with a failing lint.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/gatecli/scope.go:349
    summary: An inert-only scope-all gate still runs a tier configured with carry false.
    fix: Distinguish an inert-only gate scope from ordinary empty scopes and cached green results, and make scopeInvocations suppress tier commands for that inert-only gate scope before applying NoCarry. Cover the case in the existing inert-only scope-all integration row and update the affected carry documentation and landing description.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gatecli/scopes_integration_test.go:630
    summary: The new scope-all missing-parent and unreadable-merge-base fallbacks have no regression coverage.
    fix: Extend TestGateTest_Scopes with inert_paths enabled and a scope-all gate key to cover an absent parent and an unreadable parent merge base. Assert whole-tier invocation in both cases and the fallback diagnostic in the unreadable-base case.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/gatecli/scope.go:337
    summary: CLI help still promises that scope-all gate keys run every tier whole.
    fix: Update internal/gatecli/cli.go's test command help to name the inert-only exception for scope-all gate keys and distinguish explicit --all, which retains whole scope.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:23
    summary: Darn engine documentation still describes an unconditional prohibition on writer gate runs.
    fix: Update internal/darnengine/doc.go and the GateFixed and DarnTable comments in internal/darnengine/spec.go to describe the narrow measurement exception while retaining the normal prohibition and parent-run rule for a reported failure. Align the fragment's opening descriptive comment too.
    origin: lens:code-affected-set
---

### [BLOCKING] Measurement guidance violates the gate-key rule

**Location:** contracts/stencils/darn/darn-fragment-make-change-gated.md:25
**Issue:** The new instruction supplies `--tier <n>` and a package or project target.
`pattern/PATTERN-test-speed.md:228` requires a site or stencil to name a gate key,
and forbids tier lists, scope forms, test commands and target lists there.
No declared PATTERN amendment or departure trailer permits this exception.
The review directive makes violations of these constraints BLOCKING.

### [MEDIUM] Tierless gates skip independent checks

**Location:** internal/gatecli/run.go:83
**Issue:** A tierless key returns success before the lint loop and advisory report step.
The template declares no tiers but gives the card key comment and pattern lints.
A `--gate card --card 1` call can therefore pass without checking a lint violation.
`runLints` in `internal/gatecli/failurereport.go` needs no test tier.
The same early return bypasses the configured report duties at `internal/gatecli/run.go:224`.

### [MEDIUM] NoCarry defeats inert-only skipping

**Location:** internal/gatecli/scope.go:349
**Issue:** `allScope` returns an empty scope for an inert-only change,
but `scopeInvocations` selects `tier.NoCarry` before its empty-scope branch.
Thus a Finalize tier with `carry: false` still runs `dotnet test .` in the projectless repository from the issue.
That configuration satisfies neither alternative in the board entry's final Expected paragraph.
The landing summary acknowledges this exception; the decision record contains no amendment authorizing it.

### [MEDIUM] Scope-all fallback behavior is untested

**Location:** internal/gatecli/scopes_integration_test.go:630
**Issue:** The new row reaches the readable-parent, inert-only path alone.
The existing prime-with-no-parent row invokes the computed form, which never reaches `allScope`.
Neither it nor the failure-report tests pins whole-tier execution through the new scope-all fallback branches.

### [LOW] CLI help omits the inert-only exception

**Location:** internal/gatecli/scope.go:337
**Issue:** `internal/gatecli/cli.go:73` still says scope all runs every tier whole.
The changed gate behavior empties that scope for inert-only changes.

### [LOW] Darn documentation omits the measurement exception

**Location:** contracts/stencils/darn/darn-fragment-make-change-gated.md:23
**Issue:** `internal/darnengine/doc.go:79` and `internal/darnengine/spec.go:68` and `:80` describe no gate runs.
The fragment now authorizes a measurement run by the writer.
Its own opening descriptive comment also retains the unconditional description.

## Rejected

- Missing dotnet refusal implementation: the existing guard emits an explicit deny with a reason.
  The provider hook's successful exit carries that protocol; it does not report a successful test command.
  No evidence read establishes a lym path that silently neutralizes the command.
- Duplicate findings: the affected-set lint finding and scope lens's NoCarry finding are covered by F2 and F3.
- Redundant tests: the added API and CLI cases pin distinct outcomes; no covering assertion was identified.

## Validation

Reviewed commits `3746960e13964897b204f08b3ad79ca499e27cb5..HEAD` through the supplied diff pieces and commit log.
The board entry T7 matches the decision record's goal and has no amendment.
The targeted `TestConfig_TierUse` gate call could not acquire its slot because `_bolt/.lym/gate` is read-only.
No test executed in that attempt; the refusal is an environment limit, not a finding against the diff.
