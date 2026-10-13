# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): in `_lyx/landing/summary.md`, the "Design decisions" bullet now says a batch with no resolved or unresolved entry previously rendered the pack-on prompt, with the one-turn read instruction and a `none` or "names no member glyph" subsection per card, and now renders the pack-off prompt.
  The "Without a pack the prompt is unchanged" bullet now lists only pack off and language none, and notes that an empty pack changes the prompt of such a batch.
  The premise held against `internal/websterengine/beginbatch.go:419`.

The edit is in the working tree and not committed: the round's rules forbid any git command against a `_lyx` path, and this finding's only file is under `_lyx`.

## Disputed

None.

## Deferred

None.
