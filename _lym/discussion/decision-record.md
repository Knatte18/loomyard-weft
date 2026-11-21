## Goal

```markdown
## What was seen

Outside tests, "retired" appears in 127 files, "migrat" in 57 and "legacy" in 65 (2026-11-18);
`fabricengine` leads with 18 files, then `loomcli` 11 and `boardengine` 10.
Some of it is a deliberate refusal of an old name or key with a pointer to the new one, which is cheap and stays.
Some is real migration code: `landingshed` alone carries about 100 lines (the retired `Merge-In` row name,
a scratch-to-records attempt migration, a key-rename migration).

## Idea

A mechanical pass, module by module: classify each hit as a refusal (keep) or a migration or compatibility path
whose data no hub still holds (delete).

## Decided

- All migration code goes: the operator confirmed no hub needs it (operator, 2026-11-19).
- `lym board retire-legacy` is part of it: a one-off migration of `tasks.json` and `notes.json` into `board.json` that every hub has run.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - compatibility-readers-left-behind (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
