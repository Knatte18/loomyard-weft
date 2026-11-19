# Card 3 — one-repository-geometry

**Create:**
- `internal/fabricengine#BoltBranchName`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#recordsBranchIn`
- `internal/fabricengine#recordsLockDirName`
- `internal/fabricengine#TestRecordsGeometry_OneRepository`

**Edit:**
- `internal/fabricengine#RecordsWorktree`
- `internal/fabricengine#RecordsWorktreePath`
- `internal/fabricengine#RecordsRepoRoot`
- `internal/fabricengine#RecordsLockDirPath`
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#restoreWeftLockDir`
- `internal/fabricengine#Healthy`
- `internal/fabricengine#Topology.Status`
- `internal/fabricengine#OriginRunStatus`
- `internal/fabricengine#OriginCodeTip`
- `internal/fabricengine#Topology.reconcileMissingWeft`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#resolveMergeSources`
- `internal/fabricengine#pairHasLocalPart`
- `internal/fabricengine#primaryWeftBranch`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine#refuseTargetAnchorChange`
- `internal/fabricengine#Topology.switchOrForkWeft`
- `internal/fabricengine#resolveAddParent`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#Topology.inspectPair`
- `internal/fabricengine#Topology.RecordsBranchLive`
- `internal/fabricengine#resolvePairWarpCheckout`
- `internal/fabricengine#resolvePairWeftBranch`
- `internal/fabricengine#Topology.pairTaskBranchLocal`
- `internal/fabricengine#Topology.originPairBranch`
- `internal/fabricengine#Topology.originOtherPrefixBranch`
- `internal/fabricengine#ReadConfigChanges`
- `internal/fabricengine#Topology.keptReason`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#committedMarker`

**Intent:** D1's fabric read, and D3 and D4's geometry: in a one-repository hub the records worktree of a worktree is the worktree itself.

`oneRepositoryAt(path)` is fabric's one layout read, from git as D1 has it (the overview's `layout-read` decision):
it finds the main worktree of the repository holding `path` (its common dir's parent, read in-process through `dotgit`);
a main worktree whose name is a records sibling's (`weftname.Suffix` with its owner beside it) is a two-repository records repository and answers false;
otherwise it answers whether `.lym-anchor` is committed at HEAD of the path's own worktree (`committedMarker`),
and for the main worktree and the Bolt checkout, whose checked-out branch may lack the marker, whether the repository's `lym-bolt` branch (`BoltBranchName`, the Bolt's branch name card 5's clone uses) commits it.
A failed read answers false and is logged at Warn, so a path that is no repository keeps today's geometry.
Every member below that needs the layout calls it on the location's worktree; Add, for a worktree it has not made, reads the start point's commit.

- `RecordsWorktree(l)` returns `l.WorktreePath()` and `RecordsWorktreePath(l, slug)` returns `WorktreePath(l, slug)`.
  `RecordsRepoRoot(l)` returns the prime worktree, `<hub>/<PrimeName(l)>`, the one repository's main checkout.
  So `MergeStateActive`, the archive and owner-archive tags and every weft-side reader outside fabric (`weavercli`, `battencli`'s catch-up, `reportcli`, `reportengine`) read the one repository unchanged.
- `RecordsLockDirPath(path)` returns `<gitdir>/lym-locks` (`recordsLockDirName`) when `oneRepositoryAt(path)` holds, the gitdir read in-process through `dotgit.Read`; the Bolt checkout of a one-repository hub takes its own gitdir the same way.
  A gitdir it cannot read falls back to today's `.weft` path, where a commit fails as it does today.
  The write lock, push lock, push-holder record, move record and origin-record lock then sit in the worktree's own gitdir, which needs no exclude line, so the one repository's exclude never gets `.weft/`.
- `ensureWeftLockDirAt` in a one-repository worktree creates that directory and skips `seedWeftArtifactExcludes`, whose `.weft/` and legacy lock lines would hide code paths (D3).
- `ReadOriginFor` and `restoreWeftLockDir` build the lock directory through `RecordsLockDirPath` instead of joining `weftLockDirName` themselves.
- `recordsBranchIn(l, branch)` returns `branch` in a one-repository hub and `RecordsBranchName(branch)` otherwise.
  Every member above that derives a pair's records branch from its code branch calls it in place of `RecordsBranchName`,
  so `Healthy` raises no branch mismatch, and Add, remove, origin reads, merge sources, checkout and reconcile name the one branch.
  `RecordsBranchName` itself and its callers outside fabric stay as they are, since D4 has those readers read the one repository unchanged.
- `Topology.reconcileMissingWeft` does nothing in a one-repository hub, where the records worktree is the worktree and cannot be missing apart from it.

`TestRecordsGeometry_OneRepository` (tier1, package `fabricengine`, `gitkit` repositories standing as a hub's prime with a linked worktree for a slug) rows,
each asserting `oneRepositoryAt`, `RecordsWorktree`, `RecordsWorktreePath`, `RecordsRepoRoot`, `RecordsLockDirPath` and `recordsBranchIn` against the paths and branch the layout names:
- `OneRepository` (marker committed at the root of the prime and the slug's branch, and a `lym-bolt` branch committing it);
- `OneRepositoryPrimeOnABranchWithoutTheMarker` (the prime switched to a branch lacking the marker still reads one repository through `lym-bolt`);
- `OneRepositoryBoltCheckout` (a linked worktree named `BoltDirName` on `lym-bolt`: one repository, its lock directory in its own gitdir);
- `TwoRepositoryCodeWorktree` (no marker on the code branches and no `lym-bolt`);
- `TwoRepositorySiblingCarryingAMarker` (a records repository whose main worktree is named `<prime>` plus `weftname.Suffix` with the prime beside it, its branch committing the marker: false, and the `.weft` lock directory).
The two-repository expectations are today's: the `weftname.Suffix` sibling paths, the `.weft` lock directory and the suffixed branch.
Card 6's `TestAdd_OneRepository` reaches the switched `Topology.Status`, `Topology.RecordsBranchLive` and `ReadConfigChanges`, and card 12's `TestRemove_OneRepository` the switched remove path.

**ImpactSummary:** About thirty fabricengine members switch to layout-aware records paths and branch names; the outside callers of RecordsWorktree, RecordsWorktreePath and RecordsRepoRoot get today's paths in a two-repository hub.
