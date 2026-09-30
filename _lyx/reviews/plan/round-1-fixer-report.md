# Fixer report — plan round 1

## Fixed

- **F1 (MEDIUM):** `02-step-fold-wiring.md` gains a `step_fold_test.go` case in which a no-verdict write lands right after a fold.
  After two exempt Stucks, a `pause_requested` step must leave history unchanged and `activity.last` at `"Wait → bounced to Wait"`.
  The next step's resume write must carry it forward again, and its exempt Stuck must fold across the pause (D4), giving one entry with `Repeats` 2.
- **F2 (LOW):** The Awaiting-interleave case now scripts exempt Stuck, exempt Stuck, Awaiting, exempt Stuck, so it folds before the Awaiting as the acceptance criterion requires.
  It asserts the first entry's `Repeats` is 1 and that the last entry is fresh.
  It also states that the step after the Awaiting halt is a plain `Step` call, which re-calls `Wait` through the step-3b resume write.
- **F3 (LOW):** Card 2's ImpactSummary is rewritten, still on one line, to name the history-length readers: batten's `history_length` holding still (ly-drive hands back, D10), battencli's 20-entry window, and loomengine's readers unchanged.
- **F4 (LOW):** Card 2 adds `internal/battencli#maxStatusHistoryEntries` to its `Edit:` targets (the glyph resolves via `lyx quarry resolve`).
  Its Intent now has a paragraph that rewords the stale one-entry-per-poll doc comment and keeps the constant and `recentHistory` unchanged.
  `./internal/battencli/...` was already in `## verify:`.
- **F5 (NIT):** Card 4's `TestInnerRun_AwaitingResumesOncePerApproval` bullet now says the third Call's child history length equals the recorded one, so it is the D9 regardless-of-length case.

`lyx loom validate-plan` reports `ok: true` after the edits, with only the three informational `create-new-unit` notes.
The `approved:` key in `00-overview.md` was not touched.

## Deferred

None.
