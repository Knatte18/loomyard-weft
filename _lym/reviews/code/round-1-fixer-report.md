# Fixer report — code review round 1

## Changes

Both findings are coverage gaps, fixed as proposed, in one commit (`00570a7f`, `codexengine: fix F1, F2 — ...`).
Both findings touch `prepare_test.go`, so they share a commit instead of one each.
`lym gate test ./internal/shuttleengine/codexengine` passed (tier0, 104 passed, 0 failed).

- **F1**: `internal/shuttleengine/codexengine/fixture_test.go` gains `routeGitDirsThroughLink`, which links `main-link` to the main checkout and rewrites the worktree's `.git` gitfile to name its `gitdir:` under the link, so the `commondir` resolves under it too.
  `TestPrepare_Launch` gains a `GitDirsThroughLink` row that calls it and expects the resolved paths `wantGitDirs` returns.
  Dropping either `EvalSymlinks` call in `readCheckout` now fails the launch-line comparison.
- **F2**: `TestPrepare_ExpandsHomeInWritableRoots` in `prepare_test.go` sets `HOME` and `USERPROFILE` to a folder under `repo.base`, names `~/cache` in `writable_roots`, and expects the resolved folder among the launch's writable roots.
  It is a separate test without `t.Parallel()`, and its comment names the environment variables as the reason.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, the `GitDirsThroughLink` row does not pin the `EvalSymlinks` on the main checkout root (`home.go:88`), which sibling trust-root assertions use.
  No change made: the finding proposes an improvement the review never asked for and that the fix did not make worse, which the fix-check rubric says is not a finding.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
