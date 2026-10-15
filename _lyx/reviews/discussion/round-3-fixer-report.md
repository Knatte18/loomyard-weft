# Round 3 fixer-report — discussion

## Changes

- F1 (MEDIUM, design), fixed as proposed.
  The premise holds: `internal/loomengine/template.yaml` carries the old comments inline on each key ("empty takes haiku", "0 takes 10"), and D5's rewrite skipped a hub with any pinned key whole.
  D5's migration paragraph in `_lyx/discussion/decision-record.md` now has two steps in one pass.
  The value rewrite is unchanged.
  A per-key comment refresh runs after it, replacing any of the four keys' comments that equal the old template's text with the new template's, leaving values and hand-edited comments alone.
  Both steps are idempotent.
  The migration acceptance criterion now says a `loom.yaml` with one key off the old defaults keeps that key and every other value, each of the four keys carries the new template's comment, and a second reconcile changes nothing.

## Disputed

None.

## Deferred

None.
