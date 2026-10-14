# Batten: run window at create; new pairs and seeds push detached

`lyx batten run <slug> --window` used to take about 20 s before the terminal window opened.
Pair creation pushed both branches synchronously, the child seed push waited on the records remote, and the window opened only after the inner shed run spawned.
This change opens the window as soon as the pair exists.
It also routes every write to a new pair one way, a local commit plus a detached push, as `lyx fabric sync` already does.

## What changes

**Window at create.**
With `--open-terminal`, which `--window` passes, batten's Worktree-Create seam now opens the terminal window once `Add` and the pair's config reconcile succeed.
It opens it through the new `battenshed.OpenTerminalOnce`, under Run-Shed's existing once-marker.
Run-Shed's open after a spawn is kept, and the marker makes it a no-op unless no window opened yet in this run, such as a run resumed past Worktree-Create with `--window`.
`lyx reed attach` boots the pair's reed session when none exists, and reed serialises boots under its op lock, so the later `lyx loom start --no-attach` joins the same session.

**`Topology.Add` pushes nothing.**
`Add` keeps every origin check synchronous: both `ls-remote` probes, adoption and fast-forward of a live weft branch, and the leased deletion of an archived leftover weft branch, now step 11 and the last step.
It no longer pushes either branch.
Its callers spawn the pair's detached push through the new `fabricengine.SpawnPairPush`, which hands the code and records worktrees to the existing `SpawnDetachedPush` child.
Those callers are `lyx fabric add` and batten's Worktree-Create row.
The first push creates each branch's upstream (`push.autoSetupRemote`).
A failed or never-run push leaves the branches committed locally, and the pair's next detached push, which every fabric commit and sync spawns, carries them.

**Seed push detached.**
Seed-Child's `PushSeed` spawns the same detached pair push instead of a synchronous records push that could wait up to `StatusPushLockWait` behind the add's push child.
`CommitSeed` is unchanged.

**Envelope.**
`AddResult.Pushed` is gone.
`lyx fabric add` records one `push_spawned` mutation per side, as `sync` does, or a `push_error` field when the spawn fails, which never fails the verb.

## Design decisions

- **The callers push, not the engine.**
  Spawning inside `Add` would fork the test binary for every in-process caller (`hubforge`, `fabriccli` and `battencli` tests), and `fabricengine`'s unexported spawn seam cannot be swapped from those packages.
  So `fabriccli` and `battencli` each spawn through a package seam their `TestMain` replaces with a foreground push.
  `hubforge.AddPairWith` pushes in the foreground after `Add` unless its options skip pushing.
- **Dead rollback code removed.**
  With no push inside `Add`, its rollback never meets a pushed branch.
  The rollback step that deleted the warp branch from origin, the `branchDirtinessPushedByThisCall` kind, the `pushSeam` push-retry seam (`pushretry.go`) and `pushWeftBranch` are deleted, and `probeWarpLeftover` no longer reports an origin state.
- **Detached push failures reach the trace log.**
  The push child's stdout is discarded, so its bypass handler now logs a hard push failure with both worktree paths, which name the pair.
- **Scope beyond the board entry.**
  The parent widened the task from the early window alone to these three changes; it is recorded as a parent decision in the decision record.

## Tests

- `TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating` now also pins that the create row opens the terminal window on the new pair once, and that a re-entered row on the complete pair opens no second one.
- Rollback tests that forced a failure with a broken origin remote now force it with a refused records commit (`refuseRecordsCommits`) or, in the live-state matrix, a portal blocker.
- The leftover-replacement tests push the new pair themselves after `Add` (`pushNewPair`).
- Deleted: the push-retry and push-failure rollback tests (`add_pushretry_integration_test.go`, `add_rollback_branch_integration_test.go`, `pushretry_test.go`, `TestAddRollback_RefusedWarpBranchDeletionLogsWarn`), and `TestAddRollback_AdoptedPathPreservesOriginRecordCommit`, whose failure after the record commit can no longer occur.
- `lyx gate test`, untagged and `--tags integration`, is green for `fabricengine`, `fabriccli`, `hubforge`, `battencli` and `cmd/lyx`, and untagged for `battenshed` and `shedrecipe`.

## Docs

`internal/fabricengine/doc.go`, `internal/battenshed/doc.go`, `pattern/PATTERN-fabric-git.md`, the `lyx fabric add` and `lyx batten run` help text, `docs/overview.md` and the CLAUDE.md "Watching runs" sentence.

## Open risks

- A pair whose first detached push fails stays local-only until its next commit or sync, and another machine cannot adopt it meanwhile.
- The window boots the pair's reed session before Seed-Child writes the run settings' local config layer, so a reed key set there misses that boot.
