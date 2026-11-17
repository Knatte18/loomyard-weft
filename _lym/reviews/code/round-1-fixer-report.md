# Fixer-report, code round 1

## Changes

- F1 (MEDIUM), fixed as proposed, commit `56de8cb6`:
  - `internal/raddleparser/check.go`: new `CheckLabelDuplicate = "label-duplicate"` beside `CheckLabelUnknown`.
  - `vocabulary()` reports each repeated part label once, with the way forward "list each module label once in labels"; its doc comment covers the repeat.
  - `contracts/specs/raddle-proposal-spec.md`: `label-duplicate` bullet after `label-unknown`.
  - `internal/raddleparser/check_test.go`: `TestCheck` row `PartLabelListedTwice` wants `[label-duplicate]`.
  - `lym gate test ./internal/raddleparser` passed.
- Self-check finding (fix-consistency), follow-up commit: the spec's `labels` field line now says each label is listed once.

- Gate finding, follow-up commit: the `vocabulary` doc comment's line that wrapped mid-sentence in `internal/raddleparser/check.go` is split into two sentences, one per line.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, stale `labels` field line in the spec; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
