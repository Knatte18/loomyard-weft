---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/orchcli/rules.go:30
    summary: A hub's existing Bolt or per-orch _lym/orch/rules.md is silently no longer read, and neither the verb help nor the role text says so
    origin: lens:code-affected-set
    fix: In the long help of the rules command in internal/orchcli/rules.go and in the "Orch rules" section of contracts/stencils/orch/orch-template-role.md, add one sentence saying that a rules file kept at _lym/orch/rules.md (hub-wide on Bolt, or per orch) is no longer read, and that its content is carried over by copying it to a draft file and running lym orch rules --from. Add no migration code.
  - id: F2
    severity: LOW
    class: design
    location: internal/heddlecli/cli.go:49
    summary: ChangeStore and the writeOrchRules store-error and warning paths have no test; only the success path is reached through the tier1 integration test
    origin: lens:code-tests
    fix: Add rows to internal/orchcli/rules_test.go for writeOrchRules with a cwd outside a hub (envelope is an error and the store is unchanged) and with a sync that did not start (envelope carries warning), using the heddlecli save and sync seams or a fixture hub, so ChangeStore's error and warning returns are pinned.
---

### [MEDIUM] Existing rules files are dropped without a word

**Location:** internal/orchcli/rules.go:30, contracts/stencils/orch/orch-template-role.md (Orch rules section)
**Issue:** The Goal says the two rules files "go away", and the diff stops reading them. A hub that already holds rules in the Bolt `_lym/orch/rules.md` or in a per-orch copy loses them from every orch at the next launch with no signal. The verb help and the role text tell the operator that a draft starts as a copy of `lym heddle get orch-rules`, an entry that does not exist on such a hub, so the old content is not offered as a starting point. The Goal does not ask for migration, so the gap is the missing statement, not missing code.

### [LOW] ChangeStore error and warning paths are untested

**Location:** internal/heddlecli/cli.go:49
**Issue:** The new public write seam is reached only by the tier1 `TestOrchIntegration_RulesStoreInHeddle` on its success path. A cwd outside a drivable hub, a store change that errors, and a sync that fails to start (the `warning` return copied into the envelope by `writeOrchRules`) have no row that fails if broken. `TestWriteOrchRules_RefusesBeforeTheStore` covers only the refusals before the store.

## Rejected

- Blank draft with no entry saves an unchanged store and starts a sync (lens:code-correctness): matches the Idea's handled clear case, harmless, and the envelope outcome is accurate.
- `cleared` predicate written twice in `writeOrchRules` and `applyOrchRules` (lens:code-correctness): the two blank tests are identical and one line each; extracting a helper would be a premature abstraction.
- `_lym/orch/rules.md` still named as a fixture in `internal/configcli/configcli_integration_test.go` (lens:code-affected-set): the test uses it as an arbitrary unrelated tracked file, and no behavior depends on the feature.
