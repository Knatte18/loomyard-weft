# Card 10 — fabric-worktree-seam

**Create:**
- `plan:internal/fabricengine#WorktreeEvent` -> `type WorktreeEvent struct`
- `plan:internal/fabricengine#WorktreeSeam` -> `type WorktreeSeam interface`
- `plan:internal/fabricengine#installedWorktreeSeam` -> `var installedWorktreeSeam atomic.Pointer[WorktreeSeam]`
- `plan:internal/fabricengine#InstallWorktreeSeam` -> `func InstallWorktreeSeam(s WorktreeSeam) (restore func())`
- `plan:internal/fabricengine#EventFor` -> `func EventFor(l *lymcwd.Location, paths []string) (WorktreeEvent, error)`
- `plan:internal/fabricengine#fireWorktreeEvent` -> `func fireWorktreeEvent(l *lymcwd.Location, paths []string, made bool) string`
- `plan:internal/fabricengine#NotifyWorktreeMade` -> `func NotifyWorktreeMade(l *lymcwd.Location) string`
- `plan:internal/fabricengine_test#TestWorktreeSeam_Events` -> `func TestWorktreeSeam_Events(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology.add`
- `internal/fabriccli#CloneAndWire`
- `internal/fabricengine#PullResult`
- `internal/fabricengine#Fabric.pullWarpArm`
- `internal/fabricengine#Fabric.reanchorAfterRewrite`
- `internal/fabricengine#Fabric.FastForwardPair`
- `internal/fabricengine#BranchOutcome`
- `internal/fabricengine#originMover.move`
- `internal/fabricengine#OriginReport.Summary`
- `internal/fabricengine#pullPairSide`
- `internal/fabriccli#pullResultMap`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#SeedWarpExclude`
- `internal/fabricengine#pathsChangedBetween`
- `internal/fabricengine#RecordsWorktreePath`
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#Bolt.Pull`
- `internal/hubforge#CloneSecond`
- `internal/fabricengine_test#buildReconcileFixture`
- `internal/testkit/logcapture#Capture`
- `internal/fabriccli/doc.go`

**Intent:** Add fabric's process-wide worktree seam with its two events, "made" and "took in commits", fired where D6 says (D6).

The seam (in a new file `internal/fabricengine/worktreeseam.go`; its names carry no side token, since heddle and `cmd/lym` spell them and PATTERN-fabric-vocabulary bars a side token outside its owner set):

- `WorktreeEvent` holds `Root`, the worktree's root; `ScratchDir`, the pair's never-tracked `.lym` directory (`RecordsWorktreePath(l, slug)` joined with the anchor and `lymdirs.DotLymDirName`, as `scratchlink.go` builds it); `BoltDir`, `BoltDir(l.HubPath)`; `Excluded`, the exclude-list paths; and `Paths`, the paths that differ between the old and new HEAD, empty for "made".
  Fabric hands paths and names no heddle file.
- `WorktreeSeam` has `Made(WorktreeEvent) error` and `TookIn(WorktreeEvent) error`.
  `InstallWorktreeSeam` stores it in an `atomic.Pointer` and returns a restore function, as `InstallPushNotifier` does; a missing seam is no event, so a test binary needs no install.
- `EventFor(l, paths)` first seeds the exclude list through `SeedWarpExclude(l)`, under the exclude lock, then builds the event; `lym heddle render` builds its event through it too (card 19), so every render writes only after the lines are seeded.
- `fireWorktreeEvent(l, paths, made)` returns "" with no seam installed, else builds the event and calls `Made` or `TookIn`.
  A failure to build the event or a seam error is never returned as an error: it is logged at Warn naming the worktree and `lym heddle render` as the way forward, and returned as that warning text for the caller's result.
  It is called with no Bolt write lock held, since the seam's render takes that lock and a second acquire in one process blocks.
- `NotifyWorktreeMade(l)` fires "made" for `l`'s worktree and returns the warning.

Where it fires:

- `Topology.add` (so `Add` and `AddLocal`) calls `NotifyWorktreeMade` for the new pair last, after the wiring has seeded the exclude lines and every step succeeded; the warning is logged only, since `AddResult` is printed by callers that report nothing of the kind.
- `fabriccli.CloneAndWire` calls `NotifyWorktreeMade` for the prime right after its `WireJunctionsWith`: `CloneHub` hands wiring to that CLI function (it cannot import `configsync`), so this is the point where the clone's worktree is wired, its lines seeded and the Bolt checkout present, which is what D6 asks of `CloneHub`'s "made".
- `Fabric.pullWarpArm`, after its fast-forward `ResetHard`, and `Fabric.reanchorAfterRewrite`, after each of its `ResetHard` calls, fire "took in" with `pathsChangedBetween(warpPath, localHEAD, upstreamSHA)`; `PullResult` gains `SeamWarning string`, set from the returned warning, and the pull's result and move are otherwise unchanged.
  The weft arm and every Bolt or records move fire nothing.
  The results that print a pull carry the warning: `fabriccli`'s `pullResultMap` adds a `seam_warning` key when it is set, so `lym fabric pull` prints it, and `pullPairSide` appends it to the pair side's `Detail`, so a hub pull prints it on that pair's line.
  `battencli`'s land pull prints no pull result, so there the warning stays in the log, as D6 allows for a process that prints none.
- `Fabric.FastForwardPair`, after its `ResetHard` to the upstream, fires "took in" with `pathsChangedBetween(warpPath, localHEAD, upstreamSHA)`, since it moves a checked-out code branch to origin's commits; D6's landing exception covers a parent merge or landing bringing commits into a beam, while this call, though only Finalize makes it, moves the parent pair to origin's commits, a take-in; the warning is logged only, since `FastForwardResult` is not printed.
- `originMover.move`'s code-side branch fires "took in" once `ffWorktree` returns `BranchLevel`, with the paths between the head it read before the move and `OriginHead`, for the location `lymcwd.ResolveWorktree(target.worktree)`; `BranchOutcome` gains `Warning string`, and `OriginReport.Summary` prints a line `warning: <branch>: <warning>` for each branch carrying one.
  Records and Bolt branches moved by `reconcileRecordsBranch` fire nothing.

`TestWorktreeSeam_Events` (tier1, not parallel because it installs the process-wide seam and captures the logger; its recording fake ignores events whose root lies outside its own hubs, so parallel tests elsewhere in the binary cannot disturb it) asserts:

- a hub made by `hubforge.NewHub` (through `CloneAndWire`) and a pair added to it each record one "made" event whose root is the worktree, whose `BoltDir` exists and whose exclude lines are in the warp `info/exclude` when the fake runs;
- with a failing seam, a pair added to the hub still exists with its `AddResult`, and the captured log holds the warning naming the pair and `lym heddle render`;
- a `Fabric.Pull` that fast-forwards warp over a commit, pushed from `hubforge.CloneSecond`'s hub, changing `AGENTS.md` records one "took in" with exactly `["AGENTS.md"]`, and one over a commit changing `src/x.go` records `["src/x.go"]`;
- a `Fabric.Pull` over a rewritten warp history, built as `TestPull_ReanchorsSingleCommitBack` builds it through `buildReconcileFixture`, records one "took in" with the paths between the old and new head;
- a Bolt pull and a weft-only fast-forward record nothing;
- `Fabric.FastForwardPair` moving a pair over an upstream commit, pushed from `hubforge.CloneSecond`'s hub, changing `AGENTS.md` records one "took in" with exactly `["AGENTS.md"]`, and with a failing seam it still returns the upstream as `NewHead` and the captured log holds the warning naming the worktree and `lym heddle render`;
- `ReconcileOrigin` moving a pair's code branch by fast-forward records one "took in" with the paths the moved commits change, while its fast-forwards of that pair's records branch and of Bolt in the same reconcile record nothing;
- with a failing seam, the pull still reports `WarpAdvanced` and `NewWarpHEAD` with warp at the new head, `SeamWarning` names the worktree and `lym heddle render`, the captured log holds the same warning, and `pullResultMap` of that result holds it under `seam_warning`;
- with the hub's fabric config made unparseable after the pair was added, so `EventFor` cannot build the event, the pull still moves warp and `SeamWarning` names the worktree and `lym heddle render`;
- with a failing seam, `ReconcileOrigin` moving a pair's code branch leaves the outcome `Moved` at `BranchLevel`, its `Warning` set, `Summary()` holding the warning line, and the log holding it;
- with no seam installed, the same pull returns an empty `SeamWarning`.

**ImpactSummary:** Adds optional seam calls to fabric's add, clone, pull, origin-reconcile and parent-pair fast-forward code moves plus two result fields that `lym fabric pull`, hub pull and `OriginReport.Summary` print; with no seam installed every verb behaves and prints as before.
