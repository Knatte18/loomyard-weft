# Round 3 fixer report — discussion

Every finding was validated against `internal/websterengine/reset.go` and the decision record, and fixed in `_lyx/discussion/decision-record.md`.

## Fixed

- **F1** (MEDIUM, decision): D3 now names a closed archive-only set: no recorded start, a recorded start missing from the repository, starts with no single oldest commit and no common ancestor (`resolveResetSHA`'s octopus refusal), or a resolved start that is not an ancestor of HEAD.
  It states why none keeps a refusal: each refuses today with `lyx webster run --fresh`, which D3 retires from the loom path.
  The dropped-refusals sentence now names the no-common-ancestor refusal too, and the Acceptance line for `--to start` lists all four states.
- **F2** (LOW, consistency): D4 states once which resets list left paths: every reset to start in either mode, and every standalone reset.
  It also names what a pair reset leaves (untracked files).
  The `--to start` Acceptance line now promises the listing in either mode, which agrees with the standalone Acceptance line.
- **F3** (LOW, consistency): the D7 Acceptance clause now refuses a change outside the Card Index and names all five pinned regions, matching D7's Decision and Bound.
- **F4** (LOW, consistency): D1's guard sentence now excepts `start`'s archive-only path and points to D3 for its bound.

## Disputed

None.

## Deferred

None.
