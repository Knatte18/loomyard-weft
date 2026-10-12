---
round: 2
focus:
  - "Card 2 (`_lyx/plan/02-vocabulary-name-check.md`), the bare-token rule of the fabric-vocabulary-name check: the handle unit of an external test package is spelled `<dir>_test`, e.g. `plan:internal/planparser_test#…`, while the scan's owner check in `internal/lyxcwd/enforcement_test.go` (`fabricVocabularyFailures`) keys by the file's directory. Settle whether the card's Intent makes the check treat `internal/fabricengine_test` as its directory `internal/fabricengine` before consulting `fabricvocab.SkipsBareCheck`, and whether `TestCheckFabricVocabularyName` has a pass row for such a unit and a refuse row for a non-owner `_test` unit."
  - "Card 3 and `loom-plan-spec.md` row 41 text: settle whether the spec wording the card edits agrees with whatever unit-to-directory rule card 2 states."
---

Round 1 left one gating question, and the second entry follows from it.
