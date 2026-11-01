# Fixer-report, round 1 (code)

## Changes

- **F1** (MEDIUM), fixed as proposed, in two commits.
  `webster-body-implementer.md` line 58 now sanctions an out-of-target sweep edit only for a hit that references a name the card retires.
  It states the edit is not a failure fix, so the self-fix cap does not apply, and a hit that matches a retired string by accident is left alone.
  The first and second deviation ordering is unchanged, and line 43 now lists the sweep edit as a third bounded deviation (added after the self-check below).
- **F2** (LOW), fixed as proposed.
  Deleted the two added `inOrder` strings in `template_test.go` and restored the case title.
- **F3** (LOW), fixed as proposed.
  A new sentence in the sweep paragraph says that for a symbol rename the sweep is the same grep as the rename paragraph's zero-reference grep, widened to tests, `testdata/` and fixtures, so one grep serves both.

`lym gate test ./internal/websterengine` passes after the final commit.

## Self-check

- fix-consistency, MEDIUM: line 43 said two sanctioned deviations while line 58 sanctioned a third. Fixed as proposed by extending line 43 to three.
- fix-proposal, MEDIUM: the same finding as above, merged.
- fix-consistency, LOW: `git mv` file renames get no word on the sweep. The review raised it only as a note and its `fix` did not ask for a change, so no change was made.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
