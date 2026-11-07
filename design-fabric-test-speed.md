# Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes

`fabric-test-speed` · Note · sketch, fabric, testkit, git

A Motif (operator, 2026-11-07): settle how to take `internal/fabricengine`'s tier 1 time down, and split the work into darn tasks in an order that keeps every test's coverage.
The operator wants tests faster as a priority: a darn run's Finalize gate spends minutes on tier 1, most of it in one package.

## Seen

`docs/research/speed/tier1-test-revision-2026-11-06/report.md`, read in full before Discussion:

- A whole tier 1 pass takes 75 s on a quiet neve and a median of 111 s beside other gates; `internal/fabricengine` is 57 s of the 75 s, about 77 percent of the wall.
- The package makes 98,760 git processes a run, 63 percent of its serial time.
- None of the levers of `docs/research/speed/fabric-test-speed-survey-2026-11-03.md` has landed.
- Starting only a tier's test binaries is the report's item 1 and is `gate-tier-binaries-only`, in flight; this Motif leaves it out.

## In scope: the report's ranked items 2 to 6, 8 and 10

- **`worktree list` through `dotgit`** (item 2): read `.git/worktrees/*/{gitdir,HEAD,locked}` in `internal/dotgit` behind `fabricengine.List` and the call in `internal/websterengine/gitwrap.go`, with a test that checks the reader against `git worktree list --porcelain` on a repository with a detached, a locked and a prunable worktree. About 5 s, and fewer processes in every `lym` verb that lists worktrees.
- **Registered hub templates** (item 3): a `hubforge` entry that takes a key and a build function, builds the template once per test binary under a lock per key and copies it as a shape does; then the largest fixtures move onto it (`newAnchorHub`, `newOriginHub` and its layers, `newMergePairFixture`, the live-state arrange, `newStaleHub`, `beamHub`, `newBadReedUpFixture`), one package per task. 9 to 11 s. The tests that exercise `Add` keep building their pairs.
- **Folds of overlapping rows** (item 4), such as `TestVerbCases_CleanState` into the cross product, each fold keeping what its row alone pins. About 2.5 s.
- **A slimmer `CopyHub`** (item 5): no fingerprint walk per copy, a packed template, no `amend` and `push --force` per copy; the fingerprint stays on `SharedHub`. 1.5 to 3 s.
- **Quiet git config written from Go** in the `gitkit` and `hubforge` initialisers (item 6), after checking every test binary runs under `HermeticGitEnv`. About 2 s.
- **`CopyHub` in the `NewHub` fixtures, and the hook script without `dirname` and `basename`** (item 8). About 2 s.
- **Tier 1 tests that run no git move to tier 0** (item 10), as each is touched.

## Wanted

- The decision record states each change, the packages it touches, what it may cost a test in coverage and how that is guarded, and the order of the tasks with their dependencies.
- Each task is sized for one darn run; the template moves are one package per task.
- The record names the saving to re-measure after the tasks land, with at most one whole tier 1 pass and one `internal/fabricengine` pass through `lym gate test`.
