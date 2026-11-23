---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1 (commit 5925da521): `isTopBoxRule` accepts a trimmed line ending in ` ─` (`labelledRuleEnd`, replacing `cutLabelRuleEnd`), so a label with no leading glyph is a top rule; doc comment updated.
  A `TestInputBoxText` row covers the no-leading-glyph shape. A previous fixer session had left this change uncommitted in the tree; it holds against the finding and was kept and committed.
- F2 to F5 (commit 336d2320a, one commit since all four change `run.go` or `startup_test.go`):
  - F2: `noticeStartPromptHeld` opens with the report-not-instruction statement built from `noticeOpen` and `noticeClose`.
  - F3: `TestStartup_SendFailuresTearDown` gains an `interactive` field and a row where an interactive run whose prompt stays busy sends no notice.
  - F4: a `wantHeldReason` field pins the literal delimited reason in each held-notice row.
  - F5: the `sendStartPrompt` doc names that the notice goes out for an autonomous run's notifier.
  - The existing notice-count assertion now matches the capitalised `Agent strand-1 never took its start prompt`.

Gate: `lym gate test` over `internal/shuttleengine` and `internal/shuttleengine/claudeengine` passed, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
