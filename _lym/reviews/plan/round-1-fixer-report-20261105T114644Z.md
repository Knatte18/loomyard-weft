# Fixer report — plan round 1

## Changes

- **F1** (BLOCKING), fixed as proposed.
  In `_lym/plan/07-report-config.md`, the Create target `plan:internal/reportconfig#TestLoadConfig` is now `plan:internal/reportconfig_test#TestLoadConfig`, because `internal/reportconfig/config_test.go:3` declares `package reportconfig_test`.
  The other `plan:` Create targets of card 7 are unchanged; recovering batch 7 needs them to resolve, so that the create-not-done check that halted it (`_lym/loom/friction/loom-halt.md`) passes.
- **F2** (LOW), fixed as proposed.
  Card 7's **Edit:** list gains `pattern/PATTERN-config-layers.md` after `cmd/lym#configStrictnessDegradingSet`.
  Its Intent gains a line saying that the Degrading callers set in the file's Strictness section gains `reportconfig`.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
  Its out-of-scope note says the set already omits `sheddriver` and `codexengine`, so the new Intent line's phrase "lists the packages that load with `LoadOrTemplate`" describes what the set is meant to hold, not its current members.
  It raised no finding: the review's F2 records the gap as existing debt, and card 7 adds only its own package.
- fix-affected-set: no findings. Both findings were checked.

## Disputed

None.

## Deferred

- **Gate: create-already-exists on every Create target of cards 07–12.**
  Cards 07–12 make up Webster batch 7, and their work landed as commits `5ec85b9c` to `d7f0465e`.
  `_lym/webster/state.json` records batch 7 as terminal with status `failed`, not `done`.
  The plan gate scopes out only cards in batches recorded `done` (`internal/websterengine/beginbatch.go` `DoneCards`; `internal/planglyph/planglyph.go` `ValidateFormatAfter`).
  So it resolves these six cards against a tree that already holds their work, and every Create target reads create-already-exists (`internal/planglyph/create.go` `createFindings`).
  The review records the same findings in the baseline before this round.
  Of these targets, this round changed only card 7's test target, and that change is F1's fix.
  No plan edit clears the findings without breaking the batch:
  - Removing the targets from **Create**, or moving them to another group, drops the create-not-done check that `lym webster recover-batch 07` runs on them.
  - Dropping the `plan:` prefix, the form Webster gave done cards 02–06, does not help. `createFindings` resolves glyph-shaped Create targets the same way.
  Clearing the findings needs one of two things outside this fixer's write surface:
  - batch 7 recorded as done, for example after `lym webster recover-batch 07` succeeds;
  - or a gate that subtracts the baseline, or treats a failed batch's landed cards as begun the way `DispatchScope` does.
  My parent, `ly:orch` (creel #715), ruled to accept these findings as baseline. They come from Webster's unrecorded batch 7, not from a fault in the plan, and `lym webster recover-batch 07` settles them once the run is back in Webster. No further plan edit is made for them.
- **Gate: create-new-unit on cards 21 and 23.** These are informational, not defects. Cards 21 and 23 create new files on purpose.
