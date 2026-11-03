---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/driftengine/baseline_test.go:225
    summary: no test pins that settleConfiguredScope leaves an existing baseline.md untouched when a scope is configured
    fix: In TestDriftBaseline add a row that seeds a scope record and a baseline.md holding distinct text (seedScopeRecord with the report flag true), sets f.configScope, and asserts baseline.md still holds that text and no agent was spawned.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/driftengine/baseline.go:171
    summary: configuredBaselineReport always says the drift config set the scope, though the written scope may be a parent's replacement or an earlier record
    fix: In configuredBaselineReport drop the claim about where the scope came from; say the report is generated, that no baseline agent ran and that it expects no particular drift per test, and keep the Scope and Skipped tests lines. Update the "drift.scope" assertion in the configured-full row of TestDriftBaseline to match.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/drift/drift-template-verify.md:23
    summary: the Drift-Verify stencil calls baseline.md the baseline agent's report with the drift it expected, which a configured scope makes false
    fix: In contracts/stencils/drift/drift-template-verify.md line 23, add one clause that when the scope was configured no agent wrote the file and it names only the scope and its skipped tests, so no per-test drift is expected from it.
---

### [LOW] Existing baseline.md guard is untested

**Location:** internal/driftengine/baseline.go:150, internal/driftengine/baseline_test.go:225
**Issue:** `settleConfiguredScope` returns early when `baseline.md` exists, so an agent's or a parent's report survives a resume under a configured scope. No row seeds a report together with `configScope`, so removing the `os.Stat` guard fails no test.

### [LOW] Generated report names the wrong source for a kept scope

**Location:** internal/driftengine/baseline.go:171
**Issue:** The report text states the drift config set the scope. `settleConfiguredScope` writes it from the recorded scope, which a parent's `lym drift scope` or an earlier pass can have set differently from `cfg.Scope`. Drift-Verify's agent reads this file as its expectation, so a false provenance line misleads it.

### [LOW] Drift-Verify stencil describes baseline.md as an agent's expectation

**Location:** contracts/stencils/drift/drift-template-verify.md:23
**Issue:** The stencil says `baseline.md` is the baseline agent's report with the drift it expected. Under a configured scope the diff generates the file with no per-test expectation and no agent, and the stencil gives the verify agent no signal of that.

## Rejected

- Second `d.Config()` read in `drive` has an untested Stuck branch (lens:code-tests, NIT): `captureAgain` already reads the config again with the same untested Stuck shape, and the config is read from disk at each call, so the branch is reachable in production.
- No `LoadConfig` row for `scope: off` (lens:code-tests, NIT): `off` takes the same `checkScope` arm as `full`, and the configured-off row of `TestDriftBaseline` covers its downstream behavior; another row would repeat coverage.
- `contracts/orchrole/orch-role.md` and `docs/overview.md` edits are outside the issue's wording: they document the new key for the orch that files the entry, which is the point of the issue, and the repository's rules want docs in the same diff.
- Drift off with a configured scope: `Call` returns at `passWhenSwitchedOff` before `drive`, so the new `Config()` call is not reached and nothing is spawned, started or committed (focus item 2, not a defect).
- `UpdateScope` with `errNoRecord` (focus item 3, not a defect): `state.UpdateJSON` writes nothing when mutate errors, and `errors.Is` swallows only that sentinel, so other errors surface.
- Non-nil empty `Skip` or duplicates reaching `ScopeRecord` (focus item 5, not a defect): `checkScope` refuses an empty skip under partial, and `RecordScope` accepts duplicates the same way.
- Commit message and summary claims ("a scope already recorded is kept", "writes baseline.md itself") are borne out by `settleConfiguredScope` and its recorded-scope-wins row (focus item 8).
