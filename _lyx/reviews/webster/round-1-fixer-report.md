# Fixer-report, webster round 1

## Changes

- F1: fixed as proposed, commit `3ab5c4da`.
  `PATTERN.md` has seven entries back at their merge-base text: `PATTERN-hub-wide-config`, `PATTERN-driver-choice-single-site`, `PATTERN-stencil-ownership`, `PATTERN-friction-capture`, `PATTERN-testkit`, `PATTERN-hubforge-fixtures` and `PATTERN-markdown-link-integrity`.
  The review said to use the parent's current text where `main` changed an entry since the merge base.
  `main` changed three of the ten, so those take its text:
  - `PATTERN-push-both-sides` now reads "the pair's code branch goes along, and a status push is detached".
  - `PATTERN-durable-vs-ephemeral-state` and `PATTERN-edit-directive` already equal `main`'s shortened text, so they are unchanged.
  Against the merge base, `PATTERN.md` now differs only in the declared `PATTERN-sparse-hub-base` line and in those three entries, each equal to `main`.

- Gate finding (`PATTERN.md` past its byte headroom after F1's restored clauses): fixed, same commit as the report.
  - `PATTERN-sparse-hub-base` keeps its one-line invariant, and its unit detail moves to the new `pattern/PATTERN-sparse-hub-base.md`, linked as background.
  - `PATTERN-recipe-setting-lookup` takes `main`'s shortened one-liner, with `main`'s `pattern/PATTERN-recipe-setting-lookup.md` added as its background file.
  - `PATTERN.md` is 15676 bytes, down from 15935 and below the 15743 the gate asked to reach.

## Disputed

- F2, ground `premise false`.
  `internal/fabriccli/cli_test.go` holds `TestRunCLI_CloneEndToEnd`, which installs a pre-commit hook that rejects the base-sparse marker.
  The clone's board commit then fails, the test asserts a non-zero exit and an error naming `pre-commit`, and it asserts the marker file is absent (`os.Stat` returns not-exist on `BaseSparseMarker` of the rejected hub's board).
  That is the forced commit failure and the marker-gone assertion the finding asks for, so no test is added.

## Deferred

None.

## Self-check

- fix-code-correctness: returned no findings.
- fix-consistency, MEDIUM: the `PATTERN-durable-vs-ephemeral-state` and `PATTERN-edit-directive` one-liners disagree with their `pattern/` background files.
  Fixed as proposed, no change.
  `main` carries the same shortened one-liners and the same background text, so the disagreement comes from the parent, and the review told the fixer to take the parent's text.
  Resolving it means a change to `main`'s own decision, outside this round's surface.
- fix-consistency, LOW: the `PATTERN-push-both-sides` text is `main`'s wording, not the merge base's.
  Fixed as proposed, no change; the review's parent-text clause applies and the background file agrees in substance.
- fix-proposal, LOW: the same `PATTERN-edit-directive` disagreement.
  Same disposition as the fix-consistency MEDIUM.
- fix-affected-set, MEDIUM: the same two entries left shortened.
  Same disposition as the fix-consistency MEDIUM.
  Its other checks (the seven restored entries, the `PATTERN-push-both-sides` text equal to `main`, and the covering test for F2) found nothing.
