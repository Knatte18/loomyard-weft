# Fixer-report: darn round 1

## Changes

- **F1 (fixed as proposed):** `_lyx/landing/summary.md`, "Design decision" now states that the separate pack file departs from the board entry (which asks for the instruction and pack inline as the prompt's last section) and names commit d8d1c0956 as where the operator's decision is recorded.
  The edit is in the working tree only: the fix rules forbid any git command against a `_lyx` path, so it is not committed.
- **F2 (fixed as proposed):** the banners of `webster-fragment-pack-card.md`, `webster-fragment-pack-unresolved.md` and `webster-fragment-pack-empty.md` now place their subsections in the kick-start pack file.
  Banner-stripping is untouched; `lyx gate test` on `internal/websterengine` passes.
  Committed as "webster: fix F2 — pack fragment banners name the pack file, not the fork prompt".

## Disputed

None.

## Deferred

None.
