```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: I traced each change the diff makes to everything outside its lines that reads, asserts or describes it.

**Affected set per change**

- **New `gateslot.BatchGateKey` constant** (`internal/gateslot/tiers.go:128`):
  - Covered: `decodeGates` in `internal/gateslot/config.go:521` and `ownPackageTiers` in `internal/gatecli/scope.go:346`.
  - Not covered: these literals of the same key are left as they are:
    - `internal/gateslot/tiers.go:157` (`goReadGates`, same file as the constant)
    - `internal/runplan/cardgate.go:65` (`render("batch", ...)`)
    - `internal/weaverengine/strand.go:26` (`gateKeyBatch = "batch"`, a second constant for the same name)
    - `internal/webstercli/beginbatch.go:76`
  - Finding 1 below.
- **Batch higher tiers now run the cards' own packages** (`computeScope`, `ownPackageTiers`). Readers and describers:
  - Covered: `internal/gatecli/doc.go:154-157`, `internal/gateslot/template.yaml:18`, `contracts/specs/loom-plan-spec.md:345-346`, and both callers that issue the batch call (`runplan.RenderBatchGate` and `weaverengine/strand.go:119`), which reach the narrowing through `--gate batch`.
  - `contracts/stencils/webster/webster-body-implementer.md:73-74` describes the per-card gate step, so it stays correct.
  - `weaver-template-batch.md:87-93` and `webster-body-implementer.md:115-122` do not describe the scope, so they stay correct.
  - Not covered: `contracts/specs/loom-plan-spec.md:331` still says both keys run the cards' packages and their dependents (finding 2).
  - Not covered: the `--card` flag help at `internal/gatecli/cli.go:125` still calls the cards' packages and their dependents the scope (finding 3).
- **The stderr notice line** (`scope.go:268`): asserted in `scopes_integration_test.go` (new row). No other reader. Covered.
- **The guard step now uses `listedPackages`, replacing `listedTiers` and `coveredTiers`:**
  - Covered: `scanGuards` (`scope.go:699`) is the sole reader, and grep finds no remaining reference to the removed names.
  - Covered: the guard-step text in `internal/gatecli/doc.go:241-242` is updated.
- **`existingDirs` and `watchedSkip`, extracted helpers:** called only from `computeScope`. Covered.
- **Rule checks:**
  - Declared PATTERN changes: this directive declares none, so there is nothing to land.
  - Commit messages (d2f8313f5, c2662dece, aad258757) carry no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed.
  - Stencils: no stencil changed.

**Findings**

1. **BLOCKING — consistency** — `internal/gateslot/tiers.go:157`
   - The diff declares `BatchGateKey` but leaves the batch key's name repeated elsewhere: as a literal in `goReadGates` (same file), in `internal/runplan/cardgate.go:65` and in `internal/webstercli/beginbatch.go:76`.
   - It also leaves a second constant for it, `gateKeyBatch` in `internal/weaverengine/strand.go:26`.
   - This breaks `PATTERN-code-conventions`, Named constants: "one named constant, declared once and referenced everywhere, never a repeated literal".
   - Evidence: `internal/gateslot/tiers.go:128,157`, `internal/runplan/cardgate.go:65`, `internal/weaverengine/strand.go:26`, `internal/webstercli/beginbatch.go:76`, `pattern/PATTERN-code-conventions.md` (Named constants).
2. **LOW — consistency** — `contracts/specs/loom-plan-spec.md:331`
   - The verify-model paragraph still says the `card` and `batch` keys both run "over the cards' packages and the packages that depend on them".
   - That contradicts the diff's own bullet at lines 345-346, which says each batch tier above tier0 that does not cover tier0 runs the cards' packages alone.
   - Evidence: `contracts/specs/loom-plan-spec.md:331,345-346`.
3. **LOW — consistency** — `internal/gatecli/cli.go:125`
   - The `--card` flag help says the scope is the card's packages "and the packages that depend on them".
   - Under `--gate batch`, the narrowed higher tiers no longer run those packages.
   - Evidence: `internal/gatecli/cli.go:125`, `internal/gatecli/scope.go:263-268`.

**Tool calls**

1. Grep over the repository's md, go and yaml files for wording about dependent packages: to find docs, stencils and help text describing the batch scope that the manifest did not cover.
2. Read `internal/gatecli/test_integration_test.go:405-420`, grepped `docs/overview.md` for the gate keys, and grepped every literal `"batch"` in non-test Go files: to find every use of the key name the new constant should replace.
