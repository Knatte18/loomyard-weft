---
verdict: APPROVED
---

## Measure

The board entry `run-window-early` as amended by the decision record's parent entry "Scope: early window plus async pushes for pair creation and child seed (by: parent, 2026-10-14)".
That entry supersedes the board body's "Scope" and "Out of scope" sections and calls for three changes:

1. The terminal window opens early.
2. The child seed is committed locally and pushed detached.
3. Pair creation's code and records branch pushes go detached, with a failed push left for the next sync and never rolling the pair back.

## Diff against the measure (PR 617 at `7344092f6`, against `main`)

- Early window: `internal/battencli/wire.go` opens the window from the Worktree-Create seam, on both a fresh create and an already-complete pair, through the new exported `battenshed.OpenTerminalOnce` under Run-Shed's once-marker.
  Run-Shed keeps its own open, which the marker turns into a no-op.
  Help text, `internal/battenshed/doc.go`, `docs/overview.md` and the CLAUDE.md "Watching runs" sentence follow.
- Detached seed push: the Seed-Child `PushSeed` seam calls `spawnPairPush` in place of the synchronous `fabricengine.PushAnchored`.
- Detached pair-creation push: `internal/fabricengine/add.go` drops both branch pushes, the origin-side rollback deletion and the push-retry seam (`pushretry.go`, `pushWeftBranch`).
  `fabricengine.SpawnPairPush` is the new entry point.
  `lyx fabric add` and batten's Worktree-Create row both call it, and a failed spawn only warns or is reported in `push_error`.
  `PATTERN-fabric-git` and `internal/fabricengine/doc.go` are updated in the same change.
- Test support: `hubforge.AddPairWith` pushes in the foreground, and the `fabriccli` and `battencli` seams are swapped for synchronous pushes in tests.
- The one change the entry does not name: `internal/fabriccli/weft_verbs.go` gains a Warn log when the detached push child fails.
  It is the only place a failed detached push surfaces, so it serves point 3's "a failed push retried by the next sync".
  It adds no behaviour, so I do not count it as a deviation.
- The parent's out-of-scope items are untouched: the pre-flight `ls-remote` probes, the board claim and the hub-config reconcile.

No point of the amended entry is left out, and the diff holds nothing that belongs to another board entry.

## Carried-over open risks

The Darn-Review carry-over lists three round-1 findings:

- `batten-create-and-seed-push-spawns-unasserted`
- `fabric-add-bypasses-spawnpairpush`
- `fabric-add-envelope-and-spawnpairpush-untested`

The fixer report and commits `b98ad928e`, `fb0739535` and `df6aeddef` fix all three.
None of them names an unrecorded parent correction, so none is escalated.
The record's two other open risks follow from the parent's own decision and need no correction:

- A pair stays local-only until its next push when its first detached push fails.
- The once-marker carries over from an older status file.

## Overlap

- `hibernate-trigger` (row 17, Merge-In) also changes `docs/overview.md`, but at different places: the module tree, the shortcuts line and the tender line.
  This run changes only the batten `--window` sentence.
  The overlap is text-only, with no design conflict.
  Landing order: this run (row 22) is further along, so it lands first, and `hibernate-trigger` takes its changes at Merge-In.
- The other peers (`overview-seats`, `perch-budget-arbiter`, `twb-f3-ir`, `twb-f3-wr` and `twb-f3-pr`) list no changed paths, so none overlaps.
- `lyx loom peers` lists no parent commits.
  `main`'s tip `bf4a61460` ("reed: an attached run window follows the newest agent window") is already merged into this branch.
  It changes which window an attached session follows, not when the run window opens, so it does not conflict with this run's decisions.

## Bookkeeping files

`docs/overview.md` changes one prose sentence of the batten module entry, not a module-table line.
This run does not touch `cmd/lyx/testdata/test-budget.yaml` or add a PATTERN index line.

## Superseded

The board body's deferred note on making the add's pushes async is folded into this run by the parent's decision.
It has no board entry of its own, so no slug is superseded.
