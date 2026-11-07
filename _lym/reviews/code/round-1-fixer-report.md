# Fixer-report — code review round 1

## Changes

- F1 — fixed as proposed (commit `ec148f42`).
  `TestStoreDecisionRecord`'s `StoresBytesAndFlag` row now reads `Home.md` and `proposal-old.md` after the store and before `Rerender`.
  It asserts they contain `[Sketch](decision-old.md)` and `Decision record: [Sketch](decision-old.md)`.
- F2 — fixed as proposed (commit `d20d4b7e`).
  `TestRenderToDiskLinksOnlyAStoredDecisionRecord` asserts `tasks[1].Discussed` is still true after `RenderToDisk`.

`lym gate test ./internal/boardengine` passed (tier0, 755 passed, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
