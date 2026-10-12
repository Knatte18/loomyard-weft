---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/fabricengine/mergevocab_test.go:173
    summary: the doc comment of mergeVocabContainsBareToken points at bareVocabularyToken in internal/lyxcwd/enforcement_test.go, which card 1 deleted
    fix: in internal/fabricengine/mergevocab_test.go, change the last comment line of mergeVocabContainsBareToken from "internal/lyxcwd/enforcement_test.go's bareVocabularyToken." to "internal/fabricvocab's BareToken." so the comment names the predicate that now holds the rule.
---

### [LOW] Stale pointer to a deleted predicate

**Location:** internal/fabricengine/mergevocab_test.go:173
**Issue:** Card 1 moved `bareVocabularyToken` out of `internal/lyxcwd/enforcement_test.go` into `fabricvocab.BareToken` and deleted the original.
A grep for the deleted names (`configsyncOwnerDir`, `fabricVocabularyOwners`, `weftnamePackagePattern`, `vocabScanDocRoots`, `bareVocabularyToken`, `shouldSkipBareVocabularyCheck`, `failsBareVocabularyCheck`) finds one survivor, this comment, which now names a symbol that does not exist.
The other two "mirroring internal/lyxcwd/enforcement_test.go's ..." comments in the same file (`hostPhrases`, `fabricSenseHostPhrase`) still name symbols that exist.

## Review notes

No other defects found against the plan; the checks below were made against the committed range (merge base with `main`, nine commits).

- Card 1: `BareTokenIn` keeps the old early-return, strip, then re-check order, so the moved predicates behave as the old scan did.
  `owners` gains `internal/fabricvocab` as stated, and `docRoots`, `hostGeometryIdentifiers` and `FailsBareCheck` match the deleted originals.
  Every deleted name is gone from the tree apart from F1, and `internal/lyxcwd/enforcement_test.go` calls the package.
- Cards 2 and 3: `brokenFabricVocabularyRule` reproduces the scan's `bareScope` (`isTest || internal || doc root`) and host scope (`!isTest`, `internal` or `cmd`).
  `dir` strips `_test` from the unit, which equals stripping it from the last segment.
  The index appends the check in `ValidateFormat`, `ValidateRework` and `ValidateDispatch`, dropping completed cards at dispatch.
  Spec row 41, the exceptions paragraph, `planglyph/doc.go` and the `validate-plan` help are updated.
- Cards 4, 5 and 9: the spawn point, rejection rule, severity rule and per-fork line in `internal/loomengine/twill.go` match the card text, the `twill-plan` fan lists its four lenses in the stated order, and `ForkSubagents` is set only under `twill`.
  The refusal-spec rows and `docs/overview.md` line are present.
- Card 9 and `single`: the template marker sits on the line that was blank, `setOffParagraph("")` returns `""`, and `TestPlanSpec` pins the rendered prompt equal to the template with that line blank.
- Cards 6, 7 and 8: `plan_producer` defaults to `single` and refuses other values; `PreparePlanSpawn` runs before the rotation and is skipped on attach and relaunch; `lens-review.md` is admitted in `knownNonCardFiles`, the fingerprint and the rotator's announcement only.
- Test economy: new top-level tests are one per card as budgeted (`fabricvocab` 1, `planparser`, `planglyph`, `twilldirective`, `loomengine` each +1); `TestPlanWriteEntry_PreparePlanSpawn` has no covering test (`TestPlanWriteEntry_ConstructionFailures` covers only the nil seam), and the other additions are rows in existing tables.
- Build and tests: `gofmt -l` reports nothing, and `lyx gate test` over the touched packages (`fabricvocab`, `lyxcwd`, `planparser`, `planglyph`, `twilldirective`, `loomengine`, `loomshed`, `shedrecipe`, `loomcli`, `websterengine`, `burlerengine`, `battencli`, `contracts/stencils`, `cmd/lyx`) passes.
