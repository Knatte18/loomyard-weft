# Fixer-report — darn review, round 2

## Changes

- F1 (LOW), fixed as proposed, commit `configsync: fix F1 — report a former default dropped from a prime-seeded source`.
  - `internal/configsync/movekeys.go`: added `moveFile.reported`, set in `takeKey` on `reportOn` before any branch appends a line.
  - `moveKeys` lists files with `changed` or `reported` in its results, while the write loop still writes only `changed` files.
    A reported-only Result is never `Applied`.
  - `internal/configsync/configsync_test.go`: new `TestReconcile_MovesKeysAcrossModules` row with `darn.yaml` absent from the board dir, the prime holding `review_hard_max_bounces: 2` and `SeedDefaults` on, asserting the "dropped, a former template default" line.
    The row fails with results built from `changed` only.
  - The proposal named the drop branch, but `reported` is set once at the top of `takeKey`, so the kept-target branches of a virtual source now surface their lines too.
- Self-check follow-up, commit `configsync: fix F1 — moveKeys doc names the report-only Result`: the `moveKeys` doc comment now says it returns a Result for a file that only carries report lines.

## Self-check

- fix-consistency, LOW: `moveKeys` doc comment still said "one Result per file it changes or would change". Fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
