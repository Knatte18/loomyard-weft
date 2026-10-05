---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D6 Bound (line 97) and D10 (line 143-144)
    summary: D6 Bound states as fact that the plan verify runs -tags tmux and "keeps them run", while D10 says a verify only may run tmux and gate choice is left to test-tiers
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D6 (lines 90-94)
    summary: The retag splits a package's smoke files between tmux and llm without a decision on shared helpers, so a package whose files share helpers across the split no longer compiles under a single tag
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:Constraints (line 151)
    summary: Test-isolation constraint rekeys the "has a tagged test file" rule but not the paired !integration TestMain files, which the tag split changes the meaning of
---

### [MEDIUM] D6 Bound promises a verify the task cannot set, and contradicts D10

**Location:** `decision-record.md` D6 Bound, D10.
**Issue:** Focus point 2.
`cardGateCommand` (`internal/websterengine/cardgate.go:31`) emits `go test -tags integration`, and the record rightly leaves it alone.
The Bound's first sentence is honest about card gates.
Its second half says "the plan verify, the final gate, runs `-tags tmux` ... and that is what keeps them run".
That holds only for this task's own plan verify, which Plan-Write writes.
Verifies of other tasks come from the stencil text, and D10 says that text only says a verify "may" run `tmux`, with gate selection left to `test-tiers`.
So until `test-tiers` lands, nothing guarantees any other task's verify runs `tmux`, and "keeps them run" over-claims.
The acceptance criterion ("still runs under some gate's verify (`-tags tmux` until `test-tiers` decides)") repeats the same unscoped claim.
D6's `llm` wording, by contrast, is consistent: no sentence promises a gate runs an `llm` file, `go vet -tags llm` is the stated compile check, and the loss from `-tags smoke` and `-tags integration` runs is stated as intended and bounded by "run by hand".
**Fix:** Scope the claim: this task's plan verify runs `-tags tmux`; other verifies run it only if their plan writer follows the D10 text, and until `test-tiers` decides, the `tmux` tier outside this task's verify is run by hand or by an explicit verify.

### [MEDIUM] Retag split can break helper sharing inside a package

**Location:** `decision-record.md` D6.
**Issue:** Tags do not nest, and D6 sends each `smoke` file to `tmux` or `llm` and each tmux-starting `integration` file to `tmux`.
Files in one package share helpers: for example `internal/loomcli/smoke_test.go` defines `smokeStatusRel` and others that sibling `smoke_*_test.go` files use.
If siblings land in different tiers, or a `tmux` file uses a helper left in an `integration` file, `go test -tags tmux` and `go vet -tags llm` fail to compile, while the acceptance command `go vet -tags integration,tmux,llm ./...` still passes and hides it.
The record states no rule for where such helpers go or that each tier must compile alone.
**Fix:** Add to D6 a requirement that each tag compiles on its own (`go vet -tags tmux ./...`, `go vet -tags llm ./...`, `go vet -tags integration ./...` separately), with helpers shared across tiers moved to an untagged `_test.go` file or to a kit.
Add the per-tag vet to the acceptance criteria.

### [LOW] Paired `!integration` TestMain files are not covered by the rekeying

**Location:** `decision-record.md` Constraints, `PATTERN-test-isolation` line.
**Issue:** Ten packages carry a `testmain_test.go` tagged `!integration` that pairs with a `testmain_integration_test.go`.
The pair is keyed on one tag; once `tmux` and `llm` exist, the `!integration` file is the one compiled under `-tags tmux` or `-tags llm` alone.
The record does not say whether the pair is rekeyed or left, and `PATTERN-tmux-test-isolation` requires `tmuxkit.Main` in every `TestMain` of a package with tagged files.
**Fix:** Add one line to the constraint: the `!integration` TestMain pairs keep their meaning (the non-integration TestMain wraps `tmuxkit.Main` already), or are rekeyed, and the check covers every tag combination.

## Settled focus points

- Focus 3 (D2 spawn recognition): decided and consistent.
  The static scan names its triggers, a test it cannot classify counts as spawning, and the acceptance criterion matches.
  Not a defect.
- Focus 4 (Constraints PATTERN IDs): every ID named as a rule to follow exists today or is created by D8 (`test-speed`, `test-isolation`); the retired IDs appear only as "absorbed" context.
  Not a defect.
