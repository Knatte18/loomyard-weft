---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-runner-engine-set.md:57
    summary: Cards 7, 11 and 12 prescribe unresolvable-default guard branches with test rows whose named input, a set whose default is not registered, is rejected upstream by shuttleengines build, the sole production builder of an EngineSet.
    fix: Treat the unresolvable-default branch as a fail-loud wiring guard with no per-row test, as the plan template requires for a branch upstream validation rules out. In card 7 (07-runner-engine-set.md lines 57, 71-72), keep the toldErr join and the newParentReviewConfig lifetime-error fallback but state in the Intent that shuttleengines.New refuses an unregistered cfg.Engine (card 5's build), so no production input reaches them; drop the unregistered-default rows from TestNewRunner_RefusesUnusableToldPaths, TestNewDetachedRunner_RefusesUnusableToldPaths and TestNewParentReviewConfig_KeepAliveInterval (those three stay targets for the shuttlefake.EngineSet wrapping). In card 11 (11-fork-audit-capability.md lines 75 and 86), state the same about AuditEngineFrom's lookup-error return and drop the TestAuditEngineFrom row whose set default is unregistered. In card 12 (12-orch-resume-context.md lines 12 and 20), state the same about resume-context's unresolved default and drop the row built with shuttleengine.NewEngineSet("missing", ...). Card 1's TestEngineSet_Lookup already pins the Lookup error for an unregistered default. No target list changes.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-model-choice.md:20
    summary: Card 4 bundles modelspec.EngineNames, a separate public surface in another package, with ModelChoice and lands it with no coverage, deferring its only test to card 5.
    fix: Move the Create bullet plan:internal/modelspec#EngineNames -> func EngineNames() []string from card 4 to card 5's Create list (after plan:internal/shuttleengines#Names), and move card 4's EngineNames paragraph (lines 18-20) into card 5's Intent as one sentence, where TestNames_EqualModelspecEngineNames covers it. Trim card 4's Intent opening to the ModelChoice half (D11 only), and update the Card Index lines in 00-overview.md to 4 — model-choice — add ModelChoice and 5 — shuttleengines-registry — add the registry package, export modelspec's engine names and move SessionNamer into it. Checked with validate-plan --overlay together with F3; no new blocking finding.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/14-provider-seam-scan.md:4
    summary: Card 14 widens the provider-seam scan but lists only the TestProviderSeamImportRule member, so seam_enforcement_test.go's file header, which still says the file enforces only that internal/shuttleengine never imports claudeengine, stays stale.
    fix: In card 14's Edit list, replace internal/shuttleengine#TestProviderSeamImportRule with the whole file internal/shuttleengine/seam_enforcement_test.go (a file beside its own member is refused by redundant-file-target), and add one Intent sentence saying the file header states the module-wide rule, only internal/shuttleengines and an engine's own directory importing a concrete engine. Checked with validate-plan --overlay together with F2; no new blocking finding.
---

### [MEDIUM] Unresolvable-default guards tested through an input no production path produces

**Location:** `_lyx/plan/07-runner-engine-set.md:57`, also `_lyx/plan/11-fork-audit-capability.md:75,86` and `_lyx/plan/12-orch-resume-context.md:12,20`

**Issue:** Card 1 makes `NewEngineSet` total, so every holder of a set must handle a `Lookup("")` failure.
Card 5's `build` refuses a `cfg.Engine` naming no registered engine before it returns a set, and `shuttleengines.New` is the only production builder of an `EngineSet`.
Even so, card 7 prescribes the `toldErr` join and the `newParentReviewConfig` lookup-error fallback, card 11 prescribes `AuditEngineFrom`'s lookup-error return, and card 12 prescribes resume-context's unresolved-default failure, each with a test row built from a hand-made set whose default is unregistered.
That is the rubric's case of a named input an upstream check rejects, the upstream check being card 5's `build`.
The plan template's rule for such a branch is to drop it or write it as a fail-loud wiring guard with no per-row test.
This is one recurring method gap across three cards, so it is filed once.

### [MEDIUM] Card 4 lands `modelspec.EngineNames` without its coverage

**Location:** `_lyx/plan/04-model-choice.md:20`

**Issue:** `EngineNames` is part of `modelspec`'s public surface and does not depend on `ModelChoice`, which belongs to `shuttleengine`, so card 4 joins two units that can each be reviewed on their own.
Card 4's own text says `EngineNames`' test is card 5's, which breaks the template's "carries the coverage for the behavior it introduces" rule for card 4's commit.
Card 5 is `EngineNames`' only consumer and holds its pinning test, so moving the creation there fixes both problems.

### [LOW] Stale file header in the widened seam test

**Location:** `_lyx/plan/14-provider-seam-scan.md:4`

**Issue:** `internal/shuttleengine/seam_enforcement_test.go` opens with a header saying the file enforces "internal/shuttleengine must never import internal/shuttleengine/claudeengine".
After card 14 the file enforces the module-wide registry rule, so the header describes the old, narrower rule.
The header lies outside `TestProviderSeamImportRule`'s span, and card 14 lists only that member, so no card's targets reach it.
Card 13 settled the same case for `startup.go` by listing the file whole.

## Focus verdicts

- Card order: card 6 uses card 5's `shuttleengines.New` and card 1's `shuttlefake.EngineSet` through `Uses:`, card 7's `orchCLI.engines` is what card 12 reads, card 9 uses card 8's `engineFor` and edits card 8's `enginedispatch_test.go`, card 10 uses card 8's start-time resolution and card 3's `Requirement`, and card 11 edits card 5's `build`.
  No earlier card uses anything a later card defines; card 8 has the four guid-less reads resolve the default until card 9 gives them a guid.
- Fidelity: every Decision D1–D13 and every Constraint has a carrying card (D1/D2 cards 1 and 5, D3 card 2, D4 cards 7–10, D5/D6 cards 6 and 8, D7 cards 11 and 13, D8 cards 3, 10 and 11, D9 cards 3 and 12, D10 card 3, D11 card 4, D12 cards 4 and 5, D13 card 14; docs in cards 3, 5–8, 10, 11 and 14).
  Card 14's module-wide scan is what D13's third bullet decides, and card 12's resume-context move is D9; neither adds unlicensed scope.
  `OpenResult.Engine` in card 8 carries D4's "record the resolved name" for `Open`, which `open.go` confirms writes no run record.
- Attack surface and refusals: card 2's empty-engine refusal names its input (`engine: ""` in a seeded `shuttle.yaml`, which wins over the template, since configengine replaces scalars key by key) and its test (`TestLoadConfig_EngineKey`).
  Card 10's `Requires` checks state their bound (refuse only, an empty list behaving as today) and name the inputs and `TestRunner_Requires` rows that reach each branch.
  Card 11's fork-audit webster refusal follows D7, which decides it as a refusal row with a test.
  The unresolvable-default branches are F1.
- Edit glyphs: card 7's `NewRunner`, `NewDetachedRunner` and `newParentReviewConfig`, card 8's `checkResume`, `skillLoader`, `sessionCycler`, `Runner.newSendContext` and `reconstruct`, and card 9's four `Runner` methods with their adapters and fakes all carry the arrow, and each arrow matches a real change from the current signature.
  The re-listed members with no arrow (`start`, `attach`, `Run.newSendContext`, the interface methods) keep their signatures.
- Verify: `go build`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...` cover every targeted package, in the order the template states, and the module has no nested module outside `testdata`.
- Test economy and granularity: each new test function covers a surface no existing test does (the engine-key test follows the file's per-key `TestLoadConfig_*` convention), and the other changes extend existing tests.
  Card 5's grouping of the registry, the `SessionNamer` move and the `cliwire` shrink is one unit by D1; card 4's split is F2.

## Rejected

- Card 7's construction-site list against `grep -rn "claudeengine\."`: every production `NewRunner`/`NewDetachedRunner` and reader site maps to a listed member, `battencli/drive.go:319` included (it sits in `loadedTacklerDeps`).
- Card 11's webster test fixtures: `webstercli`'s `reset_test.go`/`verbs_test.go` and `websterengine`'s `recoverbatch_test.go`, `reset_integration_test.go` and `parentmerge_test.go` pass concrete `*shuttlefake.Engine` values, which still satisfy `ForkAuditor` and `AuditEngine`, so no unlisted fixture stops compiling.
- `lyx reed remove` in card 10's attach way forward is not in `PATTERN-no-denied-recovery`'s closed deny list.
