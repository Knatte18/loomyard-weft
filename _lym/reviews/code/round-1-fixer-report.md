# Fixer-report, round 1

## Changes

- F1 (MEDIUM): added two batcher rows to `TestReconcileAll_MigratesToSparse` in `internal/configsync/configsync_test.go`.
  An unmarked worktree holding only `active: "cautious"` is stripped, the file deleted, the marker written, and the resolution moves to `wide`.
  A marked worktree holding the same line keeps it byte for byte.
  Fixed as proposed. Commit `d3ad489c`.
- F2 (LOW): `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` line 5 now says the `wide` cost profile by default.
  Fixed as proposed. Commit `06136120`.
- F3 (LOW): fixed differently from the proposal.
  Proposed: extend `templateProfilesShareTheMeasuredWeights` using batcher accessors, and drop the finding if no accessor exists.
  No accessor exposes a profile's budget or `max_cards`, but the premise holds and a sound change exists that adds none.
  I added the subtest `templateWideDiffersFromCautiousOnlyByBudget` to `TestProfileWeights` in `internal/batcher/config_test.go`.
  It decodes `batcher.ConfigTemplate()` directly and asserts budgets 300000 and 200000 and equal `batchifier` and `max_cards`.
  Commit `7ea62285`.

`lym gate test ./internal/batcher ./internal/configsync` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

No fork returned without a report. Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.
