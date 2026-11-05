---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/plan/08-report-cost.md:45
    summary: No card's Intent names the docs/overview.md lines the new internal/reportconfig package forces, the package-tree line and the report module line's package list
    fix: In _lym/plan/08-report-cost.md, extend the Intent sentence beginning with docs/overview.md's report module line so it also says that the overview's package tree gains an internal/reportconfig/ line describing the hub-wide report.yaml config, and that the report module line's package list gains internal/reportconfig beside internal/reportcli and internal/reportengine. The Edit list already holds //docs/overview.md, so no target changes.
    origin: lens:affected-set
---

### [LOW] The overview lines for the new reportconfig package are in no card's Intent

**Location:** `_lym/plan/08-report-cost.md:45`, with `_lym/plan/07-report-config.md:13-17`

**Issue:** The amendment moves the report config module into its own package, `internal/reportconfig`.
`docs/overview.md` lists every internal package in its package tree, and its report module line names the module's packages.
The new package forces both lines, but card 07 does not target `docs/overview.md`, and card 08's Intent says only that the overview's report module line and tree line name the cost verb.
Commit 807263d5 already wrote both lines (`docs/overview.md:319` and `:480`), so the plan describes less than the batch's code changes.
The repo-invariants lens raised the same defect, and it is merged here.

## Scope of this round

The amendment base `cde2a832ae` lives in the weft repository that `_lym` links to (`/home/knatte/Code/loomyard-LYMHUB/raddle-recipe-weft`), so the changed span comes from `git diff cde2a832ae` there rather than from file times.
The authored changes are in cards 07, 08, 23 and 25 (uncommitted in the weft).
The changes to cards 02, 03, 05 and 26 come from Webster's record-batch commits and only drop `plan:` prefixes on symbols that now exist.

Holistic verdicts on the focus questions:

- Fidelity: the amendment's decision, "The report config module lives in internal/reportconfig", is carried. Card 07 creates `internal/reportconfig/template.yaml` and `reportconfig#Config`, `#ConfigTemplate`, `#ConfigSpec`, `#LoadConfig` and `#TestLoadConfig`, and card 08 uses `reportconfig#LoadConfig`. Card 07's other new targets (`doc.go`, `testmain_test.go`, `TestModules_SeedOnlyAndHubWideFlags`, `configStrictnessDegradingSet`, the `PATTERN-config-layers.md` Degrading list) are edits that existing tests and docs force on a new hub-wide module, so D13 and the amendment license them. Dropping `configreg#TestNames` (it pins order, not membership) and `reportengine#allowedImports` (only `reportcli` imports the module) is correct.
- Frozen cards: no finding falls on cards 01–06. Cards 07–12 belong to the failed batch 7, so Webster's auto-rebaseline accepts edits to them.
- Card order: card 08 uses card 07's `LoadConfig`; card 26 uses card 08's `usageFiles` and `costCommand`. Each uses an earlier card.
- Card 23: the amendment moves the five recipe-list `doc.go` files from Extends to Edit and names each sentence. Each named sentence exists in the tree. `shedrecipe/doc.go` stays under Extends because it only gains entry sections. The amendment adds no verb, flag, guard or signature.
- Card 25: the amendment moves `pawlengine/doc.go` to Edit and names its two sentences (lines 8 and 23). It adds no refusal or guard branch.
- Cards 07 and 08: neither changed card names a raw test command, and neither plans a test for prose. `TestLoadConfig`'s rows are the only test that pins the new module's load.

## Rejected

- The baseline `lym loom validate-plan --overlay` reports `create-already-exists` on cards 07–12, because batch 7's commits already built their targets. That is a gate check, and this round does not flag it.
