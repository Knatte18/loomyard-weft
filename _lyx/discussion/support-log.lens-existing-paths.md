```yaml
lens: existing-paths
outcome: reported
```

# existing-paths lens report: umbrella-anchor decision record

I traced the existing code paths that write, reset or read the marker, the anchor and the anchored paths the record changes. Findings come first, then the per-decision trace list.

## Findings

**BLOCKING — D7 recovery (and D10's pull rewire): `lyx fabric reconcile` does not wire a pair that has no `_lyx` at its anchor.**
- `isRawWarpWorktree` (`reconcile.go`) classifies a warp worktree as raw/unmanaged when `<AnchorPath>/_lyx` is absent.
- The scratch section of `doc.go` says such a pair "never reaches the repair".
- After D7 step 2, the junctions at `A` are gone, so with the marker still `A`, reconcile sees the pair as raw.
- After the step-3 commit, the marker reads `B` and nothing is wired at `B` yet, so the pair is raw again.
- A D10 pull that moved the marker before the rewire leaves the same state.
- So the claim "before the step-3 commit the worktree is at `A`, after it at `B`, and reconcile wires whichever the marker names" breaks exactly in the windows it is meant to cover.
- Fix options: reconcile classifies a pair as managed by its own committed marker as well as by `_lyx`, or the record names the D7 re-run as the only recovery and drops the reconcile claim.

**BLOCKING — D1 with D7's failure path: the reader reads the working tree, but the rollback restores only the index and `.lyx`.**
- D1 has `lyxcwd` read `.lyx-anchor` from the working tree.
- D7's step 3 writes the marker file and moves tracked files with `git mv`, which changes the working tree as well as the index.
- "A failure before the commit restores `A/.lyx` and the index" leaves the marker file reading `B` and the moved files at `B` on disk.
- After that failure, every resolve already answers `B`, while HEAD and the junctions say `A`.
- The weft is also dirty outside the anchored pathspec, so `refuseDirtyWeftWorktree` (`remove.go`) refuses later removes.
- Fix: the rollback restores the working-tree files and the marker file too, or the marker is written last, just before the commit.

**MEDIUM — D10 pull rewire: an interrupted rewire cannot be resumed.**
- The rewire triggers on "the marker differs before vs after the weft pull".
- If the process dies after the pull and before the rewire, the next pull sees no difference.
- What is left behind:
  - dangling links at `A`;
  - `<weft>/A/.lyx` orphaned, holding locks and logs;
  - a root `.scratch` link still pointing at `A/.lyx/scratch`, which the scratch step leaves as found.
- Reconcile does not repair this either (see the first finding).
- The record names no recovery. It should say that `lyx fabric anchor --set B` re-run finishes it, which needs D7's re-run to also move `A/.lyx`, or give the pull a resumable trigger.

**MEDIUM — D7 for the prime: hub-wide state and processes sit at the prime's anchor, outside the refusals.**
- The prime's `.lyx` (`loomyard/.lyx`) holds `hub/`, `gate/`, `orch/`, `shuttle-open`, `notices.log`, `beam/` and `shed/`.
- Processes are spawned with cwd at `prime.AnchorPath()`:
  - the hibernate watch daemon (`hibernatecli/deps.go`);
  - the adopt finalize suite (`adoptcli/deps.go`);
  - batten passes (`dobbydeps.go`).
- Renaming the prime's `A/.lyx` while another worktree's gate run holds a slot, or while the hibernate daemon lives, pulls state out from under live processes.
- A daemon whose cwd is the old anchor fails the cwd gate on its next lyx call.
- D7 refuses only live batten runs and dobbies for the prime. It needs refusals, or ends, for the gate-slot holders, the hibernate daemon and the adopt suite, or the record should state the hub-wide quiet condition it requires.

**MEDIUM — D8 census: origin weft branches that no machine checks out keep the Bolt marker forever.**
- D8 migrates only a managed pair "whose weft worktree is present".
- It removes the Bolt marker only when "every weft branch on origin" carries a marker.
- Orphaned remote weft branches never get one: leftovers of removed pairs that `remoteleftover.go` and `cleanupremotewarp.go` exist to handle, and branches of pairs no machine holds.
- So the census never completes, which contradicts the entry's "no marker stays on Bolt".
- The census should skip, or itself mark, weft branches with no warp sibling on origin.

**LOW — D3 scan: a Location literal that omits `AnchorRel` slips past it.**
- The scan matches literals that set `AnchorRel`.
- A literal that leaves it out gets `""`, which acts as the root anchor. `clone.go:290`'s `hookLocation` is one today.
- A new sibling builder can reintroduce the one-anchor bug that way. The scan should flag any production `lyxcwd.Location{...}` literal with `WorktreeName` set outside `lyxcwd`, or the record should name the exception.

**LOW — D7: reed strand state survives a dead session.**
- The refusal checks only that the reed session is live.
- `.lyx/reed.json` keeps its strands after a session dies, for example after a hibernate, and moves along with `.lyx`.
- A later `lyx reed resume` or `lyx reed up` replays the stored commands, and their pane cwd may be the old anchor. `reed.json` records each strand's worktree; I did not confirm whether it stores a pane cwd.
- The record should say whether the verb clears the stored strands, or refuse while any are stored.

## Paths traced

**D1/D2 — the marker read**
- `lyxcwd.resolveCore` and `ResolveWorktree`: they read the new marker, so their callers follow it. Survives. These callers are `orchcli/live.go`, `tendercli/deps.go`, `creelcli/delivery.go` (four sites), `battencli/overview.go`, `loomcli/landingdeps.go` (the parent's runs root) and `addparent.go`.
- `ResolveWithAnchor` (clone, gitkit): bypasses the marker. Unchanged.
- `refuseEmptyAnchorMarker`: reads Bolt today, made per pair by D8. Covered.
- The post-checkout hook: warns only, never reads the anchor. Survives.
- The D7 failure path against the D1 working-tree read: breaks (finding 2).

**D3 — another worktree's Location**
- `dobbydeps.go:232` and `teardown.go:336`: both are literals the scan catches. Covered.
- `warpLayoutFor`: covered.
- `clone.go:290`: a literal that omits `AnchorRel`. LOW.
- `hubworkspace.go:228` (prime anchor from `l.AnchorRel`): covered by D3's list.
- `configcli/hubwide.go` (prime anchor from a worktree's `Anchor` field): survives if fed per worktree.
- The spawns at `prime.AnchorPath()` (adopt, hibernate, batten pass): survive for a beam change; break for a prime change (finding 4).

**D5 — forks**
- `createWeftWorktreeDroppingRuns`: scoping changed. Covered.
- `checkout.go`'s `switchOrForkWeft` fork inherits the current marker. Survives.
- `replaceRecordsContent` (promote): guarded by D10. Survives for equal anchors.

**D7 — the anchor change**
- The scratch step leaving a link to another target as found: covered by step 2.
- `isRawWarpWorktree` in reconcile: breaks the recovery claim (finding 1).
- `refuseDirtyWeftWorktree` after a failed step 3: breaks (finding 2).
- Reed strand state: LOW.
- Dobby `PrimeAnchor` cache: covered by step 1.
- The prime's `.lyx/gate`, `.lyx/hub` and the hibernate daemon: break (finding 4).
- `RegenerateHubWorkspace`, also called by add, remove and reconcile: survives under D3.
- The launcher writes reached from reconcile: survive if keyed per worktree.

**D8 — migration**
- The census against orphaned remote weft branches: breaks (finding 5).
- The Bolt-marker removal commit through Bolt: survives; clone already commits Bolt files CLI-side.
- Unwire leaves the marker. Survives.

**D9 — the exclude census**
- `unseedGitExclude` and `namesWiredInSiblingWorktrees`: addressed.
- `ideengine.ExcludeAnchoredDir` leaves a stale `/A/.vscode` line. Harmless.

**D10 — checkout and pull**
- Checkout's refusal happens before either switch. Survives.
- An interrupted pull rewire: breaks (finding 3).
- Weft merges: none exist, so no merge path touches the parent's marker. Survives.

## Tool calls

1. Grepped `AnchorRel`/`AnchorPath()` in hubreconcile, reed, hubgeom, configsync, adopt, cleanup, batten and landing: the context lacked the readers outside the board entry's named files.
2. Grepped how task and parent Locations are built (`lyxcwd.Location{`, `parentLocation`): to find literals and `ResolveWorktree` callers the context did not show.
3. Read this run's `seed.json`, the prime's `_lyx` listing and `reed.json`: to see whether run seeds and reed state hold absolute anchor paths the move would leave stale.
4. Grepped `reed.json` for cwd fields and read `post-checkout.sh` and `hook.go`: the hook is an existing path triggered by branch switches, which the context had not read.
5. Grepped sibling-path builders in configsync, cliwire, orch, tender and creel, and listed the prime's and Bolt's `.lyx` plus the `PrimeAnchor` users: to find hub-wide state at the prime's anchor that D7's prime case renames.
6. Read `reconcile.go`'s `isRawWarpWorktree`: to check the record's claim that reconcile recovers an interrupted change.
