---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-prerun-seed-pull-first.md
    summary: Pull-first runs once per subtree, so the specs pass drops the stencils seed commit the same pre-run just made and reverts those files to upstream
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: Card 7's Bolt tests use the hand-assembled plain-repo fixture, against the record's constraint that Bolt tests build on hubforge and against PATTERN-hubforge-fixtures
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-housekeeping-inflight-prune.md
    summary: IsNoServer matches only tmux's no server running text, but tmux reports an absent socket as error connecting to ... (No such file or directory), so a hub with no tmux server skips gc with a Warn, contrary to D8
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: The drop's containment and ownership are both derived from the target itself, so only a base-name test binds the new destructive primitive, and the Intent states no other bound
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: dirtyScopeTracked refuses the drop on any dirty tracked board file, which the record assumes is a normal state, and the refusal surfaces as a divergence Warn naming git pull --rebase
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: Bolt.Push gains a destructive drop but has no recorder and no result type, and the card does not say which Mutations the push path's dropSeedCommits appends to or how a caller sees it
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: The integration table of commits the drop refuses to classify as seed repeats the IsSeedCommit refuse rows card 6's TestSeedCommitSignature already pins
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: Card 7 bundles three independently reviewable units, the gated drop primitive, the pull-first verb and the push retry
  - id: F9
    severity: LOW
    class: scope
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: The new KindCommitsDropped is not added to manifestObservableKind in livestate_mutationoracle_test.go, which states that every Kind but file_written is declared there
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md
    summary: internal/fabricengine#Bolt is an Edit target, but the Intent describes no change to the Bolt type
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/plan/03-stencilstore-forward-refresh.md
    summary: The ImpactSummary says a production binary stamps writer keys from this card on, but Source.Writer stays zero until card 4 sets it in StencilSource
---

### [BLOCKING] The per-subtree pull-first drops the pre-run's own stencils seed commit

**Location:** `_lyx/plan/08-prerun-seed-pull-first.md` (`seedSubtree`), with `_lyx/plan/07-bolt-pull-first-seed-drop.md` (`pullFirst`).

**Issue:** Card 8 makes `seedSubtree` call `PullThenCommitWritten` once per subtree.
`seedStencilsAt` runs the stencils pass and then the specs pass (`cmd/lyx/stencilseed.go`).
Card 7's `pullFirst` drops every seed commit ahead of upstream, as D4 says, by `ResetKeep` to upstream and then replaying only the non-seed commits.
When both subtrees have a write due, the stencils pass commits `lyx: seed stencils (…)`.
The specs pass's `pullFirst` then finds that commit ahead and drops it.
`reset --keep` sets every path that differs between HEAD and upstream back to upstream's content, so the stencils just written revert to the pulled copy.
The process ends holding only the specs commit.
The next invocation finds stencils due again and drops the specs commit in its stencils pass, and then the reverse happens.
Until a board push lands one of the commits in between, every invocation fetches twice, drops and rewrites, and producers read the reverted stencils.
That defeats the Goal's forward-only seeding.
D4 describes one pass: classify every registry name, then under the board write lock fetch, drop, fast-forward, classify again, write what is due and commit.
No card plans a single pull for both subtrees, and card 8's scenario test never has both subtrees due together.

**Fix:** Run pull-first once per pre-run, before either subtree is written.
`seedStencilsAt` asks `WritesDue` over both registries.
With a write due, it takes the board write lock, runs `pullFirst` once, then reconciles and commits each subtree under its own `SeedCommitMessage` within the same lock hold.
For example, `PullThenCommitWritten` could take an ordered list of message-and-write pairs, or card 7 could expose a lock-holding variant that card 8 drives.
Add a scenario step with both subtrees due that asserts both seed commits survive the pass.

### [BLOCKING] Bolt tests are not built on hubforge

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`, the tests paragraph (`TestBolt_PullThenCommitWritten`, `TestBolt_Push_DropsSeedCommitsOnRebaseConflict`).

**Issue:** The decision record's Constraints say "Bolt tests build on `hubforge`", citing `PATTERN-hubforge-fixtures`.
Card 7 puts its new tests in `bolt_integration_test.go` "against its plain-repo-plus-bare-remote fixture".
`bolt_integration_test.go` is in package `fabricengine` and builds that fixture by hand with `newBoltRepo`/`newBoltBareRemote`.
The drop's gate needs a `_board`-named worktree inside a container (card 7's `ownedBoardWorktree`).
So the test would hand-assemble a piece of hub geometry, which `PATTERN-hubforge-fixtures` forbids ("never hand-assembled").
`hubforge` is reachable here: `warpforward_integration_test.go` and `housekeeping_integration_test.go` are `fabricengine_test` files that use it.
The verbs under test are exported (`Bolt.PullThenCommitWritten`, `Bolt.Push`), so an external test package works.

**Fix:** Put the two new tests in a `package fabricengine_test` integration file, for example a new `internal/fabricengine/boltpullfirst_integration_test.go` on `**Create:**`.
Build the hub through `hubforge`, and move the upstream with a second clone of the records bare, as cards 8 and 10 do.
Leave the existing `CommitWritten` subtests where they are.

### [MEDIUM] IsNoServer misses tmux's absent-socket error

**Location:** `_lyx/plan/12-housekeeping-inflight-prune.md`, the `IsNoServer` and `TestListSessions` sentences.

**Issue:** D8 says "A listing that finds no tmux server on the socket reports no sessions", and an acceptance criterion requires the gc to run when there is "no tmux server".
Card 12's `IsNoServer` "reports whether a tmux error wraps tmux's `no server running` stderr".
tmux 3.6, the version on this machine, gives a socket that does not exist a different message.
`tmux -L lyxreviewprobe-nonexistent list-sessions` printed `error connecting to /tmp/tmux-1000/lyxreviewprobe-nonexistent (No such file or directory)` and exited 1.
tmux prints `no server running on …` only for a socket file that is still there but refuses connections.
A hub whose reed server never started, or whose socket was removed (after a reboot, for example), would therefore count as in flight at every removal and skip the gc with a Warn.
The planned test row fakes the `no server running` text, so it passes while the real case fails.

**Fix:** Make `IsNoServer` admit both forms tmux uses for "no server": `no server running on` and `error connecting to … (No such file or directory)`.
Add a `TestListSessions` row for each, plus a row for an unrelated tmux failure that `IsNoServer` must reject.

### [MEDIUM] The drop's containment and ownership checks bind nothing but a base name

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`, the paragraph that fixes `pullFirst`'s request.

**Issue:** The request sets the container to "the parent of the board dir" and the target to the board dir, so containment always holds.
`ownedBoardWorktree` holds when the base name is `BoardDirName` and `isAnyWorktreeOf(target, target)` holds.
`isAnyWorktreeOf` runs `List` on `repoDir` and checks whether `target` is among the entries (`destroy.go`).
Any git worktree lists itself, so the second condition is always true.
In effect, any git worktree named `_board` anywhere passes the gate that guards a `ResetKeep` and a history replay.
The checks run in the gate's order, as D5 and `PATTERN-fabric-destruction-chokepoint` require.
But the new ownership kind's bound is only a base-name test, and the Intent never says so.
The rubric's attack-surface rule requires a new guard edge to state what bounds it.

**Fix:** Tie ownership to hub geometry rather than to the target alone.
For example, `ownedBoardWorktree` could require that `filepath.Dir(target)` passes the existing `looksLikeHub` predicate and that `target` equals `BoardDir` of that hub, with the container being that hub.
State the resulting bound in the Intent.

### [MEDIUM] The tracked-dirtiness scope refuses the drop in a normal board state

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`: "dirtiness `dirtyScopeTracked`", the `BoltSkipDiverged` mapping and the "a tracked-dirty board refused and restored" subtest.

**Issue:** Board writes only touch the filesystem until a detached `lyx board sync` commits them (`internal/boardengine/sync.go`), so tracked dirty board files are an ordinary state.
The record's Auto-mode assumptions say as much: "`ResetKeep` suffices for the drop: a seed commit touches only seed subtrees, so uncommitted board writes never collide with it."
`dirtyScopeTracked` refuses on any dirty tracked path.
So whenever a board write is pending, the pre-run's drop refuses.
Card 7 maps that refusal to `BoltSkipDiverged`, whose Warn names `git pull --rebase` and `lyx board sync`.
That way forward is wrong for this cause.
The push-retry drop refuses the same way whenever a write lands between `commitDirty` and the push.

**Fix:** Use `dirtyTrackedExcept` scoped to the paths the drop rewrites, or refuse only on dirty tracked paths under the seed subtrees, which is the collision `ResetKeep` cannot carry.
Give a dirtiness refusal its own skip detail, naming the next board sync and a re-run rather than `git pull --rebase`.

### [MEDIUM] The push path's drop has no recorder

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`, the `Bolt.Push` paragraph.

**Issue:** `dropSeedCommits` takes `rec *Mutations`, and the record's Constraints apply `PATTERN-mutation-record` to the drop primitive.
`Bolt.Push(opts SyncOptions) error` has no recorder and no result type, and card 7 changes neither.
The card does not say which `Mutations` the push path passes, or whether the `KindCommitsDropped` entry is discarded.
The callers of `Bolt.Push` (board sync, `hubreconcile`, `fabric reconcile`, `configcli`'s hub-wide edit) would get a destructive history rewrite that no envelope reports.

**Fix:** Decide and state the choice.
Either give the retry path a recorder that a caller can surface, for example a `Bolt.Push` sibling returning a result that embeds `MutationRecord`, joining `destructiveGuardMutatingResultTypes`.
Or state that the push path passes a local recorder and logs the dropped SHAs at Info, and explain why `PATTERN-mutation-record` admits that.

### [LOW] Card 7 repeats card 6's IsSeedCommit refuse rows at the integration tier

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`: "a table of commits the drop refuses to classify as seed: bare `lyx: seed stencils`, `lyx: sync stencils (…)`, a hub-wide config message, and a labelled seed message touching a path outside the subtrees".

**Issue:** Card 6's untagged `TestSeedCommitSignature` already covers these exact four refuse cases for `IsSeedCommit`.
Card 7's table would rerun the same classification through git.
The only behavior card 7 adds is that `seedCommitsAhead` feeds `CommitDetail`'s message and paths into `IsSeedCommit`, and one row covers that (`PATTERN-test-economy`).

**Fix:** Keep one refused-commit row in the integration test, for example the labelled seed message touching a path outside the subtrees, which exercises `CommitDetail`'s paths.
Leave the rest to `TestSeedCommitSignature`.

### [LOW] Card 7 bundles three reviewable units

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`.

**Issue:** The card holds three units.
The gated drop primitive is `dropSeedCommits`, the new ownership kind and the guard-list rows in `destroy.go` and `destructiveguard_test.go`.
The pull-first verb is `pullFirst`, `PullThenCommitWritten`, `BoltSkip` and `BoltWriteResult`.
The push retry is the change to `Bolt.Push`.
Each builds and can be reviewed on its own, provided the primitive comes first.
Bundling them makes the riskiest unit, the destructive primitive behind the gate, the hardest to review in isolation.

**Fix:** Split card 7 into a drop-primitive card, a pull-first card and a push-retry card, in that order.
Each carries its own tests and `doc.go` sentences.

### [LOW] The new mutation Kind is missing from the oracle's classification table

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`: `plan:internal/fabricengine#KindCommitsDropped`.

**Issue:** `internal/fabricengine/livestate_mutationoracle_test.go` declares `manifestObservableKind` for "every fabricengine.Kind except KindFileWritten".
The table exists so that "a newly added Kind fail[s] loud … instead of silently falling through unclassified".
Card 7 adds `KindCommitsDropped`, a git-state kind, without adding it to that file.

**Fix:** Add `internal/fabricengine/livestate_mutationoracle_test.go` to card 7's `**Edit:**` and classify `KindCommitsDropped` as `false`, a git-state kind like `KindWorktreeReset`.

### [NIT] The Bolt type is an Edit target with no stated change

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md`: `internal/fabricengine#Bolt`.

**Issue:** The Intent adds methods and a result type beside `Bolt`, but describes no change to the `Bolt` struct or its doc comment.

**Fix:** Drop the target, or state in the Intent what changes on `Bolt`.

### [NIT] Card 3's ImpactSummary claims writer keys before any card sets a writer

**Location:** `_lyx/plan/03-stencilstore-forward-refresh.md`, `**ImpactSummary:**`.

**Issue:** The summary says "a production binary stamps writer keys from this card on".
After card 3 alone, every production caller still builds `Source` through the old `StencilSource` or a zero `Source`, so `Source.Writer` is zero and no writer key is written.
Card 4 is the card that sets `Writer`.

**Fix:** Say that writer keys are written from card 4 on, once `StencilSource` fills `Writer`.

## Settled directives with no finding

- Fidelity: D1 is carried by cards 1, 3 and 4, D2 by cards 2 and 3, D3 by cards 6, 8, 9 and 10, D4 by cards 3, 7, 8, 10 and 11, D5 by cards 5, 6 and 7, D6 by cards 7 and 11, D7 by card 11, and D8 and D9 by card 12.
  The doc updates are carried as follows: the `doc.go` of `buildvcs`, `buildinfo`, `stencilstore`, `gitrepo`, `fabricengine` and `hubreconcile` by their cards.
  `cmd/lyx`, `stencilcli`, `fabriccli` and `tools/deploy` have no `doc.go`.
  `docs/overview.md` is carried by card 8, and `PATTERN.md` and `pattern/PATTERN-hub-store-gc.md` by card 12.
  Card 2's `tools/deploy#run` edit is licensed by Scope's "the deploy tool's stamping of it".
  Card 2's `pattern/PATTERN-leaf-packages.md` edit is a forced doc update, because that file says `buildinfo` "exposes `Channel` and `IsDev()` only".
  Card 9's sync subject is D3.
  Card 11 correctly finds that `contracts/specs/refusal-spec.md` lists no fabric refusals.
- Card order: every `plan:` handle a card uses is created by an earlier card.
  Card 10 deletes `revisionLabel` and `shortRevisionLength` after card 1 creates `Identity.Label`, and `ensure` in `walk.go` is their only caller.
- Member glyphs and re-sign arrows: every Edit card names member glyphs.
  Arrows are present on `ModeFor`, `reconcileOne`, `writeStamped`, `ForceRefresh`, `StencilSource`, `seedStencilsAt`, `reconcileHubWide` and `housekeepStores`.
  `Mode`, `Source`, `Identity` and `Topology` change only fields or constants.
  `CommitSeededStencils` is not re-signed.
- Verify coverage: the section's order matches the template.
  The repository has one module (only `go.mod` and a testdata `go.mod`), so `./...` covers `tools/deploy` and `cmd/lyx` in both the untagged and the `integration` tiers.
- Attack surface: card 7's seed drop states its bound, apart from the gate predicate in F4.
  The skip paths in cards 7, 8 and 10 state what they skip and the Warn that names the way forward.
  Card 12's nil-probe default is bounded by the `pairteardown` chokepoint test.
- ImpactSummary: every Edit and Delete card's summary is a blast-radius conclusion rather than a restatement of its Intent, apart from F11.
