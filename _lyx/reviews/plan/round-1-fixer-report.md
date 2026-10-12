# Plan — round 1 fixer report

## Changes

- **F1** (BLOCKING, fixed as proposed).
  Premise confirmed: `fabricVocabularyFailures` keys the owner check by `filepath.Dir(rel)` (`internal/lyxcwd/enforcement_test.go:630,644`), and `internal/fabricengine` holds external test files.
  Card 2's Intent now defines `dir`, the unit with a trailing `_test` stripped from its last path segment, and passes it to `fabricvocab.SkipsBareCheck`, stating that it is the directory the scan's owner check reads.
  `TestCheckFabricVocabularyName` gains a pass row `plan:internal/fabricengine_test#Test<Side>…` and a refuse row for a `Test<Side>…` member in `plan:internal/configsync_test`.
  Card 3's `loom-plan-spec.md` row 41 text says an external `_test` unit counts as its directory for the owner check.
- **F2** (LOW, fixed as proposed).
  Card 2's Intent iterates `OwnedHandles(card)` (`internal/planparser/handle.go:85`), and its Uses swaps `internal/planparser#IsHandleRef` for `internal/planparser#OwnedHandles`.
- **F3** (LOW, fixed as proposed).
  Card 8 drops `TestPRRework_FirstRoundMovesGenerationZero` from Edit and from Tests.
  The PR-Rework Intent bullet keeps the no-change sentence and cites the test's existing `amendments.md` assertion (`internal/loomshed/prrework_test.go:295,304`) as the coverage.
- **F4** (NIT, fixed with the fix key's first option).
  The Shared Decision at `00-overview.md:28` now says the rejection rule and spawn point carry D5's and D3's content in the prompt's second-person voice, and the severity rule and per-fork line are verbatim.

`lyx loom validate-plan` reports `ok: true` after the changes, with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
