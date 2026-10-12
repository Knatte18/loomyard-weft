---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/04-twill-host-values.md (Edit list, contracts/stencils#stencilClaims); _lyx/plan/09-plan-twill-directive.md (Edit list, contracts/stencils#stencilClaims)
    summary: Cards 4 and 9 list the row struct type stencilClaims as the member they edit, while the claim rows their Intents change live in the var wordingClaims, so the target list names a member the change never touches and omits the one it does.
    fix: In the Edit lists of _lyx/plan/04-twill-host-values.md and _lyx/plan/09-plan-twill-directive.md, replace `contracts/stencils#stencilClaims` with `contracts/stencils#wordingClaims`, and in each card's Intent bullet that starts with `stencilClaims'` write `wordingClaims'` instead (the twill-directive.md row and the loom-template-plan.md row are entries of that var).
---

### [LOW] Cards 4 and 9 name the claim-row type instead of the claim table

**Location:** `_lyx/plan/04-twill-host-values.md` and `_lyx/plan/09-plan-twill-directive.md`, the `contracts/stencils#stencilClaims` bullet under `**Edit:**`.

**Issue:** In `contracts/stencils/claims_test.go`, `stencilClaims` is `type stencilClaims struct { file, text, claims }` (line 87), the shape of one row.
The rows themselves are entries of `var wordingClaims = []stencilClaims{` (line 107): the `loom-template-plan.md` row sits at line 188 and the `twill-directive.md` row at line 302.
Card 4's Intent changes the `twill-directive.md` entry's claims, and card 9's Intent adds a `{{.twill_directive}}` claim to the `loom-template-plan.md` entry, so both change `wordingClaims` and neither changes the struct.
The glyph resolves, so no gate catches it, but the card's target list then names a member that stays byte-identical and leaves out the member that changes.
That misleads the implementer's kickstart pack, and webster's informational `scope-outside-plan` guard reports the real change as outside the plan.
`hostNeutralClaims` on card 4 is correct: it is the func at line 768.

## Focus directive verdicts

**Card 2, the `_test` unit rule: not a defect.**
Handle units of an external test package are spelled `<dir>_test` (`internal/planglyph/glyphchain_test.go:203-205` uses `shapes_test#…`), and `planparser.HandleUnit` returns the unit after stripping `plan:` (`internal/planparser/handle.go:37-44`).
The scan keys its owner check by `filepath.Dir(rel)` (`internal/lyxcwd/enforcement_test.go:631`), so an external test file in `internal/fabricengine` is judged as `internal/fabricengine`.
Card 2's Intent defines `dir` as the unit with a trailing `_test` stripped from its last path segment and passes `dir`, not `unit`, to `fabricvocab.SkipsBareCheck`, which is what the scan does.
`first` is unaffected by the strip, so the scope test stays right.
`TestCheckFabricVocabularyName` has the pass row `plan:internal/fabricengine_test#Test<Side>…`, which fails if the strip is missing.
It has the refuse row `plan:internal/configsync_test#Test<Side>…`, which fails if an implementation skips every `_test` unit or treats every `_test` unit as an owner.
A row for a `_test` unit with no owner-set row at all, such as `internal/planparser_test`, would add no coverage: it refuses with or without the strip, and the configsync row already catches a blanket `_test` skip.

**Card 3 and spec row 41: agree with card 2.**
Card 3's row 41 text says the two rules are stated "exactly as card 2's Intent states them, with an external `_test` unit counting as its directory for the owner check", which is the same rule.
Row 41 is the next free number: `contracts/specs/loom-plan-spec.md` currently ends at `40. done-card-edited`.

## Checked and found sound

- Card 1 covers every reference to the moved names: they appear only in `internal/lyxcwd/enforcement_test.go`, and `vocabscan_test.go` calls only `fabricVocabularyFailures`, which stays.
  The `md`/`yaml` branch's `fabricVocabularyOwners[dir]` and the subtest at line 938 map to `IsOwner`, which the card names.
- `planparser` is not a leaf package in `pattern/PATTERN-leaf-packages.md`, so importing `internal/fabricvocab` needs no allowlist change.
- Card 2's literal host-geometry rows spell `hostBranch` only in test files, which the host rule (`hostScope := !isTest && …`, line 638) leaves out.
- Card 3's dispatch drop of completed cards and its `language: none` fixture use existing `cardIDSet` and `minimalPlan`; the parity tests in `internal/loomcli/parity_test.go` keep passing because the new check refuses nothing in their fixtures.
- Every existing test the cards extend exists under the name given.
