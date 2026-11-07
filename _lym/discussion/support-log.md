# Support log — fabric-test-speed

## Interview

### Turn 1 — the board and the report (autonomous)

Read the entry, the board list, `test-real-time-waits` and `gate-tier-binaries-only`, and the report in full.
No `Decision for fabric-test-speed:` note is on the board.
`test-real-time-waits` owns the report's items 7 and 9 and `gate-tier-binaries-only` its item 1, so both stay out of scope.

### Turn 2 — exploration (autonomous)

Four subagents answered: the `worktree list` callers and `dotgit`, `hubforge`'s templates and copy, the fixture sites, and git's worktree format.
Findings that shaped the design:

- `fabricengine.List` parses path, HEAD, branch, detached and prunable; `websterengine` uses paths alone; `internal/dotgit` is the sole `.git` parser, standard-library only.
- `CopyHub` walks a fingerprint per copy, relocates by text rewrite, then amends and force-pushes the board to fold in the relocated `.lym-warp`; one global mutex serialises template builds; the fixture-build env mark is already ref-counted.
- `HermeticGitEnv` sets the gc, maintenance and fsmonitor keys globally, and `cmd/lym/hermeticenv_test.go` guards its use, with an allowlist.
- `newMergePairFixture` is already a plain copy plus `OpenFabric`; `newStaleHub` always takes `pair-a`; `newBadReedUpFixture` varies only its seed; a second `beamHub` lives in `internal/battencli`.
- A scratch repository on neve under git 2.53.0 settled the facts the subagent gave from memory: linked worktrees list in path order; a fresh worktree whose directory is gone is prunable; a locked one is not.

### Turn 3 — design picks (autonomous)

The picks are in the question ledger.

### Turn 4 — lens review

Six forks returned reports: interactions, existing-paths, bound-breaker, walkthrough, board-scope and leanness.
Two git facts the bound-breaker raised were settled on the scratch repository first: an entry with no `gitdir` file is left out of `list`, and one with no `HEAD` file is listed detached with the all-zero SHA.

- bound-breaker 1, BLOCKING, D3 go-git fold skips the tracking ref's reflog that `reflog.go` reads: fixed; the fold stays on git, dropping only the separate `add`.
- walkthrough 1, BLOCKING, the live-state arrange returns a fixture value and some arranges call `TempDir`: fixed; D7 splits each arrange into a build and a derivation from the copy, keeps arranges with outside paths in the test, and D2 refuses `TempDir` in a build.
- interactions 1, MEDIUM, D3's fold against builders that commit on the board or leave it behind origin: fixed; D2 requires the board's HEAD to equal its upstream after a build, D3 folds into the tip, and the criterion reads "history as built".
- interactions 2 and bound-breaker 4, MEDIUM, D6's new `SharedHub` breaks D3's bound: fixed; the three anchor refusal rows share one `CopyHub`, and D3's bound says the split adds no `SharedHub` site.
- interactions 4, walkthrough 3 and bound-breaker 5, MEDIUM, tasks 5 and 6 both edit `replay_integration_test.go`: fixed; the `EndedSet` rows move into D5.
- existing-paths 1, MEDIUM, the main path must be resolved through symlinks: fixed in D1, with a criterion through a symlinked path.
- existing-paths 2 and bound-breaker 6, MEDIUM, where a registered template's base is built: fixed; D2 builds it in its own `lymtemp` root as a shape is built, never from a copy.
- walkthrough 2, MEDIUM, a registration's identity cannot compare closures: fixed; a registration is a package-level value compared by identity, and a build captures no per-call data.
- bound-breaker 2, MEDIUM, a missing `gitdir` is left out, not prunable: fixed in D1 after the scratch check, with a criterion.
- bound-breaker 3, MEDIUM, a half-written entry during `worktree add`: fixed in D1, as git lists it, with a criterion.
- leanness, MEDIUM, PATTERN bullets with unchanged overview text: fixed in part; the bullets stay, since the spec lists every changed entry, and the Notes say the change is to the backgrounds.
- interactions 3 and walkthrough 5, LOW, pack and fingerprint order: fixed; D2 and D3 state build, build function, pack, fingerprint.
- walkthrough 4, LOW, where the folded preconditions go in `runCell`: fixed; after the arrange, before the state is applied.
- existing-paths 3, LOW, the copy's lookup by key beside `Shapes`: fixed in D2.
- existing-paths 4, LOW, `git gc` does more than packing: fixed; D3 names `repack -ad` and `pack-refs --all`.
- existing-paths 5, LOW, installed hubs keep the old hook: fixed; D5 says so.
- bound-breaker 7, LOW, the arrange's fixture value: fixed with walkthrough 1.
- bound-breaker 8, LOW, case-folded path order where `core.ignorecase` is true: fixed in D1.
- bound-breaker 9, LOW, the tracking ref written outside the remote-refs lock: rejected; the fold stays on `git push`, as today.
- board-scope 1, LOW, the 48 s holds only once the peer's test targets are switched on: fixed in D10 and Open risks.
- board-scope 2 and 3, LOW, the narrowed `newMergePairFixture` and the dropped `HermeticGitEnv` check: rejected as defects; the lens itself found them reasoned and recorded.
- leanness, LOW, Notes repeating Constraints and D1: fixed; the Notes keep the file pointers.
- leanness, LOW, Auto-mode assumptions restating decisions: fixed; each names its pick and points at its decision.
- leanness, LOW, the gate-rule constraint and the child-processes constraint: fixed; both cut.
- leanness, LOW, the `SharedHub` site count in D3: fixed; D3 says today's sites only read.
- leanness, LOW, the report's figures in rationales: fixed in part; D7 cites the report's section, and the other figures stay as each decision's one headline number.
- leanness, LOW, D6's survey rows left to reading: fixed; the rows the report did not read again are out of scope.

## Rejected alternatives

- More `Shapes` in place of registered templates: the closed set cannot hold start states with fixed commits, pushes and slugs.
- Dropping the per-repository quiet config onto `HermeticGitEnv`: it rests on every subprocess inheriting the environment, which the allowlist leaves open; writing the same keys from Go removes the processes without that risk.
- Folding the relocated binding in-process with go-git: go-git writes no reflog, and `internal/fabricengine/reflog.go` reads the tracking ref's reflog, so copies would differ from today's in what the origin tests see; hand-writing reflogs in `hubforge` would be a fragile second writer of git's format.
- Marking `.lym-warp` skip-worktree in copies: it hides a real change to the file from tests that read dirtiness.
- One `internal/fabricengine` template task: about 150 sites is more than one darn run, so it splits by fixture group, each still one package.
- `hubforge.SharedHub` for `TestAnchorChange`'s refusal rows: without the per-copy walk, a stray write would reach other copies; one copy shared by the three rows costs one copy.
- Template moves for `newResetFixture` in `internal/webstercli`: the entry lists the fixtures to move, and it is not among them.
- A shared repository across rows, or longer flow tests: the report weighs both and finds the template the shape that removes the build without hiding a row.

## Review rounds

_No rounds yet._

## Question ledger

- Q1, resolved, auto-pick: where the worktree reader lives. `internal/dotgit`, as the sole `.git` parser, resolving refs itself so it stays standard-library only.
- Q2, resolved, auto-pick: reftable repositories. Fall back to the porcelain path; lym's code holds no reftable handling, and the fallback keeps `List` right there.
- Q3, resolved, auto-pick: what a registered build returns. Nothing; tests derive paths from the copy's `Hub`, so nothing needs a generic rebase.
- Q4, resolved, auto-pick: a builder's cleanups. `Cleanup`, `Setenv` and `TempDir` are refused through the `TB` the build gets, so no build ties template state to its first caller.
- Q5, resolved, auto-pick, revised after the lens review: how to slim the per-copy fold. It stays on git for the reflogs, dropping the separate `add`; this departs from the entry's "no `amend` and `push --force` per copy", which a go-git fold would meet only by losing the reflog entries tests read.
- Q6, resolved, auto-pick: quiet config. Written from Go into each repository's `config`, keeping the layer.
- Q7, resolved, auto-pick: `newMergePairFixture`. Moves only for layers repeated at least three times, since it is already a plain copy.
- Q8, resolved, auto-pick: the `internal/battencli` `beamHub`. Included, as the entry names `beamHub` and it has four sites.
- Q9, resolved, auto-pick: the hubreconcile `NewHub` sites. In the `internal/hubreconcile` template task, which edits the same file.
- Q10, resolved, auto-pick: task order. Folds before the `internal/fabricengine` moves; D3 after D2; the `EndedSet` rows with D5 so tasks 5 and 6 edit disjoint files.
- Q11, resolved, auto-pick: the survey's unread overlap rows. Out of scope, since no fold should rest on an unconfirmed overlap.
- Q12, environment: the scratch repository for the git facts sits under `.scratch/wt`; removing it with `rm -rf` was denied, so it stays for the operator to remove.
