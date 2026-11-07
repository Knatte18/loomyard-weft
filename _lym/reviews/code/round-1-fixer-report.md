# Round 1 fixer-report

## Changes

- F1 (fixed as proposed): rows added to `TestListWorktrees` in `internal/dotgit/worktrees_test.go` for explicit `refStorage = files`, a five-link symbolic ref chain, a six-link chain error, an `ENOTDIR` loose ref falling through to `packed-refs`, a plain file under `worktrees/`, a relative gitdir through a symlinked admin entry and a relative gitdir with a missing target behind a symlinked prefix.
  Commits `7c9be83a` and a follow-up from the self-check.
- F2 (fixed as proposed): `parseGitVersion` extracted from `SkipBelowGitVersion` in `internal/gitkit/query.go`, with `TestGitVersionGate` in `internal/gitkit/query_test.go` covering plain, Windows-suffixed and Apple-suffixed output, the exact minimum and numeric ordering.
  Commit `7455b2b4`.
- F3 (fixed as proposed): comment in `internal/fabricengine/commit_integration_test.go` now says the worktree list read fails in the anchor path.
  Commit `175e56c2`.

## Self-check

- fix-code-correctness, LOW: the missing-target row did not pin the walk-up in `resolveExistingPrefix`; fixed with own change, the missing tail now sits behind a symlinked directory so only the walk-up yields `root/real/gone`.
- fix-proposal, LOW: symlinked-admin-entry row might not discriminate; no change, the correctness lens traced that `FindRoot` keeps the link path, so lexical resolution gives `root/deep/rel` against the wanted `root/rel`.
- fix-proposal, LOW: depth limit not pinned at its boundary; fixed, the success row is now a five-link chain.
- fix-affected-set, LOW: `internal/preflight/preflight_integration_test.go:266` comment may be stale in the way F3 describes; disputed, see below.
- fix-consistency: no findings.

## Disputed

- Ground `premise false`, fork finding on `internal/preflight/preflight_integration_test.go:266`: the lens itself could not confirm that `PrimeName` reads through `dotgit`, and found no `dotgit` use in the `preflight` production files.
  The review rejected the same-class loose wording at three sibling sites.

## Deferred

Nothing deferred.
