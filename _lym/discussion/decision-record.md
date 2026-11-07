# Decision record — fabric-test-speed

## Goal

Take `internal/fabricengine`'s tier 1 time down, and with it a whole tier 1 pass, by removing git processes and per-test hub builds.
The work is the report's ranked items 2 to 6, 8 and 10 (`docs/research/speed/tier1-test-revision-2026-11-06/report.md`), split into darn tasks, each sized for one darn run, in an order that keeps every test's coverage.

## Scope

In:

- `worktree list` read in-process through `internal/dotgit`, behind `fabricengine.List` and the call in `internal/websterengine/gitwrap.go` (D1).
- Registered hub templates in `internal/hubforge` (D2), and the listed fixtures moved onto them, one package per task (D7).
- A slimmer `hubforge.CopyHub` (D3).
- Quiet git config written from Go in the `gitkit` and `hubforge` repository initialisers (D4).
- `hubforge.CopyHub` in place of `hubforge.NewHub` in the fixtures that need no fresh clone, and the post-checkout hook script without `dirname` and `basename` (D5).
- Folds of overlapping `internal/fabricengine` tests the report read (D6).
- Tier 1 tests that run no git move to tier 0 when a task of this split touches their file (D8).
- The task order (D9) and the re-measure (D10).

Out:

- Starting only a tier's test binaries: `gate-tier-binaries-only`, in flight.
- The report's items 7 and 9, real-time waits and repeated builds: `test-real-time-waits`.
- go-git or `dotgit` for the other read sites of the report's section 6: the report ranks them as one low task, and the entry does not list it.
- Template moves for fixtures the entry does not list, such as `newResetFixture` in `internal/webstercli`.
- The survey's overlap rows the report did not read again (its section 4's last row): no fold rests on an overlap nobody confirmed.

## Decisions

### D1 — `worktree list` through a `dotgit` reader

`internal/dotgit` gains a reader of a repository's worktrees from the common dir: the main worktree first, then each `worktrees/<id>/` entry.
Per entry it reads `gitdir`, `HEAD` and `locked`, and resolves `HEAD` to a SHA through the loose ref, then `packed-refs`, in the common dir.
`fabricengine.List` and the `worktree list` call in `internal/websterengine/gitwrap.go` read through it and start no git process; `List` keeps its `WorktreeEntry` result and callers.
The reader matches `git worktree list --porcelain` as git 2.53.0 prints it:

- The main worktree's path is resolved through symlinks, as git prints it; a linked worktree's path is the directory of the `.git` its `gitdir` names.
- Linked worktrees follow the main one sorted by path, compared case-folded where the repository's `core.ignorecase` is true, as git's path compare does.
- A relative `gitdir` (written under `worktree.useRelativePaths`, git 2.48 and later) resolves against its `worktrees/<id>/` directory; an absolute one is taken as is.
- An entry whose `gitdir` file is missing or unreadable is left out, as git leaves it out.
- An entry is prunable when it is not locked and its `gitdir` names a `.git` that does not exist; the file's age plays no part in `list`.
- An entry with no `HEAD` file, as one `git worktree add` is still writing, is listed detached with the all-zero SHA, as git lists it; `List` and its callers never fail on it.
- A detached `HEAD` gives `List`'s detached branch value; a branch whose ref does not exist yet gives the all-zero SHA.
- A bare main repository stays an error, as `List` returns today.

Bound: a repository whose `extensions.refStorage` is not `files` has no ref files to read, so the reader refuses it with a named error and `List` falls back to the porcelain path, which stays for that case alone.
The reader stays standard-library only and spawns nothing, as `PATTERN-leaf-packages` holds `internal/dotgit`.

Rationale: `worktree list` is about 15 percent of `internal/fabricengine`'s git processes and about 5 s of the package's wall, and `lym` calls `List` three or more times in each `Add` and `Remove`.
`internal/dotgit` is already the sole `.git` parser, and `PATTERN-gitrepo-client-boundary` gives local reads to it.
Evidence: git 2.53.0 on neve; `man git-worktree` (porcelain format, `list` ignores expiry); git 2.48.0 release notes (relative worktree links).
A scratch repository on neve, reached through a symlink, showed:

- The main path printed resolved.
- Linked worktrees in path order.
- A fresh worktree with its directory moved marked `prunable gitdir file points to non-existent location`, and a locked one not marked.
- An entry with its `gitdir` file removed left out.
- An entry with its `HEAD` file removed listed as `detached` with the all-zero SHA.

### D2 — Registered hub templates in `hubforge`

`internal/hubforge` gains a registration, a package-level value holding a key, a base `Shape` and a build function, and a copy call that takes the registration.

- The template is built the way a shape's template is built today, in its own `lymtemp` root: the bares, the clone and the base shape's pairs, then the build function on that hub; never from a copy.
  Then come D3's pack and the fingerprint, in that order.
- It is built once per test binary on first use, and copied through the same relocation and rebase as a shape's copy, through a lookup by key beside the closed `Shapes` set.
- The build function returns nothing and captures no per-call data: a test derives the paths it needs from the copy's `Hub`, and a start state that varies needs its own registration.
- Builds take a lock per key, so templates of different keys build at once; the shared map takes its own short lock, and the existing ref-counted fixture-build mark already allows concurrent builds.
- Two different registrations with one key fail the second caller naming the key; a registration is compared by identity.
- A build that fails marks the template failed: its first caller fails with the build's error, and every later caller fails naming the key without rebuilding.
- The build function gets a `testing.TB` whose `Cleanup`, `Setenv` and `TempDir` fail the build naming the key, so no build ties template state to its first caller's lifetime or places a path outside the template root.
  The helpers a shape build uses today, `AddPair`, `AddPairWith` and the `gitkit` commit helpers, register none of them.
- A build leaves the board's HEAD equal to its upstream tip, or the build fails naming the key, so D3's fold and push never drop a commit the builder made.

Bound: a template built wrongly fails every test that copies it, loudly and naming the key; a test that wrote to its copy touches nothing shared, as with a shape's copy.

Rationale: the fixtures' start states are a hub plus commits, pushes and pairs under fixed slugs, which the closed shape set cannot hold; a registered builder holds any of them, built once at about 0.18 s and copied at about 17 ms, and pays from its third use.

### D3 — A slimmer `CopyHub`

- `hubforge.CopyHub` no longer runs the fingerprint walk; it still fails on a template marked poisoned.
  `hubforge.SharedHub` keeps the walk before and after the test, and poisons the template on a change.
- A template's repositories are packed once at build time with `git repack -ad` and `git pack-refs --all`, after the build function and before the fingerprint, for shapes and registered templates alike.
  A copy then copies a few pack files in place of the loose objects; relocation rewrites only text files, and a pack or `packed-refs` holds no template path.
- The fold of the relocated `.lym-warp` binding stays on git, so HEAD, the branch and the tracking ref keep the reflog entries `internal/fabricengine/reflog.go` reads.
  Its separate `git add` goes: the amend names the binding file, and the push follows, two processes in place of three.

Bound: without the walk on each copy, a stray write by a `hubforge.SharedHub` test can reach copies taken before that test's cleanup poisons the template; copies taken after fail.
Today's `hubforge.SharedHub` sites only read, and this split adds none.
A copy's board stays clean against its HEAD and upstream, with its history as built and the binding folded into the tip.

Rationale: copies are about 13 percent of the package's serial time, and the walk and the loose objects are most of each copy.

### D4 — Quiet git config written from Go

The `gitkit` and `hubforge` repository initialisers write their per-repository keys, `user.name`, `user.email`, `core.fsmonitor`, `maintenance.auto` and `gc.auto`, into the new repository's `config` file from Go in one write after `git init`, in place of one `git config` process per key.
The keys and values stay as today, so the per-repository layer stays and nothing comes to rest on `gitkit.HermeticGitEnv`.

Rationale: about 5,600 processes a package run, about 2 s of wall.
Keeping the layer removes the report's open question, whether a test runs without `gitkit.HermeticGitEnv`: the result does not depend on it.

### D5 — `CopyHub` in the `NewHub` fixtures, and the hook without `dirname` and `basename`

- `newRecordsFixture` and the `EndedSet` rows of `TestRecordsReplay`, both in `internal/fabricengine/replay_integration_test.go`, and the `hubforge.NewHub` sites of `internal/adoptcli` take `hubforge.CopyHub` of the same anchor.
  A site whose test is about the clone or the wiring itself keeps `hubforge.NewHub`.
  The `internal/hubreconcile` sites go with D7's `internal/hubreconcile` task, which edits the same file.
- `internal/fabricengine/post-checkout.sh` derives the weft sibling with shell parameter expansion in place of `dirname` and `basename`, and the chain wrapper in `internal/fabricengine/hook.go` derives its directory the same way, falling back to `.` when `$0` holds no slash.
  `git rev-parse --show-toplevel` answers an absolute path with no trailing slash, for which the expansion and the two commands agree.
  A hub that already holds the hook keeps its old script, which still works, since the installer leaves a hook carrying its sentinel alone.

Rationale: a fresh `NewHub` is about 103 ms against 17 ms for a copy; the hook runs on every checkout of a test's pairs, and the two commands are two processes each time.

### D6 — Folds of overlapping `internal/fabricengine` tests

- `TestVerbCases_CleanState` goes: its three preconditions, Add's launcher blocker is a directory, Checkout's branch differs and Pull's bare advanced, move into the cross product's `runCell` in `internal/fabricengine/livestate_matrix_test.go`, checked right after the verb's arrange and before the state is applied.
- The report's section 4 rows that it read or found again by name fold as it names each one:
  - The three setup-free refusal rows of `TestAnchorChange` share one `hubforge.CopyHub`.
  - The two probe subtests of `TestStoreHousekeeping` fold into the gc-prune row.
  - `owned_elsewhere` of `TestReconcileOrigin_Moves` runs its six replies on one pair.
  - The paired settle and backoff rows of `TestPushFailureClasses` assert on the first push, and the backoff rows become one table.
  - The three subtest pairs of `TestBolt_PullThenCommitWritten` become two-row tables.

Each fold keeps every assertion its row alone made, named in the fold's commit, and the report's "alone pins" column is the list a review checks.

Rationale: about 6 s of the package's serial time, about 2 s of wall, with no assertion lost.

### D7 — Fixtures onto registered templates, one package per task

A start state moves onto a D2 template when at least three sites build it; the builder is the fixture's present build steps, moved as they are, so the start state is the same by construction.
Tests that exercise `Add` keep adding their pairs, and a pair whose slug a test chooses per row stays added in the test.
A step that cannot run in a build stays in the test after the copy: one that places a path outside the hub, or one that leaves the board behind its upstream.

- `internal/fabricengine`, anchor and origin: `newAnchorHub` and `newOriginHub`, with `originPair` and `endedHub` where a fixed slug set repeats at least three times.
- `internal/fabricengine`, live state and merge pairs:
  - Each verb's arrange in the cross product splits into a build, registered per verb and anchor, and a derivation of the verb's fixture value from the copy's `Hub`.
  - An arrange that places a directory outside the hub stays whole in the test.
  - The `newMergePairFixture` layers whose branch commits repeat at least three times move; `newMergePairFixture` itself is already a plain copy and gains nothing alone.
- `internal/hubreconcile`: `newStaleHub` with its fixed slug `pair-a`, and the package's `hubforge.NewHub` sites onto `hubforge.CopyHub` per D5's rule.
- `internal/loomcli`: `beamHub` and `newBadReedUpFixture`; the latter's base, a hub with the corrupted reed line and the pair, is the template, and each site's seed runs on its copy.
- `internal/battencli`: its own `beamHub`, a beam, a member and a plain pair.

Rationale: 9 to 11 s of a pass (report, section 2, rank 1), with 2 to 4 s more across the other packages.

### D8 — Tier 0 moves as each file is touched

A task of this split that edits a file holding one of the report's section 4 tier 1 tests that run no git moves that test to tier 0, as `PATTERN-test-speed` places it.
In this split that is `TestEnsure_HeldLockTimesOutWithoutStamping` in the `internal/hubreconcile` task and `TestNoticeSite` in the `internal/battencli` task, each rewritten onto in-memory inputs with its assertions kept.
No task opens a file only to move a test.

Rationale: under 1 s of wall in all, so the move rides on edits that happen anyway.

### D9 — Task order

Each line is one darn task, sized for one darn run, with what it waits for.

1. D1, the `dotgit` worktree reader: waits for nothing.
2. D2, registered templates: waits for nothing.
3. D3, the slimmer copy: waits for task 2, which edits the same file.
4. D4, quiet config: waits for nothing.
5. D5 without the `internal/hubreconcile` sites: waits for nothing.
6. D6, the folds: waits for nothing.
7. D7 `internal/fabricengine` anchor and origin: waits for tasks 2 and 6.
8. D7 `internal/fabricengine` live state and merge pairs: waits for tasks 2 and 6.
9. D7 `internal/hubreconcile`, with D5's sites there and D8's move: waits for task 2.
10. D7 `internal/loomcli`: waits for task 2.
11. D7 `internal/battencli`, with D8's move: waits for task 2.

Rationale: the folds edit the tests the `internal/fabricengine` moves rebuild, so they land first and each fold is judged against today's fixtures.
The rest of the pairs edit disjoint files: D5 alone edits `internal/fabricengine/replay_integration_test.go`, and D2's constraints on a build hold whether D3 lands before or after a move.

### D10 — The re-measure

Once every task has landed, one whole tier 1 pass, `lym gate test --tier 1 --all -- -count=1`, and one `internal/fabricengine` pass, `lym gate test --tier 1 ./internal/fabricengine -- -count=1`, on a quiet neve, read against the report's W1: 75.0 s and 57.0 s.
The expected package time is about 36 s.
The expected pass is about 56 s from these tasks alone, and about 48 s once `gate-tier-binaries-only` has landed and the hub's tier commands carry its opt-in test targets.
The split files the re-measure as a board note.

Rationale: two runs answer whether the package fell below the 35 s at which the other packages set the wall.

## Constraints

- `PATTERN-leaf-packages`: the D1 reader keeps `internal/dotgit` standard-library only and spawning nothing; the background's list of what `dotgit` parses gains the worktree entries and refs.
- `PATTERN-gitrepo-client-boundary`: D1 is a local read, which the entry gives to `dotgit`; the porcelain fallback stays on `gitexec`.
- `PATTERN-fabric-boundary`: D1's change to `fabricengine.List` stays inside `internal/fabricengine`; `internal/websterengine` reads its own worktree's siblings through `dotgit`, as it reads them through git today.
- `PATTERN-test-isolation`: every template is a hub `fabriccli.CloneAndWire` built plus the build function's steps; no hub is hand-assembled, and no test package wraps `hubforge` in its own fixture type.
  The background's hub-fixture paragraph gains the registered template.
- `PATTERN-test-speed`: each test keeps one `tiergate.Require` at the lowest tier that fits; D8 follows it.

## Auto-mode assumptions

- D1: a reftable repository falls back to the porcelain path.
- D3: the binding fold stays on git, for the reflogs, in place of the entry's "no `amend` and `push --force` per copy".
- D4: the per-repository config layer stays.
- D6: the survey's rows the report did not read again are out.
- D7: `newMergePairFixture` moves only for repeated layers, and the `internal/battencli` `beamHub` joins the moves.

## Open risks

- `gate-tier-binaries-only` is in flight with its test targets opt-in; the D10 pass depends on whether it has landed and been switched on, and D10 gives both expectations.
- The porcelain parity rests on git 2.53.0 on neve and the documented format; an older git at the floor, 2.43.0, could order or mark entries differently, which the D1 parity test against the installed git would show.
- D2's builds by key at once raise the load at a binary's start, where several templates build together; the gain assumes those builds no longer queue.

## Acceptance criteria

- D1: on a repository with a main worktree, a branch worktree, a detached one, a locked one with a reason, a locked one without, and a prunable one, the reader's entries equal what `git worktree list --porcelain` reports: path, HEAD SHA, branch or detached, and prunable, in the same order.
- D1: the same parity holds when the repository is reached through a symlinked path.
- D1: an entry with its `gitdir` file removed is left out, and an entry with its `HEAD` file removed is listed detached with the all-zero SHA, as git reports both.
- D1: a linked worktree whose `gitdir` holds a relative path resolves to the same path git reports.
- D1: a branch worktree whose branch ref sits only in `packed-refs` resolves to its SHA.
- D1: a repository with `extensions.refStorage` set to `reftable` gets `List`'s result through the porcelain fallback.
- D1: `fabricengine.List` and the `internal/websterengine` sibling read start no git process on a files-backend repository.
- D2: two registered templates of different keys build concurrently, each exactly once per binary, under parallel first callers.
- D2: a copy of a registered template holds the build function's commits and pairs, and a write to the copy leaves the template unchanged.
- D2: a build that fails fails every later caller naming the key; a build function that calls `Cleanup`, `Setenv` or `TempDir`, or leaves the board behind its upstream, fails naming the key.
- D2: two different registrations with one key fail naming the key.
- D3: a copy's board is clean against HEAD and upstream with its history as built, the copy's records bare's branch tip is the board's HEAD, and the tracking ref's newest reflog entry names that SHA.
- D3: a write by a `hubforge.SharedHub` test poisons the template at cleanup, and a later `hubforge.CopyHub` of it fails.
- D3: a template's repositories hold no loose objects after the build, and a `hubforge.SharedHub` taken right after the build sees no change.
- D4: a repository from each initialiser reads the five keys at today's values through `git config --get`.
- D5: the post-checkout hook still warns on a warp and weft branch mismatch and stays silent when they match; the chain wrapper still runs `post-checkout.user` when it is executable.
- D5: Review: each `hubforge.NewHub` site kept names the clone or wiring behavior that needs it.
- D6: Review: each fold's commit names the assertions its folded row alone made, and each stands in the remaining test.
- D7: each moved package's tests pass with the same assertions, and the tests that exercise `Add` still add their pairs.
- D8: each moved test runs at tier 0 with its assertions kept.
- D9: Review: the split's darn tasks match the order and dependencies of D9.
- D10: Review: the split files the re-measure note, naming the two commands and the expected times.

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.
- `PATTERN-test-isolation` — changes: Testing package: hermetic git and `tmuxkit.Main`; shared support in one `internal/testkit/<kit>/`; hub fixtures via `hubforge`. (test)

## Notes for the plan writer

- Both PATTERN changes are background edits; the overview lines stay as they read.
- `hubforge.CopyHub`'s fold today is `commitRelocatedBinding` in `internal/hubforge/template.go`.
- `List` today parses porcelain in `internal/fabricengine/worktreelist.go`.
