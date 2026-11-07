# Round 1 fixer-report — discussion

## Changes

All changes are in `_lym/discussion/decision-record.md`.

- **F2**, fixed as proposed.
  D2's `testing.TB` bullet refuses `Chdir` and `ArtifactDir` beside `Cleanup`, `Setenv` and `TempDir`, and its criterion names all five.
  D2's Bound now names what the TB misses: `os.Setenv`, `os.MkdirTemp`, `git config --global` into the hermetic global config, and a kept `Context()`.
  Review bounds them, and a new D7 Review criterion checks it.
- **F3**, fixed as proposed.
  D1's Bound gives the `internal/websterengine/gitwrap.go` worktree read the same porcelain fallback on the reader's reftable error as `List`, and keeps the porcelain path in both callers for that case.
  The D1 reftable criterion covers the webster read too.
- **F4**, fixed as proposed.
  `builtin/worktree.c` and `worktree.c` were read at tags `v2.43.0` and `v2.53.0`, `refs.c` was read in part at both tags, and the 2.31.0 release notes were read too.
  D1 gains a paragraph for the 2.43.0 floor, with its sources.
  The floor matches 2.53.0 on ordering (`pathsort`, `fspathcmp`), the porcelain `locked` and `prunable` lines (since 2.31.0), the skipped entry with no `gitdir`, the detached all-zero entry with no `HEAD`, and the resolved main path.
  The one difference is that 2.43.0 does not resolve a relative `gitdir`.
  D1 states that difference, and the relative-`gitdir` parity test skips below git 2.48.
  The Open risks line is dropped.
- **F5**, fixed as proposed, through its second option.
  `TestNoticeSite` is dropped from D8's move and from D9 task 11.
  D8 says it stays at tier 1 because the battencli task does not edit `internal/battencli/notices_integration_test.go`.
  `noticeHub` has two sites, below D7's three-site threshold, so naming it as a second fixture would break D7's own rule.
- **F6**, fixed as proposed.
  D4 says the non-bare initialisers write the five keys and the bare ones write `core.fsmonitor`, `maintenance.auto` and `gc.auto`, each as today.
  The D4 criterion now reads the keys each initialiser writes today.
- **F7**, fixed as proposed.
  D7 keeps in the test any step that leaves the board's HEAD unequal to its upstream tip, ahead or behind.
  The twin wording in the D2 criterion ("behind its upstream") now matches.

## Self-check

- fix-consistency, LOW: the relative-`gitdir` criterion lacked the skip below git 2.48. Fixed as proposed: the criterion now names it.
- fix-consistency, LOW: in D2, "register none of them" no longer fit `Chdir` and `ArtifactDir`. Fixed as proposed: it now reads "call none of them".
- fix-consistency, LOW: the `PATTERN-fabric-boundary` constraint did not mention the webster porcelain fallback. Fixed as proposed: the constraint names it.
- fix-claims, LOW: D2's Bound framed `os.Setenv` and `git config --global` as leaving a path in the template. Fixed: the Bound now gives the `os.MkdirTemp` path harm and the harm to every later test in the binary separately.
- fix-claims, LOW: the "Sources: refs.c" claim overstated a truncated fetch. Fixed: the missing-`HEAD` bullet now rests on `add_head_info` being the same at both tags plus the 2.53.0 experiment, with `refs.c` named as a partial read that agrees.
- fix-proposal: no findings.
- fix-way-forward, LOW: a build step making one of the process-wide calls in D2's Bound had nowhere to go in D7. Fixed: D7's stay-in-the-test sentence now names such a step.

## Disputed

- **F1**, premise false.
  The finding says a build step that commits through fabric spawns a detached push that outlives the build.
  No such child spawns in a test binary, and every template build runs in one.
  `SpawnDetachedPush` returns `ErrSpawnUnderTest` and forks nothing when `testing.Testing()` is true (`internal/fabricengine/spawn.go:71-73`; doc: "Returns ErrSpawnUnderTest, forking no child, in a test binary").
  `internal/fabricengine/doc.go:335` says "`SpawnDetachedPush` returns `ErrSpawnUnderTest` in a test binary, so no test forks a real push child".
  `Commit` discards the result (`_ = spawnDetachedPushFn(pushRequest)`, `internal/fabricengine/commit.go:152`), and the `UnderTest` row of the test in `internal/fabricengine/spawn_test.go` pins this.
  So no second writer reaches the template after the pack, and D3's Bound needs no extension.
  The real effect of such a step is the board left ahead of its upstream, which F7's change keeps out of the build.

## Deferred

None.
