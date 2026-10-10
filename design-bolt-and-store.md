# Bolt and hub store: seeding without ping-pong, reconcile on a fresh Bolt, gc that stays out of the way

`bolt-and-store` · Task · bug, enhancement, stencil, fabric, config · running · Discussion-Write

## Goal

lyx's own git writes to Bolt and the shared hub store stay correct and quiet:

- Stencil and spec seeding never flips Bolt between two binaries' versions, and every seed commit names the binary that made it.
- Seeding and the hub-config write start from an up-to-date Bolt, a conflicting seed commit is regenerated instead of left behind, and reconcile reports a failed push as a failure.
- The hub store's gc at Remove does not repack under other runs, and does not repack at every teardown without lowering the loose count.

Parts 3 and 4 come from the PR-Gate review of `git-robustness` and were not observed in use; the operator chose to fold them in (2026-10-10).
The four parts below are the merged notes, each kept as written.

## Part 1: stencil seeding ping-pongs on Bolt (#477)

Imported from [issue #477](https://github.com/Knatte18/loomyard/issues/477).

Bolt took 150 stencil and spec seed commits from 2026-10-05 to 2026-10-08 (86 `lyx: seed stencils`, 64 `lyx: seed specs`), next to 1678 `board sync` commits.
Seeding should commit only when a shipped default changes, so most of these are two binaries overwriting each other.

### What the history shows

`_lyx/stencils/loom/loom-template-plan.md` changed in 73 of the stencil seeds and `loom/loom-rubric-plan-review.md` in 66.
The plan template's blob hash in Bolt switches between two versions within minutes:

| Time (10-08) | Blob |
| --- | --- |
| 10:56 | 7285c5d → 2f77cd9 |
| 10:58 | 7285c5d → 2f77cd9 |
| 10:59 | 7285c5d → 2f77cd9 |
| 11:12 | 7285c5d |
| 14:10 | 12f5d3d |
| 16:49 | 2f77cd9 → 12f5d3d |
| 17:05 | 2f77cd9 → 12f5d3d |
| 17:06 | 2f77cd9 → 12f5d3d |

Run `git log -- _lyx/stencils/loom/loom-template-plan.md` on Bolt and hash each version to reproduce the table.

### Why it can happen

Every lyx command's root pre-run seeds the hub-level stencils and specs from the running binary's embedded registry (`cmd/lyx/stencilseed.go`, `seedStencilsAt`), commits and pushes.
In `ModeProduction`, `stencilstore.reconcileOne` refreshes an untouched stencil whenever its body differs from the shipped one, with no notion of which version is newer.
The comment there says so itself: an older binary in prod mode downgrades the board's stencils.
`ModeProduction` is the zero value, so every unstamped binary seeds in prod mode: a plain `go build ./cmd/lyx`, `go install`, and `internal/testkit/lyxbin`'s builds (empty ldflags).
Only `tools/deploy` stamps `Channel=dev` for `./deploy-dev`.

Two prod-mode binaries built from different commits, both run against one hub, flip each changed stencil on every invocation, and every flip becomes a commit and push on Bolt.
On 2026-10-08 the installed `lyx` was rebuilt several times from `main`, while task worktrees whose branches edit the plan template and rubric (plan-gate work) were running.
Which second binary did the flipping is not established; candidates are an unstamped build run inside a task worktree and an installed binary from before the latest `update-plugins.sh`.

### Effects

- Producers read whichever version the last command seeded, so a running task's Plan or review prompt can change under it mid-run.
- Bolt's history fills with seed noise.
- A clone that has not pulled Bolt for a while seeds on its stale copy and then fails to push (separate issue: reconcile seeds before pulling Bolt).

### Possible directions

- Stamp the content with the source commit and refresh only forward, never to an older commit's default.
- Or seed only from the installed production binary (`update-plugins.sh` stamps a production channel, and an unstamped build seeds nothing).
- Log which binary (path, revision) made each seed commit, in the commit message, so the next flip can be traced.


## Part 2: reconcile on a stale Bolt (#478)

Imported from [issue #478](https://github.com/Knatte18/loomyard/issues/478).

Seen 2026-10-09 on a clone that had not been used since 2026-10-05: its local Bolt (`<hub>/_board`) was 1829 commits behind `origin/main`.
After a freshly deployed `lyx`, `lyx fabric reconcile` in the prime reported `ok: true` while `hub_config_detail` and `warp_binding_detail` both said the push failed:

```
gitrepo: git pull --rebase: Rebasing (1/3)
error: could not apply cd6b67cf... lyx: seed stencils
```

The root pre-run had seeded stencils and specs, and reconcile had written the hub-wide `board.yaml`, as three local commits on the stale Bolt.
Upstream already had newer versions of the same files, so the rebase conflicted.
The rebase was aborted, leaving Bolt 3 ahead and 1829 behind, and every later lyx command repeated the failure.
The fix was by hand: `git reset --hard origin/main` in `_board`, then `lyx fabric reconcile` again, which was then clean with nothing to seed.

### Expected

- Seeding and the hub-config write pull Bolt before they write, so they compare against upstream instead of a stale local copy.
- A seed commit is regenerated content: when its push conflicts, lyx drops it, fast-forwards Bolt and seeds again, rather than leaving a conflicting local commit for the operator.
- `ok: true` is not reported when the hub config or warp binding could not be pushed; the failure is surfaced as a failure, with the command that resolves it.

Related: the stencil seed ping-pong issue (two prod-mode binaries overwriting each other's stencils on Bolt).


## Part 3: store gc while other runs are busy

Since `git-robustness`, Remove runs a foreground `git gc --auto` on the shared hub store, which repacks it while other pairs' steps run against the same store.
`readGoGit` now absorbs a changed pack set for lyx's own reads, but the repack still costs time and I/O during other runs.
Seen in the PR-Gate review of PR #483.

Idea: skip the gc at Remove while other runs are in flight, and run it at the next teardown that finds none.


## Part 4: store gc and loose objects

Since `git-robustness`, the hub store's auto gc is off and Remove runs `git gc --auto` at teardown.
Loose objects younger than the prune expiry survive each repack, so `gc --auto` can find the loose count over its threshold at every teardown and repack without lowering it.
Seen in the PR-Gate review of PR #483.

Idea: run `git prune --expire` now and then, such as at landing, so the loose count falls and `gc --auto` stops repacking every time.
